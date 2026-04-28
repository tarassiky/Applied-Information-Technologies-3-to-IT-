# Лабораторная работа №3: Несколько статических сайтов на одном сервере (Nginx)

**Выполнила:** Тарасова Дарья  
**Домен:** tarassiky.ru  
**Сервер:** 80.249.144.226  

## Цель работы
Настроить веб-сервер Nginx для обслуживания нескольких статических сайтов на субдоменах, защитить их SSL-сертификатами Let's Encrypt.

## Ход выполнения

### 1. DNS-записи
В панели управления хостингом добавлены A-записи для субдоменов:

| Субдомен | Тип | IP-адрес |
|----------|-----|----------|
| site1.tarassiky.ru | A | 80.249.144.226 |
| site2.tarassiky.ru | A | 80.249.144.226 |

### 2. Подготовка каталогов и файлов

```bash
mkdir -p /var/www/site1.tarassiky.ru/html
mkdir -p /var/www/site2.tarassiky.ru/html
```
В каждый каталог помещён HTML-файл (резюме для site1, страница проекта для site2).

### 3. Конфигурация Nginx
Созданы файлы в `/etc/nginx/sites-available/`:

**`site1.tarassiky.ru`**
```nginx
server {
    listen 80;
    server_name site1.tarassiky.ru;
    root /var/www/site1.tarassiky.ru/html;
    index index.html;
}
```

**`site2.tarassiky.ru`** – аналогично.

### 4. Символические ссылки
Активированы конфигурации через `sites-enabled`:

```bash
ln -s /etc/nginx/sites-available/site1.tarassiky.ru /etc/nginx/sites-enabled/
ln -s /etc/nginx/sites-available/site2.tarassiky.ru /etc/nginx/sites-enabled/
nginx -t && systemctl reload nginx
```

### 5. SSL-сертификаты
Выпущены сертификаты командой Certbot:

```bash
certbot --nginx -d site1.tarassiky.ru
certbot --nginx -d site2.tarassiky.ru
```
(Для каждого выбран вариант «1» — переустановить существующий сертификат)

### 6. Проверка
- `nginx -t` – синтаксис корректен.
- `curl -I https://site1.tarassiky.ru` → `HTTP/2 200`
- В браузере открыты `https://site1.tarassiky.ru` и `https://site2.tarassiky.ru` – зелёный замок, корректное отображение.

<img width="967" height="1223" alt="2026-04-28_07-21-49" src="https://github.com/user-attachments/assets/2ca24620-7660-4077-8a6d-d58b1643caaf" />

---

<img width="948" height="508" alt="2026-04-28_07-22-02" src="https://github.com/user-attachments/assets/07f3970c-b32d-4676-9456-a1fda91d5320" />

## Результат
Два статических сайта доступны по HTTPS на субдоменах `site1.tarassiky.ru` и `site2.tarassiky.ru`. Nginx использует отдельные конфигурации, включённые через символические ссылки. Сертификаты автоматически обновляются через systemd-таймер Certbot.

## Вывод
В ходе работы освоена базовая конфигурация Nginx для виртуальных хостов, работа с символическими ссылками, получение SSL-сертификатов для нескольких доменов.
