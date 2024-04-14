Настройте сервер так, чтобы в журнал сообщений сбрасывалась информация о блокировках, удерживаемых более 200 
миллисекунд. Воспроизведите ситуацию, при которой в журнале появятся такие сообщения.
```
sudo -u postgres psql

ALTER SYSTEM SET log_lock_waits = on;
ALTER SYSTEM SET deadlock_timeout = 200;
SELECT pg_reload_conf();

-- terminal 1
BEGIN;
postgres=# UPDATE test2 SET b = 11 WHERE a = 1;

-- terminal 2
BEGIN;
postgres=# UPDATE test2 SET b = 12 WHERE a = 1;
```
```
-- Зависание во второй транзакции
2024-04-15 01:31:42.848 +07 [38612] postgres@postgres LOG:  process 38612 still waiting for ShareLock on transaction 7191707 after 229.716 ms
2024-04-15 01:31:42.848 +07 [38612] postgres@postgres DETAIL:  Process holding the lock: 38592. Wait queue: 38612.
2024-04-15 01:31:42.848 +07 [38612] postgres@postgres CONTEXT:  while updating tuple (0,1) in relation "test2"
2024-04-15 01:31:42.848 +07 [38612] postgres@postgres STATEMENT:  UPDATE test2 SET b = 12 WHERE a = 1;
```

Смоделируйте ситуацию обновления одной и той же строки тремя командами UPDATE в разных сеансах. 
Изучите возникшие блокировки в представлении pg_locks и убедитесь, что все они понятны. 
Пришлите список блокировок и объясните, что значит каждая.

```
-- В трех терминалах:
begin;
postgres=# UPDATE test2 SET b = 123 WHERE a = 2;
```
```
SELECT relation::REGCLASS, locktype, mode, granted, pid, pg_blocking_pids(pid) AS wait_for
FROM pg_locks WHERE relation = 'test2'::regclass AND locktype = 'relation';
 relation | locktype |       mode       | granted |  pid  | wait_for 
----------+----------+------------------+---------+-------+----------
 test2    | relation | RowExclusiveLock | t       | 40111 | {38612}
 test2    | relation | RowExclusiveLock | t       | 38612 | {38592}
 test2    | relation | RowExclusiveLock | t       | 38592 | {}
```
```
Блокировку RowExclusiveLock получает любая команда, которая изменяет данные в таблице.
```

Воспроизведите взаимоблокировку трех транзакций. Можно ли разобраться в ситуации постфактум, изучая журнал сообщений?

```
-- terminal 1

postgres=# BEGIN;
BEGIN
postgres=# UPDATE test2 SET b = 111 WHERE a = 1;
UPDATE 1
postgres=# UPDATE test2 SET b = 222 WHERE a = 2;
ERROR:  deadlock detected
DETAIL:  Process 38592 waits for ShareLock on transaction 7191714; blocked by process 38612.
Process 38612 waits for ShareLock on transaction 7191715; blocked by process 40111.
Process 40111 waits for ShareLock on transaction 7191713; blocked by process 38592.
HINT:  See server log for query details.
CONTEXT:  while updating tuple (0,2) in relation "test2"
```
```
-- terminal 2

postgres=# begin;
BEGIN
postgres=# UPDATE test2 SET b = 211 WHERE a = 2;
UPDATE 1
postgres=# UPDATE test2  SET b = 322 WHERE a = 3;
```
```
-- terminal 3

postgres=# begin;
BEGIN
postgres=# UPDATE test2 SET b = 311 WHERE a = 3;
UPDATE 1
postgres=# UPDATE test2 SET b = 122 WHERE a = 1;
UPDATE 1
```

```
-- Лог

2024-04-15 02:09:19.132 +07 [38592] postgres@postgres LOG:  process 38592 detected deadlock while waiting for ShareLock on transaction 7191714 after 180012.064 ms
2024-04-15 02:09:19.132 +07 [38592] postgres@postgres DETAIL:  Process holding the lock: 38612. Wait queue: .
2024-04-15 02:09:19.132 +07 [38592] postgres@postgres CONTEXT:  while updating tuple (0,2) in relation "test2"
2024-04-15 02:09:19.132 +07 [38592] postgres@postgres STATEMENT:  UPDATE test2 SET b = 222 WHERE a = 2;
2024-04-15 02:09:19.132 +07 [38592] postgres@postgres ERROR:  deadlock detected
2024-04-15 02:09:19.132 +07 [38592] postgres@postgres DETAIL:  Process 38592 waits for ShareLock on transaction 7191714; blocked by process 38612.
	Process 38612 waits for ShareLock on transaction 7191715; blocked by process 40111.
	Process 40111 waits for ShareLock on transaction 7191713; blocked by process 38592.
	Process 38592: UPDATE test2 SET b = 222 WHERE a = 2;
	Process 38612: UPDATE test2  SET b = 322 WHERE a = 3;
	Process 40111: UPDATE test2 SET b = 122 WHERE a = 1;
2024-04-15 02:09:19.132 +07 [38592] postgres@postgres HINT:  See server log for query details.
2024-04-15 02:09:19.132 +07 [38592] postgres@postgres CONTEXT:  while updating tuple (0,2) in relation "test2"
2024-04-15 02:09:19.132 +07 [38592] postgres@postgres STATEMENT:  UPDATE test2 SET b = 222 WHERE a = 2;
```
```
По логу видно, что разобраться можно, "ERROR:  deadlock detected"
```

Могут ли две транзакции, выполняющие единственную команду UPDATE одной и той же 
таблицы (без where), заблокировать друг друга?

```
Могут, обновляться и блокироваться строки таблиц в разных транзакциях могут в разном порядке, это может
привести к дедлоку.
```

