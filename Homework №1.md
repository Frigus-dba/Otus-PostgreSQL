# Otus-PostgreSQL
My Otus homework
## Домашнее задание №1

* Тестовая среда была развернута в Yandex Cloud.
* Проброшен ssh ключ с моего устройва на ВМ.
* Установлен PostgreSQL 15 версии.
* Заполнена таблица данными из примера, а так же были проведены тесты с транзакциями. В данном примере 
>  Начать новую транзакцию в обоих сессиях с дефолтным (не меняя) уровнем изоляции. В первой сессии добавить новую запись insert into persons(first_name, second_name) values('sergey', 'sergeev'); Сделать select from persons во второй сессии. Видите ли вы новую запись и если да то почему? 

во второй сессии мы не увидели новых записей, так как транзакция в первой сессии не была завершена и это считается грязным чтением, что не допустимо в постгресе.

* Во второй части задания действия выполнялись внутри транзакции repeatable read. В данном случае мы не видим изменений данных во второй сессии, даже когда завершим транзакцию в первой. Здесь срабатывает защита от фантомного чтения, которую нам обеспечивает уровень изоляции repeatable read.
