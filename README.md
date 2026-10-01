# Expense Tracker API

REST API для учёта расходов на FastAPI + MySQL. Позволяет записывать траты (название, сумма, категория, дата, заметка).

## Что внутри

- Python 3
- FastAPI
- SQLAlchemy 2
- MySQL
- Uvicorn

## Структура проекта
```text
expense-tracker-api/
├── app/
│   └── main.py
├── .env.example
├── requirements.txt
```

## API методы

### Проверка состояния
```bash
curl http://127.0.0.1:8020/health
```

### Создать запись
```bash
curl -X POST http://127.0.0.1:8020/expenses \
  -H 'Content-Type: application/json' \
  -d '{"title":"Обед","amount":12.50,"category":"Еда","spent_on":"2026-10-01","notes":"кафе на углу"}'
```

### Получить список записей (с фильтром по категории)
```bash
curl http://127.0.0.1:8020/expenses
curl "http://127.0.0.1:8020/expenses?category=Еда"
```

### Получить запись по ID
```bash
curl http://127.0.0.1:8020/expenses/1
```

### Обновить запись
```bash
curl -X PUT http://127.0.0.1:8020/expenses/1 \
  -H 'Content-Type: application/json' \
  -d '{"amount":15.00}'
```

### Удалить запись
```bash
curl -X DELETE http://127.0.0.1:8020/expenses/1
```

---

## Задача: запустить через Docker Compose (MySQL вместо PostgreSQL)

Приложения в репозитории ещё нет ни `Dockerfile`, ни `docker-compose.yml` — это и есть задача, написать их самостоятельно.

1. **Dockerfile** для `app/` — базовый образ Python, установка зависимостей из `requirements.txt`, запуск через `uvicorn app.main:app --host 0.0.0.0 --port <порт>`.
2. **docker-compose.yml** с двумя сервисами:
   - `db` — официальный образ `mysql` (не забудь про историю с `postgres:latest` из прошлой задачи — какой тег стоит выбрать здесь?);
   - `app` — собирается из твоего `Dockerfile`, пробрасывает порт наружу.
3. Выясни, какие переменные окружения ожидает официальный образ `mysql` для инициализации базы, пользователя и пароля (имена переменных отличаются от Postgres — подумай, где искать: Docker Hub страница образа).
4. Добавить **volume** для `db`, чтобы данные не терялись при пересоздании контейнера (уточни, в какую директорию MySQL пишет свои данные внутри контейнера — тоже отличается от Postgres).
5. Настроить **`healthcheck`** для `db` — у MySQL для проверки готовности используется не `pg_isready`, а другая утилита. Какая?
6. Настроить **`depends_on`**, чтобы `app` поднимался после `db` только когда тот реально готов принимать соединения.
7. Важный момент про `DATABASE_URL`: внутри Docker-сети хост базы — не `localhost`, а имя сервиса из `docker-compose.yml`. Также обрати внимание на драйвер в строке подключения — для MySQL используется `pymysql`, а не `psycopg2` (уже прописано в `.env.example` и `requirements.txt`, просто имей в виду разницу).
8. Поднять всё одной командой (`docker compose up -d --build`), проверить `docker compose ps`.
9. Проверить все пять эндпоинтов через `curl` (см. примеры выше, порт уже наружу проброшен).
10. Убедиться, что данные переживают перезапуск: `docker compose down`, затем `docker compose up -d` — записи должны остаться (volume сработал).
