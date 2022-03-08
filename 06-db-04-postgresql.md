## Домашнее задание к занятию "6.4. PostgreSQL"

###  Задача 1

Используя docker поднимите инстанс PostgreSQL (версию 13). Данные БД сохраните в volume.
Подключитесь к БД PostgreSQL используя  `psql`.
Воспользуйтесь командой  `\?`  для вывода подсказки по имеющимся в  `psql`  управляющим командам.
**Найдите и приведите**  управляющие команды для:
-   вывода списка БД
-   подключения к БД
-   вывода списка таблиц
-   вывода описания содержимого таблиц
-   выхода из psql

Поднял инстанс PostgreSQL (версию 13) используя `docker-compose.yml` из предыдущего задания (только заменив версию). Подключился к контейнеру. И подключаюсь к базе используя `psql`.
```
root@ad49188eb121:/# psql -U admin_test -d test_db
psql (13.6 (Debian 13.6-1.pgdg110+1))
Type "help" for help.

test_db=#
```
- Воспользовавшись `\?` нашел управляющие команды для `psql`
- Вывода списка БД: `\l`
- Подключения к БД: `\c[onnect] {[DBNAME|- USER|- HOST|- PORT|-] | conninfo} connect to new database`
- Вывод списка таблиц: `\dt[S+] [PATTERN] list tables`
- Вывод описания содержимого таблиц: `\d[S+] NAME describe table, view, sequence, or index`
- Выход из psql: `\q`

### Задача 2

Используя  `psql`  создайте БД  `test_database`.
Изучите  [бэкап БД](https://github.com/netology-code/virt-homeworks/tree/master/06-db-04-postgresql/test_data).
Восстановите бэкап БД в  `test_database`.
Перейдите в управляющую консоль  `psql`  внутри контейнера.
Подключитесь к восстановленной БД и проведите операцию ANALYZE для сбора статистики по таблице.
Используя таблицу  [pg_stats](https://postgrespro.ru/docs/postgresql/12/view-pg-stats), найдите столбец таблицы  `orders`  с наибольшим средним значением размера элементов в байтах.
**Приведите в ответе**  команду, которую вы использовали для вычисления и полученный результат.

Создаем базу 
```
test_db=# CREATE DATABASE test_database; 
CREATE DATABASE
```
Восстанавливаем бэкап БД в  `test_database`.
```
root@ad49188eb121:/var/lib/postgresql/data# psql -U admin_test -f ./test_dump.sql test_database
```
Подключились к восстановленной БД и проводим операцию ANALYZE
```
test_database=# ANALYZE VERBOSE public.orders;
INFO:  analyzing "public.orders"
INFO:  "orders": scanned 1 of 1 pages, containing 8 live rows and 8 dead rows; 8 rows in sample, 8 estimated total rows
ANALYZE
```
Используя таблицу  `pg_stats`, находим столбец таблицы  `orders`  с наибольшим средним значением размера элементов в байтах.
В результате получим:
```
 test_database=# select avg_width from pg_stats where tablename='orders';
 avg_width 
-----------
         4
        16
         4
(3 rows)
 ```
 
 ### Задача 3

Архитектор и администратор БД выяснили, что ваша таблица orders разрослась до невиданных размеров и поиск по ней занимает долгое время. Вам, как успешному выпускнику курсов DevOps в нетологии предложили провести разбиение таблицы на 2 (шардировать на orders_1 - price>499 и orders_2 - price<=499).

Предложите SQL-транзакцию для проведения данной операции.

Можно ли было изначально исключить "ручное" разбиение при проектировании таблицы orders?

Преобразуем существующую таблицу в партиционированную.
```
test_database=# alter table orders rename to orders_simple;
ALTER TABLE
test_database=# create table orders (id integer, title varchar(80), price integer) partition by range(price);
CREATE TABLE
test_database=# create table orders_less499 partition of orders for values from (0) to (499);
CREATE TABLE
test_database=# create table orders_more499 partition of orders for values from (499) to (999999999);
CREATE TABLE
test_database=# insert into orders (id, title, price) select * from orders_simple;
INSERT 0 8
```
Посмотрим наполнение созданных таблиц:
```
test_database=# select * from orders_less499;
 id |        title         | price 
----+----------------------+-------
  1 | War and peace        |   100
  3 | Adventure psql time  |   300
  4 | Server gravity falls |   300
  5 | Log gossips          |   123
(4 rows)

test_database=# select * from orders_more499;
 id |       title        | price 
----+--------------------+-------
  2 | My little database |   500
  6 | WAL never lies     |   900
  7 | Me and my bash-pet |   499
  8 | Dbiezdmin          |   501
(4 rows)
```
Думаю создание триггера будет ответом. Т.е. на старте проекта при разработке мы можем уже сделать шардирование и при каждом insert-запросе данные автоматически распределять по разным таблицам.

### Задача 4

Используя утилиту  `pg_dump`  создайте бекап БД  `test_database`.

Как бы вы доработали бэкап-файл, чтобы добавить уникальность значения столбца  `title`  для таблиц  `test_database`?

Создадим бекап БД `test_database`.
```
root@ad49188eb121:/backup# pg_dump -U admin_test -d test_database >test_database_dump.sql
```
Для уникальности можно добавить индекса для таблицы orders поля title.
```
create unique index orders_title_uindex on orders (title);
```
Доработка.
Погуглив, нашел вот такой ответ:
```
Индекс PostgreSQL `UNIQUE` обеспечивает уникальность значений в одном или нескольких столбцах. Для создания `UNIQUE`индекса вы можете использовать следующий синтаксис:

`CREATE UNIQUE INDEX index_name
ON table_name(column_name, [...]);`

Обратите внимание, что только индексы B-дерева могут быть объявлены как уникальные индексы.
Когда вы определяете `UNIQUE`индекс для столбца, столбец не может хранить несколько строк с одинаковыми значениями.
Если вы определяете `UNIQUE`индекс для двух или более столбцов, объединенные значения в этих столбцах не могут дублироваться в нескольких строках.
PostgreSQL рассматривает `NULL` как отдельные значения, поэтому вы можете иметь несколько `NULL`значений в столбце с `UNIQUE`индексом.
Когда вы определяете первичный ключ или уникальное ограничение для таблицы, PostgreSQL автоматически создает соответствующий `UNIQUE`индекс.
```
Ну и если я правильно понял, уникальность нужно было создать до преобразования таблицы в партиционированную. Для уникальности можно добавить индекс или первичный ключ.
`CREATE INDEX ON orders ((lower(title)));`
