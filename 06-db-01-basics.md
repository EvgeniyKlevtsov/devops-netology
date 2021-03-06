## Домашнее задание к занятию "6.1. Типы и структура СУБД"

### Задача 1

Архитектор ПО решил проконсультироваться у вас, какой тип БД лучше выбрать для хранения определенных данных.

Он вам предоставил следующие типы сущностей, которые нужно будет хранить в БД:

-   Электронные чеки в json виде
-   Склады и автомобильные дороги для логистической компании
-   Генеалогические деревья
-   Кэш идентификаторов клиентов с ограниченным временем жизни для движка аутентификации
-   Отношения клиент-покупка для интернет-магазина

Выберите подходящие типы СУБД для каждой сущности и объясните свой выбор.
1. Для данного кейса лучше подойдёт документо-ориентированная СУБД. MongoDB - она хранит данные в виде JSON.
2. Подойдёт графовая БД, где вершины это города, а дороги это ребра графа.
3. Подойдет БД имеющая древовидную структуру.
4. Для этого подойдёт Memcached или Redis, т.к. поддерживают TTL и отлично подходят в качестве инструментов кэширования.
5. Подойдёт реляционная бд, так как позволяет реализовать отношение многие-ко-многим (клиент делает много покупок, одинаковые продукты могут покупать разные люди)

### Задача 2

Вы создали распределенное высоконагруженное приложение и хотите классифицировать его согласно CAP-теореме. Какой классификации по CAP-теореме соответствует ваша система, если (каждый пункт - это отдельная реализация вашей системы и для каждого пункта надо привести классификацию):

-   Данные записываются на все узлы с задержкой до часа (асинхронная запись)
-   При сетевых сбоях, система может разделиться на 2 раздельных кластера
-   Система может не прислать корректный ответ или сбросить соединение
1. По CAP-теореме это считается доступная система, поэтому (мне так кажется) возможны два варианта: CA - данные доступны и согласованы и AP - данные доступны и устойчивы к разделению, но не согласованы в каждый момент времени
2. AP - данные доступны и устойчивы к разделению, но не согласованы в каждый момент времени.
3. Скорее всего это, CP-система в которой в случае сбоя запросы могут теряться.

А согласно PACELC-теореме, как бы вы классифицировали данные реализации?
1.  PA/EC - данные доступны, но не согласованы для всех узлов если они есть. Либо согласованы и доступны если всего 1 узел.
2.  Здесь, как мне кажется, возможны как PA/EL, так и PA/EC в зависимости от бизнес процесса.
3. PC/EC т.к. соединение сбросится, значит мы не получим ответ быстро и (L)atency, не будет выполнено.

### Задача 3

Могут ли в одной системе сочетаться принципы BASE и ACID? Почему?

Не может, потому что это два противоположных друг другу принципа.
Основная цель принципа BASE добиться высокой доступности/производительности в угоду согласованности данных.
Основная цель принципа ACID добиться согласованности данных в угоду доступности/производительности.
Но допускается совмещение разных СУБД в рамках одного проекта для реализации конкретных бизнес потребностей.

### Задача 4

Вам дали задачу написать системное решение, основой которого бы послужили:

-   фиксация некоторых значений с временем жизни
-   реакция на истечение таймаута

Вы слышали о key-value хранилище, которое имеет механизм  [Pub/Sub](https://habr.com/ru/post/278237/). Что это за система? Какие минусы выбора данной системы?

Данным требованиям удовлетворяет Redis. Это СУБД типа "ключ-значение", имеющая поддержку pub/sub и TTL. 
-   фиксация некоторых значений с временем жизни  - это обычная установка ключей с ttl.
- реакция на истечение таймаута  - это подписка на канал где будут сообщения с информацией о ключах, которые истекли.

Минусы данной системы:
-   Размер БД ограничен доступной памятью.
-   Шардинг (техника масштабирования) ведет к увеличению задержки.
-  Нет сегментации на пользователей или группы пользователей. Отсутствует контроль доступа.
-   Это нереляционная СУБД!
