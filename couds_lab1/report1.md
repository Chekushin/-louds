# Лабораторная работа №1

## Классификация облачных сервисов AWS на основе Billing Data

Для выполнения задания использовался файл
`Mapping Rules AWS team 4.csv`.

---

## Шаг 1. Импорт данных

CSV-файл был импортирован в Microsoft Excel с разделителем `;`.

После импорта были получены следующие основные столбцы:

* IT Tower
* Service Family
* Service Type
* Service Sub Type
* Service Usage Type
* Product Code
* Usage Type

Первые пять столбцов заполнялись вручную на основании анализа AWS Billing.

---

## Шаг 2. Определение IT Tower

`IT Tower` — верхний уровень классификации, отражающий глобальную технологическую категорию сервиса.

В соответствии с образцом использовались следующие значения:

* `Compute`
* `Database`
* `Storage & Content Delivery`
* `Networking`
* `Cloud Services`
* `Support`

Логика определения:

1. Если сервис относится к **IaaS**, он распределяется по инфраструктурным категориям
   (`Compute`, `Database`, `Storage`, `Networking`).
2. Если сервис относится к **PaaS / SaaS**, он относится к категории
   `Cloud Services`.
3. Все строки типа `Tax` классифицируются как `Support`.

---

## Шаг 3. Service Family

`Service Family` — это функциональная группа сервисов внутри IT Tower.

Логика заполнения:

* Для инфраструктурных сервисов значение часто совпадает с IT Tower.
* Для `Cloud Services` значение уточняется (например, Monitoring, Application Integration и т.д.).
* Определение выполнялось по `Product Code` и официальной документации AWS.

---

## Шаг 4. Service Type

`Service Type` — читаемое название сервиса на основе `Product Code`.

Пример:

| Product Code   | Service Type    |
| -------------- | --------------- |
| AmazonS3       | Amazon S3       |
| AmazonEC2      | Amazon EC2      |
| AmazonSNS      | Amazon SNS      |
| AmazonRedshift | Amazon Redshift |

---

## Шаг 5. Service Sub Type

Отражает логический подтип использования сервиса.

Определялся на основании:

* `Usage Type`
* официальной документации AWS
* характера тарификации

Например:

* Data Transfer
* Storage
* API Requests
* Logs Storage
* Concurrency Scaling

---

## Шаг 6. Service Usage Type

Минимальная тарифицируемая единица.

Определяется по шаблону `Usage Type` и документации.

Примеры:

* Tier 1 Requests
* Data Transfer (GB)
* Heavy Usage
* Data Scanned
* Node Hours
* Tax

Используемая документация (пример):

[https://docs.aws.amazon.com/AmazonS3/latest/userguide/aws-usage-report-understand.html](https://docs.aws.amazon.com/AmazonS3/latest/userguide/aws-usage-report-understand.html)

---

# Пример итоговой таблицы

| IT Tower       | Service Family             | Service Type    | Service Sub Type    | Service Usage Type | Product Code   | Usage Type          |
| -------------- | -------------------------- | --------------- | ------------------- | ------------------ | -------------- | ------------------- |
| Storage        | Storage & Content Delivery | Amazon S3       | Data Transfer       | Data Transfer      | AmazonS3       | %DataTransfer%      |
| Storage        | Storage & Content Delivery | Amazon S3       | Storage             | Data Retrieval     | AmazonS3       | %Retrieval-Bytes%   |
| Database       | Data Warehouse             | Amazon Redshift | Concurrency Scaling | Heavy Usage        | AmazonRedshift | %HeavyUsage%        |
| Networking     | VPC Services               | Amazon VPC      | NAT Gateway         | Data Transfer      | AmazonVPC      | %DataTransfer%Bytes |
| Cloud Services | Application Integration    | Amazon SNS      | API Requests        | Tier 1             | AmazonSNS      | %Requests-Tier1%    |
| Support        | Support Charges            | Amazon Redshift | Additional Charges  | Tax                | AmazonRedshift | Tax%                |

---

# Итоговая иерархия

В работе реализована пятиуровневая модель:

```
IT Tower
   ↓
Service Family
   ↓
Service Type
   ↓
Service Sub Type
   ↓
Service Usage Type
```

Данная структура позволяет:

* анализировать затраты сверху вниз,
* агрегировать данные по уровням,
* детализировать потребление до конкретной модели тарификации.

---

# Классификация по уровням абстракции

В рамках работы сервисы были разделены на:

### IaaS

* Amazon EC2
* Amazon S3
* Amazon VPC

### PaaS

* AWS Lambda
* Amazon EMR
* Amazon CloudWatch

### SaaS

* Amazon WorkMail
* Amazon WorkSpaces

---

## Разбиение сложных сервисов

Крупные сервисы были дополнительно декомпозированы.

Например, Amazon CloudWatch был разделён на:

* Logs Storage
* Metrics
* Data Scanned
* General

Это позволило корректно классифицировать разные типы потребления внутри одного сервиса.

---

# Вывод

В ходе выполнения лабораторной работы:

* построена иерархическая модель классификации AWS-сервисов;
* реализовано разделение по уровням абстракции (IaaS / PaaS / SaaS);
* выполнена детализация сервисов до уровня тарифицируемых метрик;
* освоен анализ AWS Billing и официальной документации.

В результате получена связная и формализованная модель, пригодная для анализа облачных затрат и дальнейшего масштабирования (в том числе для кросс-провайдерной модели во второй лабораторной работе).