## Студент Киселев Виктор Валерьевич
## Создание таблиц
```
create table customer (
    customer_id int4,
    first_name varchar(50),
    last_name varchar(50),
    gender varchar(30),
    DOB varchar(50), 
    job_title varchar(50),
    job_industry_category varchar(50),
    wealth_segment varchar(50),
    deceased_indicator varchar(50),
    owns_car varchar(30),
    address varchar(50),
    postcode varchar(30),
    state varchar(30),
    country varchar(30),
    property_valuation int4
);
```
![Alt text](/imgs/customer1.png?raw=true "Optional Title")
![Alt text](/imgs/customer2.png?raw=true "Optional Title")
```
create table transaction (
	transaction_id int4,
	product_id int4, 	
	customer_id int4, 
	transaction_date varchar(30),  
	online_order varchar(30), 
	order_status varchar(30), 
	brand varchar(30),
	product_line varchar(30), 
	product_class varchar(30),
	product_size varchar(30), 
	list_price float4, 
	standard_cost float4 
)

```
![Alt text](/imgs/transaction.png?raw=true "Optional Title")

## Преобразование типов столбцов
Для более удобной работы преобразуем типы у некоторых столцов:
```
alter  table "transaction" 
alter column transaction_date type date using TO_DATE(transaction_date,'DD.MM.YYYY')

alter  table "customer" 
alter column dob type date using TO_DATE(dob,'YYYY-MM-DD')

ALTER TABLE transaction  ALTER COLUMN online_order TYPE bool USING cast((case online_order when '' then null else online_order end) as bool);
```
![Alt text](/imgs/cast_transaction_date.png?raw=true "Optional Title")
![Alt text](/imgs/cast_dob.png?raw=true "Optional Title")
![Alt text](/imgs/cast_online_order.png?raw=true "Optional Title")


## Выполнение запросов

### Вывести распределение (количество) клиентов по сферам деятельности, отсортировав результат по убыванию количества. — (1 балл)
```
select job_industry_category, count(customer_id) as customers_quantity
from customer c 
group by job_industry_category
order by count(customer_id) desc
```
![Alt text](/imgs/task1.png?raw=true "Optional Title")

### Найти сумму транзакций за каждый месяц по сферам деятельности, отсортировав по месяцам и по сфере деятельности. — (1 балл)
Задание можно понять двояко: 1. Нужно группировать по имени месяца, учитывая год, в котором был этот месяц 2. Нужно группировать только по названию месяца

1.
```
select date_part('year', transaction_date) as year, date_part('month', transaction_date) as month, job_industry_category, sum(list_price), sum(standard_cost)
from "transaction" t 
join customer c 
on t.customer_id = c.customer_id 
group by date_part('year', transaction_date), date_part('month', transaction_date), job_industry_category
order by date_part('year', transaction_date), date_part('month', transaction_date), job_industry_category
```
![Alt text](/imgs/task2.1.png?raw=true "Optional Title")

2.
```
select date_part('month', transaction_date), job_industry_category, sum(list_price), sum(standard_cost)
from "transaction" t 
join customer c 
on t.customer_id = c.customer_id 
group by date_part('month', transaction_date), job_industry_category
order by date_part('month', transaction_date), job_industry_category
```
![Alt text](/imgs/task2.2.png?raw=true "Optional Title")


### Вывести количество онлайн-заказов для всех брендов в рамках подтвержденных заказов клиентов из сферы IT. — (1 балл)
```
select brand, count(*) as online_order_quantity_of_IT_client_Approved
from "transaction" t 
join customer c 
on t.customer_id = c.customer_id 
where online_order and order_status = 'Approved' and job_industry_category = 'IT'
group by brand 
```
![Alt text](/imgs/task3.png?raw=true "Optional Title")

### Найти по всем клиентам сумму всех транзакций (list_price), максимум, минимум и количество транзакций, отсортировав результат по убыванию суммы транзакций и количества клиентов. Выполните двумя способами: используя только group by и используя только оконные функции. Сравните результат. — (2 балла)

Если надо вывести информацию по каждому клиенту, то нет смысла сортировать по количеству клиентов.

Используя только group by:
```
select 
c.customer_id
, sum(list_price) as list_price_sum
, max(list_price) as list_price_max
, min(list_price) as list_price_min
, count(transaction_id) as transactions_quantity
from "transaction" t 
join customer c 
on t.customer_id = c.customer_id 
group by c.customer_id
order by list_price_sum desc
```

