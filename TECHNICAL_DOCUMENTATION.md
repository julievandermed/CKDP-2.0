# Техническая документация: Анализатор трудозатрат

## Обзор проекта

### Назначение
Веб-приложение для комплексного анализа трудозатрат сотрудников с возможностью загрузки данных СSV, их обработки и формирования структурированных отчётов в CSV формате.

### Технологический стек
- **Frontend**: HTML5, CSS3, JavaScript ES6+
- **Backend Logic**: Python 3.11+ (Pyodide)
- **Data Processing**: Pandas 1.5+, OpenPyXL 3.0+
- **Runtime**: WebAssembly (WASM) через Pyodide
- **Browser APIs**: FileReader, Blob, LocalStorage

### Архитектурный подход
Гибридная архитектура, объединяющая клиентское приложение с серверной логикой, выполняемой непосредственно в браузере через WebAssembly. Это обеспечивает полную автономность приложения без необходимости внешнего сервера.

### Ключевые особенности
- **Офлайн-работа**: Все вычисления выполняются локально
- **Кроссплатформенность**: Работает в любом современном браузере
- **Масштабируемость**: Обработка больших объёмов данных
- **Безопасность**: Данные не покидают устройство пользователя

---

## Архитектура системы

### Общая архитектура

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           Браузер (Клиент)                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│  Presentation Layer  │  Business Logic  │  Data Access  │  State Management │
│  ┌─────────────────┐ │ ┌──────────────┐ │ ┌───────────┐ │ ┌──────────────┐  │
│  │ HTML Interface  │ │ │ JavaScript   │ │ │ File I/O  │ │ │ LocalStorage │  │
│  │ CSS Styling     │ │ │ Controller   │ │ │ Validation│ │ │ Session      │  │
│  │ Responsive UI   │ │ │ Event Handler│ │ │ Processing│ │ │ Cache        │  │
│  └─────────────────┘ │ └──────────────┘ │ └───────────┘ │ └──────────────┘  │
├─────────────────────────────────────────────────────────────────────────────┤
│                    Pyodide Runtime (WebAssembly)                            │
│  ┌────────────────────────────────────────────────────────────────────────┐ │
│  │ Python 3.11+ Interpreter  │  Package Manager  │  Memory Management     │ │
│  └────────────────────────────────────────────────────────────────────────┘ │
├─────────────────────────────────────────────────────────────────────────────┤
│                        Data Processing Layer                                │
│  ┌─────────────────┐ ┌─────────────────┐ ┌────────────────┐ ┌─────────────┐ │
│  │ Pandas Engine   │ │ OpenPyXL Engine │ │ NumPy Arrays   │ │ File Utils  │ │
│  │ DataFrames      │ │ XLSX Processing │ │ Mathematical   │ │ I/O Helpers │ │
│  │ Data Analysis   │ │ Format Support  │ │ Operations     │ │ Validation  │ │
│  └─────────────────┘ └─────────────────┘ └────────────────┘ └─────────────┘ │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Детальный поток данных

```
┌─────────────┐    ┌──────────────┐    ┌─────────────┐    ┌─────────────┐
│   Файлы     │───▶│  Загрузка    │───▶│ Конвертация │───▶│ Обработка   │
│ (CSV/XLSX)  │    │              │    │             │    │             │
└─────────────┘    └──────────────┘    └─────────────┘    └─────────────┘
                           │                    │                  │
                           ▼                    ▼                  ▼
                    ┌──────────────┐    ┌─────────────┐    ┌─────────────┐
                    │ FileReader   │    │ Pyodide     │    │ Pandas      │
                    │ API          │    │ Runtime     │    │ Algorithms  │
                    └──────────────┘    └─────────────┘    └─────────────┘
                                                                    │
                                                                    ▼
                    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
                    │   Отчёт     │◀───│ Генерация   │◀───│ Результат   │
                    │   (CSV)     │    │             │    │             │
                    └─────────────┘    └─────────────┘    └─────────────┘
                           │                    │                  │
                           ▼                    ▼                  ▼
                    ┌──────────────┐    ┌─────────────┐    ┌─────────────┐
                    │ Download     │    │ CSV Format  │    │ Data        │
                    │ API          │    │ with BOM    │    │ Validation  │
                    └──────────────┘    └─────────────┘    └─────────────┘
```

### Компонентная архитектура

#### 1. Presentation Layer (Слой представления)
- **HTML Structure**: Семантическая разметка с accessibility поддержкой
- **CSS Framework**: Custom CSS с CSS Grid и Flexbox
- **Responsive Design**: Mobile-first подход с breakpoints
- **UI Components**: Модульные компоненты интерфейса

#### 2. Business Logic Layer (Слой бизнес-логики)
- **Event Management**: Обработка пользовательских событий
- **State Management**: Управление состоянием приложения
- **Validation Logic**: Валидация входных данных
- **Error Handling**: Централизованная обработка ошибок

#### 3. Data Processing Layer (Слой обработки данных)
- **File Processing**: Обработка различных форматов файлов
- **Data Transformation**: Преобразование и нормализация данных
- **Algorithm Engine**: Вычислительные алгоритмы
- **Report Generation**: Генерация итоговых отчётов

