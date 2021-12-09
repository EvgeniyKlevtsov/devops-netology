## Домашнее задание к занятию "3.8. Компьютерные сети, лекция 3"

### 1. Подключитесь к публичному маршрутизатору в интернет. Найдите маршрут к вашему публичному IP
```
telnet route-views.routeviews.org
Username: rviews
show ip route x.x.x.x/32
show bgp x.x.x.x/32
```
Смог воспользоваться этим http://lg.zsttk.net
```
show ip bgp 109.254.245.6
BGP routing table entry for 109.254.224.0/19
Paths: (4 available, best #4, table Default-IP-Routing-Table)

  21127 20485 12389 48276 20590 20590 20590 20590 20590
    82.200.23.77  (Omk) from 82.200.23.77  (Omk) (10.7.55.253)
      Origin IGP, localpref 100, valid, external
      Community: 20485:11154
      Last update: Thu Dec  9 05:31:58 2021

  21127 20485 12389 48276 20590 20590 20590 20590 20590
    82.200.55.209 (Kmr)  from 82.200.55.209 (Kmr)  (10.7.42.252)
      Origin IGP, localpref 100, valid, external
      Community: 20485:11154
      Last update: Thu Dec  9 05:31:39 2021

  21127 20485 12389 48276 20590 20590 20590 20590 20590
    82.200.27.252 (Nkz)  from 82.200.27.252 (Nkz)  (10.7.43.201)
      Origin IGP, localpref 100, valid, external
      Community: 20485:11154
      Last update: Thu Dec  9 05:31:32 2021

  21127 20485 12389 48276 20590 20590 20590 20590 20590
    82.200.1.133  (Tmk) from 82.200.1.133  (Tmk) (10.7.70.253)
      Origin IGP, localpref 100, valid, external, best
      Community: 20485:11154
      Last update: Thu Dec  9 05:31:10 2021
```
### 3. Проверьте открытые TCP порты в Ubuntu, какие протоколы и приложения используют эти порты? Приведите несколько примеров.
```
vagrant@vagrant:~$ sudo lsof -i tcp
COMMAND   PID            USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
systemd     1            root   35u  IPv4  15016      0t0  TCP *:sunrpc (LISTEN)
systemd     1            root   37u  IPv6  15020      0t0  TCP *:sunrpc (LISTEN)
rpcbind   602            _rpc    4u  IPv4  15016      0t0  TCP *:sunrpc (LISTEN)
rpcbind   602            _rpc    6u  IPv6  15020      0t0  TCP *:sunrpc (LISTEN)
systemd-r 603 systemd-resolve   13u  IPv4  19926      0t0  TCP localhost:domain (LISTEN)
sshd      749            root    3u  IPv4  22364      0t0  TCP *:ssh (LISTEN)
sshd      749            root    4u  IPv6  22375      0t0  TCP *:ssh (LISTEN)
sshd      934            root    4u  IPv4  24987      0t0  TCP vagrant:ssh->_gateway:56734 (ESTABLISHED)
sshd      981         vagrant    4u  IPv4  24987      0t0  TCP vagrant:ssh->_gateway:56734 (ESTABLISHED)
```
### 4. Проверьте используемые UDP сокеты в Ubuntu, какие протоколы и приложения используют эти порты?
```
vagrant@vagrant:~$ sudo lsof -i udp
COMMAND   PID            USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
systemd     1            root   36u  IPv4  15017      0t0  UDP *:sunrpc 
systemd     1            root   38u  IPv6  15023      0t0  UDP *:sunrpc 
systemd-n 401 systemd-network   19u  IPv4  17497      0t0  UDP vagrant:bootpc 
rpcbind   602            _rpc    5u  IPv4  15017      0t0  UDP *:sunrpc 
rpcbind   602            _rpc    7u  IPv6  15023      0t0  UDP *:sunrpc 
systemd-r 603 systemd-resolve   12u  IPv4  19925      0t0  UDP localhost:domain
```

### 5. Используя diagrams.net, создайте L3 диаграмму вашей домашней сети или любой другой сети, с которой вы работали.
[![imageban](https://i7.imageban.ru/out/2021/12/09/00cf0003b9828303c8b51c61ad82c06a.png)](https://imageban.ru)
