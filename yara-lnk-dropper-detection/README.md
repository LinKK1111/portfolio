# Детектирование вредоносного LNK-дроппера с помощью YARA

**Категория:** Digital Forensics & Threat Detection (DFIR)  
**Инструменты:** `YARA 4.5.5`, `strings`, `Kali Linux`, `MalwareBazaar`

## 🎯 Цель кейса
Проанализировать реальный образец вредоносного ярлыка Windows (LNK-дроппер), извлечь хостовые индикаторы компрометации (IoC) и написать кастомное `YARA`-правило для его обнаружения, сравнив эффективность с публичными базами сигнатур.

## 🔍 Исследуемый образец
* **Имя файла:** `DOC_Ef1xaU.lnk` (Windows Shortcut)
* **SHA256:** `ef7d83d56f7f9e13b05b5d6de77bf15d3645efc86cc5b3bfd4f3bc8a3497b196`
* **Источник:** MalwareBazaar (`abuse.ch`)

## 🛠 Ход анализа и результаты

### 1. Статический анализ (`strings`)
При извлечении строк из бинарного LNK-файла выявлены артефакты, указывающие на запуск командной строки Windows с последующим обращением к браузеру для загрузки полезной нагрузки (payload):
* `C:\Windows\System32\cmd.exe` / `cmd.exe`
* `%ProgramFiles%\Microsoft\Edge\Application\msedge.exe`
* `desktop-ke013jj` (имя сборочного/скомпрометированного хоста)
<img width="652" height="398" alt="изображение" src="https://github.com/user-attachments/assets/559f9bcc-0e35-4ca5-bf48-40d22a16102a" />


### 2. Проверка по публичным базам Yara-Rules
Сканирование образца по категориям `malware/`, `maldocs/` и `apt/` из репозитория `Yara-Rules/rules` не дало срабатываний — публичные базы в основном ориентированы на PE-файлы и офисные документы конкретных APT-группировок.

### 3. Разработка кастомного YARA-правила (`lnk_detected.yar`)
На основе выявленных строк создано собственное правило для детекта подозрительных LNK-файлов, вызывающих `cmd.exe` в связке с системным путём или браузером:
<img width="706" height="542" alt="изображение" src="https://github.com/user-attachments/assets/75113882-4cc9-4568-899f-8163be4c99ac" />
Результат:
<img width="716" height="190" alt="изображение" src="https://github.com/user-attachments/assets/786250de-3ea6-4efb-b682-a06389823407" />
