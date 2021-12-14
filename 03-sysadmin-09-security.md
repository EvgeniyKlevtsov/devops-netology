## Домашнее задание к занятию "3.9. Элементы безопасности информационных систем"
### 1. Установите Bitwarden плагин для браузера. Зарегистрируйтесь и сохраните несколько паролей.
Установил, поработал. Впечатления не однозначные. Пока оставил. 
### 2. Установите Google authenticator на мобильный телефон. Настройте вход в Bitwarden аккаунт через Google authenticator OTP.
Также попробовал. 
### 3. Установите apache2, сгенерируйте самоподписанный сертификат, настройте тестовый сайт для работы по HTTPS.
Система установилась и доступна по сетевому адресу по не защищенному протоколу. С помощью OpenSSL создадим пару самоподписанных ключей и сертификатов. После настройки, в моём случае Nginx, для использования SSL, зайдем на страницу по протоколу `https`. Получим такое сообщение: 
[![imageban](https://i4.imageban.ru/out/2021/12/13/3da1bed192c4886832ff956052be84ec.png)](https://imageban.ru)
поскольку созданный нами сертификат не подписан одним из доверенных центров сертификации нашего браузера. Если нажать на кнопку `Принять риск и продолжить` откроется наш сайт, но рядом с адресом будет значок замка с восклицательным знаком. 
[![imageban](https://i6.imageban.ru/out/2021/12/13/ea320c833430433e13f9babba9f86f2a.png)](https://imageban.ru)

Это означает, что сертификат не может быть проверен.

### 4. Проверьте на TLS уязвимости произвольный сайт в интернете.
Скачиваем скрипт, используя команду `git clone --depth 1 https://github.com/drwetter/testssl.sh.git`. Заходим в папку и запускаем проверку.
```
evgeniy@evgeniy-ms7788:/testssl.sh$ ./testssl.sh -U --sneaky https://wn.dn.ua/

###########################################################
    testssl.sh       3.1dev from https://testssl.sh/dev/
    (6da72bc 2021-12-10 20:16:28 -- )

      This program is free software. Distribution and
             modification under GPLv2 permitted.
      USAGE w/o ANY WARRANTY. USE IT AT YOUR OWN RISK!

       Please file bugs @ https://testssl.sh/bugs/

###########################################################

 Using "OpenSSL 1.0.2-chacha (1.0.2k-dev)" [~179 ciphers]
 on evgeniy-ms7788:./bin/openssl.Linux.x86_64
 (built: "Jan 18 17:12:17 2019", platform: "linux-x86_64")


 Start 2021-12-13 17:42:16        -->> 195.95.139.27:443 (wn.dn.ua) <<--

 rDNS (195.95.139.27):   ip-195-95-139-27.wn.dn.ua.
 Service detected:       HTTP


 Testing vulnerabilities 

 Heartbleed (CVE-2014-0160)                not vulnerable (OK), timed out
 CCS (CVE-2014-0224)                       not vulnerable (OK)
 Ticketbleed (CVE-2016-9244), experiment.  not vulnerable (OK)
 ROBOT                                     not vulnerable (OK)
 Secure Renegotiation (RFC 5746)           supported (OK)
 Secure Client-Initiated Renegotiation     not vulnerable (OK)
 CRIME, TLS (CVE-2012-4929)                not vulnerable (OK)
 BREACH (CVE-2013-3587)                    no gzip/deflate/compress/br HTTP compression (OK)  - only supplied "/" tested
 POODLE, SSL (CVE-2014-3566)               not vulnerable (OK)
 TLS_FALLBACK_SCSV (RFC 7507)              Downgrade attack prevention supported (OK)
 SWEET32 (CVE-2016-2183, CVE-2016-6329)    VULNERABLE, uses 64 bit block ciphers
 FREAK (CVE-2015-0204)                     not vulnerable (OK)
 DROWN (CVE-2016-0800, CVE-2016-0703)      not vulnerable on this host and port (OK)
                                           make sure you don't use this certificate elsewhere with SSLv2 enabled services
                                           https://censys.io/ipv4?q=35CD8C210BCC9EE73E94B8FB5B5060961A608C482BA4EC1C6F3884D18620E351 could help you to find out
 LOGJAM (CVE-2015-4000), experimental      common prime with 2048 bits detected: RFC3526/Oakley Group 14 (2048 bits),
                                           but no DH EXPORT ciphers
 BEAST (CVE-2011-3389)                     TLS1: ECDHE-RSA-AES256-SHA
                                                 DHE-RSA-AES256-SHA
                                                 DHE-RSA-CAMELLIA256-SHA AES256-SHA
                                                 CAMELLIA256-SHA ECDHE-RSA-AES128-SHA
                                                 DHE-RSA-AES128-SHA
                                                 DHE-RSA-CAMELLIA128-SHA AES128-SHA
                                                 CAMELLIA128-SHA
                                                 ECDHE-RSA-DES-CBC3-SHA
                                                 EDH-RSA-DES-CBC3-SHA DES-CBC3-SHA 
                                           VULNERABLE -- but also supports higher protocols  TLSv1.1 TLSv1.2 (likely mitigated)
 LUCKY13 (CVE-2013-0169), experimental     potentially VULNERABLE, uses cipher block chaining (CBC) ciphers with TLS. Check patches
 Winshock (CVE-2014-6321), experimental    not vulnerable (OK) - CAMELLIA or ECDHE_RSA GCM ciphers found
 RC4 (CVE-2013-2566, CVE-2015-2808)        no RC4 ciphers detected (OK)


 Done 2021-12-13 17:43:00 [  46s] -->> 195.95.139.27:443 (wn.dn.ua) <<--
```
### 5. Установите на Ubuntu ssh сервер, сгенерируйте новый приватный ключ. Скопируйте свой публичный ключ на другой сервер. Подключитесь к серверу по SSH-ключу.
Генерация ключей происходит при помощи команды `ssh-keygen`. После чего копируем данные на нужный нам хост, используя `ssh-copy-id username@remote_host`. После успешного выполнения команды мы можем подключаться по SSH, используя `ssh username@remote_host`
### 6. Переименуйте файлы ключей из задания 5. Настройте файл конфигурации SSH клиента, так чтобы вход на удаленный сервер осуществлялся по имени сервера.
Переименовать файл ключа например в authorized_keys. `cat  id_rsa.pub  &gt;&gt;  authorized_keys` Если такой файл уже существует, то нужно добавить в его конец содержимое id_rsa.pub

Для входа на удаленный сервер по имени хоста используется файл `.ssh/config`. Создадим его и пропишем настройки для подключения. В моем примере это 

```
Host localhost
	User evgeniy
```
Пробуем подключится. Результатом получим
```
evgeniy@evgeniy-ms7788:~$ ssh localhost
evgeniy@localhost's password: 
Linux evgeniy-ms7788 5.10.0-9-amd64 #1 SMP Debian 5.10.70-1 (2021-09-30) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Tue Dec 14 09:37:46 2021 from ::1
```
### 7. Соберите дамп трафика утилитой tcpdump в формате pcap, 100 пакетов. Откройте файл pcap в Wireshark.
Воспользовавшись утилитой `tcpdump` и командой `sudo tcpdump -c 100 -w test.pcap` собрали дамп трафика и открыли его в Wireshark.
[![imageban](https://i3.imageban.ru/out/2021/12/14/9fc44521b0b9a852ad49330926ccc6b5.png)](https://imageban.ru)