#### 4. Runtime Layer (Слой выполнения)
- **Pyodide Engine**: WebAssembly Python runtime
- **Package Management**: Управление Python пакетами
- **Memory Management**: Оптимизация использования памяти
- **Cross-language Bridge**: Мост между JavaScript и Python

---

## Структура проекта

```
CKDP/
├── CKDP.html               # Основное приложение
├── README.md               # Пользовательская документация
└── TECHNICAL_DOCUMENTATION.md  # Техническая документация
```

---

## Технические компоненты

### 1. Инициализация Pyodide

#### Основная функция инициализации
```javascript
async function initializePyodide() {
    showNotification('Инициализация Python...', 'warning');
    try {
        // Загрузка основного движка Pyodide
        pyodide = await loadPyodide({
            indexURL: "https://cdn.jsdelivr.net/pyodide/v0.24.1/full/"
        });
        console.log('Pyodide загружен');
        
        // Загрузка обязательных пакетов
        await pyodide.loadPackage(['pandas']);
        console.log('Пакет pandas загружен');
        
        // Условная загрузка OpenPyXL
        try {
            await pyodide.loadPackage(['openpyxl']);
            console.log('Пакет openpyxl загружен для конвертации XLSX');
            window.xlsxConversionAvailable = true;
        } catch (e) {
            console.log('openpyxl недоступен, конвертация XLSX будет ограничена');
            window.xlsxConversionAvailable = false;
        }
        
        // Загрузка пользовательского Python кода
        await loadPythonCode();
        showNotification('Python готов к работе', 'success');
    } catch (error) {
        console.error('Ошибка инициализации Pyodide:', error);
        showNotification('Ошибка инициализации Python: ' + error.message, 'error');
    }
}
```

#### Детали инициализации
- **Асинхронная загрузка**: Использование async/await для неблокирующей загрузки
- **CDN интеграция**: Загрузка Pyodide с jsdelivr CDN для быстрого доступа
- **Условная загрузка пакетов**: Graceful degradation при недоступности библиотек
- **Глобальные флаги**: Установка `window.xlsxConversionAvailable` для контроля функций
- **Обработка ошибок**: Детальное логирование и пользовательские уведомления

#### Загрузка Python кода
```javascript
async function loadPythonCode() {
    const pythonCode = `
        import pandas as pd
        import io
        import re
        from typing import Dict, List, Tuple, Any
        
        # Основные функции обработки данных
        def normalize_name(name):
            # Детальная реализация нормализации имён
            pass
            
        def process_files(file1_content, file2_content, file3_content, file4_content, year, month, work_days, work_hours):
            # Основная логика обработки
            pass
            
        def create_csv_report(df):
            # Генерация отчёта
            pass
    `;
    
    await pyodide.runPython(pythonCode);
    console.log('Python код загружен и готов к выполнению');
}
```

### 2. Обработка файлов

#### Система загрузки файлов

##### Основная функция обработки загрузки
```javascript
async function handleFileUpload(fileNumber, input) {
    const file = input.files[0];
    if (!file) return;
    
    const area = document.getElementById(`file${fileNumber}-area`);
    const status = document.getElementById(`file${fileNumber}-status`);
    
    try {
        showNotification(`Загружаем файл ${fileNumber}...`, 'warning');
        
        // Определение типа файла и выбор стратегии обработки
        if (file.name.endsWith('.csv') || fileNumber === 1) {
            // Обработка CSV файлов
            await handleCsvFile(file, fileNumber, area, status);
        } else if (file.name.endsWith('.xlsx')) {
            // Обработка XLSX файлов
            await handleXlsxFile(file, fileNumber, area, status);
        } else {
            throw new Error('Неподдерживаемый формат файла. Используйте .xlsx или .csv');
        }
        
        checkReadyToProcess();
    } catch (error) {
        console.error(`Ошибка загрузки файла ${fileNumber}:`, error);
        showNotification(`Ошибка загрузки файла ${fileNumber}: ${error.message}`, 'error');
    }
}
```

##### Обработка CSV файлов
```javascript
async function handleCsvFile(file, fileNumber, area, status) {
    return new Promise((resolve, reject) => {
        const reader = new FileReader();
        
        reader.onload = function(e) {
            try {
                // Валидация содержимого CSV
                const content = e.target.result;
                if (!content || content.trim().length === 0) {
                    throw new Error('Файл пуст или повреждён');
                }
                
                // Сохранение в глобальном хранилище
                uploadedFiles[`file${fileNumber}`] = content;
                area.classList.add('uploaded');
                status.style.display = 'block';
                
                showNotification(`Файл ${fileNumber} загружен успешно`, 'success');
                resolve(content);
            } catch (error) {
                reject(error);
            }
        };
        
        reader.onerror = function() {
            reject(new Error('Ошибка чтения файла'));
        };
        
        // Чтение файла как текст с кодировкой UTF-8
        reader.readAsText(file, 'UTF-8');
    });
}
```

