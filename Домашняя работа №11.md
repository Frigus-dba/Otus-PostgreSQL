## Домашняя работа №11 

1. Прежде всего скачиваем скрипт, приложенный к заданию. Выполянем его, а именно создаем 2 таблицы goods и sales и сразу наполняем их исходными данными. Плюс создаем третью таблицу good_sum_mart, но пока пустую, это и будет наша витрина.

2. Создадим функцию для триггера, которая сначала очищает имеющиеся данные в таблице good_sum_mart, а затем наполняет ее же новыми данными на основе таблиц goods и sales. 

```sql
CREATE OR REPLACE FUNCTION tf_goods() 
RETURNS trigger 
AS 
$tg_goods$
BEGIN
DELETE FROM good_sum_mart *;
INSERT INTO good_sum_mart(good_name, sum_sale)
SELECT G.good_name, sum(G.good_price * S.sales_qty)
FROM goods G
INNER JOIN sales S on S.good_id = G.goods_id
GROUP BY G.good_name;
RETURN NULL;   
END;
$tg_goods$ 
LANGUAGE plpgsql;
```

3. Далее создадим сам триггер.
```sql
CREATE TRIGGER trig_goods 
AFTER  INSERT OR UPDATE OR DELETE 
ON sales
FOR EACH STATEMENT
EXECUTE FUNCTION tf_goods();
```

4. Можно приступать к проверкам работоспособности. На данный момент значения в таблице sales дефолтные, т.е. А таблица good_sum_mart пустая.

```sql
shop=# select * from sales;
 sales_id | good_id |          sales_time           | sales_qty
----------+---------+-------------------------------+-----------
        1 |       1 | 2025-02-04 16:10:46.729114+03 |        10
        2 |       1 | 2025-02-04 16:10:46.729114+03 |         1
        3 |       1 | 2025-02-04 16:10:46.729114+03 |       120
        4 |       2 | 2025-02-04 16:10:46.729114+03 |         1
(4 rows)
```

5. Попробуем обновить данные
```sql
UPDATE sales SET sales_qty = 300 WHERE sales_id = 2;
```
Видны изменения и а таблице sales 

```sql
shop=# select * from sales;
 sales_id | good_id |          sales_time           | sales_qty
----------+---------+-------------------------------+-----------
        1 |       1 | 2025-02-04 16:45:56.290231+03 |        10
        3 |       1 | 2025-02-04 16:45:56.290231+03 |       120
        4 |       2 | 2025-02-04 16:45:56.290231+03 |         1
        2 |       1 | 2025-02-04 16:45:56.290231+03 |       300
(4 rows)
```
И в good_sum_mart данные появились
```sql
shop=# select * from good_sum_mart;
        good_name         |   sum_sale
--------------------------+--------------
 Автомобиль Ferrari FXX K | 185000000.01
 Спички хозайственные     |       215.00
(2 rows)
```
6. Попробуем добавить новую позицию
```sql
INSERT INTO goods(goods_id, good_name, good_price)
VALUES(3, 'Коробка конфет', 125.70);

INSERT INTO sales(good_id,sales_qty) 
VALUES(3, 3000);
```
Как видим, данные успешно обновились
```sql
shop=# select * from sales;
 sales_id | good_id |          sales_time           | sales_qty
----------+---------+-------------------------------+-----------
        1 |       1 | 2025-02-04 16:45:56.290231+03 |        10
        3 |       1 | 2025-02-04 16:45:56.290231+03 |       120
        4 |       2 | 2025-02-04 16:45:56.290231+03 |         1
        2 |       1 | 2025-02-04 16:45:56.290231+03 |       300
        5 |       3 | 2025-02-04 17:17:02.806088+03 |      3000
(5 rows)
```
```sql
shop=# select * from good_sum_mart;
        good_name         |   sum_sale
--------------------------+--------------
 Автомобиль Ferrari FXX K | 185000000.01
 Спички хозайственные     |       215.00
 Коробка конфет           |    377100.00
(3 rows)
```

7. Проверим операцию DELETE
```sql
DELETE FROM sales WHERE sales_id = 2;
```

Все данные успешно обновились
```sql
shop=# select * from sales;
 sales_id | good_id |          sales_time           | sales_qty
----------+---------+-------------------------------+-----------
        1 |       1 | 2025-02-04 16:45:56.290231+03 |        10
        3 |       1 | 2025-02-04 16:45:56.290231+03 |       120
        4 |       2 | 2025-02-04 16:45:56.290231+03 |         1
        5 |       3 | 2025-02-04 17:17:02.806088+03 |      3000
(4 rows)
```

```sql
shop=# select * from good_sum_mart;
        good_name         |   sum_sale
--------------------------+--------------
 Автомобиль Ferrari FXX K | 185000000.01
 Спички хозайственные     |        65.00
 Коробка конфет           |    377100.00
(3 rows)
```
