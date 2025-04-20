# Проверка о соответствии выполненной работы ТЗ
## Формализация технического задания

В данном документе представлен формальный анализ соответствия разработанного бота-напоминания требованиям технического задания.
Функциональность системы описана в терминах конечного автомата, визуализированного с помощью детализированных блок-схем.

Отчет разделен на три ключевых раздела:
- Блок-схемы конечного автомата
- Соответствие реализации ТЗ
- Выводы

## **1. Блок-схемы конечного автомата**
### 1. Создание напоминания (Creation Flow)
```mermaid
stateDiagram-v2
    S0_MainMenu --> S1_NameInput: /add
    S1_NameInput --> S2_DateInput: text
    S2_DateInput --> S3_PeriodicPrompt: date(parse dateparser)
    S3_PeriodicPrompt --> S4_PeriodDaysInput: 'Yes'
    S3_PeriodicPrompt --> S5_FilesPrompt: 'No'
    S4_PeriodDaysInput --> S5_FilesPrompt: days(int input)
    S5_FilesPrompt --> S6_FileProcessing: 'Yes'
    S5_FilesPrompt --> S0_MainMenu: 'No'
    S6_FileProcessing --> S0_MainMenu: file(upload to S3)
```

---

### 2. Управление задачами (Management Flow)
```mermaid
stateDiagram-v2
    S0_MainMenu --> S7_UncompletedList: /list
    S7_UncompletedList --> S8_Editing: callback(reminder_<id>)
    S8_Editing --> S7_UncompletedList: 'back_to_list'
    S8_Editing --> S7a_DeleteConfirm: 'delete'
    S7a_DeleteConfirm --> S7_UncompletedList: 'confirm'
    S8_Editing --> S8a_EditName: edit_name
    S8a_EditName --> S8b_EditDate: text
    S8b_EditDate --> S8c_PeriodicEdit: date
    S8c_PeriodicEdit --> S8d_FilesEdit: config
    S8d_FilesEdit --> S0_MainMenu: confirm

    S7_UncompletedList --> S9_MarkCompleted: mark_completed_<id>
    S9_MarkCompleted --> S0_MainMenu: auto
```

---

### 3. Работа с файлами (File Processing)
```mermaid
stateDiagram-v2
    S6_FileProcessing --> F1_SingleFile: single file
    S6_FileProcessing --> F2_MediaGroup: media_group_id
    
    F2_MediaGroup --> F2a_Buffer: add to queue
    F2a_Buffer --> F2b_TimeoutCheck: last_check(10s timeout)
    F2b_TimeoutCheck --> F2c_BatchUpload: upload all(missing impl)
    F2c_BatchUpload --> S0_MainMenu
    
    F1_SingleFile --> F1a_Upload: s3.upload_file
    F1a_Upload --> S0_MainMenu
```

---

### 4. Просмотр и восстановление (List Management)
```mermaid
stateDiagram-v2
    S0_MainMenu --> L1_UncompletedList: /list(pagination)
    L1_UncompletedList --> L1a_PageChange: page_<N>
    
    S0_MainMenu --> L2_CompletedList: /list_completed
    L2_CompletedList --> L2a_ReturnItem: mark_uncompleted_<id>
    L2a_ReturnItem --> S10_ReturnDate: set new date
    S10_ReturnDate --> S2_DateInput: date input
```
---

## **2. Соответствие реализации ТЗ**

### 1. Создание напоминания

| Переход | Условие | Соответствие ТЗ |
|---|---|---|
| S0 → S1 | Пользователь вводит команду `/add` | Да (П.1 ТЗ) |
| S1 → S2 | Введено текстовое описание напоминания | Да (П.1 ТЗ) |
| S2 → S3 | Указана дата (базовый формат) | Частично: нет парсинга сложных форматов |
| S3 → S4 | Выбрана периодичность ("Да") | Да (П.6 ТЗ) |
| S3 → S5 | Отказ от периодичности ("Нет") | Да |
| S4 → S5 | Введен интервал повторения в днях | Да (П.6 ТЗ) |
| S5 → S6 | Пользователь согласен прикрепить файлы | Да (П.2 ТЗ) |
| S5 → S0 | Пользователь отказывается от загрузки файлов | Да |
| S6 → S0 | Файлы успешно загружены в S3 | Да (П.2 ТЗ) |

### 2. Редактирование задач

| Переход | Условие | Соответствие ТЗ |
|---|---|---|
| S0 → S7 | `/list` | Да (П.3) |
| S7 → S8 | Выбор напоминания | Да (П.4) |
| S8 → S7 | 'back_to_list' | Да |
| S8 → S7a | 'delete' | Да (П.4) |
| S7a → S7 | 'confirm' | Да (П.4) |
| S8 → S8a | 'edit' | Да (П.4) |
| S8a → S8b | Новое название | Да (П.4) |
| S8b → S8c | Новая дата | Да (П.4) |
| S8c → S8d | Настройки | Частично: нет выбора "для всех повторов" |
| S8d → S0 | Подтверждение | Да |
| S8 → S9 | 'mark_completed' | Да (П.5) |
| S9 → S0 | Автоматически | Да |

### 3. Работа с файлами

| Переход | Условие | Соответствие ТЗ |
|---|---|---|
| S6 → F1 | Загружен одиночный файл | Да (П.2 ТЗ) |
| S6 → F2 | Обнаружена медиагруппа | Да (П.2 ТЗ) |
| F2 → F2a | Файл добавлен в очередь | Нет: отсутствует таймаут |
| F2a → F2c | Истекло время ожидания (10 сек) | Нет: не реализовано |
| F2c → S0 | Все файлы загружены пакетно | Нет: не реализовано |

### 4. Просмотр и восстановление

| Переход | Условие | Соответствие ТЗ |
|---|---|---|
| S0 → L1 | /list | Да |
| L1 → L1a | Пагинация | Да |
| S0 → L2 | /completed | Да |
| L2 → L2a | Выбор | Да |
| L2a → S10 | Восстановление | Да |
| S10 → S2 | Новая дата | Да |
---

### Ключевые несоответствия:
1. **Парсинг дат**: Не поддерживаются такие фразы, как "через 3 дня" (переход S2 → S3)
2. **Медиагруппы**: Отсутствует обработка таймаута (переходы F2 → F2a → F2c)
3. **Периодичность**: Нет выбора режима редактирования по периодичности/без периодичности (переход S8c → S8d)

## **3. Выводы**
Вообще говоря, ТЗ соблюдено - бот работает корректно, однако необходимо изменить следующее:
- Требуется улучшение парсера для корректной интерпретации сложных временных форматов
- Необходима обработка исключений при нестандартном вводе дат
- Требуется доработка механизма загрузки вложений
- Важно добавить обработку ошибок при загрузке в облачное хранилище
- Необходимо разработать раздельные меню управления для:
    - Одноразовых напоминаний
    - Периодических уведомлений
