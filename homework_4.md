# Домашнее задание №4

 Этапы выполнения:

- Создана таблица accounts(id integer, amount numeric);  
```postgresql
create table if not exists accounts(id integer, amount numeric);
```
- Добавлено несколько записей. Запущено две SSH сессии,создана взаимоблокировка (deadlock).
```postgresql
insert into accounts values (1, 100), (2, 150);
```
- В обеих сессиях начата транзакция, используя `begin`
В первой сессии: 
```postgresql
update accounts set amount = 300 where id = 1;
```
Во второй сессии:
```postgresql
update accounts set amount = 400 where id = 1;
```
Выполнение транзакции во второй сессии повисло. Так произошло из - за блокировки, возникшей из - за выполнения транзакции в первой сессии.  
Такая ситуация не возникла бы, если бы данные изменялись в разных строках. 

- Проверено, что информация о блокировке (deadlock) зафиксировалась в логе.
```postgresql
select * from pg_locks where mode = 'RowExclusiveLock';
```
