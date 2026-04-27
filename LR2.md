# Лабораторная работа №2: Получение SSL-сертификата для домена tarassiky.ru

**Выполнил:** Тарасова Даша  
**Дата:** 27 апреля 2026 г.  
**Домен:** tarassiky.ru  
**IP сервера:** 80.249.144.226  

---

## Цель работы

Настроить защищённое соединение по протоколу HTTPS для домена `tarassiky.ru` с использованием SSL-сертификата от Let's Encrypt. В ходе работы необходимо:
- установить и настроить веб-сервер Nginx;
- получить доверенный сертификат с помощью Certbot;
- обеспечить автоматическое обновление сертификата;
- продемонстрировать работоспособность сайта по HTTPS.

---

## Предварительные условия

- Зарегистрировано доменное имя `tarassiky.ru` (регистратор – reg.ru).
- Создан облачный сервер (VPS) с IP-адресом `80.249.144.226`.
- Настроены DNS-серверы регистратора: `ns1.hosting.reg.ru`, `ns2.hosting.reg.ru`.
- В DNS-зону добавлена A-запись для основного домена и поддомена `www`, указывающая на IP сервера.

> **Проверка DNS:**  
> ```bash
> dig tarassiky.ru +short
> 80.249.144.226
> dig www.tarassiky.ru +short
> 80.249.144.226
> ```

---

## Ход выполнения работы

### 1. Установка и настройка веб-сервера Nginx

На сервере установлен Nginx, создана корневая директория для сайта и тестовая страница.

```bash
apt update && apt install nginx -y
mkdir -p /var/www/tarassiky.ru/html
echo "<h1>tarassiky.ru works</h1>" > /var/www/tarassiky.ru/html/index.html
```
Создан файл конфигурации /etc/nginx/sites-available/tarassiky.ru:

```bash
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    server_name tarassiky.ru www.tarassiky.ru;
    root /var/www/tarassiky.ru/html;
    index index.html;
}
```

Для активации сайта создана символическая ссылка из sites-available в sites-enabled. Это стандартная практика Nginx: файлы конфигурации хранятся в sites-available, а включённые сайты – через симлинки в sites-enabled.

```bash
ln -s /etc/nginx/sites-available/tarassiky.ru /etc/nginx/sites-enabled/
rm -f /etc/nginx/sites-enabled/default
nginx -t && systemctl reload nginx
```

После запуска сайт стал доступен по HTTP:

http://tarassiky.ru → отображается страница "tarassiky.ru works".

### 2. Выбор стратегии валидации домена

Для получения сертификата от Let's Encrypt необходимо подтвердить владение доменом. Использован HTTP-01 challenge с плагином --nginx Certbot.

Обоснование:

Nginx уже установлен и корректно обслуживает порт 80.

HTTP-01 challenge автоматизирован, не требует ручного добавления DNS-записей (в отличие от DNS-01).

Плагин --nginx сам вносит изменения в конфигурацию и настраивает HTTPS.

