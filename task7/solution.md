Настройте выполнение контрольной точки раз в 30 секунд.
```
в postgresql.conf checkpoint_timeout = 30s
```

10 минут c помощью утилиты pgbench подавайте нагрузку.

```
pgbench -i postgres -s 50
pgbench -c8 -P 6 -T 600 -U postgres postgres
```

Измерьте, какой объем журнальных файлов был сгенерирован за это время. Оцените, какой объем приходится в среднем на одну контрольную точку.

```
SELECT * FROM pg_ls_waldir();
```
```
файлы по 16мб, 25 файлов, 400мб. 400/20 = 20
```

Проверьте данные статистики: все ли контрольные точки выполнялись точно по расписанию. Почему так произошло?
```
Все контрольные точки были выполнены по расписанию, checkpoints_timed = 30, а checkpoints_req = 0.
```

Сравните tps в синхронном/асинхронном режиме утилитой pgbench. Объясните полученный результат.

```
Асинхронный режим

synchronous_commit = off в postgresql.conf

sudo -u postgres pgbench -c 8 -P 10 -T 60 -U postgres postgres

progress: 10.0 s, 10028.0 tps, lat 0.785 ms stddev 0.272
progress: 20.0 s, 10260.8 tps, lat 0.769 ms stddev 0.270
progress: 30.0 s, 10107.3 tps, lat 0.781 ms stddev 0.260
progress: 40.0 s, 10136.2 tps, lat 0.779 ms stddev 0.270
progress: 50.0 s, 10167.5 tps, lat 0.776 ms stddev 0.265
progress: 60.0 s, 10333.1 tps, lat 0.764 ms stddev 0.206
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 50
query mode: simple
number of clients: 8
number of threads: 1
duration: 60 s
number of transactions actually processed: 610337
latency average = 0.775 ms
latency stddev = 0.258 ms
tps = 10171.976527 (including connections establishing)
tps = 10172.469144 (excluding connections establishing)


Синхронный режим synchronous_commit = on
sudo -u postgres pgbench -c 8 -P 10 -T 60 -U postgres postgres
progress: 10.0 s, 5699.7 tps, lat 1.387 ms stddev 0.562
progress: 20.0 s, 5477.7 tps, lat 1.447 ms stddev 0.669
progress: 30.0 s, 5760.0 tps, lat 1.375 ms stddev 0.541
progress: 40.0 s, 4924.9 tps, lat 1.611 ms stddev 0.846
progress: 50.0 s, 5074.9 tps, lat 1.562 ms stddev 0.936
progress: 60.0 s, 5770.3 tps, lat 1.374 ms stddev 0.483
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 50
query mode: simple
number of clients: 8
number of threads: 1
duration: 60 s
number of transactions actually processed: 327083
latency average = 1.453 ms
latency stddev = 0.689 ms
tps = 5451.255178 (including connections establishing)
tps = 5451.542832 (excluding connections establishing)
```
```
В асинхронном режиме latency меньше, tps больше, потому что фиксация изменений не ждет физической записи на диск.
```

Создайте новый кластер с включенной контрольной суммой страниц. Создайте таблицу. 
Вставьте несколько значений. Выключите кластер. Измените пару байт в таблице. Включите кластер и 
сделайте выборку из таблицы. Что и почему произошло? как проигнорировать ошибку и продолжить работу?

```
sudo -u postgres /usr/lib/postgresql/15/bin/pg_checksums -e -P -D /var/lib/postgresql/15/main

create table test (id integer, data varchar(10));
insert into test values (1, 'sdf'), (2, 'bvc'), (3, 'ghk');

/usr/lib/postgresql/15/bin/pg_ctl -D /var/lib/postgresql/15/extra stop

echo foobar >> /var/lib/postgresql/15/extra/base/5/16387

select * from test;
ERROR:  invalid page in block 0 of relation base/4/14215
```
```
При выборке ошибка, контрольные суммы не совпали. Для исправления можно отключить проверку чексуммы.

set ignore_checksum_failure = on;

select * from test;
 id | data 
----+------
  1 | sdf
  2 | bvc
  3 | ghk
(3 rows)
```






