создать ВМ с Ubuntu 20.04/22.04 или развернуть докер любым удобным способом<br />
поставить на нем Docker Engine<br />

```
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

 sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
сделать каталог /var/lib/postgres
```
sudo mkdir /var/lib/postgres
```

развернуть контейнер с PostgreSQL 15 смонтировав в него /var/lib/postgresql

```
docker network create postgresql

docker run -d \
    -p 5432:5432 \
    -v /var/lib/postgresql/data:/var/lib/postgresql/data \
    -e "POSTGRES_PASSWORD=postgres" \
    --network postgresql \
    --name postgresql-server \
    postgres:15
```

развернуть контейнер с клиентом postgres<br>
подключится из контейнера с клиентом к контейнеру с сервером и сделать таблицу с парой строк<br>

```
docker run -it \
    --rm \
    --network postgresql \
    --name postgresql-client \
    postgres:15 \
    psql -h postgresql-server -U postgres
```

```
postgres=# CREATE DATABASE testdb1;

postgres=# \c testdb1 postgres

testdb1=# CREATE TABLE otus (id integer PRIMARY KEY, name varchar(255));

testdb1=# INSERT INTO otus VALUES (1, 'data1');
INSERT 0 1
testdb1=# INSERT INTO otus VALUES (2, 'data2');
INSERT 0 1

```
подключится к контейнеру с сервером с ноутбука/компьютера извне инстансов GCP/ЯО/места установки докера

```
psql -h 192.168.64.6 -p 5432 -U postgres -d testdb1
```

удалить контейнер с сервером

```
docker stop postgresql-server && docker rm postgresql-server
```

создать его заново
```
docker run -d \
    -p 5432:5432 \
    -v /var/lib/postgresql/data:/var/lib/postgresql/data \
    -e "POSTGRES_PASSWORD=postgres" \
    --network postgresql \
    --name postgresql-server \
    postgres:15
```

подключится снова из контейнера с клиентом к контейнеру с сервером

```
docker run -it \
    --rm \
    --network postgresql \
    --name postgresql-client \
    postgres:15 \
    psql -h postgresql-server -U postgres -d testdb1
```

проверить, что данные остались на месте

```
SELECT * FROM otus;
 id | name
----+------
  1 | data1
  2 | data2
```




























