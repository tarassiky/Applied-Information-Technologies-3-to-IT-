# Лабораторная работа №6: Docker Compose – приложение Ping Pong

**Выполнила:** Тарасова Дарья  
**Сервер:** tarassiky.ru (80.249.144.226)  
**Дата:** 28.04.2026  

## Цель работы
Разработать простое веб-приложение (стиль «ping-pong»), принимающее порт и ответ из переменных окружения. С помощью Docker Compose запустить два независимых экземпляра приложения на разных портах с разными ответами.

## 1. Приложение на Python (Flask)

Файл `app.py`:

```python
import os
from flask import Flask

app = Flask(__name__)

MESSAGE = os.environ.get('MESSAGE', 'Default Pong')

@app.route('/')
def pong():
    return f'{MESSAGE}\n'

if __name__ == '__main__':
    port = int(os.environ.get('PORT', 5000))
    app.run(host='0.0.0.0', port=port)
```

## 2. Docker Compose для двух экземпляров

Файл `docker-compose.yml`:

```yaml
version: '3.8'

services:
  pong1:
    build: .
    container_name: pong_service_1
    ports:
      - "5001:5000"
    environment:
      - PORT=5000
      - MESSAGE=Pong from instance 1

  pong2:
    build: .
    container_name: pong_service_2
    ports:
      - "5002:5000"
    environment:
      - PORT=5000
      - MESSAGE=Pong from instance 2
```

## 3. Запуск и проверка

Выполненные команды (см. скриншот ниже):

```bash
cd ~/pong-app
docker-compose up -d
docker-compose ps
curl localhost:5001
curl localhost:5002
```

## 4. Результат работы

- Контейнеры `pong_service_1` и `pong_service_2` запущены и слушают порты 5001 и 5002.
- На порту 5001 возвращается текст `Pong from instance 1`.
- На порту 5002 возвращается текст `Pong from instance 2`.

### Скриншот с демонстрацией всех команд и выводов:

<img width="1083" height="891" alt="2026-04-28_09-22-40" src="https://github.com/user-attachments/assets/7f68035c-2ae1-4c91-9de7-e6c63f4181d3" />

---

<img width="1203" height="282" alt="2026-04-28_09-18-26" src="https://github.com/user-attachments/assets/14f9fdde-7ba2-40c5-8f60-464523321348" />

---

<img width="1154" height="198" alt="2026-04-28_09-18-35" src="https://github.com/user-attachments/assets/6a033c8b-d12b-4c4f-8809-bd5273cbd432" />

---

## 5. Вывод

В ходе работы создано веб-приложение, порт и ответ которого управляются через переменные окружения. С помощью Docker Compose запущены два экземпляра на разных портах. Задание выполнено в полном объёме.
