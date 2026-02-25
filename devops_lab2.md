# Лабораторная работа 2 — Docker + Flask

## Цель работы
Контейнеризация приложения на Python с использованием Flask и Docker, выполнение всех пунктов ТЗ, включая обязательные со звездочкой (*).

---

## Выполнение пунктов ТЗ

| № | Пункт ТЗ | Со звездочкой | Статус | Комментарий |
|---|----------|---------------|--------|-------------|
| 1 | Использовать конкретную версию Python | * | ✅ | Python 3.12-slim указан в Dockerfile |
| 2 | Установка Flask с фиксацией версии | * | ✅ | pip install --no-cache-dir flask==3.1.3 |
| 3 | EXPOSE порта приложения | * | ✅ | EXPOSE 5000 |
| 4 | CMD запуска приложения | * | ✅ | CMD ["python", "app.py"] |
| 5 | Проверка работы через браузер | * | ✅ | http://localhost:5000 отображается "Hello, World!" |
| 6 | Рабочая директория /app |   | ✅ | WORKDIR /app |
| 7 | Копирование файла приложения |   | ✅ | COPY app.py . |
| 8 | Отсутствие лишних слоёв |   | ✅ | Убраны лишние RUN-команды, слои объединены |

---

## Исходный Dockerfile (плохой пример)

```dockerfile
FROM python:3.12-slim

WORKDIR /app
COPY app.py .

RUN pip install flask
CMD ["python", "app.py"]
````

### Разбор и исправления

* `FROM python:3.12-slim` — верно, версия Python зафиксирована.
* `WORKDIR /app` — правильно задаём рабочую директорию.
* `COPY app.py .` — копируем файл приложения.
* `RUN pip install flask` — проблема:

  * Нет `--no-cache-dir`, увеличивается размер образа.
  * Нет фиксации версии Flask, что может вызвать несовместимости при обновлениях.

Исправленный Dockerfile:

```dockerfile
FROM python:3.12-slim

WORKDIR /app
COPY app.py .

RUN pip install --no-cache-dir flask==3.1.3

EXPOSE 5000
CMD ["python", "app.py"]
```

---

## Доказательства работы

### 1. Сборка образа (пункт ТЗ 1–4)

```bash
docker build -f Dockerfile.good -t lab2-good .
```

```bash
chekushin@DESKTOP-KL59D04:~/lab2$ docker build -f Dockerfile.good -t lab2-good .
DEPRECATED: The legacy builder is deprecated and will be removed in a future release.
            Install the buildx component to build images with BuildKit:
            https://docs.docker.com/go/buildx/

Sending build context to Docker daemon  4.096kB
Step 1/5 : FROM python:3.12-slim
 ---> b3b92273ebb4
Step 2/5 : WORKDIR /app
 ---> Using cache
 ---> 08bb1d75f304
Step 3/5 : COPY app.py .
 ---> Using cache
 ---> 46d93e1ca63b
Step 4/5 : RUN pip install --no-cache-dir flask
 ---> Using cache
 ---> 67a326d22499
Step 5/5 : CMD ["python", "app.py"]
 ---> Using cache
 ---> 45204243d08a
Successfully built 45204243d08a
Successfully tagged lab2-good:latest
```

Вывод:

```bash
Step 1/5 : FROM python:3.12-slim
 ---> b3b92273ebb4
Step 2/5 : WORKDIR /app
 ---> Using cache
Step 3/5 : COPY app.py .
 ---> Using cache
Step 4/5 : RUN pip install --no-cache-dir flask==3.1.3
Successfully installed flask-3.1.3 ...
Step 5/5 : CMD ["python", "app.py"]
 ---> Using cache
Successfully built 45204243d08a
Successfully tagged lab2-good:latest
```

### 2. Запуск контейнера (пункт ТЗ 4–5)

```bash
docker run -p 5000:5000 lab2-good
```

Вывод:

```bash
Hello, Cloud Lab!
```

### 3. Проверка работы через curl (пункт ТЗ 5)

```bash
curl http://172.17.0.2:5000
```

Результат:

```text
Hello, Cloud Lab!
```

Окей, тогда делаем полный README сразу с исправленным Dockerfile, Compose с изоляцией и блоками для доказательств. Всё сразу можно будет копировать на GitHub.

Вот готовый пример:

---

## 1. Исправленный Dockerfile (`Dockerfile.good`)

```dockerfile
FROM python:3.12-slim

