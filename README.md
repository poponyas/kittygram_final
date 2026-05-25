# Kittygram

Kittygram - социальный сервис для публикации фотографий котов. Пользователи могут регистрироваться, входить в аккаунт, добавлять карточки котов, редактировать и удалять свои публикации, загружать изображения и указывать достижения питомцев.

Проект подготовлен для запуска в Docker-контейнерах и автоматического обновления образов через GitHub Actions.

## Возможности

- регистрация и авторизация пользователей;
- получение токена авторизации;
- просмотр карточек котов;
- создание, редактирование и удаление карточек;
- загрузка изображений;
- хранение пользовательских файлов в Docker volume;
- работа с PostgreSQL;
- раздача фронтенда, media и static через Nginx;
- автоматическая проверка кода, тестирование и публикация образов на Docker Hub.

## Стек технологий

- Python 3.9;
- Django 3.2.3;
- Django REST Framework;
- Djoser;
- PostgreSQL 13;
- Gunicorn;
- Nginx;
- React;
- Docker и Docker Compose;
- GitHub Actions;
- Docker Hub.

## Структура проекта

```text
kittygram_final/
├── backend/                 # Django API
├── frontend/                # React-приложение
├── nginx/                   # Конфигурация gateway
├── docker-compose.yml       # Локальный запуск со сборкой образов
├── docker-compose.production.yml
├── .env.example             # Пример переменных окружения
└── .github/workflows/main.yml
```

## Переменные окружения

Перед запуском создайте файл `.env` в корне проекта. Можно скопировать пример:

```bash
cp .env.example .env
```

Пример `.env`:

```env
SECRET_KEY=django-insecure-change-me
DEBUG=False
ALLOWED_HOSTS=127.0.0.1 localhost backend your-kittygram-domain.example
CSRF_TRUSTED_ORIGINS=https://your-kittygram-domain.example

POSTGRES_DB=kittygram
POSTGRES_USER=kittygram_user
POSTGRES_PASSWORD=kittygram_password
DB_NAME=kittygram
DB_HOST=db
DB_PORT=5432

DOCKER_USERNAME=poponyas
```

Описание переменных:

- `SECRET_KEY` - секретный ключ Django;
- `DEBUG` - режим отладки, в production должен быть `False`;
- `ALLOWED_HOSTS` - список разрешенных хостов Django;
- `CSRF_TRUSTED_ORIGINS` - доверенные HTTPS-домены для CSRF;
- `POSTGRES_DB` и `DB_NAME` - имя базы данных;
- `POSTGRES_USER` - пользователь PostgreSQL;
- `POSTGRES_PASSWORD` - пароль пользователя PostgreSQL;
- `DB_HOST` - имя сервиса базы данных в Docker Compose;
- `DB_PORT` - порт PostgreSQL;
- `DOCKER_USERNAME` - логин Docker Hub для production-образов.

## Локальный запуск

Соберите и запустите контейнеры:

```bash
docker compose up -d --build
```

После запуска приложение будет доступно по адресу:

```text
http://localhost:9000
```

Проверить состояние контейнеров:

```bash
docker compose ps
```

Посмотреть логи backend:

```bash
docker compose logs -f backend
```

Остановить проект:

```bash
docker compose down
```

## Production-запуск

Production compose использует готовые образы из Docker Hub:

- `poponyas/kittygram_backend`;
- `poponyas/kittygram_frontend`;
- `poponyas/kittygram_gateway`.

На сервере создайте директорию проекта и положите туда `docker-compose.production.yml` и `.env`.

Загрузите свежие образы:

```bash
docker compose -f docker-compose.production.yml pull
```

Запустите проект:

```bash
docker compose -f docker-compose.production.yml up -d
```

Проверьте, что контейнеры работают:

```bash
docker compose -f docker-compose.production.yml ps
```

Gateway публикует приложение на порту `9000`. Внешний Nginx на сервере должен проксировать домен Kittygram на этот порт:

```nginx
server {
    server_name kittygram.example.com;

    location / {
        proxy_pass http://127.0.0.1:9000;
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

После настройки домена выпустите HTTPS-сертификат, например через Certbot:

```bash
sudo certbot --nginx -d kittygram.example.com
```

## CI/CD

Workflow находится в `.github/workflows/main.yml` и запускается при push в ветку `main`.

GitHub Actions выполняет:

- проверку backend-кода через Ruff;
- запуск тестов backend;
- запуск тестов frontend;
- сборку Docker-образов;
- публикацию образов на Docker Hub;
- отправку сообщения в Telegram после успешного завершения.

Для работы workflow в настройках GitHub-репозитория нужно добавить secrets:

```text
DOCKER_USERNAME
DOCKER_PASSWORD
TELEGRAM_TO
TELEGRAM_TOKEN
```

## Проверка перед отправкой

В корне проекта должен быть файл `tests.yml`:

```yaml
repo_owner: poponyas
kittygram_domain: https://kittygram.example.com
taski_domain: https://taski.example.com
dockerhub_username: poponyas
```

В `kittygram_domain` и `taski_domain` нужно указать реальные HTTPS-домены развернутых проектов, а не ссылки на GitHub.

Запуск автотестов:

```bash
pytest
```

## Автор

poponyas
