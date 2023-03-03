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

Установлен индекс на поле payment_date с названием index_for_test

-> Limit: 200 row(s)  (cost=0.00..0.00 rows=0) (actual time=36013.935..36013.966 rows=200 loops=1)
    -> Table scan on <temporary>  (cost=2.50..2.50 rows=0) (actual time=36013.934..36013.955 rows=200 loops=1)
        -> Temporary table with deduplication  (cost=0.00..0.00 rows=0) (actual time=36013.931..36013.931 rows=391 loops=1)
            -> Window aggregate with buffering: sum(payment.amount) OVER (PARTITION BY c.customer_id,f.title )   (actual time=32843.331..35831.496 rows=642000 loops=1)
                -> Sort: c.customer_id, f.title  (actual time=32843.294..32895.442 rows=642000 loops=1)
                    -> Stream results  (cost=10413378.80 rows=16008000) (actual time=14868.944..32368.733 rows=642000 loops=1)
                        -> Nested loop inner join  (cost=10413378.80 rows=16008000) (actual time=14868.930..32135.498 rows=642000 loops=1)
                            -> Nested loop inner join  (cost=4810578.80 rows=16008000) (actual time=0.318..8787.620 rows=16044000 loops=1)
                                -> Nested loop inner join  (cost=3205776.80 rows=16008000) (actual time=0.314..5039.139 rows=16044000 loops=1)
                                    -> Inner hash join (no condition)  (cost=1600974.80 rows=16008000) (actual time=0.305..1210.460 rows=16044000 loops=1)
                                        -> Table scan on r  (cost=1.67 rows=16008) (actual time=0.021..4.651 rows=16044 loops=1)
                                        -> Hash
                                            -> Covering index scan on f using idx_title  (cost=103.00 rows=1000) (actual time=0.043..0.215 rows=1000 loops=1)
                                    -> Single-row index lookup on c using PRIMARY (customer_id=r.customer_id)  (cost=0.00 rows=1) (actual time=0.000..0.000 rows=1 loops=16044000)
                                -> Single-row covering index lookup on i using PRIMARY (inventory_id=r.inventory_id)  (cost=0.00 rows=1) (actual time=0.000..0.000 rows=1 loops=16044000)
                            -> Index lookup on p using index_for_test (payment_date=r.rental_date), with index condition: (cast(p.payment_date as date) = '2005-07-30')  (cost=0.25 rows=1) (actual time=0.001..0.001 rows=0 loops=16044000)








