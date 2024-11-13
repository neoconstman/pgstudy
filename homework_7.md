# Домашнее задание №7

 Этапы выполнения:

- Развёрнута ВМ в WB Cloud.
- Установлен PostgreSQL 14й версии.
- В СУБД загружен дамп thai_small.tar.gz.
- Включено отображение времени выполнения запросов `\timing on`.
- Выполнен запрос:
```
WITH all_place AS (
    SELECT count(s.id) as all_place, s.fkbus as fkbus
    FROM book.seat s
    group by s.fkbus
),
order_place AS (
    SELECT count(t.id) as order_place, t.fkride
    FROM book.tickets t
    group by t.fkride
)
SELECT r.id, r.startdate as depart_date, bs.city || ', ' || bs.name as busstation,  
      t.order_place, st.all_place
FROM book.ride r
JOIN book.schedule as s
      on r.fkschedule = s.id
JOIN book.busroute br
      on s.fkroute = br.id
JOIN book.busstation bs
      on br.fkbusstationfrom = bs.id
JOIN order_place t
      on t.fkride = r.id
JOIN all_place st
      on r.fkbus = st.fkbus
GROUP BY r.id, r.startdate, bs.city || ', ' || bs.name, t.order_place,st.all_place
ORDER BY r.startdate
limit 10;
```
В результате время выполнения составило ~ 2,6 секунды:
```
 id | depart_date |       busstation       | order_place | all_place 
----+-------------+------------------------+-------------+-----------
  2 | 2000-01-01  | Bankgkok, Suvarnabhumi |          38 |        40
  3 | 2000-01-01  | Bankgkok, Suvarnabhumi |          34 |        40
  4 | 2000-01-01  | Bankgkok, Eastern      |          33 |        40
  5 | 2000-01-01  | Bankgkok, Eastern      |          36 |        40
  6 | 2000-01-01  | Bankgkok, Eastern      |          32 |        40
  7 | 2000-01-01  | Bankgkok, Chatuchak    |          37 |        40
  8 | 2000-01-01  | Bankgkok, Chatuchak    |          33 |        40
  9 | 2000-01-01  | Bankgkok, Chatuchak    |          37 |        40
 10 | 2000-01-01  | Bankgkok, Suvarnabhumi |          38 |        40
  1 | 2000-01-01  | Bankgkok, Suvarnabhumi |          38 |        40
(10 rows)

Time: 2593.083 ms (00:02.593)

```
- Созданы индексы:
`create index on book.seat(fkbus);` для первого табличного выражения.  
`create index on book.tickets(fkride);` для второго табличного выражения.  
`create index on book.ride(startdate);` для конечной сортировки.

- Повторно выполнен тот же самый запрос, в результате время выполнения сократилось в 2.5 раза.
```
 id | depart_date |       busstation       | order_place | all_place 
----+-------------+------------------------+-------------+-----------
  2 | 2000-01-01  | Bankgkok, Suvarnabhumi |          38 |        40
  3 | 2000-01-01  | Bankgkok, Suvarnabhumi |          34 |        40
  4 | 2000-01-01  | Bankgkok, Eastern      |          33 |        40
  5 | 2000-01-01  | Bankgkok, Eastern      |          36 |        40
  6 | 2000-01-01  | Bankgkok, Eastern      |          32 |        40
  7 | 2000-01-01  | Bankgkok, Chatuchak    |          37 |        40
  8 | 2000-01-01  | Bankgkok, Chatuchak    |          33 |        40
  9 | 2000-01-01  | Bankgkok, Chatuchak    |          37 |        40
 10 | 2000-01-01  | Bankgkok, Suvarnabhumi |          38 |        40
  1 | 2000-01-01  | Bankgkok, Suvarnabhumi |          38 |        40
(10 rows)

Time: 1066.496 ms (00:01.066)
```

## PS
Изначально я использовал PostgreSQL 17й версии, как и в предыдущих домашних работах.
- Время выполнения запроса до создания индексов:
```
 id | depart_date |       busstation       | order_place | all_place 
----+-------------+------------------------+-------------+-----------
  2 | 2000-01-01  | Bankgkok, Suvarnabhumi |          38 |        40
  3 | 2000-01-01  | Bankgkok, Suvarnabhumi |          34 |        40
  4 | 2000-01-01  | Bankgkok, Eastern      |          33 |        40
  5 | 2000-01-01  | Bankgkok, Eastern      |          36 |        40
  6 | 2000-01-01  | Bankgkok, Eastern      |          32 |        40
  7 | 2000-01-01  | Bankgkok, Chatuchak    |          37 |        40
  8 | 2000-01-01  | Bankgkok, Chatuchak    |          33 |        40
  9 | 2000-01-01  | Bankgkok, Chatuchak    |          37 |        40
 10 | 2000-01-01  | Bankgkok, Suvarnabhumi |          38 |        40
  1 | 2000-01-01  | Bankgkok, Suvarnabhumi |          38 |        40
(10 rows)

Time: 888.508 ms
```
- Время выполнения запроса после создания индексов:
```
 id | depart_date |       busstation       | order_place | all_place 
----+-------------+------------------------+-------------+-----------
  2 | 2000-01-01  | Bankgkok, Suvarnabhumi |          38 |        40
  3 | 2000-01-01  | Bankgkok, Suvarnabhumi |          34 |        40
  4 | 2000-01-01  | Bankgkok, Eastern      |          33 |        40
  5 | 2000-01-01  | Bankgkok, Eastern      |          36 |        40
  6 | 2000-01-01  | Bankgkok, Eastern      |          32 |        40
  7 | 2000-01-01  | Bankgkok, Chatuchak    |          37 |        40
  8 | 2000-01-01  | Bankgkok, Chatuchak    |          33 |        40
  9 | 2000-01-01  | Bankgkok, Chatuchak    |          37 |        40
 10 | 2000-01-01  | Bankgkok, Suvarnabhumi |          38 |        40
  1 | 2000-01-01  | Bankgkok, Suvarnabhumi |          38 |        40
(10 rows)

Time: 874.243 ms
```
При использовании PostgreSQL 17й версии в данном конкретном случае запрос отрабатывает гораздо быстрее и без создания индексов (по сравнению с 14й версией), создание индексов же не оказало существенного влияния на время выполнения (индексы создавались идентичные).
