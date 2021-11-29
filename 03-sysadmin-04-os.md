## Домашнее задание к занятию "3.4. Операционные системы, лекция 2

### 1. На лекции мы познакомились с node_exporter. В демонстрации его исполняемый файл запускался в background. Этого достаточно для демо, но не для настоящей production-системы, где процессы должны находиться под внешним управлением. Используя знания из лекции по systemd, создайте самостоятельно простой unit-файл для node_exporter:

-   поместите его в автозагрузку,
-   предусмотрите возможность добавления опций к запускаемому процессу через внешний файл (посмотрите, например, на systemctl cat cron),
-   удостоверьтесь, что с помощью systemctl процесс корректно стартует, завершается, а после перезагрузки автоматически поднимается.

 Установим NodeExporter согласно документации на сайте:
`wget https://github.com/prometheus/node_exporter/releases/download/v1.3.0/node_exporter-1.3.0.linux-amd64.tar.gz`
`tar -xf node_exporter-1.3.0.linux-amd64.tar.gz`
`sudo mv node_exporter-1.3.0.linux-amd64/node_exporter /usr/local/bin`

Далее мы создадим пользователей и служебные файлы для node_exporter.
_По соображениям безопасности всегда рекомендуется запускать любые службы / демоны в отдельных учетных записях._
`sudo useradd -rs / bin / false node_exporter`

Затем мы создадим unit-файл.
`sudo nano /etc/systemd/system/node_exporter.service`

```
[Unit]  
Description=Node Exporter  
After=network.target  
[Service]  
User=node_exporter  
Group=node_exporter  
Type=simple
EnvironmentFile=/etc/options/node_exporter  
ExecStart=/usr/local/bin/node_exporter $OPTIONS
[Install]  
WantedBy=multi-user.target
```
Создаём файл с опциями:

`$OPTIONS="--collector.disable-defaults --collector.cpu --collector.meminfo --collector.netdev --collector.diskstats"`

Перезапускаем демона systemd, добавляем сервис в автозагрузку, запускаем его, проверяем статус
```
sudo systemctl daemon-reload  
sudo systemctl enable node_exporter  
sudo systemctl start node_exporter  
sudo systemctl status node_exporter
```
Перезагружаем систему и смотрим, что программа запускается при старте. Всё ок. 
```
vagrant@vagrant:~$ sudo systemctl status node_exporter
● node_exporter.service - Node Exporter
     Loaded: loaded (/etc/systemd/system/node_exporter.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2021-11-29 13:07:08 UTC; 58min ago
   Main PID: 607 (node_exporter)
      Tasks: 5 (limit: 1112)
     Memory: 17.6M
     CGroup: /system.slice/node_exporter.service
             └─607 /usr/local/bin/node_exporter

Nov 29 13:07:14 vagrant node_exporter[607]: ts=2021-11-29T13:07:14.647Z caller=node_exporter.go:115 level=info>
Nov 29 13:07:14 vagrant node_exporter[607]: ts=2021-11-29T13:07:14.647Z caller=node_exporter.go:115 level=info>
Nov 29 13:07:14 vagrant node_exporter[607]: ts=2021-11-29T13:07:14.647Z caller=node_exporter.go:115 level=info>
Nov 29 13:07:14 vagrant node_exporter[607]: ts=2021-11-29T13:07:14.647Z caller=node_exporter.go:115 level=info>
Nov 29 13:07:14 vagrant node_exporter[607]: ts=2021-11-29T13:07:14.647Z caller=node_exporter.go:115 level=info>
Nov 29 13:07:14 vagrant node_exporter[607]: ts=2021-11-29T13:07:14.647Z caller=node_exporter.go:115 level=info>
Nov 29 13:07:14 vagrant node_exporter[607]: ts=2021-11-29T13:07:14.647Z caller=node_exporter.go:115 level=info>
Nov 29 13:07:14 vagrant node_exporter[607]: ts=2021-11-29T13:07:14.647Z caller=node_exporter.go:115 level=info>
Nov 29 13:07:14 vagrant node_exporter[607]: ts=2021-11-29T13:07:14.649Z caller=node_exporter.go:199 level=info>
Nov 29 13:07:14 vagrant node_exporter[607]: ts=2021-11-29T13:07:14.651Z caller=tls_config.go:195 level=info ms>
lines 1-19/19 (END)
```

### 2. Ознакомьтесь с опциями node_exporter и выводом /metrics по-умолчанию. Приведите несколько опций, которые вы бы выбрали для базового мониторинга хоста по CPU, памяти, диску и сети.

