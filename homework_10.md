# Домашнее задание №10

 Этапы выполнения:

- Развёрнута ВМ в WB Cloud.
- Установлен PostgreSQL 17й версии.
- Создана БД "salary", в неё загружена структура и данные из https://www.db-fiddle.com/f/eQ8zNtAFY88i8nB4GRB65V/0.
- Выведены данные о зарплатах сотрудников и их росте с использованием оконной функции. "NULL" заменяются на "0":
```
SELECT
    s.fk_employee,
    s.from_date,
    s.amount AS current_salary,
    LAG(s.amount) OVER (PARTITION BY s.fk_employee ORDER BY s.from_date) AS previous_salary,
    COALESCE(s.amount - LAG(s.amount) OVER (PARTITION BY s.fk_employee ORDER BY s.from_date), 0) AS salary_increase
FROM
    salary s
ORDER BY
    s.fk_employee, s.from_date;
```
```
 fk_employee | from_date  | current_salary | previous_salary | salary_increase 
-------------+------------+----------------+-----------------+-----------------
           1 | 2024-01-01 |         100000 |                 |               0
           1 | 2024-02-01 |         200000 |          100000 |          100000
           1 | 2024-03-01 |         300000 |          200000 |          100000
           2 | 2023-01-01 |         200000 |                 |               0
           3 | 2024-03-01 |         200000 |                 |               0
(5 rows)
```
