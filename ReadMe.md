1. Создаем простой проект на Django.

Создаем виртуальное окружение:
	cd app
    python -m venv env
    или с определенной версией python:
    py -3.8 -m venv env
    ///py -3.9 -m venv env

PSh
Активируем виртуальное окружение:
	env\scripts\activate
    # source env/bin/activate
    можно проверить версию python -V
                           Python 3.8.10
                           python:3.8.10-alpine
Устанавливаем Django в виртуальное окружение:
	pip3 install django==3.2.6

(env)$ django-admin.py startproject docker_django .
(env)$ python manage.py migrate
(env)$ python manage.py runserver 

Создаем пустой проект:
	django-admin startproject docker_django
    cd docker_django
    python manage.py migrate
    python manage.py runserver

Запускаем проект:
Переходим в папку project (уже перешли) и выполняем:
	python manage.py runserver
Перейдите по адресу http://localhost:8000/ (или http://localhost:8000/)
    для просмотра экрана приветствия Django.
Остановить сервер: нажми CTRL + C

Создаем админа:
    python manage.py createsuperuser
    myadmin
    docsecret
Superuser created successfully.
(python manage.py changepassword)
Проверим:
    Запускаем проект:
        python manage.py runserver
    Переходим по адресу
        http://localhost:8000/admin
    вводим логин и пароль
    наблюдаем админку с двумя модулями:
        Groups	
        Users
Остановить сервер: нажми CTRL + C

Выйдите из виртуальной среды
    deactivate
и удалите ее. Можно закрыть VSC и удалить каталог env.
Теперь у нас есть простой проект Django для работы.

---------------------------
Поскольку мы перейдем к Postgres, удалите файл db.sqlite3 из каталога app.

---------------------------
Создайте файл requirements.txt в каталоге app и добавьте Django в качестве зависимости:
    Django==3.2.6

---------------------------

Каталог проекта должен выглядеть следующим образом:

└── app
    ├── docker_django
    │   ├── __init__.py
    │   ├── asgi.py
    │   ├── settings.py
    │   ├── urls.py
    │   └── wsgi.py
    ├── manage.py
    └── requirements.txt

По умолчанию создается структура проекта:
└── app
    ├── docker_django
        ├── docker_django
        │   ├── __init__.py
        │   ├── asgi.py
        │   ├── settings.py
        │   ├── urls.py
        │   └── wsgi.py
        ├── manage.py
        └── requirements.txt
поэтому, перенес вложенную папку docker_django наверх, чтобы получилась выше описанная структура
================================================================================================

2. Добавьте Dockerfile в каталог app

Так, мы начали с Alpine на основе образа Docker для Python 3.9.6.
Затем мы установили рабочий каталог и две переменные окружения:

PYTHONDONTWRITEBYTECODE: Запрещает Python записывать файлы pyc на диск (эквивалент опции python -B)
PYTHONUNBUFFERED: Запрещает Python буферизовать stdout и stderr (эквивалент опции python -u)

Наконец, мы обновили Pip,
скопировали файл requirements.txt,
установили зависимости
и скопировали сам проект Django.

================================================================================================

3. Создаем образ docker-compose с проектом Django.

Создаем файл docker-compose.yml в корне проекта

Обновите переменные SECRET_KEY, DEBUG и ALLOWED_HOSTS в settings.py

Добавим импорт наверху:
import os

Затем создайте файл .env.dev в корне проекта (в app не получилось, значит в docker_django)
    для хранения переменных среды для разработки
    (это не тот же файл, в котором храним переменные для сборки всего проекта docker-compose)

Собираем образ:
    docker-compose build

После создания образа запустите контейнер:
    docker-compose up -d
    ✔ Network docker-djago_default  Created
    ✔ Container docker-djago-web-1  Started

Перейдите по адресу http://localhost:8000/, чтобы снова просмотреть экран приветствия.

Остановим docker-compose:
    docker-compose stop
или
    docker-compose down
Команда docker-compose stop остановит ваши запущенные docker-контейнеры, однако не удалит их.
В отличии от нее, команда docker-compose down остановит запущенные docker-контейнеры
и удалит их, а также все docker-сети (networks) созданные при запуске связки контейнеров
из файла docker-compose.yml.

Можно добавить аргумент -v (--volumes) при запуске команды для удаления созданных томов:
    docker-compose down -v

================================================================================================

4. Настроим Postgres

Для сохранения данных после окончания жизни контейнера мы настроили том.
Эта конфигурация привяжет postgres_data к директории "/var/lib/postgresql/data/" в контейнере.

