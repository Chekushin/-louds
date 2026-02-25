# Отчет по лабораторной работе №1: Настройка Nginx

**Цель работы:** Настроить веб-сервер Nginx для работы с несколькими проектами, обеспечить безопасность через HTTPS и настроить перенаправление трафика.

### 1. Установка и подготовка

В качестве операционной среды использовалась **Ubuntu через WSL**. Установка Nginx выполнена стандартным менеджером пакетов:

```bash
sudo apt update && sudo apt install nginx -y
chekushin@DESKTOP-KL59D04:~$ sudo nginx -t
[sudo] password for chekushin:
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

Для демонстрации работы виртуальных хостов была создана структура директорий и проверочные файлы:

```bash
chekushin@DESKTOP-KL59D04:~$ curl -k https://project1.test --resolve project1.test:443:127.0.0.1
<h1>Project ONE</h1>
chekushin@DESKTOP-KL59D04:~$ curl -k https://project2.test --resolve project2.test:443:127.0.0.1
<h1>Project TWO</h1>
chekushin@DESKTOP-KL59D04:~$ curl -I http://project1.test --resolve project1.test:80:127.0.0.1
HTTP/1.1 301 Moved Permanently
Server: nginx/1.24.0 (Ubuntu)
Date: Tue, 24 Feb 2026 08:49:38 GMT
Content-Type: text/html
Content-Length: 178
Connection: keep-alive
Location: https://project1.test/
```


### 2. Генерация SSL-сертификата

Для реализации протокола HTTPS (порт 443) был сгенерирован самоподписанный SSL-сертификат и приватный ключ:

```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
-keyout /etc/ssl/private/nginx-selfsigned.key \
-out /etc/ssl/certs/nginx-selfsigned.crt -subj "/CN=localhost"

```

### 3. Конфигурация Nginx

Конфигурация была вынесена в отдельный файл `/etc/nginx/sites-available/projects.conf`.

**Реализация требований ТЗ:**

* **HTTPS и Virtual Hosts:** Созданы два блока `server` для `project1.test` и `project2.test` с прослушиванием порта 443 и указанием путей к сертификатам.
* **Принудительный редирект:** Настроен отдельный блок на порту 80, который перенаправляет все запросы на HTTPS с кодом 301.
* **Alias:** Использована директива `alias` для доступа к статической папке `/static/`.

```nginx
# Перенаправление с HTTP на HTTPS
server {
    listen 80;
    server_name project1.test project2.test;
    return 301 https://$host$request_uri;
}

# Конфигурация Project 1
server {
    listen 443 ssl;
    server_name project1.test;

    ssl_certificate /etc/ssl/certs/nginx-selfsigned.crt;
    ssl_certificate_key /etc/ssl/private/nginx-selfsigned.key;

    location / {
        root /var/www/project1/html;
        index index.html;
    }

    location /static/ {
        alias /var/www/static-files/;
    }
}


chekushin@DESKTOP-KL59D04:~$ curl -k https://project1.test/static/test.txt --resolve project1.test:443:127.0.0.1
This is content
```

### 4. Проверка работы

Для проверки использовалась утилита `curl` с флагом `--resolve`, так как доменные имена являются локальными.

* **Проверка редиректа:** Команда `curl -I http://project1.test` подтвердила возврат кода `301 Moved Permanently`.
* **Проверка HTTPS:** Запрос к `https://project1.test` успешно вернул контент `<h1>Project ONE</h1>`.
* **Проверка Alias:** Запрос к `/static/test.txt` вернул содержимое файла из внешней директории.

### 5. Выявленные проблемы

В ходе работы возникла проблема с тем, что система не распознавала домены `.test`. Вместо редактирования файла `hosts` в Windows, была использована подмена IP на уровне запроса через `curl`, что позволило изолировать тесты внутри среды WSL. Также была исправлена ошибка прав доступа (permission denied) путем смены владельца папок на пользователя `www-data`.

---


