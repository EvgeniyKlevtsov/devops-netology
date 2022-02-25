## Домашнее задание к занятию "6.3. MySQL"

## Задача 1

Используя docker поднимите инстанс MySQL (версию 8). Данные БД сохраните в volume.
Изучите  [бэкап БД](https://github.com/netology-code/virt-homeworks/tree/master/06-db-03-mysql/test_data)  и восстановитесь из него.
Перейдите в управляющую консоль  `mysql`  внутри контейнера.
Используя команду  `\h`  получите список управляющих команд.
Найдите команду для выдачи статуса БД и  **приведите в ответе**  из ее вывода версию сервера БД.
Подключитесь к восстановленной БД и получите список таблиц из этой БД.
**Приведите в ответе**  количество записей с  `price`  > 300.
В следующих заданиях мы будем продолжать работу с данным контейнером.

1. Запускаем docker-comose
```yaml
version: '3.9'

services:
  mysql_db:
    platform: linux/x86_64
    container_name: mysql8
    image: mysql:8
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: test_db
      MYSQL_USER: test-admin-user
      MYSQL_PASSWORD: test
      MYSQL_ALLOW_EMPTY_PASSWORD: "yes"
    volumes:
      - /home/evgeniy/mysql/data:/var/lib/mysql
      - /home/evgeniy/mysql/etc/mysql/conf.d:/etc/mysql/conf.d
    ports:
      - '3306:3306'
```
`evgeniy@evgeniy-ms7788:~/mq$ docker-compose up -d`

```
Восстанавливаем базу:
root@d7779e1112b0:/# mysql -u root -p test_db < /var/lib/mysql/test_dump.sql 
Enter password: 
```
```
Статус БД:
mysql> \s
--------------
mysql  Ver 8.0.28 for Linux on x86_64 (MySQL Community Server - GPL)

Connection id:		10
Current database:	
Current user:		root@localhost
SSL:			Not in use
Current pager:		stdout
Using outfile:		''
Using delimiter:	;
Server version:		8.0.28 MySQL Community Server - GPL
Protocol version:	10
Connection:		Localhost via UNIX socket
Server characterset:	utf8mb4
Db     characterset:	utf8mb4
Client characterset:	latin1
Conn.  characterset:	latin1
UNIX socket:		/var/run/mysqld/mysqld.sock
Binary data as:		Hexadecimal
Uptime:			9 min 20 sec

Threads: 2  Questions: 5  Slow queries: 0  Opens: 117  Flush tables: 3  Open tables: 36  Queries per second avg: 0.008
--------------
```
```
Список таблиц из этой БД:

mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| test_db            |
+--------------------+
5 rows in set (0.57 sec)

Database changed
mysql> SHOW TABLES FROM test_db;
+-------------------+
| Tables_in_test_db |
+-------------------+
| orders            |
+-------------------+
1 row in set (0.01 sec)

mysql> SELECT * FROM orders;
+----+-----------------------+-------+
| id | title                 | price |
+----+-----------------------+-------+
|  1 | War and Peace         |   100 |
|  2 | My little pony        |   500 |
|  3 | Adventure mysql times |   300 |
|  4 | Server gravity falls  |   300 |
|  5 | Log gossips           |   123 |
+----+-----------------------+-------+
5 rows in set (0.02 sec)
```
```
Количество записей с price > 300.

mysql> SELECT * from orders o where o.price > 300;
+----+----------------+-------+
| id | title          | price |
+----+----------------+-------+
|  2 | My little pony |   500 |
+----+----------------+-------+
1 row in set (0.00 sec)
```
### Задача 2

Создайте пользователя test в БД c паролем test-pass, используя:

-   плагин авторизации mysql_native_password
-   срок истечения пароля - 180 дней
-   количество попыток авторизации - 3
-   максимальное количество запросов в час - 100

атрибуты пользователя:

-   Фамилия "Pretty"
-   Имя "James"
```
Создадим пользователя, используя команду:
mysql> CREATE USER 'test'@'localhost'
    -> IDENTIFIED WITH caching_sha2_password BY 'test-pass'
    -> REQUIRE NONE
    -> WITH MAX_QUERIES_PER_HOUR 100
    -> PASSWORD EXPIRE INTERVAL 180 DAY FAILED_LOGIN_ATTEMPTS 3 PASSWORD_LOCK_TIME 2
    -> ATTRIBUTE '{"fname": "Pretty", "lname": "James"}';

Даём привелегии пользователю test:
mysql> GRANT SELECT ON test_db.* TO 'test'@'localhost';
Query OK, 0 rows affected (0.28 sec)

Данные по пользователю test
mysql> SELECT * FROM INFORMATION_SCHEMA.USER_ATTRIBUTES WHERE USER = "test";
+------+-----------+---------------------------------------+
| USER | HOST      | ATTRIBUTE                             |
+------+-----------+---------------------------------------+
| test | localhost | {"fname": "Pretty", "lname": "James"} |
+------+-----------+---------------------------------------+
```
### Задача 3

Установите профилирование  `SET profiling = 1`. Изучите вывод профилирования команд  `SHOW PROFILES;`.

Исследуйте, какой  `engine`  используется в таблице БД  `test_db`  и  **приведите в ответе**.

Измените  `engine`  и  **приведите время выполнения и запрос на изменения из профайлера в ответе**:

-   на  `MyISAM`
-   на  `InnoDB`
```
1. 
mysql> SET profiling = 1;
Query OK, 0 rows affected, 1 warning (0.04 sec)
mysql> DROP TABLE IF EXISTS t1;
Query OK, 0 rows affected, 1 warning (0.10 sec)
mysql> SHOW PROFILES;
+----------+------------+-------------------------+
| Query_ID | Duration   | Query                   |
+----------+------------+-------------------------+
|        1 | 0.12523275 | DROP TABLE IF EXISTS t1 |
+----------+------------+-------------------------+
1 row in set, 1 warning (0.02 sec)
```
```
2. 
mysql> SELECT TABLE_NAME, ENGINE FROM information_schema.TABLES where TABLE_SCHEMA = 'test_db';
+------------+--------+
| TABLE_NAME | ENGINE |
+------------+--------+
| orders     | InnoDB |
+------------+--------+
1 row in set (0.00 sec)
```
```
3.
mysql> ALTER TABLE orders ENGINE = MyIsam;
Query OK, 5 rows affected (4.79 sec)
Records: 5  Duplicates: 0  Warnings: 0

mysql> SELECT TABLE_NAME, ENGINE FROM information_schema.TABLES where TABLE_SCHEMA = 'test_db';
+------------+--------+
| TABLE_NAME | ENGINE |
+------------+--------+
| orders     | MyISAM |
+------------+--------+
1 row in set (0.03 sec)
```
```
mysql> SHOW PROFILES;
+----------+------------+-----------------------------------------------------------------------------------------+
| Query_ID | Duration   | Query                                                                                   |
+----------+------------+-----------------------------------------------------------------------------------------+
|        1 | 0.12523275 | DROP TABLE IF EXISTS t1                                                                 |
|        2 | 0.01287500 | SELECT TABLE_NAME, ENGINE FROM information_schema.TABLES where TABLE_SCHEMA = 'test_db' |
|        3 | 4.78983150 | ALTER TABLE orders ENGINE = MyIsam                                                      |
|        4 | 0.02738425 | SELECT TABLE_NAME, ENGINE FROM information_schema.TABLES where TABLE_SCHEMA = 'test_db' |
+----------+------------+-----------------------------------------------------------------------------------------+
4 rows in set, 1 warning (0.00 sec)

mysql> SHOW PROFILE FOR QUERY 4;
+--------------------------------+----------+
| Status                         | Duration |
+--------------------------------+----------+
| starting                       | 0.000126 |
| Executing hook on transaction  | 0.000007 |
| starting                       | 0.000009 |
| checking permissions           | 0.000007 |
| Opening tables                 | 0.026479 |
| init                           | 0.000035 |
| System lock                    | 0.000025 |
| optimizing                     | 0.000047 |
| statistics                     | 0.000191 |
| preparing                      | 0.000077 |
| executing                      | 0.000136 |
| checking permissions           | 0.000068 |
| end                            | 0.000012 |
| query end                      | 0.000009 |
| waiting for handler commit     | 0.000020 |
| closing tables                 | 0.000028 |
| freeing items                  | 0.000086 |
| cleaning up                    | 0.000026 |
+--------------------------------+----------+
18 rows in set, 1 warning (0.03 sec)
```

### Задача 4

Изучите файл  `my.cnf`  в директории /etc/mysql.

Измените его согласно ТЗ (движок InnoDB):

-   Скорость IO важнее сохранности данных
-   Нужна компрессия таблиц для экономии места на диске
-   Размер буффера с незакомиченными транзакциями 1 Мб
-   Буффер кеширования 30% от ОЗУ
-   Размер файла логов операций 100 Мб

Приведите в ответе измененный файл  `my.cnf`.

Вывод файла конфигурации:
```
root@d7779e1112b0:/# cat /etc/mysql/my.cnf
# Copyright (c) 2017, Oracle and/or its affiliates. All rights reserved.
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; version 2 of the License.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA  02110-1301 USA

#
# The MySQL  Server configuration file.
#
# For explanations see
# http://dev.mysql.com/doc/mysql/en/server-system-variables.html

[mysqld]
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /var/lib/mysql
secure-file-priv= NULL

# Custom config should go here
!includedir /etc/mysql/conf.d/
root@d7779e1112b0:/# 
```
```
Добавим требуемые параметры:

[mysqld]
innodb_flush_log_at_trx_commit = 2
innodb_buffer_pool_size = 582M
innodb_log_buffer_size = 1M
innodb_log_file_size = 100M
[mysql]
```
