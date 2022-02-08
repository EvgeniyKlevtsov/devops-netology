## Домашнее задание к занятию "5.5. Оркестрация кластером Docker контейнеров на примере Docker Swarm"

### Задача 1: Дайте письменные ответы на следующие вопросы:

-   В чём отличие режимов работы сервисов в Docker Swarm кластере: replication и global?

В режиме  `replication`  необходимо указывать число реплик (сервисов), которые будут распределены между устройствами кластера. В режиме  `global`  сервис автоматически запустится на всех доступных устройствах кластера.

-   Какой алгоритм выбора лидера используется в Docker Swarm кластере?

RAFT.

-   Что такое Overlay Network?

Распределенная сеть между множественными хостами Docker для использования сервисами. В Swarm по умолчанию используется сеть типа overlay с именем ingress для распределения нагрузки, а так же сеть типа bridge с названием docker_gwbridge для коммуникации самих Docker daemon.

### Задача 2: Создать ваш первый Docker Swarm кластер в Яндекс.Облаке

Для получения зачета, вам необходимо предоставить скриншот из терминала (консоли), с выводом команды:

`docker node ls`

После выполнения всех действий по развёртыванию системы выполняем  требуемую команду и получаем вывод:

```bash
[centos@node01 ~]$ sudo docker node list
ID                            HOSTNAME             STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
c8f4mv08y96uw5no9edyqhmrw *   node01.netology.yc   Ready     Active         Leader           20.10.12
w6u2ej63se8w46tjq5ht40tad     node02.netology.yc   Ready     Active         Reachable        20.10.12
ufs5i83cx7gd6kr9wvvyv7jcw     node03.netology.yc   Ready     Active         Reachable        20.10.12
ofxkoqj8crkre2a5rc2vc967j     node04.netology.yc   Ready     Active                          20.10.12
jvhgs581e9e5aj715bsa5sera     node05.netology.yc   Ready     Active                          20.10.12
h9mk5k8csft99iwbdihm89s8k     node06.netology.yc   Ready     Active                          20.10.12
[centos@node01 ~]$ 
```
### Задача 3: Создать ваш первый, готовый к боевой эксплуатации кластер мониторинга, состоящий из стека микросервисов.

Для получения зачета, вам необходимо предоставить скриншот из терминала (консоли), с выводом команды:

`docker service ls`

Вывод команды следующий:
```bash
[centos@node01 ~]$ sudo docker service ls
ID             NAME                                MODE         REPLICAS   IMAGE                                          PORTS
suubjh5mzx2g   swarm_monitoring_alertmanager       replicated   1/1        stefanprodan/swarmprom-alertmanager:v0.14.0    
rq7pof0n82zn   swarm_monitoring_caddy              replicated   1/1        stefanprodan/caddy:latest                      *:3000->3000/tcp, *:9090->9090/tcp, *:9093-9094->9093-9094/tcp
o12r5hldqou2   swarm_monitoring_cadvisor           global       6/6        google/cadvisor:latest                         
oqcsqbh7d9e3   swarm_monitoring_dockerd-exporter   global       6/6        stefanprodan/caddy:latest                      
a8k26edwf6ht   swarm_monitoring_grafana            replicated   1/1        stefanprodan/swarmprom-grafana:5.3.4           
604zqf8mf8gm   swarm_monitoring_node-exporter      global       6/6        stefanprodan/swarmprom-node-exporter:v0.16.0   
xrafzd9eoaph   swarm_monitoring_prometheus         replicated   1/1        stefanprodan/swarmprom-prometheus:v2.5.0       
08f7t62cpyl1   swarm_monitoring_unsee              replicated   1/1        cloudflare/unsee:v0.8.0                        
[centos@node01 ~]$
```
### Задача 4: Выполнить на лидере Docker Swarm кластера команду (указанную ниже) и дать письменное описание её функционала, что она делает и зачем она нужна:

(см.документацию:  [https://docs.docker.com/engine/swarm/swarm_manager_locking/](https://docs.docker.com/engine/swarm/swarm_manager_locking/))

`docker swarm update --autolock=true`

Данная команда шифрует сертификаты всех нод дополнительным ключём. 
После ввода команды и синхронизации изменений между менеджерами в swarm, в случае рестарта Docker Engine или менеджера целиком, 
Docker демон не сможет после старта прочитать сертификат кластера и будет считаться не доступным для обработки задач кластера. 
После ввода команды  `docker swarm unlock`  на менеджере и ввода ключа (который генерируется при первом запуске  `docker swarm update --autolock=true`) 
связь восстанавливается и менеджер снова становится доступным для деплоя на него сервисов.
