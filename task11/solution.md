На 1 ВМ создаем таблицы test для записи, test2 для запросов на чтение.

```
CREATE TABLE test (name text PRIMARY KEY);
CREATE TABLE test2 (name text PRIMARY KEY);
```
```
-- В pghba.conf
host    postgres        postgres        192.168.1.41/32      trust
```

Создаем публикацию таблицы test и подписываемся на публикацию таблицы test2 с ВМ №2.
```
-- Ставлю wal_level=logical
ALTER SYSTEM SET wal_level = logical;

CREATE PUBLICATION test_pub_repl0 FOR TABLE test;

CREATE SUBSCRIPTION test_sub_repl1 CONNECTION 'host=192.168.1.41 user=postgres dbname=postgres' PUBLICATION
test_pub_repl1 WITH (copy_data = TRUE);
```

```
SELECT * FROM pg_stat_subscription \gx

subid                 | 90236
subname               | test2_sub_repl1
pid                   | 58243
relid                 | 
received_lsn          | 0/169D890
last_msg_send_time    | 2024-04-18 04:27:41.969014+07
last_msg_receipt_time | 2024-04-18 04:26:54.333669+07
latest_end_lsn        | 0/169D890
latest_end_time       | 2024-04-18 04:27:41.969014+07
```

На 2 ВМ создаем таблицы test2 для записи, test для запросов на чтение.

```
CREATE TABLE test (name text PRIMARY KEY);
CREATE TABLE test2 (name text PRIMARY KEY);
```
```
-- В pghba.conf
host    postgres        postgres        192.168.1.37/32      trust
```

Создаем публикацию таблицы test2 и подписываемся на публикацию таблицы test1 с ВМ №1.

```
ALTER SYSTEM SET wal_level = logical;

CREATE PUBLICATION test_pub_repl1 FOR TABLE test2;

CREATE SUBSCRIPTION test_sub_repl0 CONNECTION 'host=192.168.1.37 user=postgres dbname=postgres' PUBLICATION
test_pub_repl0 WITH (copy_data = TRUE);
```

3 ВМ использовать как реплику для чтения и бэкапов (подписаться на таблицы из ВМ №1 и №2 ).

```
CREATE TABLE test (name text PRIMARY KEY);
CREATE TABLE test2 (name text PRIMARY KEY);

CREATE SUBSCRIPTION test_sub_repl01 CONNECTION 'host=192.168.1.37 user=postgres dbname=postgres' PUBLICATION
test_pub_repl0 WITH (copy_data = TRUE);

CREATE SUBSCRIPTION test2_sub_repl11 CONNECTION 'host=192.168.1.41 user=postgres dbname=postgres' PUBLICATION
test_pub_repl1 WITH (copy_data = TRUE);
```

