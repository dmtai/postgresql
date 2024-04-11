создайте новый кластер PostgresSQL 14
```
sudo apt install postgresql-14
```

зайдите в созданный кластер под пользователем postgres
```
sudo -u postgres psql
```

создайте новую базу данных testdb<br>
зайдите в созданную базу данных под пользователем postgres<br>

```
CREATE DATABASE testdb;
\c testdb

```

создайте новую схему testnm

```
CREATE SCHEMA testnm;
```

создайте новую таблицу t1 с одной колонкой c1 типа integer

```
CREATE TABLE t1 (c1 integer);
```

вставьте строку со значением c1=1

```
INSERT INTO t1 VALUES (1);
```

создайте новую роль readonly

```
CREATE ROLE readonly;
```

дайте новой роли право на подключение к базе данных testdb
```
testdb=# GRANT CONNECT ON DATABASE testdb TO readonly;
```

дайте новой роли право на использование схемы testnm
```
testdb=# GRANT USAGE ON SCHEMA testnm TO readonly;
```

дайте новой роли право на select для всех таблиц схемы testnm
```
testdb=# GRANT SELECT ON ALL TABLES IN SCHEMA testnm TO readonly;
```

создайте пользователя testread с паролем test123
```
testdb=# CREATE USER testread PASSWORD 'test123';
```

дайте роль readonly пользователю testread
```
GRANT readonly TO testread;
```

зайдите под пользователем testread в базу данных testdb

```
psql -U testread -h 127.0.0.1 -d testdb
```

сделайте select * from t1;

```
select * from t1;
ERROR: permission denied for relation t1
```

получилось? (могло если вы делали сами не по шпаргалке и не упустили один существенный момент про который позже)
```
Нет, permission denied for relation t1
```

напишите что именно произошло в тексте домашнего задания<br>
у вас есть идеи почему? ведь права то дали?<br>
посмотрите на список таблиц<br>

```
\d
        List of relations
 Schema | Name | Type  |  Owner   
--------+------+-------+----------
 public | t1   | table | postgres
```

подсказка в шпаргалке под пунктом 20<br>
а почему так получилось с таблицей (если делали сами и без шпаргалки то может у вас все нормально)<br>

```
t1 в public, права есть на testnm. Так получилось, потому что дефолтная схема public
```

вернитесь в базу данных testdb под пользователем postgres
удалите таблицу t1

```
drop table t1;
```

создайте ее заново но уже с явным указанием имени схемы testnm

```
create table testnm.t1 (c1 integer);
```

вставьте строку со значением c1=1

```
insert into testnm.t1 values(1);
```

зайдите под пользователем testread в базу данных testdb<br>
сделайте select * from testnm.t1;<br>

```
testdb=> select * from testnm.t1;
ERROR: permission denied for relation t1
```

получилось?
есть идеи почему? если нет - смотрите шпаргалку

```
Нет, таблица создана после того, как роли readonly установлены права на чтение таблиц в testnm,
права выданы только на существующие таблицы
```

как сделать так чтобы такое больше не повторялось? если нет идей - смотрите шпаргалку

```
Нужно поменять привилегии по умолчанию, выдаваемые при создании таблиц
```
```
alter default privileges in schema testnm grant select on tables to readonly;
create table testnm.t2 (c2 integer);
insert into testnm.t2 values(2);
```

сделайте select * from testnm.t1; в базе testdb под пользователем testread<br>
получилось?<br>

```
testdb=> select * from testnm.t1;
ERROR: permission denied for relation t1
```

есть идеи почему? если нет - смотрите шпаргалку

```
Новые привилегии применяются к объектам, которые были созданы после их изменения
```

сделайте select * from testnm.t1; под пользователем testread<br>
получилось?<br>
```
да
```
ура!

теперь попробуйте выполнить команду create table t2(c1 integer); insert into t2 values (2);<br>
```
create table t2(c1 integer);
insert into t2 values (2);
```
а как так? нам же никто прав на создание таблиц и insert в них под ролью readonly?<br>

```
Роль public имеет права на использование схемы public. Insert сработал, т.к. создатель таблицы имеет права на это
```

есть идеи как убрать эти права? если нет - смотрите шпаргалку<br>
Если вы справились сами то расскажите что сделали и почему, если 
смотрели шпаргалку - объясните что сделали и почему выполнив указанные в ней команды

```
Нужно отозвать у роли public права на схему public
```
```
REVOKE ALL ON SCHEMA public FROM public;
```

теперь попробуйте выполнить команду create table t3(c1 integer); insert into t2 values (2);<br>
расскажите что получилось и почему<br>

```
Прав нет на public, поэтому создать новую таблицу и сделать вставку в существующую не получилось
```
