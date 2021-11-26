# Домашнее задание к занятию "3.3. Операционные системы, лекция 1"

## 1. Какой системный вызов делает команда cd? В прошлом ДЗ мы выяснили, что cd не является самостоятельной программой, это shell builtin, поэтому запустить strace непосредственно на cd не получится. Тем не менее, вы можете запустить strace на /bin/bash -c 'cd /tmp'. В этом случае вы увидите полный список системных вызовов, которые делает сам bash при старте. Вам нужно найти тот единственный, который относится именно к cd.

evgeniy@evgeniy-ms7788:~$ strace /bin/bash -c 'cd /tmp' 2>&1 | grep tmp

execve("/bin/bash", ["/bin/bash", "-c", "cd /tmp"], 0x7ffd67766470 /* 55 vars */) = 0

stat("/tmp", {st_mode=S_IFDIR|S_ISVTX|0777, st_size=4096, ...}) = 0

**chdir("/tmp")**
системный вызов команды в нашем случае chdir("/tmp"). 

### 2. Попробуйте использовать команду file на объекты разных типов на файловой системе. Например:

`vagrant@netology1:~$ file /dev/tty`
`/dev/tty: character special (5/0)`
`vagrant@netology1:~$ file /dev/sda`  
`/dev/sda: block special (8/0)`  
`vagrant@netology1:~$ file /bin/bash`
`/bin/bash: ELF 64-bit LSB shared object, x86-64`

### Используя strace выясните, где находится база данных file на основании которой она делает свои догадки.

Файл базы данных типов - /usr/share/misc/magic.mgc
```
openat(AT_FDCWD, "/usr/share/misc/magic.mgc", O_RDONLY) = 3
```
### 3. Предположим, приложение пишет лог в текстовый файл. Этот файл оказался удален (deleted в lsof), однако возможности сигналом сказать приложению переоткрыть файлы или просто перезапустить приложение – нет. Так как приложение продолжает писать в удаленный файл, место на диске постепенно заканчивается. Основываясь на знаниях о перенаправлении потоков предложите способ обнуления открытого удаленного файла (чтобы освободить место на файловой системе).

Создадим процесс и будем писать результат в файл test `top > test`. Откроем сессию в другом терминале и через `lsof | grep test` увидим, что есть файл, в который пишется результат. Удалим файл `rm test`.  Зная PID процесса, из команды ранее,  выполним `cat /dev/null > /proc/PID/fd/1`. Содержимое файла будет удалено.

### 4. Занимают ли зомби-процессы какие-то ресурсы в ОС (CPU, RAM, IO)?

Зомби процессы, в отличии от сирот освобождают свои ресурсы, но не освобождают запись в таблице процессов. 
Запись освободиться при вызове wait() родительским процессом.

### 5. В iovisor BCC есть утилита opensnoop

`root@vagrant:~# dpkg -L bpfcc-tools | grep sbin/opensnoop` 
`/usr/sbin/opensnoop-bpfcc`

### На какие файлы вы увидели вызовы группы  `open`  за первую секунду работы утилиты? Воспользуйтесь пакетом  `bpfcc-tools`  для Ubuntu 20.04.

>`vagrant@vagrant:~$ sudo opensnoop-bpfcc`

PID    COMM               FD ERR PATH
746    vminfo              4   0 /var/run/utmp
567    dbus-daemon        -1   2 /usr/local/share/dbus-1/system-services
567    dbus-daemon        20   0 /usr/share/dbus-1/system-services
567    dbus-daemon        -1   2 /lib/dbus-1/system-services
567    dbus-daemon        20   0 /var/lib/snapd/dbus-1/system-services/
381    systemd-udevd      14   0 /sys/fs/cgroup/unified/system.slice/systemd-udevd.service/cgroup.procs
381    systemd-udevd      14   0 /sys/fs/cgroup/unified/system.slice/systemd-udevd.service/cgroup.threads

