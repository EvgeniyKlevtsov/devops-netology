## Домашнее задание к занятию "6.5. Elasticsearch"

### Задача 1

В этом задании вы потренируетесь в:

-   установке elasticsearch
-   первоначальном конфигурировании elastcisearch
-   запуске elasticsearch в docker

Используя докер образ  [centos:7](https://hub.docker.com/_/centos)  как базовый и  [документацию по установке и запуску Elastcisearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/targz.html):

-   составьте Dockerfile-манифест для elasticsearch
-   соберите docker-образ и сделайте  `push`  в ваш docker.io репозиторий
-   запустите контейнер из получившегося образа и выполните запрос пути  `/`  c хост-машины

Требования к  `elasticsearch.yml`:

-   данные  `path`  должны сохраняться в  `/var/lib`
-   имя ноды должно быть  `netology_test`

В ответе приведите:

-   текст Dockerfile манифеста
-   ссылку на образ в репозитории dockerhub
-   ответ  `elasticsearch`  на запрос пути  `/`  в json виде

Подсказки:

-   возможно вам понадобится установка пакета perl-Digest-SHA для корректной работы пакета shasum
-   при сетевых проблемах внимательно изучите кластерные и сетевые настройки в elasticsearch.yml
-   при некоторых проблемах вам поможет docker директива ulimit
-   elasticsearch в логах обычно описывает проблему и пути ее решения

Далее мы будем работать с данным экземпляром elasticsearch.

Dockerfile
```json
FROM centos:7
ENV PATH=/usr/lib:/usr/lib/jvm/jre-11/bin:$PATH

RUN yum install java-11-openjdk -y 
RUN yum install wget -y 

RUN wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.17.0-linux-x86_64.tar.gz \
    && wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.17.0-linux-x86_64.tar.gz.sha512 
RUN yum install perl-Digest-SHA -y 
RUN shasum -a 512 -c elasticsearch-7.17.0-linux-x86_64.tar.gz.sha512 \ 
    && tar -xzf elasticsearch-7.17.0-linux-x86_64.tar.gz \
    && yum upgrade -y
    
ADD elasticsearch.yml /elasticsearch-7.17.0/config/
ENV JAVA_HOME=/elasticsearch-7.17.0/jdk/
ENV ES_HOME=/elasticsearch-7.17.0
RUN groupadd elasticsearch \
    && useradd -g elasticsearch elasticsearch
    
RUN mkdir /var/lib/logs \
    && chown elasticsearch:elasticsearch /var/lib/logs \
    && mkdir /var/lib/data \
    && chown elasticsearch:elasticsearch /var/lib/data \
    && chown -R elasticsearch:elasticsearch /elasticsearch-7.17.0/
RUN mkdir /elasticsearch-7.17.0/snapshots &&\
    chown elasticsearch:elasticsearch /elasticsearch-7.17.0/snapshots
    
USER elasticsearch
CMD ["/usr/sbin/init"]
CMD ["/elasticsearch-7.17.0/bin/elasticsearch"]
```
elasticsearch.yml файл
```yml
cluster.name: netology_test
discovery.type: single-node
node.name: netology_test
path.data: /var/lib/data
path.logs: /var/lib/logs
path.repo: /elasticsearch-7.17.0/snapshots
network.host: 0.0.0.0
http.port: 9200
discovery.seed_hosts: ["127.0.0.1", "[::1]"]
```
```
$ docker build . -t evgeniyklevtsov/elasticsearch:7.17
$ docker login -u "evgeniyklevtsov"
$ docker push evgeniyklevtsov/elasticsearch:7.17
```
- ссылку на образ в репозитории dockerhub: https://hub.docker.com/r/evgeniyklevtsov/elasticsearch

Чтобы запустить, нужно обязательно изменить параметр ядра  `vm.max_map_count`, иначе упадёт с ошибкой:
```
sudo sysctl -w vm.max_map_count=262144
sudo docker run --rm -d --name elastic -p 9200:9200 -p 9300:9300 evgeniyklevtsov/elasticsearch:7.17
```
evgeniy@home:~/elastic$ curl -X GET 'localhost:9200/'
-   ответ  `elasticsearch`  на запрос пути  `/`  в json виде
```json
{
  "name" : "netology_test",
  "cluster_name" : "netology_test",
  "cluster_uuid" : "SkAwP8n1TGGpJrBe1-GBPA",
  "version" : {
    "number" : "7.17.0",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "bee86328705acaa9a6daede7140defd4d9ec56bd",
    "build_date" : "2022-01-28T08:36:04.875279988Z",
    "build_snapshot" : false,
    "lucene_version" : "8.11.1",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

### Задача 2

В этом задании вы научитесь:

-   создавать и удалять индексы
-   изучать состояние кластера
-   обосновывать причину деградации доступности данных

Ознакомьтесь с  [документацией](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-create-index.html)  и добавьте в  `elasticsearch`  3 индекса, в соответствии со таблицей:

| Имя | Количество реплик | Количество шард |
|-----|-------------------|-----------------|
| ind-1| 0 | 1 |
| ind-2 | 1 | 2 |
| ind-3 | 2 | 4 |
Добавляем индексы:
```
$ curl -X PUT "localhost:9200/ind-1?pretty" -H 'Content-Type: application/json' -d'
{
 "settings": {
 "number_of_shards": 1,
 "number_of_replicas": 0
 }
}
'
$ curl -X PUT "localhost:9200/ind-2?pretty" -H 'Content-Type: application/json' -d'
{
 "settings": {
 "number_of_shards": 2,
 "number_of_replicas": 1
 }
}
'
$ curl -X PUT "localhost:9200/ind-3?pretty" -H 'Content-Type: application/json' -d'
{
 "settings": {
 "number_of_shards": 4,
 "number_of_replicas": 2
 }
}
'
```
Получите список индексов и их статусов, используя API и **приведите в ответе** на задание.
```
$ curl 'localhost:9200/_cat/indices?v'
health status index            uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   .geoip_databases 0MNAZBAfQgaY0v1HFC6PrQ   1   0         45            0     42.5mb         42.5mb
green  open   ind-1            p0tI-w3vSROPbtaVKjouow   1   0          0            0       226b           226b
yellow open   ind-3            YwWB2IS7RouvEKPJh6qhAw   4   2          0            0       904b           904b
yellow open   ind-2            mB1mJFsFQSWTCVF2U6faXQ   2   1          0            0       452b           452b
```
Получите состояние кластера `elasticsearch`, используя API.

```
$ curl -X GET "localhost:9200/_cluster/health?pretty"
{
  "cluster_name" : "netology_test",
  "status" : "yellow",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 10,
  "active_shards" : 10,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 10,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 50.0
}
```
 Как вы думаете, почему часть индексов и кластер находится в состоянии yellow?

У них должны быть реплики, но в кластере всего одна нода, поэтому размещать их негде. В таком случае кластер помечает их желтыми.

Удалите все индексы.
```
$ curl -X DELETE 'http://localhost:9200/_all'
```
**Важно**

При проектировании кластера elasticsearch нужно корректно рассчитывать количество реплик и шард, иначе возможна потеря данных индексов, вплоть до полной, при деградации системы.

### Задача 3

В данном задании вы научитесь:

-   создавать бэкапы данных
-   восстанавливать индексы из бэкапов

Задачи:

-   Создайте директорию {путь до корневой директории с elasticsearch в образе}/snapshots.

`path.repo: /elasticsearch-7.17.0/snapshots`  указали в elasticsearch.yml.

-   Используя API зарегистрируйте данную директорию как snapshot repository c именем netology_backup.
-   Приведите в ответе запрос API и результат вызова API для создания репозитория.
```
$ curl -X PUT "localhost:9200/_snapshot/snapshot_repository?pretty" -H 'Content-Type: application/json' -d'
{
  "type": "fs",
  "settings": {
    "location": "/elasticsearch-7.17.0/snapshots/netology_backup/",
    "compress": true
  }
}
'
{
  "acknowledged" : true
}
```
-   Создайте индекс test с 0 реплик и 1 шардом и приведите в ответе список индексов.
```
$ curl -XPUT -H "Content-Type: application/json" http://localhost:9200/test?pretty -d '{
  "settings": {
    "index": {
      "number_of_shards": 1,
      "number_of_replicas": 0
    }
  }
}'
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "test"
}
$ curl -X GET "localhost:9200/_cat/indices?v"
health status index            uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   .geoip_databases 0MNAZBAfQgaY0v1HFC6PrQ   1   0         45            0     42.5mb         42.5mb
green  open   test             NOq7FTIET-W-gUYA4y8Ihw   1   0          0            0       226b           226b
```
-   Создайте snapshot состояния кластера elasticsearch.
```
evgeniy@home:~/elastic$ curl -X PUT "localhost:9200/_snapshot/snapshot_repository/snapshot_1?wait_for_completion=true&pretty"
{
  "snapshot" : {
    "snapshot" : "snapshot_1",
    "uuid" : "mJoHkDVMTbKtXLaeFc1h3Q",
    "repository" : "snapshot_repository",
    "version_id" : 7170099,
    "version" : "7.17.0",
    "indices" : [
      ".geoip_databases",
      ".ds-ilm-history-5-2022.03.13-000001",
      ".ds-.logs-deprecation.elasticsearch-default-2022.03.13-000001",
      "test"
    ],
    "data_streams" : [
      "ilm-history-5",
      ".logs-deprecation.elasticsearch-default"
    ],
    "include_global_state" : true,
    "state" : "SUCCESS",
    "start_time" : "2022-03-13T21:01:27.126Z",
    "start_time_in_millis" : 1647205287126,
    "end_time" : "2022-03-13T21:01:32.528Z",
    "end_time_in_millis" : 1647205292528,
    "duration_in_millis" : 5402,
    "failures" : [ ],
    "shards" : {
      "total" : 4,
      "failed" : 0,
      "successful" : 4
    },
    "feature_states" : [
      {
        "feature_name" : "geoip",
        "indices" : [
          ".geoip_databases"
        ]
      }
    ]
  }
}
```
-   Приведите в ответе список файлов в директории со snapshotами.

Зайдём на контейнер, используя  `sudo docker exec -it elastic bash`  и посмотрим файлы в подключённом репозитории:
```
evgeniy@home:~/elastic$ sudo docker exec -it elastic bash
[elasticsearch@49596d2a338b /]$ ll /elasticsearch-7.17.0/snapshots/netology_backup/
total 28
-rw-r--r-- 1 elasticsearch elasticsearch 1422 Mar 13 21:01 index-0
-rw-r--r-- 1 elasticsearch elasticsearch    8 Mar 13 21:01 index.latest
drwxr-xr-x 6 elasticsearch elasticsearch 4096 Mar 13 21:01 indices
-rw-r--r-- 1 elasticsearch elasticsearch 9732 Mar 13 21:01 meta-mJoHkDVMTbKtXLaeFc1h3Q.dat
-rw-r--r-- 1 elasticsearch elasticsearch  457 Mar 13 21:01 snap-mJoHkDVMTbKtXLaeFc1h3Q.dat
```
-   Удалите индекс test и создайте индекс test-2. Приведите в ответе список индексов.
```
[elasticsearch@49596d2a338b /]$ curl -X DELETE "localhost:9200/test?pretty"
{
  "acknowledged" : true
}
```
создадим индекс test-2:
```
[elasticsearch@49596d2a338b /]$ curl -XPUT -H "Content-Type: application/json" http://localhost:9200/test-2?pretty -d '{
>   "settings": {
>     "index": {
>       "number_of_shards": 1,
>       "number_of_replicas": 0
>     }
>   }
> }'
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "test-2"
}
```
Список индексов:
```
evgeniy@home:~/elastic$ curl -X GET "localhost:9200/_cat/indices?v"
health status index            uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   test-2           YSWx60b3SrGIfjtPg-bwiA   1   0          0            0       226b           226b
green  open   .geoip_databases 0MNAZBAfQgaY0v1HFC6PrQ   1   0         45            0     42.5mb         42.5mb
```
-   Восстановите состояние кластера elasticsearch из snapshot, созданного ранее.
-   Приведите в ответе запрос к API восстановления и итоговый список индексов.
```
curl -X POST "localhost:9200/_snapshot/netology_backup/snapshot_1/_restore?pretty" -H 'Content-Type: application/json' -d'
{
 "indices": "*",
 "include_global_state": true
}
'
```
```
[elasticsearch@d57d88102dac snapshots]$ curl -X GET "localhost:9200/_cat/indices?v"
health status index            uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   .geoip_databases XH6ej89ASd28lwGvj9uz_g   1   0         45            0     42.5mb         42.5mb
green  open   test             -ZCdE7SsTTq8JDjfpGKqVA   1   0          0            0       226b           226b
[
```