##### Обработка XLSX файлов
```javascript
async function handleXlsxFile(file, fileNumber, area, status) {
    // Проверка доступности конвертации
    if (!window.xlsxConversionAvailable) {
        throw new Error('Конвертация XLSX недоступна. Пожалуйста, сконвертируйте файл в CSV вручную или дождитесь полной инициализации.');
    }
    
    showNotification(`Конвертируем XLSX файл ${fileNumber} в CSV...`, 'warning');
    
    try {
        // Чтение файла как ArrayBuffer
        const arrayBuffer = await readFileAsArrayBuffer(file);
        
        // Конвертация через Pyodide
        const csvContent = await convertXlsxToCsv(arrayBuffer, fileNumber);
        
        // Сохранение результата
        uploadedFiles[`file${fileNumber}`] = csvContent;
        area.classList.add('uploaded');
        status.style.display = 'block';
        
        showNotification(`Файл ${fileNumber} сконвертирован и загружен`, 'success');
    } catch (error) {
        throw new Error(`Ошибка конвертации XLSX: ${error.message}`);
    }
}
```

##### Вспомогательная функция чтения ArrayBuffer
```javascript
function readFileAsArrayBuffer(file) {
    return new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.onload = (e) => resolve(e.target.result);
        reader.onerror = () => reject(new Error('Ошибка чтения файла как ArrayBuffer'));
        reader.readAsArrayBuffer(file);
    });
}
```

#### Конвертация XLSX → CSV

##### JavaScript функция конвертации
```javascript
async function convertXlsxToCsv(arrayBuffer, fileNumber) {
    try {
        // Преобразование ArrayBuffer в Uint8Array
        const uint8Array = new Uint8Array(arrayBuffer);
        
        // Вызов Python функции через Pyodide
        const result = await pyodide.runPython(`
            import io
            import pandas as pd
            
            # Конвертация JavaScript Uint8Array в Python bytes
            xlsx_data = bytes(js_proxy_object.to_py())
            
            # Чтение XLSX файла
            df = pd.read_excel(io.BytesIO(xlsx_data))
            
            # Конвертация в CSV
            csv_content = df.to_csv(index=False, encoding='utf-8')
            csv_content
        `.replace('js_proxy_object', `js('${uint8Array}')`));
        
        return result;
    } catch (error) {
        console.error('Ошибка конвертации XLSX:', error);
        throw new Error(`Не удалось конвертировать XLSX файл: ${error.message}`);
    }
}
```

##### Python функция конвертации (встроенная)
```python
def convert_xlsx_to_csv(xlsx_data, file_number):
    """
    Конвертирует XLSX данные в CSV формат
    
    Args:
        xlsx_data: bytes - данные XLSX файла
        file_number: int - номер файла для логирования
    
    Returns:
        str - CSV содержимое
    """
    try:
        # Создание DataFrame из XLSX
        df = pd.read_excel(io.BytesIO(xlsx_data))
        
        # Очистка данных
        df = df.dropna(how='all')  # Удаление полностью пустых строк
        df = df.fillna('')         # Заполнение NaN значений
        
        # Конвертация в CSV
        csv_content = df.to_csv(
            index=False, 
            encoding='utf-8',
            sep=','
        )
        
        print(f"Файл {file_number} успешно сконвертирован: {len(df)} строк, {len(df.columns)} колонок")
        return csv_content
        
    except Exception as e:
        print(f"Ошибка конвертации файла {file_number}: {str(e)}")
        raise
```

### 3. Алгоритм обработки данных

#### Структура входных данных

| Файл | Назначение | Обязательные колонки | Опциональные колонки | Формат |
|------|------------|---------------------|---------------------|---------|
| 1 | Список сотрудников | ФИО, Должность | - | CSV |
| 2 | Прошлый месяц | ФИО, Остаток за позапрошлый месяц | Кол-во рабочих дней | XLSX/CSV |
| 3 | Текущий месяц | ФИО, Total (время работы) | - | XLSX/CSV |
| 4 | Отгулы | User, Total (время отгулов) | - | XLSX/CSV |

#### Основная функция обработки

```python
def process_files(file1_content, file2_content, file3_content, file4_content, year, month, work_days, work_hours):
    """
    Основная функция обработки всех файлов и расчёта итогов
    
    Args:
        file1_content: str - CSV содержимое файла сотрудников
        file2_content: str - CSV содержимое файла прошлого месяца
        file3_content: str - CSV содержимое файла текущего месяца
        file4_content: str - CSV содержимое файла отгулов
        year: int - Год для расчётов
        month: int - Месяц для расчётов
        work_days: int - Количество рабочих дней
        work_hours: float - Количество рабочих часов в месяц
    
    Returns:
        tuple: (result_df, errors) - DataFrame с результатами и список ошибок
    """
    errors = []
    
    try:
        # 1. Загрузка и валидация файлов
        df1 = load_and_validate_file(file1_content, "сотрудники", ["ФИО", "Должность"])
        df2 = load_and_validate_file(file2_content, "прошлый месяц", ["ФИО"])
        df3 = load_and_validate_file(file3_content, "текущий месяц", ["ФИО", "Total"])
        df4 = load_and_validate_file(file4_content, "отгулы", ["User", "Total"])
        
        # 2. Создание базового DataFrame
        result_df = create_base_dataframe(df1, work_days, work_hours)
        
        # 3. Обработка каждого сотрудника
        for idx, row in result_df.iterrows():
            employee_name = row['ФИО']
            employee_errors = []
            
            # Поиск данных в файле 2 (прошлый месяц)
            process_previous_month_data(df2, result_df, idx, employee_name, employee_errors)
            
            # Поиск данных в файле 3 (текущий месяц)
            process_current_month_data(df3, result_df, idx, employee_name, employee_errors)
            
            # Поиск данных в файле 4 (отгулы)
            process_vacation_data(df4, result_df, idx, employee_name, employee_errors)
            
            # Расчёт итога за текущий месяц
            calculate_monthly_total(result_df, idx, work_hours, employee_errors)
            
            # Запись ошибок для сотрудника
            if employee_errors:
                result_df.at[idx, 'Ошибки'] = "; ".join(employee_errors)
        
        return result_df, errors
        
    except Exception as e:
        errors.append(f"Критическая ошибка обработки: {str(e)}")
        return pd.DataFrame(), errors
```

