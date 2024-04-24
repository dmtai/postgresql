Запросы делал на БД - https://github.com/pthom/northwind_psql (структуру таблиц см. там)<br>

Реализовать прямое соединение двух или более таблиц
```
select 
	order_id, c.contact_name customer_contact_name, e.last_name employee
from 
	public.orders o
inner join 
	public.customers c on c.customer_id = o.customer_id
inner join 
	public.employees e on e.employee_id = o.employee_id
limit 10;

 order_id | customer_contact_name | employee  
----------+-----------------------+-----------
    10248 | Paul Henriot          | Buchanan
    10249 | Karin Josephs         | Suyama
    10250 | Mario Pontes          | Peacock
    10251 | Mary Saveley          | Leverling
    10252 | Pascale Cartrain      | Peacock
    10253 | Mario Pontes          | Leverling
    10254 | Yang Wang             | Buchanan
    10255 | Michael Holz          | Dodsworth
    10256 | Paula Parente         | Leverling
    10257 | Carlos Hernández      | Peacock
(10 rows)
```

Реализовать левостороннее (или правостороннее) соединение двух или более таблиц

```
select 
	c.contact_name, o.order_id
from 
	public.customers c
left join 
	orders o on c.customer_id = o.customer_id
limit 10;

   contact_name   | order_id 
------------------+----------
 Paul Henriot     |    10248
 Karin Josephs    |    10249
 Mario Pontes     |    10250
 Mary Saveley     |    10251
 Pascale Cartrain |    10252
 Mario Pontes     |    10253
 Yang Wang        |    10254
 Michael Holz     |    10255
 Paula Parente    |    10256
 Carlos Hernández |    10257
(10 rows)
```

Реализовать кросс соединение двух или более таблиц

```
select * from products cross join categories
```

Реализовать полное соединение двух или более таблиц

```
select 
	p.product_name, s.contact_title
from
	products p
full join
	suppliers s on p.supplier_id = s.supplier_id
limit 10;

          product_name           |    contact_title     
---------------------------------+----------------------
 Chai                            | Sales Representative
 Chang                           | Purchasing Manager
 Aniseed Syrup                   | Purchasing Manager
 Chef Anton's Cajun Seasoning    | Order Administrator
 Chef Anton's Gumbo Mix          | Order Administrator
 Grandma's Boysenberry Spread    | Sales Representative
 Uncle Bob's Organic Dried Pears | Sales Representative
 Northwoods Cranberry Sauce      | Sales Representative
 Mishi Kobe Niku                 | Marketing Manager
 Ikura                           | Marketing Manager
(10 rows)
```

Реализовать запрос, в котором будут использованы разные типы соединений

```
select
  c.contact_name, o.order_id, s.company_name shipper_company_name
from
  customers c
left join
  orders o on o.customer_id = c.customer_id
inner join
  shippers s on o.ship_via = s.shipper_id
limit 10

   contact_name   | order_id | shipper_company_name 
------------------+----------+----------------------
 Paul Henriot     |    10248 | Federal Shipping
 Karin Josephs    |    10249 | Speedy Express
 Mario Pontes     |    10250 | United Package
 Mary Saveley     |    10251 | Speedy Express
 Pascale Cartrain |    10252 | United Package
 Mario Pontes     |    10253 | United Package
 Yang Wang        |    10254 | United Package
 Michael Holz     |    10255 | Federal Shipping
 Paula Parente    |    10256 | United Package
 Carlos Hernández |    10257 | Federal Shipping
(10 rows)
```