Внести изменения в файлы:
    docker-compose.yml
    .env.dev
    Dockerfile

Создайте новый образ и запустите два контейнера:
    docker-compose build
    docker-compose up -d --build

    ✔ Network docker-djago_default         Created                                                                       0.0s 
    ✔ Volume "docker-djago_postgres_data"  Created                                                                       0.0s 
    ✔ Container docker-djago-db-1          Started                                                                       0.6s 
    ✔ Container docker-djago-web-1         Started

Запустите миграции:
    docker-compose exec web python manage.py migrate --noinput

---------------------------------------------------------------
Убедитесь, что таблицы Django по умолчанию были созданы:
    docker-compose exec db psql --username=docker_django --dbname=docker_django_dev

    psql (13.0)
    Type "help" for help.

    docker_django_dev=#

    введем
    \l
выведет список и описание таблиц

для выхода из psql
    docker_django_dev=# \q

----------------------------------------------------------------
Вы можете проверить, что том был создан, выполнив:
    docker volume inspect docker-djago_postgres_data
    docker volume inspect docker-django_postgres_data
(посмотрел в Docker Desktop - volumes
там же на вкладке Data видно его содержание,
которое соответствует /var/lib/postgresql/data/ )

Также, название volume выходит в консоль при выполнении команды 
создания образа и запуска контейнеров:
    docker-compose up -d --build
    ✔ Network docker-djago_default         Created                                                                       0.0s 
    ✔ Volume "docker-djago_postgres_data"  Created  
    
-----------------------------------------------------------------
-----
    volumes:
    db-data:
        external: true

    - db-data в данном примере этот каталог расположенный в одной директории с docker-compose.yml?
    Нет. Это именованный volume. Его фактическая папка спрятана где-то глубоко, можно посмотреть командой docker volume inspect db-data и изначально он пустой.

    В чем отличие от такой записи(является ли она корректная?):
    - ./db-data:/var/lib/mysql/data
    Такая запись первым параметром указывает не именованный volume, а подпапку в папке с docker-compose.yml Т.е. mysql получит папку со всем её содержимым по адресу, указанному вторым параметром.
----
Запускаем контейнеры через docker-compose вновь

docker-compose -f docker-compose.yml up -d
-----------------------------------------------------------------


Затем добавьте файл entrypoint.sh в каталог app, 
чтобы проверить работоспособность Postgres 
перед применением миграции и запуском сервера разработки Django
--------------------------
Обновите разрешения файлов локально:

    $ chmod +x app/entrypoint.sh
--------------------------

Затем обновите файл Dockerfile, 
чтобы скопировать файл entrypoint.sh и запустить его как команду точки входа
--------------------------
Добавьте переменную окружения DATABASE в .env.dev:
    DATABASE=postgres

--------------------------
--- !!!!!!!!!!!!!!!!!!! ---
Попробую здесь добавить админа
cd app
env\scripts\activate
python manage.py createsuperuser
    myadmin
    docsecret
Или так:
    docker-compose down -v

Создаем образ
    docker-compose build
Запускаем контейнер
    docker-compose up -d --build


    или одной командой создания образа и запуска контейнеров:
        docker-compose -f docker-compose.yml up -d
    или
        docker-compose up -d --build
    
    docker-compose exec web python manage.py createsuperuser
--------------------------
export DJANGO_SUPERUSER_CREATE=true
export DJANGO_SUPERUSER_USERNAME=admin
export DJANGO_SUPERUSER_EMAIL=admin@example.com
export DJANGO_SUPERUSER_PASSWORD=admin
python manage.py migrate

LCE-HiP-5sY-V9e

или вполнить команды после создания контейнера:
docker-compose exec web python manage.py flush --no-input
docker-compose exec web python manage.py migrate
python manage.py createsuperuser --noinput
docker run -p 8000:8000 -e POSTGRES_USER=myusername -e POSTGRES_PASSWORD=mypassword -e POSTGRES_DB=mydatabase YOUR_IMAGE_NAME:TAG

Вам также необходимо указать путь к entrypoint.sh
docker run -p 8000:8000 -v /путь/к/каталогу/с/entrypoint.sh:/usr/src/app/entrypoint.sh имя_вашего_образа

можно наверное таким образом любой файл заменить перед сборкой образа,
например, .env.dev - с настройками:
создаем/копируем .env.dev в каком-либо каталоге
при запуске указываем путь к нему:
docker run -p 8000:8000 -v ~/.env.dev:/usr/src/.env.dev image_name
docker run -p 8000:8000 -v c:\\docker\env.dev:/usr/src/.env.dev image_name
(а в docker-compose?)