#### Алгоритм нормализации имён

```python
def normalize_name(name):
    """
    Нормализация имени для сопоставления
    
    Args:
        name: str - Исходное имя
    
    Returns:
        str - Нормализованное имя
    """
    if pd.isna(name) or name == '':
        return ''
    
    # 1. Приведение к строке и удаление лишних пробелов
    normalized = str(name).strip()
    
    # 2. Приведение к заглавным буквам
    normalized = normalized.upper()
    
    # 3. Удаление множественных пробелов
    normalized = re.sub(r'\s+', ' ', normalized)
    
    # 4. Транслитерация кириллицы в латиницу
    cyrillic_to_latin = {
        'А': 'A', 'Б': 'B', 'В': 'V', 'Г': 'G', 'Д': 'D', 'Е': 'E', 'Ё': 'E',
        'Ж': 'ZH', 'З': 'Z', 'И': 'I', 'Й': 'Y', 'К': 'K', 'Л': 'L', 'М': 'M',
        'Н': 'N', 'О': 'O', 'П': 'P', 'Р': 'R', 'С': 'S', 'Т': 'T', 'У': 'U',
        'Ф': 'F', 'Х': 'KH', 'Ц': 'TS', 'Ч': 'CH', 'Ш': 'SH', 'Щ': 'SHCH',
        'Ъ': '', 'Ы': 'Y', 'Ь': '', 'Э': 'E', 'Ю': 'YU', 'Я': 'YA'
    }
    
    for cyr, lat in cyrillic_to_latin.items():
        normalized = normalized.replace(cyr, lat)
    
    # 5. Удаление специальных символов
    normalized = re.sub(r'[^\w\s]', '', normalized)
    
    return normalized

def find_name_match(target_name, df, name_column):
    """
    Поиск совпадения имени в DataFrame
    
    Args:
        target_name: str - Целевое имя для поиска
        df: pd.DataFrame - DataFrame для поиска
        name_column: str - Название колонки с именами
    
    Returns:
        pd.DataFrame - Строки с совпадениями
    """
    if df.empty or name_column not in df.columns:
        return pd.DataFrame()
    
    # 1. Точное совпадение
    exact_match = df[df[name_column].str.upper() == target_name.upper()]
    if not exact_match.empty:
        return exact_match
    
    # 2. Совпадение после нормализации
    df_normalized = df.copy()
    df_normalized['normalized'] = df_normalized[name_column].apply(normalize_name)
    target_normalized = normalize_name(target_name)
    
    normalized_match = df_normalized[df_normalized['normalized'] == target_normalized]
    if not normalized_match.empty:
        return normalized_match.drop('normalized', axis=1)
    
    # 3. Частичное совпадение (содержит целевое имя)
    partial_match = df[df[name_column].str.contains(target_name, case=False, na=False)]
    if not partial_match.empty:
        return partial_match
    
    # 4. Обратное частичное совпадение (целевое имя содержит)
    reverse_match = df[df[name_column].apply(lambda x: target_name.lower() in str(x).lower() if pd.notna(x) else False)]
    if not reverse_match.empty:
        return reverse_match
    
    return pd.DataFrame()
```

#### Обработка данных по файлам

##### Файл 2 (Прошлый месяц)
```python
def process_previous_month_data(df2, result_df, idx, employee_name, employee_errors):
    """Обработка данных прошлого месяца"""
    if not df2.empty and 'ФИО' in df2.columns:
        match_rows = find_name_match(employee_name, df2, 'ФИО')
        
        if len(match_rows) == 1:
            # Точное совпадение - берём значение
            row = match_rows.iloc[0]
            if 'Остаток за позапрошлый месяц' in row:
                val = row['Остаток за позапрошлый месяц']
                if pd.notna(val) and val != '':
                    try:
                        previous_balance = float(val)
                        result_df.at[idx, 'Остаток за позапрошлый месяц'] = previous_balance
                    except ValueError:
                        employee_errors.append("Некорректное значение остатка")
        elif len(match_rows) > 1:
            # Множественные совпадения - ошибка
            employee_errors.append("Найдено несколько совпадений в файле прошлого месяца")
        else:
            # Совпадений нет - оставляем 0
            pass
```

##### Файл 3 (Текущий месяц)
```python
def process_current_month_data(df3, result_df, idx, employee_name, employee_errors):
    """Обработка данных текущего месяца"""
    if not df3.empty and 'ФИО' in df3.columns and 'Total' in df3.columns:
        match_rows = find_name_match(employee_name, df3, 'ФИО')
        
        if len(match_rows) == 1:
            row = match_rows.iloc[0]
            total_time = 0
            
            # Суммирование всех числовых колонок
            for col in df3.columns:
                if col not in ['ФИО']:
                    val = row[col]
                    if pd.notna(val) and val != '':
                        try:
                            total_time += float(val)
                        except (ValueError, TypeError):
                            pass
            
            result_df.at[idx, 'Время без отгулов'] = total_time
        elif len(match_rows) > 1:
            employee_errors.append("Найдено несколько совпадений в файле текущего месяца")
```

