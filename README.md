# NikkiNoble_microservices
NikkiNoble microservices repository

# Технология контейнеризации. Введение в Docker

- Вывод команды `docker images` сохранен в файл `docker-monolith/docker-1.log`

* `docker-machine` - встроенный в докер инструмент для создания хостов и установки на них docker engine.

* Создан Docker хост в Yandex Cloud и настроено локальное окружение на работу с ним


        yc compute instance create \
        --name docker-host \
        --zone ru-central1-a \
        --network-interface subnet-name=default-ru-central1-a,nat-ip-version=ipv4 \
        --create-boot-disk image-folder-id=standard-images,image-family=ubuntu-1804-lts,size=15 \
        --ssh-key ~/.ssh/appuser.pub
	
        docker-machine create \
        --driver generic \
        --generic-ip-address=51.250.11.100 \
        --generic-ssh-user yc-user \
        --generic-ssh-key ~/.ssh/appuser \
        docker-host
  
        eval $(docker-machine env docker-host)

        $ docker-machine ls
        NAME          ACTIVE   DRIVER       STATE     URL             SWARM   DOCKER      ERRORS
        docker-host   -        generic      Running   tcp://51.250.11.100:2376           v20.10.11

* Созданы Dockerfile, db_config, mongod.conf, start.sh

        $ docker build -t reddit:latest .

        $ docker run --name reddit -d --network=host reddit:latest

* Образ загружен на Docker Hub

# Docker-образы. Микросервисы

- Docker-образы для сервисного приложения
- оптимизация работы с Docker-образом
- Запуск и работа приложения на основе Docker-образов

Новая структура приложения: каталог `scr`, три компонента - `post-py`, `comment`, `ui`
* ./post-py/Dockerfile
* ./comment/Dockerfile
* ./ui/Dockerfile
### Сборка приложения

        docker pull mongo:latest
        docker build -t nikkinoble/post:1.0 ./post-py
        docker build -t nikkinoble/comment:1.0 ./comment
        docker build -t nikkinoble/ui:1.0 ./ui
### Запуск приложения

        docker network create reddit
        docker run -d --network=reddit --network-alias=post_db --network-alias=comment_db mongo:latest
        docker run -d --network=reddit --network-alias=post nikkinoble/post:1.0
        docker run -d --network=reddit --network-alias=comment nikkinoble/comment:1.0
        docker run -d --network=reddit -p 9292:9292 nikkinoble/ui:1.0

### Улучшаем образ 

        FROM ubuntu:16.04
        RUN apt-get update \
        && apt-get install -y ruby-full ruby-dev build-essential \
        && gem install bundler --no-ri --no-rdoc
        ENV APP_HOME /app
        RUN mkdir $APP_HOME
        WORKDIR $APP_HOME
        ADD Gemfile* $APP_HOME/
        RUN bundle install
        ADD . $APP_HOME
        ENV POST_SERVICE_HOST post
        ENV POST_SERVICE_PORT 5000
        ENV COMMENT_SERVICE_HOST comment
        ENV COMMENT_SERVICE_PORT 9292
        CMD ["puma"]

### Docker volume:

         docker volume create reddit_db

### Перезапуск приложения с volume

        docker kill $(docker ps -q)
        docker run -d --network=reddit --network-alias=post_db --network-alias=comment_db -v reddit_db:/data/db mongo:latest
        docker run -d --network=reddit --network-alias=post nikkinoble/post:1.0
        docker run -d --network=reddit --network-alias=comment nikkinoble/comment:1.0
        docker run -d --network=reddit -p 9292:9292 nikkinoble/ui:2.0
        