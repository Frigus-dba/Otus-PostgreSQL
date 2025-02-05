## Домашняя работа №12

1. Для выполнения тестового резервного копирования создана таблица со 100 записями. 
```sql
postgres=# create table student as
select
  generate_series(1,100) as id,
  md5(random()::text)::char(10) as fio;
SELECT 100
```
Внутри она выглядит так:
```sql
postgres=# select * from student;
 id |    fio
----+------------
  1 | b66d05c970
  2 | 9f6c139da1
  3 | 167903a546
  4 | 45fcb71083
  5 | e1107811c5
  6 | e97fe6894d
  7 | 9836bf12cd
  8 | e4e18d453a
  9 | 9b15195e9e
 10 | a27dee510d
 ...
```
2. Выполним логический бэкап при помощи команды COPY
```sql
postgres=# COPY student TO '/var/lib/postgresql/otus_backup/std_bcp.sql';
COPY 100
```
3. Бэкап готов, теперь восстановим эти данные в новую таблицу. Для этого скопируем таблицу student, но не полностью, а только ее структуру. Ведь команда COPY не запоминает структуру таблицы, а только данные.
```sql
postgres=# CREATE TABLE student2 AS TABLE student WITH NO DATA;
CREATE TABLE AS
```
Сейчас таблица student2 пустая. Заполним ее данными из бэкапа.
```sql
postgres=# COPY student2 FROM '/var/lib/postgresql/otus_backup/std_bcp.sql';
COPY 100
```
Как можно увидеть, таблица заполнилась корректно
```sql
postgres=# select * from student2;
 id |    fio
----+------------
  1 | b66d05c970
  2 | 9f6c139da1
  3 | 167903a546
  4 | 45fcb71083
  5 | e1107811c5
  6 | e97fe6894d
  7 | 9836bf12cd
  8 | e4e18d453a
  9 | 9b15195e9e
 10 | a27dee510d
 ...
```

4. Далее создадим бэкап двух таблиц при помощи утилиты pg_dump
```bash
pg_dump -U postgres -d postgres -Fc -t student -t student2 > arh_bcp.sql
```
5. Таблицы намерено создавал в БД postgres и дамп делал оттуда же. Создадим новую БД, чтобы восстановиться уже в нее. 
```sql
postgres=# CREATE DATABASE studentdb;
CREATE DATABASE
```
Выполним команду восстановления только второй таблицы
```bash
pg_restore -U postgres -Fc -d studentdb -t student2 arh_bcp.sql
```
Видно, что таблица успешно восстановлена и данные на месте.
```sql
studentdb=# \conninfo
You are connected to database "studentdb" as user "postgres" via socket in "/var/run/postgresql" at port "5432".
studentdb=# \dt
          List of relations
 Schema |   Name   | Type  |  Owner
--------+----------+-------+----------
 public | student2 | table | postgres
(1 row)
```
```sql
studentdb=# select * from student2;
 id |    fio
----+------------
  1 | b66d05c970
  2 | 9f6c139da1
  3 | 167903a546
  4 | 45fcb71083
  5 | e1107811c5
  6 | e97fe6894d
  7 | 9836bf12cd
  8 | e4e18d453a
  9 | 9b15195e9e
 10 | a27dee510d
 ...
```