### 1. Работа c HTTP через телнет.

-   Подключитесь утилитой телнет к сайту stackoverflow.com `telnet stackoverflow.com 80`
-   отправьте HTTP запрос  
    ```
    GET /questions HTTP/1.0
    HOST: stackoverflow.com 
    press enter]
    press enter]
    ```
-   В ответе укажите полученный HTTP код, что он означает?  
    Результатом мы получим ответ с кодом 301  `HTTP/1.1 301 Moved Permanently`  - страница была перемещена на постоянной основе
```
evgeniy@evgeniy-ms7788:~$ telnet stackoverflow.com 80
Trying 151.101.1.69...
Connected to stackoverflow.com.
Escape character is '^]'.
GET /questions HTTP/1.0
HOST: stackoverflow.com


HTTP/1.1 301 Moved Permanently
cache-control: no-cache, no-store, must-revalidate
location: https://stackoverflow.com/questions
x-request-guid: 34f2c79a-5a91-46ca-96e8-fcb37d05664f
feature-policy: microphone 'none'; speaker 'none'
content-security-policy: upgrade-insecure-requests; frame-ancestors 'self' https://stackexchange.com
Accept-Ranges: bytes
Date: Fri, 03 Dec 2021 09:25:33 GMT
Via: 1.1 varnish
Connection: close
X-Served-By: cache-hhn4039-HHN
X-Cache: MISS
X-Cache-Hits: 0
X-Timer: S1638523534.616543,VS0,VE170
Vary: Fastly-SSL
X-DNS-Prefetch-Control: off
Set-Cookie: prov=1a0af5d6-1466-a493-8799-2fa2d80a6cbd; domain=.stackoverflow.com; expires=Fri, 01-Jan-2055 00:00:00 GMT; path=/; HttpOnly

Connection closed by foreign host.
```

### 2. Повторите задание 1 в браузере, используя консоль разработчика F12.

-   откройте вкладку Network
-   отправьте запрос  http://stackoverflow.com
-   найдите первый ответ HTTP сервера, откройте вкладку Headers
-   укажите в ответе полученный HTTP код.

