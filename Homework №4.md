## Домашняя работа

1. Для выполнения этого домашнего задания создан кластер на 15 версии постгреса.
2. Создана база данных testdb, в ней создана схема testnm, а затем таблица и небольшое наполнение для нее:

```sql
create database testdb;
create schema testnm;
create table t1(c1 integer);
insert into t1(c1) values(1);
```
3. Так же создал роль readonly и выдал все необходимые права: 

```sql
create role readonly;
grant connect on database testdb to readonly;
grant usage on schema testnm to readonly ;
grant select on all tables in schema testnm to readonly;
```
4. Однако выполнить SELECT по созданной таблице не вышло, так как она создалась в схеме public, а не testnm. А прав на public не было выдано.
5. После пересоздания таблицы, но уже с точным укfзанием схемы testnm, выполнить SELECT все равно не удалось, так как команда
```sql
GRANT SELECT ON ALL TABLES IN SCHEMA testnm TO readonly;
```
выдала права только для существующих таблиц, на новые это не распространяется. 

6.  Таким образом, чтобы эти права применялись для новых таблиц, можно использовать команду ниже, а затем пересоздать таблицу.
```sql
ALTER DEFAULT PRIVILEGES IN SCHEMA testnm GRANT SELECT ON TABLES TO readonly;
``` 
7. Создать новую таблицу под ролью readonly не вышло из-за недостатка прав. Так как у меня 15 версия постгреса, в которой и отобрали право CREATE у роли public в схеме public.