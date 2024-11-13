# Домашнее задание №8

 Этапы выполнения:

- Развёрнута ВМ в WB Cloud.
- Установлен PostgreSQL 17й версии.
- Создана и заполнена данными таблица с продажами:
```
CREATE TABLE sales (
    id SERIAL PRIMARY KEY,
    sale_date DATE NOT NULL,
    amount NUMERIC NOT NULL
);

INSERT INTO sales (sale_date, amount) VALUES
('2024-01-01', 100.00),
('2024-02-01', 200.00),
('2024-03-01', 300.00),
('2024-04-01', 400.00),
('2024-05-01', 500.00),
('2024-06-01', 600.00),
('2024-07-01', 700.00),
('2024-08-01', 800.00),
('2024-09-01', 900.00),
('2024-10-01', 1000.00),
('2024-11-01', 1100.00),
('2024-12-01', 1200.00);
```
- Создана функция (используя "CASE"):
```
CREATE OR REPLACE FUNCTION get_third_of_year_case(sale_date DATE)
RETURNS INTEGER AS $$
DECLARE
    month INTEGER;
BEGIN
    IF sale_date IS NULL THEN
        RETURN NULL;
    END IF;

    month := EXTRACT(MONTH FROM sale_date);

    RETURN CASE
        WHEN month BETWEEN 1 AND 4 THEN 1
        WHEN month BETWEEN 5 AND 8 THEN 2
        WHEN month BETWEEN 9 AND 12 THEN 3
        ELSE NULL -- Handle invalid month inputs, though this shouldn't occur
    END;
END;
$$ LANGUAGE plpgsql;
```
- Создана функция (используя математическую операцию):
```
CREATE OR REPLACE FUNCTION get_third_of_year_math(sale_date DATE)
RETURNS INTEGER AS $$
DECLARE
    month INTEGER;
BEGIN
    IF sale_date IS NULL THEN
        RETURN NULL;
    END IF;

    month := EXTRACT(MONTH FROM sale_date);

    RETURN CEIL(month / 4.0);
END;
$$ LANGUAGE plpgsql;
```
- Вызваны ранее созданые функции:
```
SELECT
    id,
    sale_date,
    amount,
    get_third_of_year_case(sale_date) AS third_case,
    get_third_of_year_math(sale_date) AS third_math
FROM sales;
 id | sale_date  | amount  | third_case | third_math
----+------------+---------+------------+------------
  1 | 2024-01-01 |  100.00 |          1 |          1
  2 | 2024-02-01 |  200.00 |          1 |          1
  3 | 2024-03-01 |  300.00 |          1 |          1
  4 | 2024-04-01 |  400.00 |          1 |          1
  5 | 2024-05-01 |  500.00 |          2 |          2
  6 | 2024-06-01 |  600.00 |          2 |          2
  7 | 2024-07-01 |  700.00 |          2 |          2
  8 | 2024-08-01 |  800.00 |          2 |          2
  9 | 2024-09-01 |  900.00 |          3 |          3
 10 | 2024-10-01 | 1000.00 |          3 |          3
 11 | 2024-11-01 | 1100.00 |          3 |          3
 12 | 2024-12-01 | 1200.00 |          3 |          3
(12 rows)
```
