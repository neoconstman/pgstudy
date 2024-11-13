# Домашнее задание №11

 Этапы выполнения:

- Развёрнута ВМ в WB Cloud.
- Установлены PostgreSQL 16й и 17й версии.
- Настроено прослушивание разными версиями PostgreSQL разных портов (PG 16 слушает порт 5416, PG 17 слушает порт 5417).
- В PostgreSQL 16й версии загружен дамп thai_medium.tar.gz.
- Выполнена проверка загруженной БД:
```
thai=# select count(*) from book.tickets;
  count
----------
 53997475
(1 row)
```
### Выполнение проверки работы логической репликации:

- На PG16 выполнена установка нужного wal_level для логической репликации.
```
postgres=# SHOW wal_level;
 wal_level
-----------
 replica
(1 row)

postgres=# ALTER SYSTEM SET wal_level = 'logical';
ALTER SYSTEM
```
```
systemctl restart postgresql@16-main.service
```
```
thai=# SHOW wal_level;
 wal_level
-----------
 logical
(1 row)
```
- На PG16 создана публикация для всех таблиц БД "thai".
```
thai=# CREATE PUBLICATION logical_publication FOR ALL TABLES;
CREATE PUBLICATION
```
- Из PG16 выгружена структура БД "thai" без данных и загружена в PG17.


```
pg_dump -p 5416 -d thai --schema-only -f /tmp/schema_thai.sql
```

```
psql -p 5417 thai < /tmp/schema_thai.sql
```
- Выполнена проверка создания структуры БД в PG17.
```
postgres=# select count(*) from book.tickets;
 count
-------
     0
(1 row)
```
- В PG17 создана подписка на публикацию, созданную в PG16.
```
postgres=# CREATE SUBSCRIPTION logical_subscription
CONNECTION 'host=localhost port=5416 user=postgres password=postgres dbname=thai'
PUBLICATION my_publication;
NOTICE:  created replication slot "logical_subscription" on publisher
CREATE SUBSCRIPTION

```

```
 subid |       subname        | worker_type | pid | leader_pid | relid | received_lsn |      last_msg_send_time       |     last_msg_receipt_time     | latest_end_lsn |        latest_end_time
-------+----------------------+-------------+-----+------------+-------+--------------+-------------------------------+-------------------------------+----------------+-------------------------------
 23442 | logical_subscription | apply       | 352 |            |       | 1/420D00A9   | 2024-11-13 20:05:03.856351+00 | 2024-11-13 20:05:03.856351+00 | 1/420D00A9      | 2024-11-13 20:05:03.856351+00
(1 row)
```

