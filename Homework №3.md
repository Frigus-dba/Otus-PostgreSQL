## Домашняя работа

1. Для выполнения ДЗ была создана ВМ в VirtualBox на ОС Debian 12
2. Установлен PostgreSQL 15 
3. Создана таблица с данными из примера 
> postgres=# create table test(c1 text); 

> postgres=# insert into test values('1'); 

4. Остановка постгреса произведена через команду 
> systemctl stop postgresql@15-main.service

5. Примонтирован жесткий диск размером 20 гб
> pvcreate /dev/sdb
>> Physical volume "/dev/sdb" successfully created.

> vgcreate vg_pgsql /dev/sdb
>> Volume group "vg_pgsql" successfully created

> lvcreate -n lv_pgsql -l 100%FREE vg_pgsql
>> Logical volume "lv_pgsql" created.

> mkfs.ext4 /dev/mapper/vg_pgsql-lv_pgsql
>> Creating filesystem with 5241856 4k blocks and 1310720 inodes
Filesystem UUID: 143b785e-5a86-4cb3-8c1b-3a6834774313
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
        4096000
        Allocating group tables: done
        Writing inode tables: done
        Creating journal (32768 blocks): done
        Writing superblocks and filesystem accounting information: done

> echo '/dev/mapper/vg_pgsql-lv_pgsql /data/postgres ext4 defaults,relatime 0 2' >> /etc/fstab

> mount -av
>> /data/postgres           : successfully mounted

6. Владельцем каталога /data/postgres был назначен юзер postgres
> chown postgres:postgres /data/postgres

7. Пытаться запустить postgres после переноса данных не имеет смысла. Будет ошибка запуска, так как в файле postgresql.conf указан старый data_dir. Меняем это значение на новое /data/postgres. 
8. После этого была ошибка связанная с правами доступа к этому каталогу. Увидеть ее можно командой 
> journalctl -xeu postgresql@15-main.service
>> FATAL:  data directory "/data/postgres" has invalid permissions 
DETAIL:  Permissions should be u=rwx (0700) or u=rwx,g=rx (0750).

9. После смены прав на каталог кластер успешно запустился. Созданная ранее таблица осталась на месте. 
