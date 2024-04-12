Создать инстанс ВМ с 2 ядрами и 4 Гб ОЗУ и SSD 10GB<br>
Установить на него PostgreSQL 15 с дефолтными настройками<br>
```
sudo apt install postgresql-15
```
Создать БД для тестов: выполнить pgbench -i postgres
```
sudo -u postgres pgbench -i postgres
dropping old tables...
creating tables...
generating data (client-side)...
100000 of 100000 tuples (100%) done (elapsed 0.06 s, remaining 0.00 s)
vacuuming...
creating primary keys...
done in 0.16 s (drop tables 0.01 s, create tables 0.00 s, client-side generate 0.07 s, vacuum 0.04 s, primary keys 0.03 s).
```
Запустить pgbench -c8 -P 6 -T 60 -U postgres postgres
```
sudo -u postgres pgbench -c8 -P 6 -T 60 -U postgres postgres
pgbench (15.6 (Ubuntu 15.6-1.pgdg20.04+1))
starting vacuum...end.
progress: 6.0 s, 2094.0 tps, lat 3.782 ms stddev 2.374, 0 failed
progress: 12.0 s, 1964.9 tps, lat 4.044 ms stddev 2.768, 0 failed
progress: 18.0 s, 2072.8 tps, lat 3.835 ms stddev 2.538, 0 failed
progress: 24.0 s, 2082.7 tps, lat 3.820 ms stddev 2.643, 0 failed
progress: 30.0 s, 2081.3 tps, lat 3.820 ms stddev 2.510, 0 failed
progress: 36.0 s, 1980.2 tps, lat 4.015 ms stddev 2.635, 0 failed
progress: 42.0 s, 1994.6 tps, lat 3.985 ms stddev 2.616, 0 failed
progress: 48.0 s, 2072.0 tps, lat 3.833 ms stddev 2.367, 0 failed
progress: 54.0 s, 2045.2 tps, lat 3.888 ms stddev 2.626, 0 failed
progress: 60.0 s, 2001.8 tps, lat 3.970 ms stddev 2.455, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 122345
number of failed transactions: 0 (0.000%)
latency average = 3.897 ms
latency stddev = 2.556 ms
initial connection time = 11.551 ms
tps = 2039.187536 (without initial connection time)
```
Применить параметры настройки PostgreSQL из прикрепленного к материалам занятия файла<br>
Протестировать заново<br>

```
sudo -u postgres pgbench -c8 -P 6 -T 60 -U postgres postgres
pgbench (15.6 (Ubuntu 15.6-1.pgdg20.04+1))
starting vacuum...end.
progress: 6.0 s, 1983.5 tps, lat 3.998 ms stddev 2.742, 0 failed
progress: 12.0 s, 1991.2 tps, lat 3.993 ms stddev 2.640, 0 failed
progress: 18.0 s, 1799.9 tps, lat 4.412 ms stddev 3.050, 0 failed
progress: 24.0 s, 1901.6 tps, lat 4.180 ms stddev 2.699, 0 failed
progress: 30.0 s, 1936.2 tps, lat 4.106 ms stddev 2.795, 0 failed
progress: 36.0 s, 1933.8 tps, lat 4.110 ms stddev 2.797, 0 failed
progress: 42.0 s, 1944.7 tps, lat 4.090 ms stddev 2.612, 0 failed
progress: 48.0 s, 1957.2 tps, lat 4.060 ms stddev 2.533, 0 failed
progress: 54.0 s, 1988.9 tps, lat 3.997 ms stddev 2.584, 0 failed
progress: 60.0 s, 1926.0 tps, lat 4.124 ms stddev 2.746, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 116190
number of failed transactions: 0 (0.000%)
latency average = 4.104 ms
latency stddev = 2.723 ms
initial connection time = 13.369 ms
tps = 1936.507036 (without initial connection time)
```

Что изменилось и почему?<br>
```
Производительность стала хуже, tps снизился, судя по всему из-за изменения настроек.
```

Создать таблицу с текстовым полем и заполнить случайными или сгенерированными данным в размере 1млн строк

```
create table student(text text);
insert into student(text) select 'data' from generate_series(1, 1000000);
```

Посмотреть размер файла с таблицей
```
select pg_size_pretty(pg_total_relation_size('student'));
 pg_size_pretty
----------------
 35 MB
(1 row)
```
5 раз обновить все строчки и добавить к каждой строчке любой символ. 
```
update student set text = 'data1';
update student set text = 'data2';
update student set text = 'data3';
update student set text = 'data4';
update student set text = 'data5';
```

Посмотреть количество мертвых строчек в таблице и когда последний раз приходил автовакуум
```
select relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%",
last_autovacuum FROM pg_stat_user_tables where relname = 'student';
 relname | n_live_tup | n_dead_tup | ratio% | last_autovacuum 
---------+------------+------------+--------+-----------------
 student |    1000000 |    4999846 |    499 | 
(1 row)
```

Подождать некоторое время, проверяя, пришел ли автовакуум<br>
```
 relname | n_live_tup | n_dead_tup | ratio% |        last_autovacuum        
---------+------------+------------+--------+-------------------------------
 student |    1000000 |          0 |      0 | 2024-04-13 01:36:34.151572+07
(1 row)
```

5 раз обновить все строчки и добавить к каждой строчке любой символ<br>
```
update student set text = 'data1';
update student set text = 'data2';
update student set text = 'data3';
update student set text = 'data4';
update student set text = 'data5';
```
Посмотреть размер файла с таблицей
```
select pg_size_pretty(pg_total_relation_size('student'));
 pg_size_pretty 
----------------
 207 MB
(1 row)
```
Отключить Автовакуум на конкретной таблице
```
ALTER TABLE student SET(autovacuum_enabled = on);
```

10 раз обновить все строчки и добавить к каждой строчке любой символ
```
update student set text = 'data1';
update student set text = 'data2';
update student set text = 'data3';
update student set text = 'data4';
update student set text = 'data5';
update student set text = 'data1';
update student set text = 'data2';
update student set text = 'data3';
update student set text = 'data4';
update student set text = 'data5';
```

Посмотреть размер файла с таблицей
```
select pg_size_pretty(pg_total_relation_size('student'));
 pg_size_pretty 
----------------
 380 MB
(1 row)
```

Объясните полученный результат
```
После изменения 5 раз было создано множество мертвых строк и вес таблицы увеличился с 35 до 207мб.
После автовакуума повторные изменения не изменили размер, т.к. было использовано освободившееся пространство.
После отключения автовакуума и изменения 10 раз размер увеличился еще до 380мб, т.к. пространство, занятое
мертвыми строками не освобождалось автовакуумом.
```

Не забудьте включить автовакуум)
```
alter table test set (autovacuum_enabled = on);
```