##### Файл 4 (Отгулы)
```python
def process_vacation_data(df4, result_df, idx, employee_name, employee_errors):
    """Обработка данных об отгулах"""
    if not df4.empty and 'User' in df4.columns and 'Total' in df4.columns:
        match_rows = find_name_match(employee_name, df4, 'User')
        
        if len(match_rows) == 1:
            row = match_rows.iloc[0]
            total_vacation = 0
            
            # Суммирование всех числовых колонок
            for col in df4.columns:
                if col not in ['User']:
                    val = row[col]
                    if pd.notna(val) and val != '':
                        try:
                            total_vacation += float(val)
                        except (ValueError, TypeError):
                            pass
            
            result_df.at[idx, 'Отгулы'] = total_vacation
        elif len(match_rows) > 1:
            employee_errors.append("Найдено несколько совпадений в файле отгулов")
```

#### Формула расчёта итога

```python
def calculate_monthly_total(result_df, idx, work_hours, employee_errors):
    """
    Расчёт итога за текущий месяц по формуле:
    Итог = N - Время без отгулов - Остаток за позапрошлый месяц + Отгулы
    
    Args:
        result_df: pd.DataFrame - DataFrame с данными
        idx: int - Индекс строки сотрудника
        work_hours: float - N (рабочие часы в месяц)
        employee_errors: list - Список ошибок для сотрудника
    """
    try:
        # Получение значений из DataFrame
        remainder = result_df.at[idx, 'Остаток за позапрошлый месяц'] or 0
        time_worked = result_df.at[idx, 'Время без отгулов'] or 0
        time_off = result_df.at[idx, 'Отгулы'] or 0
        
        # Преобразование в числа с обработкой NaN
        remainder = float(remainder) if pd.notna(remainder) else 0
        time_worked = float(time_worked) if pd.notna(time_worked) else 0
        time_off = float(time_off) if pd.notna(time_off) else 0
        
        # Применение формулы
        total = work_hours - time_worked - remainder + time_off
        
        # Сохранение результата
        result_df.at[idx, 'Итог за текущий месяц'] = total
        
        # Логирование для отладки
        print(f"Расчёт для {result_df.at[idx, 'ФИО']}: {work_hours} - {time_worked} - {remainder} + {time_off} = {total}")
        
    except Exception as e:
        employee_errors.append(f"Ошибка расчёта итога: {str(e)}")
```

**Математическая интерпретация формулы:**
```
Итог за текущий месяц = N - Время без отгулов - Остаток за позапрошлый месяц + Отгулы
```

**Где:**
- `N` - Нормативные рабочие часы в месяц (пользовательский ввод)
- `Время без отгулов` - Фактически отработанное время (файл 3)
- `Остаток за позапрошлый месяц` - Неиспользованные часы с прошлого месяца (файл 2)
- `Отгулы` - Время отгулов (файл 4)

**Логика формулы:**
- Положительный результат = переработка
- Отрицательный результат = недоработка
- Нулевой результат = точное выполнение нормы

### 4. Генерация отчёта

#### Структура итогового CSV

```python
def create_csv_report(df):
    """
    Создание итогового CSV отчёта
    
    Args:
        df: pd.DataFrame - DataFrame с обработанными данными
    
    Returns:
        str - CSV содержимое отчёта
    """
    try:
        # Заголовки колонок
        headers = ['ФИО', 'Должность', 'Кол-во рабочих дней', 
                  'Остаток за позапрошлый месяц', 'Время без отгулов', 
                  'Отгулы', 'Итог за текущий месяц', 'N (рабочие часы в месяц)', 'Ошибки']
        
        # Подготовка DataFrame для экспорта
        df_export = df.copy()
        
        # Переименование колонки с именами
        df_export = df_export.rename(columns={'Имя': 'ФИО'})
        
        # Выбор только нужных колонок
        df_export = df_export[headers]
        
        # Конвертация в CSV с правильными параметрами
        csv_content = df_export.to_csv(
            index=False, 
            sep=';',           # Разделитель для Excel
            encoding='utf-8'   # Кодировка UTF-8
        )
        
        return csv_content
        
    except Exception as e:
        print(f"Ошибка создания отчёта: {str(e)}")
        raise
```

#### Форматирование для Excel

```javascript
function downloadReport() {
    try {
        // Добавление BOM для корректного отображения кириллицы в Excel
        const bom = '\uFEFF';
        const csvContent = bom + reportData;
        
        // Создание Blob с правильным MIME типом
        const blob = new Blob([csvContent], { 
            type: 'text/csv;charset=utf-8;' 
        });
        
        // Создание ссылки для скачивания
        const link = document.createElement('a');
        const url = URL.createObjectURL(blob);
        
        link.setAttribute('href', url);
        link.setAttribute('download', `отчет_трудозатрат_${new Date().toISOString().split('T')[0]}.csv`);
        link.style.visibility = 'hidden';
        
        // Добавление в DOM и клик
        document.body.appendChild(link);
        link.click();
        document.body.removeChild(link);
        
        // Очистка URL
        URL.revokeObjectURL(url);
        
        showNotification('Отчёт успешно скачан', 'success');
    } catch (error) {
        console.error('Ошибка скачивания отчёта:', error);
        showNotification('Ошибка скачивания отчёта: ' + error.message, 'error');
    }
}
```