--------------------------
//////////////////////////
Во-первых, несмотря на добавление Postgres, 
мы все еще можем создать независимый образ Docker для Django, 
если переменная окружения DATABASE не установлена на postgres. 
Чтобы проверить, создайте новый образ, а затем запустите новый контейнер:

$ docker build -f ./app/Dockerfile -t docker_django:latest ./app
$ docker run -d \
    -p 8006:8000 \
    -e "SECRET_KEY=please_change_me" -e "DEBUG=1" -e "DJANGO_ALLOWED_HOSTS=*" \
    docker_django python /usr/src/app/manage.py runserver 0.0.0.0:8000

Надо перенести кавычки:
docker run -d -p 8006:8000 -e SECRET_KEY="please_change_me" -e DEBUG="1" -e DJANGO_ALLOWED_HOSTS="*" docker_django python /usr/src/app/manage.py runserver 0.0.0.0:8000
Вы должны увидеть страницу приветствия по адресу http://localhost:8006
////////////////////////

Во-вторых, возможно, вы захотите закомментировать команды database flush и migrate 
в сценарии entrypoint.sh, чтобы они не выполнялись при каждом запуске или перезапуске контейнера:
-----------
#!/bin/sh

if [ "$DATABASE" = "postgres" ]
then
    echo "Waiting for postgres..."

    while ! nc -z $SQL_HOST $SQL_PORT; do
      sleep 0.1
    done

    echo "PostgreSQL started"
fi

# python manage.py flush --no-input
# python manage.py migrate

exec "$@"
-------------
Вместо этого вы можете запустить их вручную, после запуска контейнеров, следующим образом:

$ docker-compose exec web python manage.py flush --no-input
$ docker-compose exec web python manage.py migrate 

//////////////////////////////

===================================================
5. Добавляем Gunicorn
   Файлы для производства

Для производственных сред (то что будет выкладываться в прод, коиенту, на сайт)
добавим Gunicorn, WSGI-сервер промышленного класса,
в файл требований:
----------------
Django==3.2.6
gunicorn==20.1.0
psycopg2-binary==2.9.1 
----------------

Поскольку мы все еще хотим использовать встроенный сервер Django в разработке, 
(для дальнейшей разработке проекта на Django, но это не надо на проде)
создайте новый файл compose под названием docker-compose.prod.yml для производства:
-----
- Обратите внимание на команду по умолчанию. 
Мы запускаем Gunicorn, а не сервер разработки Django. 
- Мы также удалили том из службы web, поскольку он не нужен нам в производстве. 
- Наконец, мы используем отдельные файлы переменных окружения, 
чтобы определить переменные окружения для обоих сервисов, 
которые будут передаваться контейнеру во время выполнения
(в соотвествующих файлах в корень проекта):
    .env.prod
    .env.prod.db
-----

Удалите контейнеры разработки (и связанные тома с флагом -v):
    $ docker-compose down -v
Затем создайте производственные образы и запустите контейнеры:
    $ docker-compose -f docker-compose.prod.yml up -d --build

 ✔ Network docker-djago_default         Created                                                                       0.0s 
 ✔ Volume "docker-djago_postgres_data"  Created                                                                       0.0s 
 ✔ Container docker-djago-db-1          Started                                                                       0.6s 
 ✔ Container docker-djago-web-1         Started   

Т.о. в docker-compose.yml файлах мы описываем конфинурацию:
что включать в проект, какие сервера, какое окружение и т.п.
А при создании образа указываем по какому docker-compose.prod.yml файлу его собирать:
    docker-compose -f docker-compose.prod.yml up -d --build

-------------------------------------------------------------------------------
Производственный Dockerfile
Заметили ли вы, что мы по-прежнему выполняем команды database flush
 (которая очищает базу данных)
  и migrate при каждом запуске контейнера? 
  Это нормально для разработки, но давайте создадим новый файл точки входа для производства.

entrypoint.prod.sh:

------------------
Обновите разрешения файлов локально:

$ chmod +x app/entrypoint.prod.sh 
- вот это не знаю как делать на виндовсе
-------------------------------------------
Чтобы использовать этот файл, создайте новый Dockerfile с именем Dockerfile.prod 
для использования с производственными сборками:

------------------

Здесь мы использовали Docker многоступенчатую сборку, 
чтобы уменьшить конечный размер образа. 
По сути, builder - это временный образ, который используется для сборки колес Python. 
Затем колеса копируются в конечный производственный образ, а образ builder удаляется.

--------------------

