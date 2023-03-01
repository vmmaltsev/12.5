# Домашнее задание к занятию 12.5. «Индексы» - `Мальцев Виктор`

---

Задание 1

Напишите запрос к учебной базе данных, 
который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.

Ответ:

select 
(SELECT 
sum(ROUND(stat_value * @@innodb_page_size / 1024 / 1024, 2)) size_in_mb
FROM mysql.innodb_index_stats
WHERE stat_name = 'size' AND index_name != 'PRIMARY' and database_name = 'sakila') * 100 /
(SELECT 
     sum(round(((data_length + index_length) / 1024 / 1024), 2)) `Size in MB` 
FROM information_schema.TABLES where TABLE_SCHEMA ='sakila');


![alt text](https://github.com/vmmaltsev/screenshot2/blob/main/Screenshot_30.png)

---

Задание 2

Выполните explain analyze следующего запроса:

select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id

    перечислите узкие места;
    оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы.


Ответ:

1. Использование оператора distinct, он ищет по всему переченю текстовых значений
2. Использование inner join, то есть происходит проверка всех значений со всеми.

Скорректированный скрипт:

select concat(c.last_name, ' ', c.first_name) , SUM(p.amount)  from payment p
left outer join rental r on p.payment_date = r.rental_date 
LEFT OUTER join customer c on r.customer_id = c.customer_id
LEFT outer join inventory i on i.inventory_id = r.inventory_id
where date(p.payment_date) = '2005-07-30'
group by concat(c.last_name, ' ', c.first_name)

Выполняется за 23 ms

![alt text](https://github.com/vmmaltsev/screenshot2/blob/main/Screenshot_31.png)

План выполнения запроса 

![alt text](https://github.com/vmmaltsev/screenshot2/blob/main/Screenshot_32.png)


Запрос из условия задание ыполнется в среднем 5 секунд

![alt text](https://github.com/vmmaltsev/screenshot2/blob/main/Screenshot_33.png)

План выполнения запроса

![alt text](https://github.com/vmmaltsev/screenshot2/blob/main/Screenshot_34.png)




