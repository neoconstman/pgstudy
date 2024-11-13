# Домашнее задание №9

 Этапы выполнения:

- Развёрнута ВМ в WB Cloud.
- Установлен PostgreSQL 17й версии.
- Создана таблица для JSONB документов и заполнена данными:
```
CREATE TABLE jsonb_data (
    id SERIAL PRIMARY KEY,
    data JSONB
);
INSERT INTO jsonb_data (data)
SELECT jsonb_build_object(
    'name', 'Item ' || i,
    'value', i,
    'description', 'This is item number ' || i
)
FROM generate_series(1, 1000000) AS s(i);

SELECt count(*) FROM jsonb_data;
  count
---------
 1000000
```
- Создан индекс: `CREATE INDEX idx_jsonb_data ON jsonb_data USING gin (data);`
- Активировано расширение "pgstattuple": `create extension pgstattuple;`
- Выведены текущие значения "dead_tuple_count" и "dead_tuple_len". На данный момент разрастание отсутствует.
```
SELECT * FROM pgstattuple('jsonb_data');
 table_len | tuple_count | tuple_len | tuple_percent | dead_tuple_count | dead_tuple_len | dead_tuple_percent | free_space | free_percent 
-----------+-------------+-----------+---------------+------------------+----------------+--------------------+------------+--------------
 134291456 |     1000000 | 124864702 |         92.98 |                0 |              0 |                  0 |    1840444 |         1.37
(1 row)
```
- Обновлена каждая десятая строка:
```
UPDATE jsonb_data
SET data = jsonb_set(data, '{description}', '"Updated"')
WHERE (data->>'value')::int % 10 = 0;
UPDATE 100000

```
- Повторно выведены "dead_tuple_count" и "dead_tuple_len". Теперь имеем разрастание.
```
SELECT * FROM pgstattuple('jsonb_data');
 table_len | tuple_count | tuple_len | tuple_percent | dead_tuple_count | dead_tuple_len | dead_tuple_percent | free_space | free_percent 
-----------+-------------+-----------+---------------+------------------+----------------+--------------------+------------+--------------
 144072704 |     1000000 | 122975807 |         85.36 |           100000 |       12486301 |               8.67 |      61968 |         0.04
(1 row)
```
- Разрастание устранено с помощью `VACUUM FULL jsonb_data;`
- Ещё раз выведены "dead_tuple_count" и "dead_tuple_len". Убеждаемся, что разрастание отсутствует.
SELECT * FROM pgstattuple('jsonb_data');
 table_len | tuple_count | tuple_len | tuple_percent | dead_tuple_count | dead_tuple_len | dead_tuple_percent | free_space | free_percent 
-----------+-------------+-----------+---------------+------------------+----------------+--------------------+------------+--------------
 130859008 |     1000000 | 122975807 |         93.98 |                0 |              0 |                  0 |      79860 |         0.06
(1 row)
