## Домашняя работа №10

1. Секционирование проводилось на скачанной готовой БД с postgrespro.
- Первым делом определил самую "тяжелую" таблицу.
- Делаю копию этой таблицы, но с указанием того, что она будет секционироваться по хэшу.
```sql
CREATE TABLE ticket_flights_new (LIKE ticket_flights including all) PARTITION BY hash(ticket_no);
```
- Создаю 3 новые таблицы, которые будут партициями предыдущей 
```sql 
CREATE TABLE test1 PARTITION of ticket_flights_new FOR VALUES WITH (modulus 3, remainder 0);
CREATE TABLE test2 PARTITION of ticket_flights_new FOR VALUES WITH (modulus 3, remainder 1);
CREATE TABLE test3 PARTITION of ticket_flights_new FOR VALUES WITH (modulus 3, remainder 2);
```
- Переношу данные из старой таблицы в новую. При этом в ней самой данных не будет, они сразу распределяются по партициям
```sql
INSERT INTO ticket_flights_new SELECT * FROM ticket_flights;
```
- На этом этапе можно остановиться, ведь партиции созданы, данные перенесены, но имя основной таблицы сейчас отличается от того, что было ```ticket_flights_new```. Так же в ней нет необходимых констрейнтов. Да и старая таблица нам не нужна.
- Чтобы это исправить, смотрю, какие констрейнты были на старой таблице ```\d ticket_flights```. 
- Добавляю внешние ключи в новую таблицу 
```sql
ALTER TABLE ticket_flights_new ADD CONSTRAINT ticket_flights_flight_id_fkey FOREIGN KEY (flight_id) REFERENCES flights(flight_id);
ALTER TABLE ticket_flights_new ADD CONSTRAINT ticket_flights_ticket_no_fkey FOREIGN KEY (ticket_no) REFERENCES tickets(ticket_no);
```
- Так же на нашу старую таблицу ссылается третья таблица. Дропаю эту ссылку с неё и после этого дропаю саму таблицу, ведь она нам уже не нужна. P.S до того, как мы удалили этот констрейнт, таблицу невозможно было удалить без CASCADE.
```sql
ALTER TABLE boarding_passes DROP CONSTRAINT boarding_passes_ticket_no_fkey;
```
- Теперь можно переименовать секционированную таблицу и вернуть удаленный в предыдущем шаге констрейнт.
```sql
ALTER TABLE ticket_flights_new rename TO ticket_flights;

ALTER TABLE boarding_passes ADD CONSTRAINT boarding_passes_ticket_no_fkey FOREIGN KEY (ticket_no, flight_id) REFERENCES ticket_flights(ticket_no, flight_id);
```
