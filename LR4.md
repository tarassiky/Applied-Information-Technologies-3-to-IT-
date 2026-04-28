# Лабораторная работа №4: Docker – установка и запуск контейнера с веб-приложением

**Выполнила:** Тарасова Дарья  
**Сервер:** tarassiky.ru (80.249.144.226)  
**Дата:** 28.04.2026  

## Цель работы
Установить Docker Engine на сервер (Ubuntu), запустить контейнер с веб-сервером Nginx и обеспечить доступ к веб-странице из браузера.

## Ход выполнения

### 1. Подключение к серверу

```bash
ssh root@80.249.144.226
```

### 2. Проверка, что Docker не установлен

```bash
docker --version
```
**Результат:** `bash: docker: command not found` (Docker отсутствует)

### 3. Установка Docker Engine (по официальной инструкции для Ubuntu)

```bash
# Обновление пакетов
apt update && apt upgrade -y

# Установка зависимостей
apt install -y apt-transport-https ca-certificates curl software-properties-common

# Добавление ключа GPG Docker
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Добавление репозитория Docker
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null

# Установка Docker Engine
apt update
apt install -y docker-ce docker-ce-cli containerd.io

# Добавление текущего пользователя в группу docker
usermod -aG docker $USER
newgrp docker
```

**Проверка установки:**

```bash
docker --version
```
**Результат:** `Docker version 27.4.1, build 9e9f930` (версия может отличаться)

### 4. Запуск контейнера с Nginx

```bash
docker run -d --name my-nginx -p 8080:80 nginx
```
- `-d` – запуск в фоновом режиме
- `--name my-nginx` – имя контейнера
- `-p 8080:80` – проброс порта 8080 хоста на порт 80 контейнера
- `nginx` – образ веб-сервера

**Проверка работающих контейнеров:**

```bash
docker ps
```
**Результат:** контейнер `my-nginx` в статусе `Up`

### 5. Доступность веб-страницы локально на сервере

```bash
curl localhost:8080
```
**Результат:** возвращается HTML-код стандартной страницы Nginx (содержит "Welcome to nginx!")

### 6. Открытие порта 8080 в фаерволе (если включён UFW)

```bash
ufw allow 8080/tcp
```

### 7. Проверка работы в браузере

Открыт URL: `http://tarassiky.ru:8080`

<img width="1889" height="594" alt="2026-04-28_07-33-26" src="https://github.com/user-attachments/assets/fb456713-a767-45da-b8a5-8785cc51987d" />

*На скриншоте видна стандартная приветственная страница Nginx, доступная по адресу `http://tarassiky.ru:8080`*

### 8. Дополнительные команды (управление контейнером)

```bash
# Остановка контейнера
docker stop my-nginx

# Запуск остановленного контейнера
docker start my-nginx

# Удаление контейнера
docker rm my-nginx
```

## Результаты

- Docker Engine успешно установлен на сервер.
- Запущен контейнер с Nginx, который слушает порт 8080.
- Веб-приложение (стандартная страница Nginx) доступно из интернета по адресу `http://tarassiky.ru:8080`.
- Получены базовые навыки работы с Docker: установка, запуск контейнера с пробросом портов, остановка и удаление.

## Вывод

В ходе лабораторной работы освоен процесс установки Docker на Ubuntu, а также запуск и проверка работы контейнеризированного веб-сервера. Это основа для дальнейшего использования Docker в разработке и развёртывании приложений.