### 6. Какой системный вызов использует  `uname -a`? Приведите цитату из man по этому системному вызову, где описывается альтернативное местоположение в  `/proc`, где можно узнать версию ядра и релиз ОС.
В `man proc` на 4078 строке описано 
/proc/version
              This string identifies the kernel version that is currently running.   It  includes  the  contents  of /proc/sys/kernel/ostype, /proc/sys/kernel/osrelease  and  /proc/sys/kernel/version. 

### 7. Чем отличается последовательность команд через ; и через && в bash? Например
`root@netology1:~# test -d /tmp/some_dir; echo Hi`
`Hi`
`root@netology1:~# test -d /tmp/some_dir && echo Hi`
`root@netology1:~#`
### Есть ли смысл использовать в bash &&, если применить set -e?
Так как `&&` - условный оператор, а `;` - разделитель последовательных команд. Отличие заключается в том, что параметры, введённые после `&&` будут выполнены лишь в случае возвращения кода выхода (0) команды до `&&`. В случае с `;` такое ограничение отсутствует. Поэтому `test -d /tmp/some_dir && echo Hi`,  `echo`  отработает только при успешном завершении команды `test`. 
`set -e` - прерывает сессию при любом ненулевом значении исполняемых команд. В случае `&&`  вместе с `set -e`  вероятно не имеет смысла, так как при ошибке, выполнение команд прекратиться. 

### 8. Из каких опций состоит режим bash set -euxo pipefail и почему его хорошо было бы использовать в сценариях?

`-e` завершает работу, если команда, лист или составная команда выдает ненулевой статус вывода.  
`-u` считывает неустановленные переменные и спецсимволы (кроме @ и * ) и завершает программу возвращая ненулевой результат.  
`-x` подставляет значения переменных (команды, case, select, арифметических и т.д.) и выводит значение переменной PS4  
`-o pipefail` устанавливает значение пайплайна на последнее (самое правое) выражение, с ненулевым кодом или выходит, в случае успешного выполнения всех введённых команд.

Использование данных параметров, спасёт от зацикливания и позволит исключить ошибки ввода. И завершит сценарий при наличии ошибок, на любом этапе выполнения.

### 9. Используя  `-o stat`  для  `ps`, определите, какой наиболее часто встречающийся статус у процессов в системе. В  `man ps`  ознакомьтесь  `(/PROCESS STATE CODES)`  Что значат дополнительные к основной заглавной буквы статуса процессов. Его можно не учитывать при расчете (считать  `S`,  `Ss`  или  `Ssl`  равнозначными).

Используя `man`к `ps`  нашёл параметр `-axo`, отображающий все статусы процессов. 

>PROCESS STATE CODES
       Here are the different values that the s, stat and state output
       specifiers (header "STAT" or "S") will display to describe the state of
       a process:`

               D    uninterruptible sleep (usually IO)
               I    Idle kernel thread
               R    running or runnable (on run queue)
               S    interruptible sleep (waiting for an event to complete)
               T    stopped by job control signal
               t    stopped by debugger during the tracing
               W    paging (not valid since the 2.6.xx kernel)
               X    dead (should never be seen)
               Z    defunct ("zombie") process, terminated but not reaped by
                    its parent
Список состояния процессов на моём компьютере.
>evgeniy@evgeniy-ms7788:~$ ps axu S | grep -c D
8
evgeniy@evgeniy-ms7788:~$ ps axu S | grep -c I
48
evgeniy@evgeniy-ms7788:~$ ps axu S | grep -c R
8
evgeniy@evgeniy-ms7788:~$ ps axu S | grep -c S
185
evgeniy@evgeniy-ms7788:~$ ps axu S | grep -c T
4
evgeniy@evgeniy-ms7788:~$ ps axu S | grep -c t
21
evgeniy@evgeniy-ms7788:~$ ps axu S | grep -c W
3
evgeniy@evgeniy-ms7788:~$ ps axu S | grep -c X
2
evgeniy@evgeniy-ms7788:~$ ps axu S | grep -c Z
3

Больше всего процессов `S - interruptible sleep`