---

## Обработка ошибок

### Многоуровневая система обработки ошибок

#### 1. JavaScript уровень (Frontend)
```javascript
// Валидация файлов
function validateFile(file, fileNumber) {
    const errors = [];
    
    // Проверка размера файла (максимум 10MB)
    if (file.size > 10 * 1024 * 1024) {
        errors.push('Файл слишком большой (максимум 10MB)');
    }
    
    // Проверка типа файла
    const allowedTypes = ['.csv', '.xlsx'];
    const fileExtension = file.name.toLowerCase().substring(file.name.lastIndexOf('.'));
    if (!allowedTypes.includes(fileExtension)) {
        errors.push('Неподдерживаемый формат файла');
    }
    
    return errors;
}
```

#### 2. Python уровень (Data Processing)
```python
def safe_float_conversion(value, default=0.0):
    """Безопасное преобразование в float с обработкой ошибок"""
    try:
        if pd.isna(value) or value == '':
            return default
        return float(value)
    except (ValueError, TypeError):
        return default

def validate_dataframe(df, required_columns, file_name):
    """Валидация DataFrame с детальными ошибками"""
    errors = []
    
    if df.empty:
        errors.append(f"Файл {file_name} пуст")
        return errors
    
    # Проверка обязательных колонок
    missing_columns = [col for col in required_columns if col not in df.columns]
    if missing_columns:
        errors.append(f"Отсутствуют обязательные колонки в {file_name}: {', '.join(missing_columns)}")
    
    return errors
```

---

## Жизненный цикл приложения

### Детальный процесс выполнения

#### 1. Инициализация (Startup)
```
Загрузка страницы → Инициализация Pyodide → Загрузка пакетов → Готовность
```

#### 2. Загрузка данных (Data Loading)
```
Выбор файлов → Валидация → Конвертация XLSX → Сохранение в памяти
```

#### 3. Обработка данных (Data Processing)
```
Передача в Python → Загрузка файлов → Обработка сотрудников → Расчёт итогов
```

#### 4. Генерация отчёта (Report Generation)
```
Создание CSV → Добавление BOM → Скачивание → Очистка ресурсов
```

---

## Структуры данных

### Входные форматы

#### Файл 1: Список сотрудников (CSV)
```csv
ФИО,Должность
Иванов Иван Иванович,Разработчик
```

#### Файл 2: Прошлый месяц (XLSX/CSV)
```csv
ФИО,Остаток за позапрошлый месяц
Иванов Иван Иванович,8.5
```

#### Файл 3: Текущий месяц (XLSX/CSV)
```csv
ФИО,Total
Иванов Иван Иванович,160.0
```

#### Файл 4: Отгулы (XLSX/CSV)
```csv
User,Total
Иванов Иван Иванович,8.0
```

### Выходной формат

#### Итоговый отчёт (CSV)
```csv
ФИО;Должность;Кол-во рабочих дней;Остаток за позапрошлый месяц;Время без отгулов;Отгулы;Итог за текущий месяц;N (рабочие часы в месяц);Ошибки
Иванов Иван Иванович;Разработчик;22;8.5;160.0;8.0;0.5;176;
```

---

## Технические особенности

### Pyodide интеграция
- Мост JavaScript ↔ Python
- Управление памятью
- Конвертация типов данных

### Производительность
- Ленивая загрузка
- Кэширование данных
- Пакетная обработка

### Безопасность
- Валидация входных данных
- Изоляция выполнения
- Graceful degradation

---

## Заключение

### Ключевые достижения
- **Автономность**: Работа без серверов
- **Безопасность**: Данные не покидают устройство
- **Производительность**: Оптимизированная обработка
- **Надёжность**: Многоуровневая обработка ошибок

### Технические особенности
- Гибридная архитектура JavaScript + Python
- Продвинутые алгоритмы обработки данных
- Современный пользовательский интерфейс
- Детальная диагностика и логирование

---

## Пользовательский интерфейс

### CSS архитектура

#### CSS переменные и тема
```css
:root {
    /* Основные цвета */
    --primary-color: #f7fafc;
    --secondary-color: #edf2f7;
    --accent-color: #4299e1;
    --success-color: #68d391;
    --error-color: #fc8181;
    --warning-color: #f6e05e;
    
    /* Дополнительные цвета */
    --text-primary: #2d3748;
    --text-secondary: #718096;
    --border-color: #e2e8f0;
    --shadow-color: rgba(0, 0, 0, 0.1);
    
    /* Интерактивные состояния */
    --button-hover: #3182ce;
    --input-focus: #4299e1;
    --transition-speed: 0.3s;
    
    /* Типографика */
    --font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'San Francisco', sans-serif;
    --font-size-base: 16px;
    --line-height-base: 1.5;
}
```

#### Компоненты UI

