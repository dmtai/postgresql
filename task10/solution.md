Создаем ВМ/докер c ПГ.<br>
Создаем БД, схему и в ней таблицу.<br>

```
create database test3;
\c test3;
create schema test_schema;
CREATE TABLE test_schema.test_table (id integer PRIMARY KEY, name text);
```

Заполним таблицы автосгенерированными 100 записями

```
INSERT INTO test_schema.test_table VALUES (generate_series(1, 100), random()::text);
```

Под линукс пользователем Postgres создадим каталог для бэкапов

```
sudo mkdir /backups
sudo chown postgres:postgres /backups
```

Сделаем логический бэкап используя утилиту COPY

```
COPY test_schema.test_table TO '/backups/test_table1.sql' (DELIMITER ',');
```

Восстановим в 2 таблицу данные из бэкапа.

```
create table test_schema.test_table2 (like test_schema.test_table including all);
copy test_schema.test_table2 from '/backups/test_table1.sql';
```

Используя утилиту pg_dump создадим бэкап в кастомном сжатом формате двух таблиц

```
pg_dump --dbname=test3 --schema=test_schema --table=test_schema.test_table* --format=c --compress=6 --file=test3.sql```
```
Используя утилиту pg_restore восстановим в новую БД только вторую таблицу

```
CREATE DATABASE test4;
\c test4;
CREATE SCHEMA test_schema;

sudo -u postgres pg_restore --dbname=test4 --table=test_table  test3.sql
```