```
2024-11-13 20:02:08.224 UTC [200] LOG:  logical replication apply worker for subscription "logical_subscription" has started
2024-11-13 20:02:08.234 UTC [201] LOG:  logical replication table synchronization worker for subscription "logical_subscription", table "fam" has started
2024-11-13 20:02:08.244 UTC [202] LOG:  logical replication table synchronization worker for subscription "logical_subscription", table "busstation" has started
2024-11-13 20:02:08.262 UTC [201] LOG:  logical replication table synchronization worker for subscription "logical_subscription", table "fam" has finished
2024-11-13 20:02:08.263 UTC [202] LOG:  logical replication table synchronization worker for subscription "logical_subscription", table "busstation" has finished
2024-11-13 20:02:08.265 UTC [203] LOG:  logical replication table synchronization worker for subscription "logical_subscription", table "bus" has started
2024-11-13 20:02:08.266 UTC [204] LOG:  logical replication table synchronization worker for subscription "logical_subscription", table "tickets" has started
2024-11-13 20:02:08.289 UTC [203] LOG:  logical replication table synchronization worker for subscription "logical_subscription", table "bus" has finished
2024-11-13 20:02:08.291 UTC [205] LOG:  logical replication table synchronization worker for subscription "logical_subscription", table "schedule" has started
2024-11-13 20:02:08.314 UTC [205] LOG:  logical replication table synchronization worker for subscription "logical_subscription", table "schedule" has finished
2024-11-13 20:02:08.317 UTC [206] LOG:  logical replication table synchronization worker for subscription "logical_subscription", table "seat" has started
2024-11-13 20:02:08.338 UTC [206] LOG:  logical replication table synchronization worker for subscription "logical_subscription", table "seat" has finished
2024-11-13 20:02:08.342 UTC [207] LOG:  logical replication table synchronization worker for subscription "logical_subscription", table "ride" has started
2024-11-13 20:02:09.985 UTC [207] LOG:  logical replication table synchronization worker for subscription "logical_subscription", table "ride" has finished
2024-11-13 20:02:09.994 UTC [208] LOG:  logical replication table synchronization worker for subscription "logical_subscription", table "nam" has started
2024-11-13 20:02:10.012 UTC [208] LOG:  logical replication table synchronization worker for subscription "logical_subscription", table "nam" has finished
2024-11-13 20:02:10.012 UTC [209] LOG:  logical replication table synchronization worker for subscription "logical_subscription", table "seatcategory" has started
2024-11-13 20:02:10.041 UTC [209] LOG:  logical replication table synchronization worker for subscription "logical_subscription", table "seatcategory" has finished
2024-11-13 20:02:10.067 UTC [210] LOG:  logical replication table synchronization worker for subscription "logical_subscription", table "busroute" has started
2024-11-13 20:02:10.234 UTC [210] LOG:  logical replication table synchronization worker for subscription "logical_subscription", table "busroute" has finished
2024-11-13 20:02:58.634 UTC [204] LOG:  logical replication table synchronization worker for subscription "logical_subscription", table "tickets" has finished

```
- Всего на логическую репликацию БД было затрачено ~48 секунд времени. 

- На PG17 выполнена проверка идентичности данных на PG16.
```
select count(*) from book.tickets;
  count
----------
 53997475
(1 row)
```

### Выполненение проверка работы Foreign-Data Wrapper:

- На PG17 создано и настроено расширение "postgres_fdw"
```
CREATE EXTENSION IF NOT EXISTS postgres_fdw;

CREATE SERVER thai_fdw
FOREIGN DATA WRAPPER postgres_fdw
OPTIONS (host 'localhost', port '5416', dbname 'thai');

CREATE USER MAPPING FOR postgres
SERVER thai_fdw
OPTIONS (user 'postgres', password 'postgres');

CREATE SCHEMA fdw_book;

IMPORT FOREIGN SCHEMA book
FROM SERVER thai_fdw
INTO fdw_book;

postgres=# select * from pg_foreign_table;
 ftrelid | ftserver |                 ftoptions
---------+----------+--------------------------------------------
   16531 |    16528 | {schema_name=book,table_name=bus}
   16534 |    16528 | {schema_name=book,table_name=busroute}
   16537 |    16528 | {schema_name=book,table_name=busstation}
   16540 |    16528 | {schema_name=book,table_name=fam}
   16543 |    16528 | {schema_name=book,table_name=nam}
   16546 |    16528 | {schema_name=book,table_name=ride}
   16549 |    16528 | {schema_name=book,table_name=schedule}
   16552 |    16528 | {schema_name=book,table_name=seat}
   16555 |    16528 | {schema_name=book,table_name=seatcategory}
   16558 |    16528 | {schema_name=book,table_name=tickets}
```
- На PG17 выполнена проверка идентичности данных на PG16.
```
select count(*) from fdw_book.tickets;
  count
----------
 53997475
```



### Выполненение проверка работы pg_dump:

- В PG17 создана пустая БД.
```CREATE DATABASE thai_pgdump TEMPLATE template0;```

- В PG17 загружен дамп из PG16
  ```pg_dump -p 5416 thai | psql -p 5417 thai_pgdump```

- На PG17 выполнена проверка идентичности данных на PG16.
```
select count(*) from fdw_book.tickets;
  count
----------
 53997475
```
