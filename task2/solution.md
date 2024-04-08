

создать новый проект в Google Cloud Platform, Яндекс облако или на любых ВМ, докере<br />
далее создать инстанс виртуальной машины с дефолтными параметрами<br />
добавить свой ssh ключ в metadata ВМ<br />
зайти удаленным ssh (первая сессия), не забывайте про ssh-add<br />

```
Создал ВМ с ubuntu и ключами
```

поставить PostgreSQL


```
# Create the file repository configuration:
sudo sh -c 'echo "deb https://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'

# Import the repository signing key:
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -

# Update the package lists:
sudo apt-get update

# Install the latest version of PostgreSQL.
# If you want a specific version, use 'postgresql-12' or similar instead of 'postgresql':
sudo apt-get -y install postgresql
```
Установлено:
```
Ver Cluster Port Status Owner    Data directory              Log file
12  main    5432 online postgres /var/lib/postgresql/12/main /var/log/postgresql/postgresql-12-main.log
```

зайти вторым ssh (вторая сессия)<br />
запустить везде psql из под пользователя postgres<br />

```
Запущены терминалы через sudo -u postgres psql
```

выключить auto commit

```
\set AUTOCOMMIT
```

сделать в первой сессии новую таблицу и наполнить ее данными create table persons(id serial, first_name text, second_name text); insert into persons(first_name, second_name) values('ivan', 'ivanov'); insert into persons(first_name, second_name) values('petr', 'petrov'); commit;

```
create table persons(id serial, first_name text, second_name text); insert into persons(first_name, second_name) values('ivan', 'ivanov'); insert into persons(first_name, second_name) values('petr', 'petrov'); commit;
```
```
CREATE TABLE
INSERT 0 1
INSERT 0 1
WARNING:  there is no transaction in progress
COMMIT
```

посмотреть текущий уровень изоляции: show transaction isolation level

```
show transaction isolation level;
```
```
 transaction_isolation 
-----------------------
 read committed
(1 row)
```

начать новую транзакцию в обоих сессиях с дефолтным (не меняя) уровнем изоляции
```
begin; в двух терминалах
```

в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sergey', 'sergeev');

```
insert into persons(first_name, second_name) values('sergey', 'sergeev');
INSERT 0 1
```

сделать select * from persons во второй сессии

```
 id | first_name | second_name 
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
(2 rows)
```

видите ли вы новую запись и если да то почему?

```
нет, уровень изоляции read commited, транзакция с insert не закоммичена, грязное чтение запрещено
```

завершить первую транзакцию - commit;
```
commit;
```

сделать select from persons во второй сессии

```
 id | first_name | second_name 
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
(3 rows)
```

видите ли вы новую запись и если да то почему?

```
да, потому что она была закоммичена и установлен уровень read commited
```

завершите транзакцию во второй сессии<br />
начать новые но уже repeatable read транзации - set transaction isolation level repeatable read;<br />
в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sveta', 'svetova');<br />

```
set transaction isolation level repeatable read;
insert into persons(first_name, second_name) values('sveta', 'svetova');

```

сделать select * from persons во второй сессии*

```
select * from persons
```
```
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
(3 rows)
```

видите ли вы новую запись и если да то почему?

```
Нет, уровень изоляции repeatable read запрещает грязное чтение
```

завершить первую транзакцию - commit;<br />
сделать select * from persons во второй сессии<br />
```
select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
```

видите ли вы новую запись и если да то почему?

```
Нет, уровень изоляции repeatable read, поэтому повторное чтение без изменений
```

завершить вторую транзакцию<br />
сделать select * from persons во второй сессии<br />

```
select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
  4 | sveta      | svetova
(4 rows)
```

видите ли вы новую запись и если да то почему?

```
Да, транзакцию завершили, уровень изоляции repeatable read позволит увидеть изменения
```
