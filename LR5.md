# Лабораторная работа №5: Создание собственного Docker-образа (Python + Flask)

**Выполнила:** Тарасова Дарья  
**Сервер:** tarassiky.ru (80.249.144.226)  
**Дата выполнения:** 28.04.2026  

## Цель работы
Создать Docker-образ на основе Ubuntu с приложением на Python (Flask), загрузить образ в Docker Hub и предоставить публичную ссылку.

## Ход работы

### 1. Подготовка файлов приложения
Создана директория `~/my-docker-app` и в ней три файла.

#### `app.py` – веб-приложение Flask:
```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello():
    return "Hello from Docker! This is Dasha's container."

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

requirements.txt:

```text
Flask==2.3.3
```

Dockerfile (на основе Ubuntu 22.04):

```dockerfile
FROM ubuntu:22.04

RUN apt-get update && apt-get install -y \
    python3 \
    python3-pip \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

COPY requirements.txt .
COPY app.py .

RUN pip3 install --no-cache-dir -r requirements.txt

EXPOSE 5000

CMD ["python3", "app.py"]
```

### 2. Сборка образа

Выполнена команда:

```bash
docker build -t tarassiky/flask-app:latest .
```

Сборка прошла успешно.

### 3. Локальный запуск и проверка

Контейнер запущен локально:

```bash
docker run -d -p 5000:5000 --name temp-app tarassiky/flask-app:latest
curl localhost:5000
```

Результат: Hello from Docker! This is Dasha's container.

### 4. Публикация в Docker Hub

Зарегистрирован аккаунт на hub.docker.com с логином tarassiky.

Выполнен вход в CLI: docker login

Образ отправлен в репозиторий:

```bash
docker push tarassiky/flask-app:latest
```

Скриншот страницы образа на Docker Hub:

<img width="2296" height="1179" alt="9T42yfXYNokMVEWS21tvkB6CYfs4aRQ7UilA8jO045WZ12owVn6nC_CjXRm_FQ3LuYppKo4ux171fJBW6SJtexBb" src="https://github.com/user-attachments/assets/bb8434ec-70fe-4236-af96-fc1f6d1a8fb5" />

<img width="1649" height="1179" alt="6l_s3KuyTJDjqF8DYvSnA0k8H8DRs1OaOBMFQ22BCBiiSZL18Dg8-c08GlKEKqI9UmmOtJNZvtiF_mTdSfYerI9k" src="https://github.com/user-attachments/assets/4793a5a0-c120-429a-9932-b0eaa668aab1" />

### 5. Ссылка на образ

Публичный образ доступен по адресу:

https://hub.docker.com/repository/docker/tarassiky/flask-app

## Результаты работы

| Этап |Выполнение |
|----------|----------|
| Dockerfile | создан |
| Программа на Python | написана |
| docker build | образ собран |
| docker push | образ загружен |
| Ссылка на Docker Hub | опубликована |

## Вывод

В ходе лабораторной работы освоены основные операции с Docker: создание Dockerfile, сборка образа на основе Ubuntu с Python-приложением, тестирование контейнера локально и публикация образа в публичный реестр Docker Hub. Полученный образ может быть использован любым пользователем командой docker run -d -p 5000:5000 tarassiky/flask-app:latest.
