# Выявление элементов инфраструктуры, затронутых инцидентом (THOR Lite)

**Категория:** Incident Response & Endpoint Forensics  
**Инструменты:** `THOR Lite`, `Windows 10`, `MalwareBazaar`

## Цель работы
Изучить методы обнаружения вредоносного программного обеспечения и отработать практический анализ системы с помощью сканера компрометации `THOR Lite`.

## Ход работы
1. **Подготовка стенда:** развёрнута виртуальная машина с ОС `Windows 10`.
2. **Имитация заражения:** загружен и запущен тестовый образец вредоносного ПО (троян) из базы `MalwareBazaar`.
3. **Настройка сканера:** загружена и настроена утилита `THOR Lite` (подключён файл лицензии).
4. **Запуск сканирования:** выполнена проверка системы из командной строки:
   ```cmd
   thor-lite.exe --quick --lookback 150 
   ```
   
   <img width="981" height="515" alt="изображение" src="https://github.com/user-attachments/assets/989c69f1-5665-4942-95cc-fc8330bf73db" />
   <img width="1903" height="662" alt="изображение" src="https://github.com/user-attachments/assets/532db8f4-2a6b-4e57-b1bd-d34fa3c65335" />
   <img width="1886" height="763" alt="изображение" src="https://github.com/user-attachments/assets/53f49461-710a-484d-85c3-618012c6ad78" />


