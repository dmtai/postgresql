Секционировать большую таблицу из демо базы flights.
```
Сделал декларативное секционирование по диапазону поля book_date таблицы bookings.bookings
```

```
-- Создал новую таблицу

CREATE TABLE IF NOT EXISTS bookings.bookings2
(
    book_ref character(6) COLLATE pg_catalog."default" NOT NULL,
    book_date timestamp with time zone NOT NULL,
    total_amount numeric(10,2) NOT NULL,
    CONSTRAINT bookings2_pkey PRIMARY KEY (book_ref, book_date)
) partition by range (book_date);


-- Сделал партиции

create table bookings.bookings_part_minvalue partition of bookings2 for values from (MINVALUE) to ('2017-08-16'); 
create table bookings.bookings_part_2017_08_16 partition of bookings2 for values from ('2017-08-16') to ('2024-11-01');

-- Скопировал данные из старой таблицы

insert into bookings.bookings2 select * from bookings.bookings


-- Переименовал старую таблицу

alter table bookings.bookings rename to bookings_archive;
alter table bookings.bookings2 rename to bookings;
```
```
\c demo
You are now connected to database "demo" as user "postgres".
demo=# SELECT
    nmsp_parent.nspname AS parent_schema,
    parent.relname      AS parent,
    nmsp_child.nspname  AS child_schema,
    child.relname       AS child
FROM pg_inherits
    JOIN pg_class parent            ON pg_inherits.inhparent = parent.oid
    JOIN pg_class child             ON pg_inherits.inhrelid   = child.oid
    JOIN pg_namespace nmsp_parent   ON nmsp_parent.oid  = parent.relnamespace
    JOIN pg_namespace nmsp_child    ON nmsp_child.oid   = child.relnamespace
WHERE parent.relname='bookings';

 parent_schema |  parent  | child_schema |          child           
---------------+----------+--------------+--------------------------
 bookings      | bookings | bookings     | bookings_part_minvalue
 bookings      | bookings | bookings     | bookings_part_2017_08_16
(2 rows)
```