##### Загрузка файлов
```css
.file-upload-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
    gap: 20px;
    margin-bottom: 30px;
}

.file-upload-item {
    border: 2px dashed var(--border-color);
    border-radius: 12px;
    padding: 25px;
    text-align: center;
    transition: all var(--transition-speed) ease;
    background: white;
    position: relative;
}

.file-upload-item:hover {
    border-color: var(--accent-color);
    background: #f8fafc;
}

.file-upload-item.uploaded {
    border-color: var(--success-color);
    background: #f0fff4;
}
```

##### Параметры ввода
```css
.parameters-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
    gap: 20px;
    margin-bottom: 30px;
}

.parameter-group {
    display: flex;
    flex-direction: column;
    gap: 8px;
}

.parameter-group label {
    font-weight: 600;
    color: var(--text-primary);
    font-size: 0.9rem;
}

.parameter-group input {
    padding: 12px 16px;
    border: 2px solid var(--border-color);
    border-radius: 8px;
    font-size: var(--font-size-base);
    transition: border-color var(--transition-speed) ease;
}

.parameter-group input:focus {
    outline: none;
    border-color: var(--input-focus);
    box-shadow: 0 0 0 3px rgba(66, 153, 225, 0.1);
}
```

##### Прогресс-бар
```css
.progress-container {
    background: var(--secondary-color);
    border-radius: 10px;
    overflow: hidden;
    margin-bottom: 15px;
    height: 8px;
}

.progress-fill {
    height: 100%;
    background: linear-gradient(90deg, var(--accent-color), var(--success-color));
    width: 0%;
    transition: width 0.3s ease;
    border-radius: 10px;
}
```

##### Кнопки действий
```css
.action-buttons {
    display: flex;
    gap: 15px;
    justify-content: center;
    margin-top: 40px;
    flex-wrap: wrap;
}

.btn {
    padding: 15px 30px;
    border: none;
    border-radius: 10px;
    font-weight: 500;
    font-size: 1rem;
    cursor: pointer;
    transition: all var(--transition-speed) ease;
    text-decoration: none;
    display: inline-flex;
    align-items: center;
    gap: 10px;
    min-width: 160px;
    justify-content: center;
}

.btn-primary {
    background: var(--accent-color);
    color: white;
}

.btn-primary:hover:not(:disabled) {
    background: var(--button-hover);
    transform: translateY(-2px);
    box-shadow: 0 4px 12px var(--shadow-color);
}

.btn:disabled {
    background: #cbd5e0;
    cursor: not-allowed;
    transform: none;
    opacity: 0.6;
}
```

### Система уведомлений

#### Основная функция уведомлений
```javascript
function showNotification(message, type = 'info', persistent = false) {
    // Создание контейнера уведомлений
    let container = document.getElementById('notification-container');
    if (!container) {
        container = document.createElement('div');
        container.id = 'notification-container';
        container.style.cssText = `
            position: fixed;
            top: 20px;
            right: 20px;
            z-index: 1000;
            display: flex;
            flex-direction: column;
            gap: 10px;
        `;
        document.body.appendChild(container);
    }
    
    // Создание уведомления
    const notification = document.createElement('div');
    notification.className = `notification ${type}`;
    
    // Содержимое уведомления
    notification.innerHTML = `
        <div class="notification-content">${message}</div>
        <button class="notification-close" onclick="closeNotification(this)">×</button>
    `;
    
    // Добавление в контейнер
    container.appendChild(notification);
    
    // Анимация появления
    setTimeout(() => notification.classList.add('show'), 100);
    
    // Автоматическое скрытие (кроме ошибок и постоянных)
    if (!persistent && type !== 'error') {
        setTimeout(() => {
            if (notification.parentNode) {
                closeNotification(notification.querySelector('.notification-close'));
            }
        }, 5000);
    }
    
    // Интеграция с футером ошибок для длинных сообщений
    if (message.length > 200) {
        showErrorFooter(message);
    }
}
```

#### Футер ошибок
```css
.error-footer {
    position: fixed;
    bottom: 0;
    left: 0;
    right: 0;
    background: #2d3748;
    color: white;
    border-top: 3px solid var(--error-color);
    transform: translateY(100%);
    transition: transform 0.3s ease;
    z-index: 1001;
    max-height: 50vh;
    overflow: hidden;
    display: flex;
    flex-direction: column;
}

.error-footer.show {
    transform: translateY(0);
}

.error-footer-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 15px 20px;
    background: #1a202c;
    border-bottom: 1px solid #4a5568;
}

.error-footer-title {
    font-weight: 600;
    font-size: 1.1rem;
}

.error-footer-close {
    background: none;
    border: none;
    color: white;
    font-size: 1.5rem;
    cursor: pointer;
    padding: 0;
    width: 30px;
    height: 30px;
    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 50%;
    transition: background-color 0.2s ease;
}

.error-footer-close:hover {
    background: rgba(255, 255, 255, 0.1);
}

.error-footer-content {
    padding: 20px;
    overflow-y: auto;
    font-family: 'Monaco', 'Menlo', 'Ubuntu Mono', monospace;
    font-size: 0.9rem;
    line-height: 1.4;
    white-space: pre-wrap;
    word-break: break-word;
}

.error-footer-actions {
    padding: 15px 20px;
    background: #1a202c;
    border-top: 1px solid #4a5568;
    display: flex;
    gap: 10px;
    justify-content: flex-end;
}

.error-footer-btn {
    padding: 8px 16px;
    background: var(--accent-color);
    color: white;
    border: none;
    border-radius: 6px;
    cursor: pointer;
    font-size: 0.9rem;
    transition: background-color 0.2s ease;
}

.error-footer-btn:hover {
    background: var(--button-hover);
}
```