Примечание: изначально были проблемы с IPv6 (AAAA-записи в DNS направляли Let's Encrypt на неработающий IPv6-адрес). После удаления AAAA-записей проверка прошла успешно.

### 3. Установка Certbot и получение сертификата

Certbot установлен через snap (рекомендованный способ для автоматических обновлений):

```bash
snap install --classic certbot
ln -s /snap/bin/certbot /usr/bin/certbot
```

Запущена команда для получения сертификата и автоматической настройки Nginx:

```bash
certbot --nginx -d tarassiky.ru -d www.tarassiky.ru
```

Certbot запросил email для уведомлений, согласие с условиями использования. После успешной проверки доменов сертификат был выдан.

Результат:

```bash
Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/tarassiky.ru/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/tarassiky.ru/privkey.pem
This certificate expires on 2026-07-26.
Deploying certificate...
Successfully deployed certificate for tarassiky.ru to /etc/nginx/sites-enabled/tarassiky.ru
Congratulations! You have successfully enabled HTTPS on https://tarassiky.ru
```

### 4. Настройка статического сервера для работы с SSL

Certbot автоматически модифицировал конфигурацию Nginx, добавив блок server для порта 443 и перенаправление с HTTP на HTTPS. Финальная конфигурация:

```bash
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    server_name tarassiky.ru www.tarassiky.ru;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl;
    listen [::]:443 ssl;
    server_name tarassiky.ru www.tarassiky.ru;
    ssl_certificate /etc/letsencrypt/live/tarassiky.ru/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/tarassiky.ru/privkey.pem;
    root /var/www/tarassiky.ru/html;
    index index.html;
}
```

### 5. Автоматическое обновление сертификата
Certbot через snap создал systemd-таймер для автоматического продления сертификата (дважды в день). Проверка:

```bash
systemctl list-timers | grep certbot
```

Вывод:

```text
Mon 2026-04-27 19:53:00 UTC  ... snap.certbot.renew.timer
```

Тест обновления в сухом режиме:

```bash
certbot renew --dry-run
```

Результат: Congratulations, all simulated renewals succeeded.

### 6. Демонстрация работы HTTPS

Открываем в браузере https://tarassiky.ru. Сайт загружается, в адресной строке отображается зелёный замок, протокол – HTTPS.

При нажатии на замок отображается информация о сертификате: выдан Let's Encrypt, действует для tarassiky.ru и www.tarassiky.ru, срок действия – до 26 июля 2026 г.

### Результаты работы

- Веб-сервер Nginx успешно установлен и настроен для обслуживания статического сайта.
- Получен доверенный SSL-сертификат от Let's Encrypt.
- Настроено автоматическое перенаправление с HTTP на HTTPS.
- Организовано автоматическое обновление сертификата через systemd-таймер.
- Сайт доступен по защищённому протоколу HTTPS, соединение признаётся браузером безопасным.

| Параметр | Значение |
|----------|----------|
| Домен | tarassiky.ru |
| IP-адрес сервера | 80.249.144.226 |
| Центр сертификации | Let's Encrypt |
| Срок действия сертификата | 89 дней (до 2026-07-26) |
| Тип ключа | ECDSA |
| Автоматическое обновление | Да (snap.certbot.renew.timer) |

### Заключение

В ходе лабораторной работы был успешно настроен HTTPS для домена tarassiky.ru. Выполнены все этапы: настройка DNS, установка и конфигурация Nginx, получение SSL-сертификата с помощью Certbot, обеспечение автоматического обновления. Продемонстрирована работа сайта по защищённому протоколу. Полученные навыки позволяют в дальнейшем безопасно разворачивать веб-приложения.

### Приложение: Использованные команды (для справки)

```bash
# Установка Nginx
apt update && apt install nginx -y

# Создание директории сайта
mkdir -p /var/www/tarassiky.ru/html
echo "<h1>tarassiky.ru works</h1>" > /var/www/tarassiky.ru/html/index.html

# Конфигурация Nginx
cat > /etc/nginx/sites-available/tarassiky.ru <<EOF
server {
    listen 80;
    server_name tarassiky.ru www.tarassiky.ru;
    root /var/www/tarassiky.ru/html;
    index index.html;
}
EOF

# Активация сайта через симлинк
ln -s /etc/nginx/sites-available/tarassiky.ru /etc/nginx/sites-enabled/
nginx -t && systemctl reload nginx

# Установка Certbot
snap install --classic certbot
ln -s /snap/bin/certbot /usr/bin/certbot

# Получение сертификата и настройка HTTPS
certbot --nginx -d tarassiky.ru -d www.tarassiky.ru

# Проверка статуса сертификата
certbot certificates

# Проверка автоматического обновления
certbot renew --dry-run
systemctl list-timers | grep certbot
```

### Все скрины:

<img width="728" height="501" alt="2026-04-27_20-03-08" src="https://github.com/user-attachments/assets/95aa240f-428a-4f26-963b-32a5ec34e777" />

---

<img width="1171" height="341" alt="2026-04-27_20-03-39" src="https://github.com/user-attachments/assets/96aacc81-f031-4dfe-aaa4-1e5f865fa022" />

---

<img width="883" height="108" alt="2026-04-27_20-04-23" src="https://github.com/user-attachments/assets/9675863a-04c3-440f-b1df-d6775137786c" />

---

<img width="857" height="663" alt="2026-04-27_20-04-52" src="https://github.com/user-attachments/assets/173bf3ca-9348-4155-8c1e-6cac4eb6041b" />

---

<img width="906" height="158" alt="2026-04-27_20-05-36" src="https://github.com/user-attachments/assets/04a5e899-2ad5-4038-8438-1cc7d33110a5" />

---

<img width="651" height="128" alt="2026-04-27_20-06-15" src="https://github.com/user-attachments/assets/99c26605-f76b-4680-8828-2428856243ff" />

---

<img width="1093" height="561" alt="2026-04-27_20-08-26" src="https://github.com/user-attachments/assets/59ccdb34-d68b-4369-b260-10036e79af50" />

---

<img width="536" height="187" alt="2026-04-27_20-09-21" src="https://github.com/user-attachments/assets/0675d838-edad-4255-a7c3-9d27192953c1" />
