## Домашняя работа

1. создана тестовая таблица 

- Для таблицы создан обычный индекс. Результат работы команды explain по таблиц с индексом. 
```sql
demo=# explain select seat_no from seats where seat_no = '8A';
                                   QUERY PLAN
---------------------------------------------------------------------------------
 Index Only Scan using idx_test_seats on seats  (cost=0.28..1.62 rows=5 width=3)
   Index Cond: (seat_no = '8A'::text)
```
2. Индекс для полнотекстового поиска был реализован на примере из лекции. 

- Создана тестовая таблица, а затем заполнена данными. 
```sql
create table orders (
    id int,
    user_id int,
    order_date date,
    status text,
    some_text text
);

insert into orders(id, user_id, order_date, status, some_text)
select generate_series, (random() * 70), date'2019-01-01' + (random() * 300)::int as order_date
        , (array['returned', 'completed', 'placed', 'shipped'])[(random() * 4)::int]
        , concat_ws(' ', (array['go', 'space', 'sun', 'London'])[(random() * 5)::int]
            , (array['the', 'capital', 'of', 'Great', 'Britain'])[(random() * 6)::int]
            , (array['some', 'another', 'example', 'with', 'words'])[(random() * 6)::int]
            )
from generate_series(100001, 1000000);
```
 - Далее используются функции to_tsvector и to_tsquery для преобразования текста в типы tsvector и tsquery соотвественно. 
 ```sql
select some_text, to_tsvector(some_text)
from orders;

select some_text, to_tsvector(some_text) @@ to_tsquery('britains')
from orders;

select some_text, to_tsvector(some_text) @@ to_tsquery('london & capital')
from orders;

select some_text, to_tsvector(some_text) @@ to_tsquery('london | capital')
from orders;
 ```
 - После этого необходимо создать дополнительную колонку в таблице, чтобы хранить в ней значения tsvector.
 ```sql
 alter table orders add column some_text_lexeme tsvector;
update orders
set some_text_lexeme = to_tsvector(some_text);
 ```
 - После этого можно выполнить запрос на поиск интереусующего нас текста, но с использования explain analyze, чтобы оценить, насколько быстро он выполнится. Итоговое время выоплнения запроса составило ``` Execution Time: 1373.226 ms```. Чтобы ускорить его, создадим индекс. 
 ```sql
CREATE INDEX search_index_ord ON orders USING GIN (some_text_lexeme);
 ```
- После сосздания индекса запрос выполнился более чем в 13 раз быстрее. ``` Execution Time: 103.974 ms```

3. Частичный индекс так же создавался на примере из лекции.
- Создана таблица с данными
- Создан частичный индекс для нее 
```sql
create index idx_test_id_100 on test(id) where id < 100;
```
- Выполнен запрос данных, которые не входят в рамки созданного индекса. Время его выполнения составило ```Execution Time: 5.467 ms``` 
```sql
explain analyze
select * from test where id > 150;
```
- И наоборот, запрос, под который построен индекс. Он выполнился куда быстрее ```Execution Time: 0.023 ms```
```sql
explain analyze
select * from test where id < 50;
```
4. Индекс на несколько полей создавался для той же таблицы, что и в предыдущем примере. 
```sql 
create index idx_test_id_is_okay on test(id, is_okay);
```
- Пример выполнения запроса до построения индекса
```sql
postgres=# explain analyze
select * from test2 where is_okay = 'True' and id = 1;
                                            QUERY PLAN
---------------------------------------------------------------------------------------------------
 Seq Scan on test2  (cost=0.00..1133.00 rows=1 width=31) (actual time=3.519..3.519 rows=0 loops=1)
   Filter: ((is_okay = 'True'::text) AND (id = 1))
   Rows Removed by Filter: 50000
 Planning Time: 0.326 ms
 Execution Time: 3.534 ms
```
- Пример после построения запроса
```sql 
postgres=# explain analyze
select * from test2 where is_okay = 'True' and id = 1;
                                                         QUERY PLAN
----------------------------------------------------------------------------------------------------------------------------
 Index Scan using idx_test_id_is_okay on test2  (cost=0.29..2.11 rows=1 width=31) (actual time=0.012..0.012 rows=0 loops=1)
   Index Cond: ((id = 1) AND (is_okay = 'True'::text))
 Planning Time: 0.910 ms
 Execution Time: 0.025 ms
```