Заметили ли вы, что мы создали пользователя, не являющегося root? 
По умолчанию Docker запускает контейнерные процессы от имени root внутри контейнера. 
Это плохая практика, поскольку злоумышленники могут получить root-доступ к хосту Docker, 
если им удастся вырваться из контейнера. 
Если вы являетесь root в контейнере, вы будете root на хосте.

--------------------

Обновите веб-службу в файле docker-compose.prod.yml для сборки с помощью Dockerfile.prod:

--------------------

Попробуйте:

$ docker-compose -f docker-compose.prod.yml down -v
$ docker-compose -f docker-compose.prod.yml up -d --build
$ docker-compose -f docker-compose.prod.yml exec web python manage.py migrate --noinput 

 ✔ Network docker-djago_default  Created                                                                              0.0s 
 ✔ Container docker-djago-db-1   Started                                                                              0.7s 
 ✔ Container docker-djago-web-1  Started
-------------------------

======================================

6. Nginx

Следующим шагом добавим Nginx, который будет работать как обратный прокси для Gunicorn 
для обработки клиентских запросов, а также для обслуживания статических файлов.

Добавьте сервис в docker-compose.prod.yml:
-------------------------------------------
Затем в локальном корне проекта создайте следующие файлы и папки:

└── nginx
    ├── Dockerfile
    └── nginx.conf

-------------------------------------------
Затем обновите веб-службу в docker-compose.prod.yml, заменив порты на expose:

web:
  build:
    context: ./app
    dockerfile: Dockerfile.prod
  command: gunicorn docker_django.wsgi:application --bind 0.0.0.0:8000
  expose:
    - 8000
  env_file:
    - ./.env.prod
  depends_on:
    - db

Сейчас порт 8000 открыт только внутри системы, для других служб Docker.
Порт больше не будет публиковаться на хост-машине.
-------------------------------------------
Попробуйте еще раз.

$ docker-compose -f docker-compose.prod.yml down -v
$ docker-compose -f docker-compose.prod.yml up -d --build

 ✔ Network docker-djago_default         Created                                                                       0.3s 
 ✔ Volume "docker-djago_postgres_data"  Created                                                                       0.0s 
 ✔ Container docker-djago-db-1          Started                                                                       0.7s 
 ✔ Container docker-djago-web-1         Started                                                                       1.1s 
 ✔ Container docker-djago-nginx-1       Started 

$ docker-compose -f docker-compose.prod.yml exec web python manage.py migrate --noinput
-------------------------------------------
Убедитесь, что приложение запущено и работает на http://localhost:1337.

-------------------------------------------

Структура вашего проекта теперь должна выглядеть следующим образом:

├── .env.dev
├── .env.prod
├── .env.prod.db
├── .gitignore
├── app
│   ├── Dockerfile
│   ├── Dockerfile.prod
│   ├── entrypoint.prod.sh
│   ├── entrypoint.sh
│   ├── docker_django
│   │   ├── __init__.py
│   │   ├── asgi.py
│   │   ├── settings.py
│   │   ├── urls.py
│   │   └── wsgi.py
│   ├── manage.py
│   └── requirements.txt
├── docker-compose.prod.yml
├── docker-compose.yml
└── nginx
    ├── Dockerfile
    └── nginx.conf
==================================================

Поскольку Gunicorn является сервером приложений, он не будет обслуживать статические файлы.

STATIC_URL = "/static/"
STATIC_ROOT = BASE_DIR / "staticfiles" 
=============================================
может быть можно статику поискать (см закомментированные команды в settings.py)
почему нет на 8000 порту
------------------------

выложить на Doker Hub:
- образ
- пример файла .env.dev
- пример команды для скачивания
- пример команды для запуска конейнера (с указанием пути к .env.dev в windows и linux)

----------------------
Postgres    :5432
docker-compose exec db psql --username=docker_django --dbname=docker_django_dev
docker-compose exec pg_isready -h localhost
DBeaver
localhost:5432 - accepting connections

nginx   :80
curl localhost:80

http://localhost:80
http://localhost:80/static/index.html

Gunicorn    /   Django  :8000
http://localhost:8000/admin



docker build -t djan1.test .
docker run -d -p 8080:8080 -p 80:80 -p 5432:5432 djan1.test

======================================================================
docker-compose работает с файлами docker-compose.yml из текущей директории, или указанными в  -f docker-compose.yml
docker-compose down -v
docker-compose -d --build

Собираем образ:
    docker-compose build

После создания образа запустите контейнер:
    docker-compose up -d

    
Остановим docker-compose:
    docker-compose stop
    docker-compose start
или
    docker-compose down
    
Создайте новый образ и запустите два контейнера:
    docker-compose up -d --build