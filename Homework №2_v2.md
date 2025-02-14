## Домашнее задание №2

1. Для выполнения дз установлен VirtualBox, а в нем ВМ на Debian 12.
2. Внутри ВМ установлен Docker.
3. Создаем docker-сеть
```bash
docker network create pg-net
```
4. Подключаем созданную сеть к контейнеру сервера Postgres
```bash
docker run --name pg-docker --network pg-net -e POSTGRES_PASSWORD=1234 -d -p 5435:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres
```
И так же второй контейнер
```bash
docker run --name pg-docker2 --network pg-net -e POSTGRES_PASSWORD=1234 -d -p 5436:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres
```
5. Запускаем отдельный контейнер с клиентом в общей сети с БД
```bash
docker run -it --rm --network pg-net --name pg-client postgres psql -h pg-docker -U postgres
```
Для второго контейнера так же.

6. Теперь мы можем попасть внутрь контейнеров и внести необохдимые изменения в hba файл
```bash
host all all 0.0.0.0/0 trust
```
А так же конфиг файл postgres
```bash
listen_addresses = '*'
```
7. Проверим подключение из контейнера с клиентом к контейнеру с сервером. 
```bash
postgres@363730cd9965:~$ psql -h 192.168.0.116 -p5435
psql (17.3 (Debian 17.3-1.pgdg120+1))
Type "help" for help.

postgres=#
```
Успешно

8. Подключение из Dbeaver на моем личном ПК к контейнеру с сервером тоже успешно
![alt text](image-1.png)

9. Далее удалил контейнер с сервером создал его заново. Все данные остались на месте
```sql
postgres=# \dt
        List of relations
 Schema | Name | Type  |  Owner
--------+------+-------+----------
 public | t1   | table | postgres
 public | t2   | table | postgres
(2 rows)
```


