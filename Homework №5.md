## Домашняя работа №5

1. Тесты производительности выполнялись на 15 версии постгреса. На ВМ выделено 1 ядро и 2 гб ОЗУ.

2. Инициализация производилась с ключом -s 100 для более "жирной" БД.

```bash
pgbench -i -s 100 test2
```

3. Команда для запуска pgbench 
```bash
pgbench -c 50 -j 2 -P 5 -T 20 testdb
```
4. Первый тест производился на дефолтных параметрах постгреса. То есть shared_buffers = 128MB, work_mem = 4MB и т.д. В результате теста значение TPS равнялось 85. 
5. Для второго теста были взяты рекомендуемые параметры с сайта pgconfig.org
```bash
# Memory Configuration
shared_buffers = 512MB
effective_cache_size = 2GB
work_mem = 5MB
maintenance_work_mem = 102MB

# Checkpoint Related Configuration
min_wal_size = 2GB
max_wal_size = 3GB
checkpoint_completion_target = 0.9
wal_buffers = -1

# Network Related Configuration
listen_addresses = '*'
max_connections = 100

# Storage Configuration
random_page_cost = 4.0
effective_io_concurrency = 2

# Worker Processes Configuration
max_worker_processes = 8
max_parallel_workers_per_gather = 2
max_parallel_workers = 2
```
В итоге получилось 83 TPS, что даже немного меньше, чем на дефолтных значениях. 

6. И для третьего теста я намеренно превысил значения параметров в надежде на более высокие результаты 
```bash
shared_buffers = 2048MB
work_mem = 32MB
maintenance_work_mem = 512MB
```
Однако это тоже не сильно что-то изменило, ведь TPS в третьем тесте стал 87. 