# Проект docker-django

Проект построен с использованием Docker Compose.
Собраны и работают серверы Linux, nginx, Django, Postgres, Gunicorn.
Собранные образы загружены на hub.docker.com.
Ссылка на исходный проект на github.com: https://github.com/alexmed-dev/docker-django

Для скачивания образов и запуска контейнеров на их основе следует использовать конфигурационный файл docker-compose.yml.

- Ссылка на github.com: https://github.com/alexmed-dev/docker-django-prod/blob/main/docker-compose.yml
- Или ссылка на скачивание: https://drive.google.com/file/d/1T02q0V1UEoSW14yZ9vykAYHvA3MMz324/view?usp=drive_link
- Команда для запуска: `docker-compose up -d`

Можно зайти в панель администратора Django: http://localhost:8000/admin
   - логин: `admin`
   - пароль: `secret`

   Доступна статическая страница: http://localhost:8000/static/index.html


## Подробности

При первом запуске с hub.docker.com загрузятся необходимые образы и запустятся три контейнера.

При запуске будет создано два тома:
- Для БД PostgreSQL (можно сохранять изменения после выключения контейнера при настройке окружения NEWDB=0 - описано ниже)
- Для статических файлов (можно внести изменения будут отображены в браузере по адресам http://localhost:8000/static/)

В БД будет выполнена миграция.

Будет создан администратор Django:
    логин: admin
    пароль: secret
    (Данные можно поменять в файле docker-compose.yml.)


## Работа с готовым проектом

### Основное

Для работы с проектом необходимо:
1. Загрузить конфигурационный файл docker-compose.yml.
   - ссылка на github.com: https://github.com/alexmed-dev/docker-django-prod/blob/main/docker-compose.yml
   - ссылка на скачивание: https://drive.google.com/file/d/1T02q0V1UEoSW14yZ9vykAYHvA3MMz324/view?usp=drive_link
2. Поместить его в удобный для работы каталог.
3. В терминале перейти в каталог и запустить команду: `docker-compose up -d`
   Можно зайти в панель администратора Django: http://localhost:8000/admin
   - логин: `admin`
   - пароль: `secret`
   Доступна статическая страница: http://localhost:8000/static/index.html

### Дополнительно

 При необходимости можно изменить параметры:
   - `DJANGO_SUPERUSER_USERNAME` - логин администратора Django;
   - `DJANGO_SUPERUSER_PASSWORD` - пароль администратора Django.

  Перед повторноым запуском можно указать `NEWDB=0`, чтобы продолжить работу с существующей БД и пользователем-админом Django, если не были удалены автоматически созданные тома.

      **!!! ВНИМАНИЕ !!!**
      Не устанавливайте `NEWDB=0` при первом запуске (docker-compose up -d).
      БД и админ уже должны быть созданы ранее (`NEWDB=1`).

   Параметры из конфигурационного файла docker-compose.yml используются для создания пользователя в новой БД и для авторизации в админке по адресу http://localhost:8000/admin/
   только при `NEWDB=1` (при `NEWDB=0` игнорируется и использутся ранее примененные).

