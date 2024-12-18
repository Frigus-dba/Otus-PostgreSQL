## Домашнее задание №6

1. Для выполнения задания был взят PostgreSQL 15. Ресурсы на ВМ: 2 ядра, 4 гб ОЗУ, SSD на 20гб.

2. Создана тестовая БД. Подана нагрузка при помощи pgbench сначала на дефолтные настройки, а затем на те, что ты были предоставлены в приложении. 

```bash
max_connections = 40
shared_buffers = 1GB
effective_cache_size = 3GB
maintenance_work_mem = 512MB
checkpoint_completion_target = 0.9
wal_buffers = 16MB
default_statistics_target = 500
random_page_cost = 4
effective_io_concurrency = 2
work_mem = 65536kB
min_wal_size = 4GB
max_wal_size = 16GB
```

3. Результаты работы pgbench не сильно отличались друг от друга (как и в предыдущем уроке). В первом случае 1109 TPS, во втором 1079 TPS. Я думаю, что я не получаю прирост в производительности, несмотря на уведичение параметров, по той причине, что запросы в тестовой нагрузке довольно простые и системе просто напросто хватает и дефолтных ресурсов на такой объем. В предыдущем уроке я пробовал занижать параметры work_mem и shared_buffers до минимально возможных и только в этом случае удалось добиться просадки по TPS.

4. Далее создана таблица с 1 млн строк.

```sql
create table t1(id serial, fio char(100));

INSERT INTO t1(fio) SELECT 'name' FROM generate_series(1,1000000);
```
Таблица весит 135мб

5. Выполнен UPDATE 5 раз. Вес таблицы уже674мб. Колличество мертвых строк было 3999929, но уже через минуту автовакуум их вычистил. 

6. UPDATE выолнен еще 5 раз. Вес таблицы 808мб. Мертвых строк было 4999589, но снова в течение минуты отработал автовакуум. 

7. На таблице t1 отключен автовакуум командой 

```sql
ALTER TABLE t1 SET (autovacuum_enabled = off);
```
8. Выполнен UPDATE еще 10 раз. Таблица распухла до 1482мб и мертвых строк ~10млн. Как и ожидалось, они не будут вычещены, так как автовакуум отключен. После его включения, мертвые строки будут удалены, но вес таблицы от этого не уменьшится. Чтобы таблица уменьшилась в размере, можно запустить vacuum full по ней, он выполнит дефрагментацию, что приведет к желанному результату.
