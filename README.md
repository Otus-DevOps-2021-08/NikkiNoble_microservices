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
