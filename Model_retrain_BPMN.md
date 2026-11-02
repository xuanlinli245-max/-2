# Процесс автоматического переобучения модели (BPMN)

## 1. Схема процесса (BPMN Diagram)

```mermaid
flowchart TD
    %% Определение стилей
    classDef trigger fill:#f9d0c4,stroke:#333,stroke-width:2px;
    classDef process fill:#d4e6f1,stroke:#333,stroke-width:1px;
    classDef decision fill:#fcf3cf,stroke:#333,stroke-width:2px;
    classDef success fill:#d5f5e3,stroke:#333,stroke-width:2px;
    classDef fail fill:#fadbd8,stroke:#333,stroke-width:2px;

    %% Элементы процесса
    Start([Событие-триггер: дрейф данных или расписание]):::trigger
    Data[Получение и подготовка новых данных]:::process
    Train[Обучение новой версии модели]:::process
    Evaluate[Проверка качества модели]:::process
    Decision{Метрики выше порога?}:::decision
    Register[Регистрация / сохранение новой версии]:::success
    Deploy[Деплой модели в эксплуатацию]:::success
    Alert[Уведомление и остановка процесса]:::fail

    %% Связи
    Start --> Data
    Data --> Train
    Train --> Evaluate
    Evaluate --> Decision
    
    Decision -- Да (Успешно) --> Register
    Decision -- Нет (Неуспешно) --> Alert
    
    Register --> Deploy

    %% Передача результатов (пояснения на стрелках)
    Start -- "Сигнал о необходимости" --> Data
    Data -- "Подготовленный датасет (Train/Val)" --> Train
    Train -- "Артефакт модели (weights)" --> Evaluate
    Evaluate -- "Отчет с метриками" --> Decision
    Decision -- "Разрешение на публикацию" --> Register
    Register -- "Версия модели в реестре" --> Deploy