### Адаптивность

#### Медиа-запросы
```css
/* Планшеты */
@media (max-width: 768px) {
    .container {
        padding: 20px;
        margin: 10px;
    }
    
    .file-upload-grid {
        grid-template-columns: 1fr;
    }
    
    .parameters-grid {
        grid-template-columns: 1fr;
    }
    
    .action-buttons {
        flex-direction: column;
        align-items: center;
    }
    
    .btn {
        width: 100%;
        max-width: 300px;
    }
}

/* Мобильные устройства */
@media (max-width: 480px) {
    .container {
        padding: 15px;
        margin: 5px;
    }
    
    .file-upload-item {
        padding: 20px 15px;
    }
    
    .notification {
        max-width: calc(100vw - 40px);
        right: 20px;
        left: 20px;
    }
    
    .error-footer {
        max-height: 70vh;
    }
}
```
```

#### Форматирование для Excel

```javascript
function downloadReport() {
    const bom = '\uFEFF'; // BOM для корректного отображения кириллицы
    const csvContent = bom + reportData;
    // Скачивание с разделителем ';' для Excel
}
```

---

## Пользовательский интерфейс

### Компоненты UI

1. **Загрузка файлов** - Drag & Drop области
2. **Параметры** - Форма ввода (год, месяц, дни, часы)
3. **Прогресс-бар** - Визуализация обработки
4. **Уведомления** - Система обратной связи
5. **Футер ошибок** - Детальная диагностика

### Адаптивность

```css
@media (max-width: 768px) {
    .container { padding: 20px; }
    .file-upload-grid { grid-template-columns: 1fr; }
}
```

---

## Обработка ошибок

### Уровни обработки

1. **JavaScript уровень** - Валидация файлов, UI ошибки
2. **Python уровень** - Обработка данных, расчёты
3. **Пользовательский уровень** - Уведомления и футер ошибок

### Система уведомлений

```javascript
function showNotification(message, type, persistent = false) {
    // Создание уведомления с кнопкой закрытия
    // Автоматическое скрытие (кроме ошибок)
    // Интеграция с футером ошибок
}
```

### Футер ошибок

```html
<div class="error-footer" id="error-footer">
    <div class="error-footer-header">
        <div class="error-footer-title">⚠️ Подробности ошибки</div>
        <button class="error-footer-close" onclick="hideErrorFooter()">×</button>
    </div>
    <div class="error-footer-content" id="error-footer-content"></div>
</div>
```

---

## Технические особенности

### Pyodide интеграция

```javascript
// Выполнение Python кода
const result = await pyodide.runPython(`
    result_df, errors = process_files(
        file1_content, file2_content, file3_content, file4_content,
        year, month, work_days, work_hours
    )
`);
```

### Конвертация типов данных

```python
# JavaScript Uint8Array → Python bytes
xlsx_data = bytes(js_proxy_object.to_py())

# Python DataFrame → JavaScript string
csv_content = df.to_csv(index=False, encoding='utf-8')
```

### Управление памятью

- Автоматическая очистка Pyodide объектов
- Ограничение размера загружаемых файлов
- Обработка больших датасетов

---

## Диагностика и отладка

### Логирование

```javascript
console.log('Pyodide загружен');
console.log('Пакет pandas загружен');
console.log('Начинаем обработку', len(result_df), 'сотрудников...');
```

### Отслеживание ошибок

```python
try:
    # Обработка данных
    total = work_hours - time_worked - remainder + time_off
except Exception as e:
    employee_errors.append("Ошибка расчёта итога: " + str(e))
```

### Валидация данных

- Проверка форматов файлов
- Валидация колонок
- Контроль типов данных
- Обработка пустых значений

---

## Производительность

### Оптимизации

1. **Ленивая загрузка** - Pyodide загружается только при необходимости
2. **Кэширование** - Параметры сохраняются в LocalStorage
3. **Пакетная обработка** - Все файлы обрабатываются за один проход
4. **Эффективные алгоритмы** - Оптимизированное сопоставление имён

### Ограничения

- Размер файлов ограничен памятью браузера
- Производительность зависит от мощности устройства
- Pyodide имеет ограничения по библиотекам

---

## Безопасность

### Клиентская безопасность

- Все данные обрабатываются локально
- Нет передачи данных на сервер
- Валидация входных файлов

### Обработка ошибок

- Graceful degradation при недоступности библиотек
- Детальная диагностика ошибок
- Восстановление после сбоев

---

## Заключение

Приложение представляет собой полнофункциональный анализатор трудозатрат с современной архитектурой, использующий передовые веб-технологии для выполнения сложных вычислений непосредственно в браузере.

**Ключевые преимущества:**
- Полная автономность (работает без сервера)
- Современный пользовательский интерфейс
- Надёжная обработка ошибок
- Детальная диагностика и логирование

---

**Версия документации**: 2.4  
**Дата создания**: 30.08.2025
**Статус**: Актуальная
