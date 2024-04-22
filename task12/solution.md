Создать индекс к какой-либо из таблиц вашей БД
```
Добавил индекс для поиска по id

create table currencies(
	currency_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
	code VARCHAR(3) NOT NULL,
	full_name VARCHAR(255) NOT NULL,
	description TEXT,
	description_tsv tsvector
);
update currencies set description_tsv = to_tsvector(description);
create index idx_currency_id ON currencies(currency_id);
```

Прислать текстом результат команды explain,

```
explain select * from currencies where currency_id = 5;
                                     QUERY PLAN                                     
------------------------------------------------------------------------------------
 Index Scan using idx_currency_id on currencies  (cost=0.28..2.79 rows=1 width=117)
   Index Cond: (currency_id = 5)
(2 rows)
```

Реализовать индекс для полнотекстового поиска

```
Сделал индекс для быстрого поиска по описанию

create index on currencies using gin(description_tsv);

explain select * from currencies
where description_tsv @@ to_tsquery('test & txt');
                                         QUERY PLAN                                          
---------------------------------------------------------------------------------------------
 Bitmap Heap Scan on currencies  (cost=5.27..9.75 rows=3 width=736)
   Recheck Cond: (description_tsv @@ to_tsquery('test & txt'::text))
   ->  Bitmap Index Scan on currencies_description_tsv_idx  (cost=0.00..5.27 rows=3 width=0)
         Index Cond: (description_tsv @@ to_tsquery('test & txt'::text))
(4 rows)
```

Реализовать индекс на часть таблицы или индекс на поле с функцией

```
Добавил индекс для поиска по имени без учета регистра

create index idx_name on currencies (lower(full_name));

explain select * from currencies where lower(full_name) = 'test name';
                               QUERY PLAN                               
------------------------------------------------------------------------
 Bitmap Heap Scan on currencies  (cost=1.63..17.56 rows=13 width=736)
   Recheck Cond: (lower((full_name)::text) = 'test name'::text)
   ->  Bitmap Index Scan on idx_name  (cost=0.00..1.63 rows=13 width=0)
         Index Cond: (lower((full_name)::text) = 'test name'::text)
(4 rows)
```

Создать индекс на несколько полей

```
create index idx_name_id on currencies (lower(full_name), currency_id);

explain select * from currencies where lower(full_name) = 'test name' and currency_id = 1;
                                      QUERY PLAN                                      
--------------------------------------------------------------------------------------
 Index Scan using idx_name_id on currencies  (cost=0.28..2.80 rows=1 width=736)
   Index Cond: ((lower((full_name)::text) = 'test name'::text) AND (currency_id = 1))
(2 rows)
```

















