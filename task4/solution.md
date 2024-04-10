создайте виртуальную машину c Ubuntu 20.04/22.04 LTS в GCE/ЯО/Virtual Box/докере<br>
поставьте на нее PostgreSQL 15 через sudo apt<br>
проверьте что кластер запущен через sudo -u postgres pg_lsclusters<br>
```
Ver Cluster Port Status Owner    Data directory              Log file
15  main    5432 online postgres /var/lib/postgresql/15/main /var/log/postgresql/postgresql-15-main.log
```

зайдите из под пользователя postgres в psql и сделайте произвольную таблицу с произвольным содержимым

```
sudo -u postgres psql
postgres=# create table test(c1 text);
CREATE TABLE
postgres=# insert into test values('1');
INSERT 0 1
postgres=# 
```

остановите postgres например через sudo -u postgres pg_ctlcluster 15 main stop

```
sudo -u postgres pg_ctlcluster 15 main stop
```

создайте новый диск к ВМ размером 10GB<br>
добавьте свеже-созданный диск к виртуальной машине - надо зайти в режим ее редактирования и дальше выбрать пункт attach existing disk<br>
проинициализируйте диск согласно инструкции и подмонтировать файловую систему, только не забывайте менять имя диска на актуальное, в вашем случае это скорее всего будет /dev/sdb - https://www.digitalocean.com/community/tutorials/how-to-partition-and-format-storage-devices-in-linux<br>

```
sudo parted /dev/vda2 mklabel gpt
sudo parted -a opt /dev/vdb mkpart primary ext4 0% 100%
sudo mkfs.ext4 -L datapartition /dev/vdb1
sudo mkdir -p /mnt/data
```
/etc/fstab:
```
LABEL=datapartition /mnt/data ext4 defaults 0 2
```

перезагрузите инстанс и убедитесь, что диск остается примонтированным (если не так смотрим в сторону fstab)

```
df -h -x tmpfs
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda2        68G   54G   11G  84% /
/dev/vdb1       9.8G   25K  9.3G   1% /mnt/data
```

сделайте пользователя postgres владельцем /mnt/data - chown -R postgres:postgres /mnt/data/

```
sudo chown -R postgres:postgres /mnt/data/
```

перенесите содержимое /var/lib/postgres/15 в /mnt/data - mv /var/lib/postgresql/15/mnt/data

```
sudo -u postgres mv /var/lib/postgresql/15 /mnt/data
```

попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 15 main start

```
sudo -u postgres pg_ctlcluster 15 main start
Error: /var/lib/postgresql/15/main is not accessible or does not exist****
```

напишите получилось или нет и почему

```
Нет, параметр data_directory указывает на старый каталог
```

задание: найти конфигурационный параметр в файлах раположенных в /etc/postgresql/15/main который надо поменять и поменяйте его
напишите что и почему поменяли

```
Поменял data_directory в файле /etc/postgresql/15/main/postgresql.conf
```
```
data_directory = '/mnt/data/15/main'
```

попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 15 main start
напишите получилось или нет и почему

```
sudo -u postgres pg_ctlcluster 15 main start
sudo -u postgres pg_lsclusters
Ver Cluster Port Status Owner    Data directory    Log file
15  main    5432 online postgres /mnt/data/15/main /var/log/postgresql/postgresql-15-main.log
```
```
Кластер запустился, т.к. в файле конфигурации путь теперь правильный
```

зайдите через через psql и проверьте содержимое ранее созданной таблицы

```
sudo -u postgres psql
postgres=# SELECT * FROM test;
 c1 
----
 1
(1 row)
```
















