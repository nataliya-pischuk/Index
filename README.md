## Домашнее задание к занятию «Индексы»
## Задание 1

Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.

## Решение
```
select sum(data_length) as SUM_Data_Length, sum(index_length) as SUM_Index_Length, sum(index_length)*100.0/sum(data_length) as Persentage_ratio
from information_schema.tables
where table_schema='sakila' and data_length is not null;
```
## Задание 2

Выполните explain analyze следующего запроса:
  
```
   select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id 
and i.inventory_id = r.inventory_id
````
- перечислите узкие места; Избыточные запросы к таблицам, которые не нужны для получения этих данных, в оконной функции лишняя таблица, лишние условия.
- оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы.
  
## Решение
Запрос сложный, выполняется долго.
Нужно убрать фрагмент over (partition by c.customer_id, f.title) и оператор distinct и убмраем избыточные таблицы film и inventory :

```
select concat(c.last_name, ' ', c.first_name), sum(p.amount)
from payment p, rental r, customer c
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id
group by c.last_name, c.first_name;
```
```
# проосматриваем индексы
select *
from INFORMATION_SCHEMA.STATISTICS
where TABLE_NAME='payment';

# создаем новый индекс
create index idx_payment_date on payment(payment_date);

# выполняем оптимизированный запрос
select concat(c.last_name, ' ', c.first_name), sum(p.amount)
from payment p, rental r, customer c
where p.payment_date >= '2005-07-30 00:00:00' and p.payment_date < date_add('2005-07-30 00:00:00', interval 1 day) and p.payment_date = r.rental_date and r.customer_id = c.customer_id
group by c.last_name, c.first_name;

```
