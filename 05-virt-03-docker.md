## Домашнее задание к занятию "5.3. Введение. Экосистема. Архитектура. Жизненный цикл Docker контейнера"
### Задача 1: 

Сценарий выполнения задачи:

-   создайте свой репозиторий на  [https://hub.docker.com](https://hub.docker.com/);
-   выберете любой образ, который содержит веб-сервер Nginx;
-   создайте свой fork образа;
-   реализуйте функциональность: запуск веб-сервера в фоне с индекс-страницей, содержащей HTML-код ниже:
```html
<html>
<head>
Hey, Netology
</head>
<body>
<h1>I’m DevOps Engineer!</h1>
</body>
</html>
```
Опубликуйте созданный форк в своем репозитории и предоставьте ответ в виде ссылки на  [https://hub.docker.com/username_repo](https://hub.docker.com/username_repo).

Для теста использовал следующие команды:

-   скачал официальный образ, используя docker pull nginx
-   создал html страницу, с указанным кодом
-   запустил контейнер, используя команду  
`docker run -d --name netology_nginx -v $PWD:/usr/share/nginx/html nginx`

После того, как проверил что всё работает, приступил к сборке простого docker image предварительно создав Dockerfile:
```
FROM nginx
COPY . /usr/share/nginx/html
ENTRYPOINT ["nginx", "-g", "daemon off;"]
```
Собираем
```
docker buid -t evgeniyklevtsov/netology_nginx:v1.0.0 .
```
Полученный образ залил на докер хаб 
```
docker push evgeniyklevtsov/netology_nginx:v1.0.0
```
[доступен по ссылке](https://hub.docker.com/r/evgeniyklevtsov/netology_nginx)


### Задача 2

Посмотрите на сценарий ниже и ответьте на вопрос: "Подходит ли в этом сценарии использование Docker контейнеров или лучше подойдет виртуальная машина, физическая машина? Может быть возможны разные варианты?"

Детально опишите и обоснуйте свой выбор.

--

Сценарий:

-   Высоконагруженное монолитное java веб-приложение;
	- Да, подходит. Это первый шаг к микросервисной архитектуре для проектов монолитов и самый простой способ это сделать используя docker.
-   Nodejs веб-приложение;
	- Да, подходит. Докер позволит дополнительно реализовать масштабируемость проекта.
-   Мобильное приложение c версиями для Android и iOS;
	- Для данного сценария, полная виртуализация избыточна. Поэтому подойдёт вариант docker и виртуальная машина. Также контейнеры позволят достичь идемпотентности, что отлично подойдёт для тестирования.
-   Шина данных на базе Apache Kafka;
	- Докер идеально подходит под эти задачи.
-   Elasticsearch кластер для реализации логирования продуктивного веб-приложения - три ноды elasticsearch, два logstash и две ноды kibana;
	- Для данного сценария отлично подойдёт Docker-контейнерезация
-   Мониторинг-стек на базе Prometheus и Grafana;
	- Для данного сценария отлично подойдёт Docker-контейнерезация
-   MongoDB, как основное хранилище данных для java-приложения;
	- MongoDB можно загнать в докер и подключать через volume источники данных.
-   Gitlab сервер для реализации CI/CD процессов и приватный (закрытый) Docker Registry.
	- Я бы, развернул это на виртуалке или полноценном сервере, из-за того что сам GitLab собирает докеры и пушит в реестр образов. Поэтому могут быть проблемы с запуском докера внутри докера.

### Задача 3

-   Запустите первый контейнер из образа  centos  c любым тэгом в фоновом режиме, подключив папку  `/data`  из текущей рабочей директории на хостовой машине в  `/data`  контейнера;
-   Запустите второй контейнер из образа  debian  в фоновом режиме, подключив папку  `/data`  из текущей рабочей директории на хостовой машине в  `/data`  контейнера;
-   Подключитесь к первому контейнеру с помощью  `docker exec`  и создайте текстовый файл любого содержания в  `/data`;
-   Добавьте еще один файл в папку  `/data`  на хостовой машине;
-   Подключитесь во второй контейнер и отобразите листинг и содержание файлов в  `/data`  контейнера.

Запустим два контейнера:
```bash
evgeniy@evgeniy-ms7788:~/data$ sudo docker run -it -d -v $PWD:/data centos
Unable to find image 'centos:latest' locally
latest: Pulling from library/centos
a1d0c7532777: Pull complete 
Digest: sha256:a27fd8080b517143cbbbab9dfb7c8571c40d67d534bbdee55bd6c473f432b177
Status: Downloaded newer image for centos:latest
03fa06e6670d8790fb258f72fbffa7075ca7f4332e0f3a6014d8231d88346c26
evgeniy@evgeniy-ms7788:~/data$ sudo docker run -it -d -v $PWD:/data debian
Unable to find image 'debian:latest' locally
latest: Pulling from library/debian
0c6b8ff8c37e: Pull complete 
Digest: sha256:fb45fd4e25abe55a656ca69a7bef70e62099b8bb42a279a5e0ea4ae1ab410e0d
Status: Downloaded newer image for debian:latest
3ac9f2bb1b7b3aa4e4a55058b60b25a6c7d48df161b675110238a4c66cb18300
evgeniy@evgeniy-ms7788:~/data$ pwd
/home/evgeniy/data
evgeniy@evgeniy-ms7788:~/data$
```
Проверим их статус:
```bash
evgeniy@evgeniy-ms7788:~/data$ sudo docker ps
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS          PORTS     NAMES
3ac9f2bb1b7b   debian    "bash"        11 minutes ago   Up 11 minutes             dreamy_gauss
03fa06e6670d   centos    "/bin/bash"   22 minutes ago   Up 21 minutes             inspiring_mendeleev
```
Выполним поставленные задачи:  
В Centos:

```bash
evgeniy@evgeniy-ms7788:~/data$ sudo docker exec -it inspiring_mendeleev /bin/bash
[root@03fa06e6670d /]# cd /data/
[root@03fa06e6670d data]# echo "this is file from centos" >> file_from_centos.txt
[root@03fa06e6670d data]# ls
file_from_centos.txt
[root@03fa06e6670d data]# cat file_from_centos.txt 
this is file from centos
[root@03fa06e6670d data]# 
```
В хостовой машине:
```bash
evgeniy@evgeniy-ms7788:~/data$ echo "I have created this file from host machine" >> file_from_host.txt
evgeniy@evgeniy-ms7788:~/data$ ls
file_from_centos.txt  file_from_host.txt
```
Зайдём в машину с Debian и проверим наличие и содержание файлов:
```bash
evgeniy@evgeniy-ms7788:~/data$ sudo docker exec -it dreamy_gauss /bin/bash
root@3ac9f2bb1b7b:/# cd /data/
root@3ac9f2bb1b7b:/data# ls
file_from_centos.txt  file_from_host.txt
root@3ac9f2bb1b7b:/data# cat file_from_centos.txt 
this is file from centos
root@3ac9f2bb1b7b:/data# cat file_from_host.txt 
I have created this file from host machine
root@3ac9f2bb1b7b:/data# exit
exit
evgeniy@evgeniy-ms7788:~/data$ 
```
### Задача 4 (*)

Воспроизвести практическую часть лекции самостоятельно.

Соберите Docker образ с Ansible, загрузите на Docker Hub и пришлите ссылку вместе с остальными ответами к задачам.

Используя следующий Dockerfile, собрал образ и залил в [репозиторий](https://hub.docker.com/r/evgeniyklevtsov/ansible):

```
FROM alpine:3.14
RUN CARGO_NET_GIT_FETCH_WITH_CLI=1 && \
    apk --no-cache add \
       sudo python3 py3-pip openssl ca-certificates sshpass openssh-client rsync git && \
    apk --no-cache add \
        --virtual build-dependencies python3-dev libffi-dev musl-dev gcc cargo openssl-dev \
         libressl-dev \
         build-base && \
    pip install --upgrade pip wheel && \
    pip install --upgrade cryptography cffi && \
    pip install ansible==2.9.24 && \
    pip install mitogen ansible-lint jmespath && \
    pip install --upgrade pywinrm && \
    apk del build-dependencies && \
    rm -rf /var/cache/apk/* && \
    rm -rf /root/.cache/pip && \
    rm -rf /root/.cargo
RUN mkdir /ansible && \
    mkdir -p /etc/ansible && \
    echo 'localhost' > /etc/ansible/hosts
WORKDIR /ansible
CMD [ "ansible-playbook", "--version"]
```