![Alt text](/imgs/task4_group_by.png?raw=true "Optional Title")

Используя только оконные функции:
```
select 
c.customer_id
, sum(list_price) over w_customer as list_price_sum
, max(list_price) over w_customer as list_price_max
, min(list_price) over w_customer as list_price_min
, count(transaction_id) over w_customer as transactions_quantity
from "transaction" t 
join customer c 
on t.customer_id = c.customer_id 
window w_customer as (partition by c.customer_id )
order by list_price_sum desc;
```
![Alt text](/imgs/task4_window_function.png?raw=true "Optional Title")


В результате при использовании group by данные каждой группы были агрегированы в одну запись, а при исопльзовании оконных функций количество записей осталось прежним. Для данного задания внутри окон все записи дублируются, поэтому разумней использовать group by.

### Найти имена и фамилии клиентов с минимальной/максимальной суммой транзакций за весь период (сумма транзакций не может быть null). Напишите отдельные запросы для минимальной и максимальной суммы. — (2 балла)

Для минимальной суммы:
```
select
first_name
, last_name
, sum(list_price) as list_price_min_sum
from "transaction" t 
join customer c 
on t.customer_id = c.customer_id 
group by first_name, last_name
having sum(list_price) =
(
	select *
	from (
		select
		sum(list_price) as list_price_min_sum
		from "transaction" t 
		join customer c 
		on t.customer_id = c.customer_id 
		group by first_name, last_name
		order by list_price_min_sum
		limit 1
	)
)
```

![Alt text](/imgs/task5_min.png?raw=true "Optional Title")

Для максимальной суммы:

```
select
first_name
, last_name
, sum(list_price) as list_price_max_sum
from "transaction" t 
join customer c 
on t.customer_id = c.customer_id 
group by first_name, last_name
having sum(list_price) =
(
	select *
	from (
		select
		sum(list_price) as list_price_max_sum
		from "transaction" t 
		join customer c 
		on t.customer_id = c.customer_id 
		group by first_name, last_name
		order by list_price_max_sum desc
		limit 1
	)
)
```

![Alt text](/imgs/task5_max.png?raw=true "Optional Title")

### Вывести только самые первые транзакции клиентов. Решить с помощью оконных функций. — (1 балл)

```
select * 
from (
select 
	row_number() over w_customer_id  as date_rank
	, t.*
	from "transaction" t
	join customer c 
	on t.customer_id = c.customer_id 
	window w_customer_id as (partition by c.customer_id order by transaction_date)
)
where date_rank = 1
```

![Alt text](/imgs/task6.png?raw=true "Optional Title")

### Вывести имена, фамилии и профессии клиентов, между транзакциями которых был максимальный интервал (интервал вычисляется в днях) — (2 балла).



Задание можно понять двояко: 1. между ближайшими транзакциями был максимальный интервал 2. между любыми транзакциями клиента был максимальный интервал.

1.

```
select *
from 
(
	select 
	first_name
	, last_name
	, job_title
	, lead(transaction_date) over w_customer_id - transaction_date as transaction_interval
	from "transaction" t 
	join customer c 
	on t.customer_id  = c.customer_id 
	window w_customer_id as (partition by c.customer_id order by transaction_date)
)
where transaction_interval  = 
(
	select 
	lead(transaction_date) over w_customer_id - transaction_date as transaction_interval
	from "transaction" t 
	join customer c 
	on t.customer_id  = c.customer_id 
	window w_customer_id as (partition by c.customer_id order by transaction_date )
	order by transaction_interval desc NULLS last
	limit 1
)
```

![Alt text](/imgs/task7.1.png?raw=true "Optional Title")

2.
```
select *
from 
(
	select 
	first_name
	, last_name
	, job_title
	, last_value(transaction_date) over w_customer_id - first_value(transaction_date) over w_customer_id  as transaction_interval
	from "transaction" t 
	join customer c 
	on t.customer_id  = c.customer_id 
	window w_customer_id as (partition by c.customer_id order by transaction_date)
)
where transaction_interval  = 
(
	select 
	last_value(transaction_date) over w_customer_id - first_value(transaction_date) over w_customer_id  as transaction_interval
	from "transaction" t 
	join customer c 
	on t.customer_id  = c.customer_id 
	window w_customer_id as (partition by c.customer_id order by transaction_date )
	order by transaction_interval desc NULLS last
	limit 1
)
```

![Alt text](/imgs/task7.2.png?raw=true "Optional Title")