[![imageban](https://i4.imageban.ru/out/2021/12/05/1a9982438010017957700f47cd2de8c7.png)](https://imageban.ru)

-   проверьте время загрузки страницы, какой запрос обрабатывался дольше всего?
-   приложите скриншот консоли браузера в ответ.
[![imageban](https://i2.imageban.ru/out/2021/12/05/f394fc33c06d70fedf0afe2c998ea382.png)](https://imageban.ru)

### 3. Какой IP адрес у вас в интернете?

Воспользуемся командой `curl`
```
deemer@Home:~$ curl 2ip.ru
195.95.139.85
```
### 4. Какому провайдеру принадлежит ваш IP адрес? Какой автономной системе AS? Воспользуйтесь утилитой whois
Исходя из результатов вывода команды `whois`, мы видим наш провайдер  `WELCOME_NET`, номер автономной системы `AS41081`
```
deemer@Home:~$ whois -h whois.ripe.net 195.95.139.85
% This is the RIPE Database query service.
% The objects are in RPSL format.
%
% The RIPE Database is subject to Terms and Conditions.
% See http://www.ripe.net/db/support/db-terms-conditions.pdf

% Note: this output has been filtered.
%       To receive output for a database update, use the "-B" flag.

% Information related to '195.95.139.0 - 195.95.139.255'

% Abuse contact for '195.95.139.0 - 195.95.139.255' is 'abuse@wn.dn.ua'

inetnum:        195.95.139.0 - 195.95.139.255
netname:        WELCOME-NET
country:        UA
org:            ORG-WN6-RIPE
admin-c:        SS7547-RIPE
tech-c:         SS7547-RIPE
status:         ASSIGNED PI
mnt-by:         RIPE-NCC-END-MNT
mnt-by:         WELCOME_NET
mnt-domains:    WELCOME_NET
mnt-routes:     WELCOME_NET
created:        2006-05-18T10:01:59Z
last-modified:  2016-04-14T08:28:35Z
source:         RIPE
sponsoring-org: ORG-SF2-RIPE

organisation:   ORG-WN6-RIPE
org-name:       Private enterprise Gorbunov A.A.
descr:          Private enterprise Gorbunov A.A.
org-type:       OTHER
address:        Ukrain, Donetsk, Enakievo, Ermishina 34-37
abuse-c:        AR19066-RIPE
mnt-ref:        WELCOME_NET
mnt-by:         WELCOME_NET
created:        2006-04-28T09:48:11Z
last-modified:  2014-03-27T15:28:20Z
source:         RIPE # Filtered

person:         Stas Saleev
address:        Ukrain, Donetsk, Enakievo, Metallurgov, 42
phone:          +380501370688
nic-hdl:        SS7547-RIPE
created:        2006-04-28T07:00:27Z
last-modified:  2014-03-07T09:15:48Z
source:         RIPE
mnt-by:         WELCOME_NET

% Information related to '195.95.139.0/24AS41081'

route:          195.95.139.0/24
descr:          Routing for WELCOME-NET network
origin:         AS41081
mnt-by:         WELCOME_NET
created:        2006-10-26T06:08:15Z
last-modified:  2006-10-26T06:08:15Z
source:         RIPE

% This query was served by the RIPE Database Query Service version 1.101 (BLAARKOP)
```
### 5. Через какие сети проходит пакет, отправленный с вашего компьютера на адрес 8.8.8.8? Через какие AS? Воспользуйтесь утилитой traceroute
Выполним `traceroute -An 8.8.8.8` результатом получим:
```
deemer@Home:~$ traceroute -An 8.8.8.8
traceroute to 8.8.8.8 (8.8.8.8), 30 hops max, 60 byte packets
 1  192.168.0.1 [*]  0.268 ms  0.357 ms  0.395 ms
 2  91.216.28.33 [AS41081]  1.276 ms  1.560 ms  1.783 ms
 3  192.168.10.4 [*]  1.161 ms  1.156 ms  1.164 ms
 4  91.216.28.18 [AS41081]  3.704 ms  4.388 ms  6.313 ms
 5  195.95.139.145 [AS41081]  1.457 ms  1.464 ms  1.470 ms
 6  92.242.110.33 [AS41039/AS3261]  28.496 ms  3.819 ms  3.244 ms
 7  5.153.143.6 [AS41039]  5.201 ms  4.374 ms  4.378 ms
 8  31.133.40.206 [AS52091]  3.987 ms  2.318 ms  2.300 ms
 9  31.133.40.221 [AS52091]  26.267 ms  26.585 ms  26.713 ms
10  31.133.40.250 [AS52091]  25.785 ms  25.791 ms  25.751 ms
11  * * *
12  108.170.250.33 [AS15169]  25.602 ms 108.170.227.70 [AS15169]  23.755 ms 108.170.250.129 [AS15169]  24.801 ms
13  108.170.250.130 [AS15169]  22.826 ms 108.170.250.34 [AS15169]  25.363 ms  25.352 ms
14  142.251.49.24 [AS15169]  37.010 ms *  37.041 ms
15  216.239.57.222 [AS15169]  42.749 ms 209.85.254.20 [AS15169]  41.008 ms 72.14.238.168 [AS15169]  45.162 ms
16  72.14.236.73 [AS15169]  39.428 ms 142.250.56.217 [AS15169]  45.387 ms 72.14.236.73 [AS15169]  38.467 ms
17  * * *
18  * * *
19  * * *
20  * * *
21  * * *
22  * * *
23  * * *
24  * * *
25  * * *
26  * 8.8.8.8 [AS15169]  37.835 ms  36.878 ms
deemer@Home:~$ 
```
### 6. Повторите задание 5 в утилите mtr. На каком участке наибольшая задержка - delay?
![imageban](https://i3.imageban.ru/out/2021/12/05/47dd25b5207e432ad865274fe58fd1b7.png)Из полученного результата видим что наибольшая задержка на участке  `72.14.238.168`
### 7. Какие DNS сервера отвечают за доменное имя dns.google? Какие A записи? воспользуйтесь утилитой dig
```
deemer@Home:~$ dig dns.google

; <<>> DiG 9.16.1-Ubuntu <<>> dns.google
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 22737
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;dns.google.			IN	A

;; ANSWER SECTION:
dns.google.		112	IN	A	8.8.4.4
dns.google.		112	IN	A	8.8.8.8

;; Query time: 3 msec
;; SERVER: 127.0.0.53#53(127.0.0.53)
;; WHEN: Вс дек 05 18:35:07 MSK 2021
;; MSG SIZE  rcvd: 71
```
### 8. Проверьте PTR записи для IP адресов из задания 7. Какое доменное имя привязано к IP? воспользуйтесь утилитой dig

```
deemer@Home:~$ dig -x 8.8.8.8

; <<>> DiG 9.16.1-Ubuntu <<>> -x 8.8.8.8
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 9764
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;8.8.8.8.in-addr.arpa.		IN	PTR

;; ANSWER SECTION:
8.8.8.8.in-addr.arpa.	4630	IN	PTR	dns.google.

;; Query time: 0 msec
;; SERVER: 127.0.0.53#53(127.0.0.53)
;; WHEN: Вс дек 05 18:37:43 MSK 2021
;; MSG SIZE  rcvd: 73
```
Из результата видно `8.8.8.8.in-addr.arpa. 4630 IN PTR dns.google.`
