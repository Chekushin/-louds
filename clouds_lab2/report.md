# Лабораторная работа №2

## Сравнение сервисов AWS и Azure. Кросс-провайдерная модель

## 1. Цель работы

Сравнить сервисы AWS и Azure на основе данных биллинга и построить единую кросс-провайдерную сервисную модель.

---

## 2. Классификация Azure

Файл `Azure lab team 4.csv` был импортирован в Excel.
Заполнены столбцы:

* IT Tower
* Service Family
* Service Type
* Service Sub Type
* Service Usage Type

### Разделение по уровням абстракции

* **IaaS**: Virtual Machines, Networking
* **PaaS / SaaS**: Data Factory, Databricks, PostgreSQL, Redis, Power BI, Machine Learning

Service Sub Type определялся на основе `Meter Name` и `Meter Sub-Category`.

---

## 3. Сравнение AWS и Azure

### Распределение по типу сервисов

| Модель      | AWS    | Azure  |
| ----------- | ------ | ------ |
| IaaS        | ~15%   | ~43%   |
| PaaS / SaaS | ~72.5% | ~59.5% |

Azure в данной выборке содержит больший процент инфраструктурных сервисов.

---

## 4. Кросс-провайдерная модель

Для сопоставления сервисов была разработана единая таксономия:

* Support & Billing
* Compute
* Databases
* Networking
* Analytics & Big Data
* Artificial Intelligence
* Developer Tools
* Business Applications
* Security & Compliance

Пример соответствия:

| AWS        | Azure            | Категория            |
| ---------- | ---------------- | -------------------- |
| Amazon EMR | Azure Databricks | Analytics & Big Data |
| Amazon RDS | Azure PostgreSQL | Databases            |
| AWS Lambda | Azure Functions  | Compute              |
| CloudWatch | Azure Monitor    | Monitoring           |

---

## 5. Вывод

Создана кросс-провайдерная сервисная модель, позволяющая сопоставлять сервисы AWS и Azure в единой системе категорий.
Достигнута цель формирования вендор-независимого представления облачной инфраструктуры.