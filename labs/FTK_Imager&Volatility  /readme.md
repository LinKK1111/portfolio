# Инструменты снятия данных: дамп ОЗУ и анализ памяти (FTK Imager & Volatility)

**Категория:** Memory Forensics & Incident Response  
**Инструменты:** `FTK Imager`, `Volatility / Volatility 3`, `Windows`

## Цель работы
Изучить инструменты создания дампов оперативной памяти (`FTK Imager`) и криминалистического анализа памяти (`Volatility`), а также освоить базовые команды для получения информации о системе, просмотра списка процессов, анализа сетевых подключений и извлечения артефактов.

## Ход работы

### 1. Получение дампа оперативной памяти
С помощью утилиты `FTK Imager` снят дамп оперативной памяти исследуемой системы в файл (`memdump.mem`) для дальнейшего анализа.

<img width="800" height="604" alt="изображение" src="https://github.com/user-attachments/assets/886228b0-6984-49d0-bac1-7f836408f724" />

### 2. Сканирование и анализ дампа памяти
Для поиска потенциально подозрительных участков памяти и признаков внедрения кода (например, регионов с правами `PAGE_EXECUTE_READWRITE`) выполнен запуск сканирования через `Volatility`:
<img width="964" height="253" alt="изображение" src="https://github.com/user-attachments/assets/dba24101-a483-4e65-9add-9d04b6e0a3b1" />
<img width="838" height="438" alt="изображение" src="https://github.com/user-attachments/assets/981cf1fb-be84-4643-9acc-ebb5f71a3ecf" />
<img width="1341" height="610" alt="изображение" src="https://github.com/user-attachments/assets/8c2c94ef-3107-4117-b9d3-efc570a6bb7b" />