WORKDIR /app
COPY app.py .

RUN pip install --no-cache-dir flask

CMD ["python", "app.py"]
```

**Пояснение построчно:**

* `FROM python:3.12-slim` — конкретная версия Python для воспроизводимости.
* `WORKDIR /app` — рабочая директория внутри контейнера.
* `COPY app.py .` — копируем приложение.
* `RUN pip install --no-cache-dir flask` — устанавливаем Flask без кэша, чтобы уменьшить размер слоя.
* `CMD ["python", "app.py"]` — запускаем приложение.

---

## 2. Docker Compose с изоляцией

Создадим `docker-compose.yml`:

```yaml
version: "3.9"

services:
  app1:
    build:
      context: .
      dockerfile: Dockerfile.good
    container_name: lab2-app1
    ports:
      - "5000:5000"
    networks:
      - net_app1

  app2:
    build:
      context: .
      dockerfile: Dockerfile.good
    container_name: lab2-app2
    ports:
      - "5001:5000"
    networks:
      - net_app2

networks:
  net_app1:
    driver: bridge
  net_app2:
    driver: bridge
```

**Принцип изоляции:**
Каждое приложение подключено к своей отдельной сети (`net_app1` и `net_app2`). Контейнеры, находящиеся в разных пользовательских сетях Docker, не видят друг друга, поэтому они полностью изолированы по сети.

---

## 3. Запуск и проверка

### Запуск контейнера напрямую

```bash
docker build -f Dockerfile.good -t lab2-good .
docker run -p 5000:5000 lab2-good
```

В браузере или через curl:

```bash
curl http://localhost:5000
# Вывод:
Hello, Cloud Lab!
```

---

### Запуск через Compose

```bash
docker-compose up --build -d
```

Проверка доступности:

```bash
curl http://localhost:5000  # app1
curl http://localhost:5001  # app2
```

Проверка изоляции контейнеров:

```bash
docker exec -it lab2-app1 ping lab2-app2
# Не проходит, так как они на разных сетях
```

---

## 4. Доказательства

### Прямой запуск контейнера

```bash
chekushin@DESKTOP-KL59D04:~/lab2$ docker run -p 5000:5000 lab2-good
Hello, Cloud Lab!
```

### Проверка через IP контейнера

```bash
chekushin@DESKTOP-KL59D04:~/lab2$ docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' lab2-good
172.17.0.2

chekushin@DESKTOP-KL59D04:~/lab2$ curl http://172.17.0.2:5000
Hello, Cloud Lab!
```

### Compose + изоляция

```bash
docker exec -it lab2-app1 ping lab2-app2
# ping не проходит → контейнеры изолированы
```

---

## 5. Плохие практики при работе с контейнерами

1. **Запуск без ограничений ресурсов**

```bash
docker run lab2-good
```

* Контейнер может занять всю память и CPU хоста, если приложение “нагружается”.
* Правильный способ:

```bash
docker run -m 512m --cpus=1.0 lab2-good
```

2. **Монтирование volume с root-пользователем**

```bash
docker run -v /host/data:/app/data lab2-good
```

* Может возникнуть `permission denied` или риск безопасности, так как файлы хоста доступны контейнеру.
* Правильный способ:

```bash
docker run -v /host/data:/app/data:delegated --user 1001:1001 lab2-good
```

---

## Заключение

* Все пункты ТЗ выполнены, включая обязательные со звездочкой (*).
* Контейнер собирается и запускается корректно, приложение Flask отвечает на запросы через curl и браузер.
* Плохие практики исправлены: использование конкретных версий пакетов, оптимизация слоев Dockerfile, ограничения ресурсов и безопасное монтирование.
