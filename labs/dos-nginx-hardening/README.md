---
title: DoS-атака Slowloris и защита Nginx (limit_req, rate limiting)
category: network-security
tools: slowhttptest, siege, Nginx
date: 2026-05-19
mitre: T1499.002 (Endpoint Denial of Service: Service Exhaustion Flood)
---

## Описание
Лабораторная работа по курсу «Расследование компьютерных инцидентов». Цель: развернуть веб-сервер (LEMP, Nginx + PHP-FPM), провести на него DoS-атаку двумя методами (Slowloris, HTTP-флуд) и настроить защиту.

Среда: собственная виртуальная лаборатория, тестирование выполнено на изолированном стенде.

## Настройка окружения
- Сетевой интерфейс целевой машины (Debian, LEMP) переведён в режим Bridge для получения отдельного IP в локальной подсети.
<img width="639" height="252" alt="изображение" src="https://github.com/user-attachments/assets/4ec42aa4-b831-4fe9-827e-4be206280c4a" />

- Настроено удалённое администрирование по SSH.
- В `/var/www/html/` создан `info.php` (`<?php phpinfo(); ?>`) для проверки работы PHP.
- В `/etc/nginx/sites-available/default` добавлен `index.php` в список индексных файлов, раскомментирован блок обработки PHP (`fastcgi_pass unix:/run/php/php8.4-fpm.sock;`).<img width="403" height="197" alt="изображение" src="https://github.com/user-attachments/assets/9e2981a4-8bb8-4983-80cb-4a7340a1652e" />

- Конфигурация проверена (`sudo nginx -t`), сервис перезапущен. Доступность подтверждена открытием `info.php` в браузере.
<img width="563" height="106" alt="изображение" src="https://github.com/user-attachments/assets/e3a0ac1f-cc6a-48a3-afbb-a3052e4e06af" />
<img width="633" height="378" alt="изображение" src="https://github.com/user-attachments/assets/f88da270-2cb2-4be8-9bc5-cfac2cf423d9" />

## Атака 1: Slowloris (slowhttptest)

**Первая попытка** — 1000–3000 медленных соединений:

slowhttptest -c 3000 -H -g -o slow_result -i 5 -r 400 -t GET -u http://10.62.239.150/info.php -x 10 -p 2
<img width="290" height="191" alt="изображение" src="https://github.com/user-attachments/assets/6be5f6c7-3115-47f1-b555-036d31b33114" />


Сервер устоял: встроенные таймауты Nginx и защита ядра Linux успевали закрывать неактивные сессии (`Service available: YES`).

**Усиленная атака** — увеличено число потоков и скорость подключения:

slowhttptest -c 4000 -H -g -o slow_result -i 3 -r 500 -t GET -u http://10.62.239.150/info.php -x 10 -p 2

Результат: `connected: 1530`, `Service available: NO`. Сервер перестал отвечать на легитимные запросы.
<img width="268" height="176" alt="изображение" src="https://github.com/user-attachments/assets/240c319d-629e-45a6-b714-c21ce8f91b2b" />


### Почему атака сработала
1. **Исчерпан лимит `worker_connections`.** По умолчанию Nginx обслуживает ограниченное число одновременных соединений (обычно 512–1024). Атака на 4000 подключений мгновенно исчерпала лимит.
2. **Механика Slowloris:** `slowhttptest` открывает соединение, но не досылает финальные символы запроса. Nginx удерживает такие «зависшие» соединения открытыми в ожидании продолжения, не сбрасывая их по таймауту, если атака достаточно быстрая и регулярно «подогревает» соединения.
3. **TCP SYN Cookies не помогают:** атакующая машина честно завершает TCP-рукопожатие, поэтому для операционной системы соединения выглядят легитимными, и штатная защита ядра от SYN-флуда здесь бессильна.

## Атака 2: HTTP-флуд (siege)

siege http://10.62.239.150/info.php -d1 -r20000 -c100000

Результат: сервер **не упал**. Из лога `access.log`:
- все запросы получили код **200 OK**;
- размер ответа стабильно 22235–22243 байт (вес `info.php`);
- источник атаки чётко виден по IP (`10.57.64.114`) и User-Agent (`Siege/4.1.6`).
<img width="599" height="520" alt="изображение" src="https://github.com/user-attachments/assets/c7e30b57-89d3-4ff2-82a3-5539ee5904a8" />


Сводка после остановки атаки: 5780 успешных транзакций, 0 ошибок, доступность 100%.
<img width="296" height="179" alt="изображение" src="https://github.com/user-attachments/assets/e9addbf8-2ccf-4843-a914-9461b6769154" />


**Вывод:** обычный HTTP-флуд без удержания соединений (как в Slowloris) Nginx парирует штатно — он успевает полностью обслуживать каждый запрос.

## Защита: rate limiting
После настройки `limit_req_zone` / `limit_req` в конфигурации Nginx повторный запуск siege с теми же параметрами начал получать блокировки — в `error.log`:

2026/05/19 22:24:12 [error] 9017#9017: *1092 limiting requests, excess: 5.186 by zone "peripreq", client: 10.57.64.114, server: _, request: "GET /info.php HTTP/1.1", host: "10.62.239.150"

Это означает, что клиент превысил допустимую скорость запросов и Nginx принудительно сбросил его запрос по правилу зоны `peripreq`.

> ⚠️ **Дополнить:** вставить сюда точный блок `limit_req_zone ... ; limit_req zone=peripreq burst=... nodelay;` из конфига, и сводку по итоговому прогону siege после включения лимитов (доступность, число заблокированных/успешных запросов).

## Взгляд защитника (детектирование)
Признаки атаки Slowloris в логах:
- Резкий рост числа `connected` в статистике `slowhttptest` / одновременных соединений к серверу с одного или нескольких IP.
- Множество долго не закрывающихся соединений от одного клиента при низком объёме переданных данных.

Признаки HTTP-флуда / brute-force по скорости в `error.log`:
- Серии записей `limiting requests, excess: N by zone` от одного `client` — верный признак превышения `limit_req`.
- Нетипичный `User-Agent` (например, `Siege/4.1.6`, `slowhttptest`), которого нет в типичном профиле легитимного трафика.

Рекомендация по детектированию в SIEM: правило на количество соединений/запросов с одного IP в единицу времени, превышающее пороговое значение, плюс алерт на появление записей `limiting requests` в `error.log` Nginx.
