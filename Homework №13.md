## Домашняя работа №13

1. Это задание выполнял не на разных ВМ, а в рамках одной, но кластера были развернуты на разных портах: 5432, 5433, 5434.

2. В конфигурационных файлах было выставлено значение wal_level = logical. 
3. В первом кластере создал 2 таблицы: test_t1 и test_t2. А со второго и третьего кластера выполнил дамп структуры, чтобы можно было начать логическую репликацию
```bash
pg_dump -p5432 -U postgres -d test --create --schema-only | psql -p5433
```

```bash 
pg_dump -p5432 -U postgres -d test --create --schema-only | psql -p5434
```
4. В первом кластере создал публикацию на первую таблицу
```sql
CREATE PUBLICATION test_pub FOR TABLE test_t1;
```
А так же подписку на эту публикацию со второго кластера
```sql
CREATE SUBSCRIPTION test_sub
CONNECTION 'host=localhost port=5432 user=postgres password=1234 dbname=test'
PUBLICATION test_pub WITH (copy_data = true);
```
5. Аналогичные действия провел на втором кластере. Публикация таблицы test_t2, а затем активация подписки на неё со стороны первого кластера
```sql
CREATE PUBLICATION test2_pub FOR TABLE test_t2;
```
```sql
CREATE SUBSCRIPTION test2_sub
CONNECTION 'host=localhost port=5433 user=postgres password=1234 dbname=test'
PUBLICATION test2_pub WITH (copy_data = true);
```
6. И в третьем кластере просто создал 2 подписки на эти таблицы в 1 и 2 кластере соответсвенно. 
```sql
CREATE SUBSCRIPTION test3_sub
CONNECTION 'host=localhost port=5432 user=postgres password=1234 dbname=test'
PUBLICATION test_pub WITH (copy_data = true);
```
```sql
CREATE SUBSCRIPTION test4_sub
CONNECTION 'host=localhost port=5433 user=postgres password=1234 dbname=test'
PUBLICATION test2_pub WITH (copy_data = true);
```
7. Таким образов имеем следующие картины в кластерах: 

 * 1 кластер
 ```sql
test=# \dRp+
                            Publication test_pub
  Owner   | All tables | Inserts | Updates | Deletes | Truncates | Via root
----------+------------+---------+---------+---------+-----------+----------
 postgres | f          | t       | t       | t       | t         | f
Tables:
    "public.test_t1"

test=# SELECT * FROM pg_stat_subscription \gx
-[ RECORD 1 ]---------+------------------------------
subid                 | 73815
subname               | test2_sub
pid                   | 2789
relid                 |
received_lsn          | 0/1B57ED0
last_msg_send_time    | 2025-02-06 16:17:34.686299+03
last_msg_receipt_time | 2025-02-06 16:17:34.686428+03
latest_end_lsn        | 0/1B57ED0
latest_end_time       | 2025-02-06 16:17:34.686299+03
 ```
 * 2 кластер
 ```sql
test=# \dRp+
                           Publication test2_pub
  Owner   | All tables | Inserts | Updates | Deletes | Truncates | Via root
----------+------------+---------+---------+---------+-----------+----------
 postgres | f          | t       | t       | t       | t         | f
Tables:
    "public.test_t2"

test=# SELECT * FROM pg_stat_subscription \gx
-[ RECORD 1 ]---------+------------------------------
subid                 | 16403
subname               | test_sub
pid                   | 2195
relid                 |
received_lsn          | 0/A6F07B08
last_msg_send_time    | 2025-02-06 16:18:46.010085+03
last_msg_receipt_time | 2025-02-06 16:18:46.010124+03
latest_end_lsn        | 0/A6F07B08
latest_end_time       | 2025-02-06 16:18:46.010085+03
 ```
 * 3 кластер 
 ```sql
postgres=# SELECT * FROM pg_stat_subscription \gx
-[ RECORD 1 ]---------+------------------------------
subid                 | 16398
subname               | test3_sub
pid                   | 2804
relid                 |
received_lsn          | 0/A6F07B08
last_msg_send_time    | 2025-02-06 16:20:16.116325+03
last_msg_receipt_time | 2025-02-06 16:20:16.116362+03
latest_end_lsn        | 0/A6F07B08
latest_end_time       | 2025-02-06 16:20:16.116325+03
-[ RECORD 2 ]---------+------------------------------
subid                 | 16399
subname               | test4_sub
pid                   | 2812
relid                 |
received_lsn          | 0/1B57ED0
last_msg_send_time    | 2025-02-06 16:20:04.8521+03
last_msg_receipt_time | 2025-02-06 16:20:04.852135+03
latest_end_lsn        | 0/1B57ED0
latest_end_time       | 2025-02-06 16:20:04.8521+03
 ```