Основные метрики можно посмотреть на github, в разделе, посвящённом node_exporter. Для мониторинга я бы взял метрики  `--collector.cpu --collector.meminfo --collector.filesystem --collector.netdev --collector.diskstats`

### 3. Установите в свою виртуальную машину Netdata. Воспользуйтесь готовыми пакетами для установки (sudo apt install -y netdata).

После успешной установки:

-   в конфигурационном файле /etc/netdata/netdata.conf в секции [web] замените значение с localhost на bind to = 0.0.0.0,
-   добавьте в Vagrantfile проброс порта Netdata на свой локальный компьютер и сделайте  `vagrant reload`:
-   `config.vm.network "forwarded_port", guest: 19999, host: 19999`
    После успешной перезагрузки в браузере на своем ПК (не в виртуальной машине) вы должны суметь зайти на localhost:19999. Ознакомьтесь с метриками, которые по умолчанию собираются Netdata и с комментариями, которые даны к этим метрикам.

[![imageban](https://i7.imageban.ru/out/2021/11/29/5f775bc39b6bbcbe0f946733f83c1284.png)](https://imageban.ru)

### 4. Можно ли по выводу  `dmesg`  понять, осознает ли ОС, что загружена не на настоящем оборудовании, а на системе виртуализации?
Судя по выводу dmesg да
```
vagrant@vagrant:~$ dmesg |grep virtua
[    0.032888] CPU MTRRs all blank - virtualized system.
[    0.110398] Booting paravirtualized kernel on KVM
[    5.462471] systemd[1]: Detected virtualization oracle.
```
### 5. Как настроен  `sysctl fs.nr_open`  на системе по-умолчанию? Узнайте, что означает этот параметр. Какой другой существующий лимит не позволит достичь такого числа  `(ulimit --help)`?
Выполнив `sysctl fs.nr_open`
```
vagrant@vagrant:~$ sysctl fs.nr_open
fs.nr_open = 1048576
```
видим `fs.nr_open = 1048576`
Это максимальное число открытых дескрипторов для ядра. Для пользователя задать больше этого числа нельзя. Число задается кратное 1024, в данном случае =1024*1024. 
Максимальный придел можно посмотреть так `cat /proc/sys/fs/file-max` получим 9223372036854775807
Для отображения мягких ограничений используют `ulimit -S`
Для отображения жестких ограничений используют `ulimit -H`
Флаг `-a` будет отображать все параметры и их конфигурацию для конкретного имени пользователя. В нашем случае:
```
vagrant@vagrant:~$ ulimit -a
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 3707
max locked memory       (kbytes, -l) 65536
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 3707
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
```
### 6. Запустите любой долгоживущий процесс (не  `ls`, который отработает мгновенно, а, например,  `sleep 1h`) в отдельном неймспейсе процессов; покажите, что ваш процесс работает под  `PID 1`  через  `nsenter`. Для простоты работайте в данном задании под  `root`  (`sudo -i`). Под обычным пользователем требуются дополнительные опции (`--map-root-user`) и т.д.
```
root@vagrant:~# ps -e |grep sleep
   2119 pts/2    00:00:00 **sleep**
root@vagrant:~# nsenter -t 2119 -p -m
root@vagrant:/# ps
    PID TTY          TIME CMD
      2 pts/0    00:00:00 bash
     11 pts/0    00:00:00 ps
```
     
### 7. Найдите информацию о том, что такое  `:(){ :|:& };:`. Запустите эту команду в своей виртуальной машине Vagrant с Ubuntu 20.04 (это важно, поведение в других ОС не проверялось). Некоторое время все будет "плохо", после чего (минуты) – ОС должна стабилизироваться. Вызов  `dmesg`  расскажет, какой механизм помог автоматической стабилизации. Как настроен этот механизм по-умолчанию, и как изменить число процессов, которое можно создать в сессии?

`:(){ :|:& };:` - так называемая fork bomb - функция, которая вызывает сама себя и плодит процессы. Последней строкой в `dmesg` была информация, о том, что `[ 2655.841433] cgroup: fork rejected by pids controller in /user.slice/user-1000.slice/session-13.scope`. 

Посмотрим на другие лимиты в системе, используя `ulimit --help` и среди них найдём параметр `-u`, отвечающий за количество процессов. Изменить ограничения можно путём корректировки файла `/etc/security/limits.conf file` и его параметра `nproc – max number of processes`. 
Например, если установить `ulimit -u 50` - число процессов будет ограниченно 50 для пользователя.
