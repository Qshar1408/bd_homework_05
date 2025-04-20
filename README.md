# Домашнее задание к занятию «Индексы»
### Грибанов Антон. FOPS-31


### Задание 1

Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.

 ![bd_005](https://github.com/Qshar1408/bd_homework_05/blob/main/img/bd_05_001.png)

### Задание 2

Выполните explain analyze следующего запроса:
```sql
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
```
- перечислите узкие места;
- оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы.

 ![bd_005](https://github.com/Qshar1408/bd_homework_05/blob/main/img/bd_05_002.png)

#### Узкие места перечислены здесь:

```bash
-> Temporary table with deduplication  (cost=0..0 rows=0) (actual time=4381..4381 rows=391 loops=1)
-> Window aggregate with buffering: sum(payment.amount) OVER (PARTITION BY c.customer_id,f.title )   (actual time=1876..4203 rows=642000 loops=1)
-> Sort: c.customer_id, f.title  (actual time=1876..1918 rows=642000 loops=1)
```
Использование оконной функции увеличивает количество строк, плюс сортировка по двум параметрам. Удаление повторяющихся строк из-за оконной функции. Плюс лишняя таблица film.

#### Оптимизация запроса:

Удаляем из запроса таблицу film, оператор distinct и оконную функцию. Добавляем GROUP BY.

```bash
EXPLAIN ANALYZE
select concat(c.last_name, ' ', c.first_name), sum(p.amount)
from payment p, rental r, customer c, inventory i
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
GROUP BY concat(c.last_name, ' ', c.first_name);
```
 ![bd_005](https://github.com/Qshar1408/bd_homework_05/blob/main/img/bd_05_003.png)

## Дополнительные задания (со звёздочкой*)
Эти задания дополнительные, то есть не обязательные к выполнению, и никак не повлияют на получение вами зачёта по этому домашнему заданию. Вы можете их выполнить, если хотите глубже шире разобраться в материале.

### Задание 3*

Самостоятельно изучите, какие типы индексов используются в PostgreSQL. Перечислите те индексы, которые используются в PostgreSQL, а в MySQL — нет.

*Приведите ответ в свободной форме.*


PostgreSQL поддерживает следующие типы индексов:

   - B-Tree - в MySQL так же используется;
   - SP-GiST
   - GIN (Inverted) - в MySQL так же используется;
   - BRIN
   - Hash - в MySQL используется только в механизме хранения Memory;
   - GiST (R-Tree) - в MySQL так же используется;
   - bloom

Для разных типов индексов применяются разные алгоритмы, ориентированные на определённые типы запросов. По умолчанию команда CREATE INDEX создаёт индексы типа B-Tree, эффективные в большинстве случаев.
