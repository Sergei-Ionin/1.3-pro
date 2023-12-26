# 1.3-pro
<h1>Docker + SQL + Python</h1>
<h2>Задание</h2>

* Аналитики решили поупражняться в написании кода и пошли дальше — они написали скрипт на Python! Скрипт не вызывает у вас большого уважения, но он, тем не менее, воспроизводит минимальные расчеты над данными в созданной вами базе. Теперь вам необходимо придумать механизм, который позволит одновременно поднимать базу, запускать sql-скрипт коллеги-джуна, а затем запускать python-скрипт от коллег-аналитиков. Учтите, последовательность должна быть именно такой, иначе ничего не выйдет!
* В качестве результата вам необходимо загрузить в ваш репозиторий проект, на основе которого можно воспроизвести логику работы программы из описания задачи. А также вам необходимо в отдельном текстовом файле описать список docker-команд для управления этим проектом + оформить README-файл с описанием функциональности проекта и его содержимого.

По итогу выполнения задания вам необходимо прислать ссылку на ваш репозиторий.

---
Используемые программы:
* VS code
* Docker
----

<h2>Выполнение</h2>

* Создаем две папки, "app" и "db", для скрипта на python и для базы данных postgres
* В каждой будет находится Dockerfile для создания образа и потом в контейнер.
* Пример Dockerfile для базы данных :
```
FROM postgres:16.0
LABEL SI="Sergei Ionin"
ENV POSTGRES_PASSWORD=postgres
ENV POSTGRES_USER=postgres
ENV POSTGRES_DB=testdb
COPY init/init.sql /docker-entrypoint-initdb.d/init.sql
WORKDIR /db
COPY . . /db/
VOLUME [ "/datavolume" ]
```
    * FROM - какой образ будет в качестве основы
    * LABEL - для метинформации(например кто автор)
    * ENV - для создания переменных окружений (например пароль)
    * COPY - копирует отдельные ( в данном случае скрипт для запуска базы), либо все как ниже
    * WORKDIR - создание рабочей директории внутри контейнера
    * VOLUME - создает том в котором будут хранится данные контейнера

* Для скрипта на python:

```
FROM python:3.9.18
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1
WORKDIR /app
COPY requirements.txt .
RUN /usr/local/bin/python -m pip install --upgrade pip
RUN --mount=type=cache,target=/root/.cache/pip \
    --mount=type=bind,source=requirements.txt,target=requirements.txt \
    python -m pip install -r requirements.txt
COPY . .
CMD [ "python", "app.py" ]
```
* Все почти как в предыдущем, только добавляется RUN посредством которой обновляется, если необходимо pip, который служит для установки модулей в соответствии с  requirements.txt 
* CMD - команда для запуска скрипта python 

---

Для запуска необходима тока команда:

```
docker compose up -d
```
Пример docker compose

```
version: '3'
services:
# APP
  app: 
    container_name: app            (название контейнера)
    build: ./app                   (из какой папки он разворачивается)
    ports:                         (порты для соединения)
      - 8080:8080 
    restart: on-failure            (перезапуск)
    environment:
      - APP_POSTGRES_HOST=postgres
    volumes:
      - api:/usr/src/app/
    depends_on:                     (отслеживает состояние других контейнеров)
      postgres:
        condition: service_healthy  (условие для запуска состояния контейнера)
    networks:                       (сеть внутри контейнера)
      - net
    links:                          (метка(название контейнера) по которой ищет нужный)
      - "postgres"
# POSTGRES
  postgres:
    build: ./db
    container_name: db
    restart: always
    user: postgres
    ports:
      - '5432:5432'
    healthcheck:                      (состояние контейнера)
      test: [ "CMD", "pg_isready" ]
      interval: 10s
      timeout: 5s
      retries: 5
    volumes:
      - database_postgres:/var/lib/postgresql/data
    networks:
      - net

volumes:
  api:
  database_postgres:
networks:
  net:
    driver: bridge
```
После запуска команды docker compose up -d скачиваются образы используемые при построении контейнера. Первым разворачивается контейнер с базой данных и пока он полностью не запуститься, второй "app" будет ждать, для чего в нем используется depends on который отслеживает состояние контейнера "db". Контейнер "app" запускает скрипт для python который работает с базой данных.