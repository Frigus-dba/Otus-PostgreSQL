## Домашнее задание №2

1. Для выполнения дз установлен VirtualBox, а в нем ВМ на Ubuntu.
2. Внутри ВМ установлен Docker.
3. Созданы 2 контейнера с PostgreSQL 17 (клиент 5433 и сервер 5434)
> docker run --name otus-pgdocker-client -e POSTGRES_PASSWORD=1234 -e POSTGRES_USER=postgres -e POSTGRES_DB=postgres -d -p 5434:5432 -v /var/lib/postgresql/data:/var/lib/postgresql/data postgres
4. Успешно удалось подключиться к контейнеру с сервером с личного компьютера, а именно через Dbeaver
5. После удаления контейнера с сервером, а затем установки обратно, данные внутри БД остались на месте. 
6. Единственный момент, который мне не удалось сделать, это подключиться к БД из одного контейнера в другой. psql по порту выдавал вот такую картинку.
> psql -p 5433 -h 127.0.0.1 -U postgres -d postgres

> psql: error: connection to server at "127.0.0.1", port 5433 failed: Connection refused
        
> Is the server running on that host and accepting TCP/IP connections?

Было бы отлично, если бы вы объяснили, как заставить их взаимодействовать друг с другом.