# Домашнее задание №3

 Этапы выполнения:

- Развёрнута ВМ в WB Cloud.
- Установлен PostgreSQL 17й версии.
- Создана таблица с текстовым полем и заполнена случайными или сгенерированными
   данным в размере 1 млн строк:  
```postgresql
create schema vactest;
create table vactest.randtable
(
    i     serial,
    t_str text
);

create extension if not exists pgcrypto;

create or replace procedure ddl_home03_batch_insert_test_data(num_of_rows integer)
    language plpgsql
as
$$
begin
    insert into vactest.randtable (t_str)
    select SUBSTRING(gen_random_bytes(10)::text, 3)
    from generate_series(0, num_of_rows);
end;
$$;

call ddl_home03_batch_insert_test_data(1000000);
```

- Запрошен размера файла с таблицей:  
`SELECT pg_size_pretty(pg_total_relation_size('vactest.randtable'));`
> 124 MB

- 5 раз подряд обновлены все строки таблицы, к каждой строке добавлен символ:
```postgresql
create or replace procedure ddl_home03_upd_all_rows(num_of_updates integer)
    language plpgsql
as
$$
begin
    for _ in 1 .. num_of_updates
        loop
            update vactest.randtable set t_str = t_str || substr(gen_random_bytes(1)::text, 4);
        end loop;
end;
$$;

call ddl_home03_upd_all_rows(5);
```

- Запрошено количество мертвых строк в таблице, а так же время, когда в последний раз обрабатывал autovacuum:  
```postgresql
SELECT schemaname, relname, n_dead_tup FROM pg_stat_all_tables WHERE relid='table_name'::regclass
```
> 5000005 - так как каждая строка обновлялась 5 раз (т.е. предыдущие данные теряли актуальность 5 раз).  

- Выполнялось ожидание отработки механизма autovacuum:
> Autovacuum отработал через ~ 1 минуту.

- Еще раз 5 раз подряд обновлены все строки таблицы, к каждой строке добавлен символ: 
```postgresql
call ddl_home03_upd_all_rows(5);
```

- Запрошен размера файла с таблицей: 
```postgresql
SELECT pg_size_pretty(pg_total_relation_size('vactest.randtable'));
```
> 391 MB.

- Отключен autovacuum на таблице "randtable"
```postgresql
ALTER TABLE vactest.randtable SET (autovacuum_enabled = false);
```

- 10 раз подряд обновлены все строки таблицы, к каждой строке добавлен символ:
```postgresql
call ddl_home03_upd_all_rows(10);
```

- Запрошен размера файла с таблицей: 
```postgresql
SELECT pg_size_pretty(pg_total_relation_size('vactest.randtable'));
```
> 811 MB

- "Объясните полученный результат"  
> Размер таблицы увеличился в ~ 2 раза. Увеличилось, так как данные в 5 из 10 циклов попали в ячейки, до этого очищенные autovacuum'ом (но не удалённые), а в остальных 5 циклах СУБД была вынуждена увеличивать таблицу, выделяя новые страницы под данные.
