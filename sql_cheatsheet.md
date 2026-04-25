# SQL Cheat Sheet for Data Engineering

A practical reference for SQL operations commonly used in data engineering, ETL pipelines, and analytics workflows.

---

## Table & Schema Management

### Create a table

```sql
CREATE TABLE orders (
    order_id INT,
    customer_id INT,
    amount DECIMAL(10, 2),
    created_at TIMESTAMP
);
```

### Create a table from a query result

```sql
CREATE TABLE orders_summary AS
SELECT customer_id, SUM(amount) AS total
FROM orders
GROUP BY customer_id;
```

### Create a table only if it does not exist

```sql
CREATE TABLE IF NOT EXISTS orders (
    order_id INT,
    amount DECIMAL(10, 2)
);
```

### Drop a table

```sql
DROP TABLE orders;
```

### Drop a table only if it exists

```sql
DROP TABLE IF EXISTS orders;
```

### Truncate a table (delete all rows, keep structure)

```sql
TRUNCATE TABLE orders;
```

### Add a column to an existing table

```sql
ALTER TABLE orders ADD COLUMN status VARCHAR(50);
```

### Drop a column from a table

```sql
ALTER TABLE orders DROP COLUMN status;
```

### Rename a column

```sql
ALTER TABLE orders RENAME COLUMN old_name TO new_name;
```

### Change a column's data type

```sql
ALTER TABLE orders ALTER COLUMN amount TYPE NUMERIC(12, 2);
```

---

## Inserting & Updating Data

### Load data from a CSV file (MySQL)

```sql
LOAD DATA LOCAL INFILE '/path/to/your_file.csv'
INTO TABLE sales_data
FIELDS TERMINATED BY ','
LINES TERMINATED BY '\n'
IGNORE 1 ROWS;
```

### Insert a single row

```sql
INSERT INTO orders (order_id, customer_id, amount) VALUES (1, 42, 99.99);
```

### Insert multiple rows at once

```sql
INSERT INTO orders (order_id, customer_id, amount)
VALUES (1, 42, 99.99), (2, 43, 149.00), (3, 44, 59.50);
```

### Insert from a query result

```sql
INSERT INTO orders_archive
SELECT * FROM orders WHERE created_at < '2024-01-01';
```

### Update rows matching a condition

```sql
UPDATE orders SET status = 'closed' WHERE created_at < '2024-01-01';
```

### Update using a value from another table

```sql
UPDATE orders
SET region = customers.region
FROM customers
WHERE orders.customer_id = customers.id;
```

### Delete rows matching a condition

```sql
DELETE FROM orders WHERE status = 'cancelled';
```

### Upsert — insert or update if key already exists

```sql
INSERT INTO orders (order_id, amount)
VALUES (1, 120.00)
ON CONFLICT (order_id) DO UPDATE SET amount = EXCLUDED.amount;
```

---

## Querying Data

### Select all columns

```sql
SELECT * FROM orders;
```

### Select specific columns

```sql
SELECT order_id, customer_id, amount FROM orders;
```

### Filter rows

```sql
SELECT * FROM orders WHERE amount > 100;
```

### Filter with multiple conditions

```sql
SELECT * FROM orders WHERE amount > 100 AND status = 'active';
```

### Filter with OR

```sql
SELECT * FROM orders WHERE status = 'pending' OR status = 'processing';
```

### Filter using a list of values

```sql
SELECT * FROM orders WHERE status IN ('pending', 'processing', 'failed');
```

### Filter rows where a column is null

```sql
SELECT * FROM orders WHERE email IS NULL;
```

### Filter rows where a column is not null

```sql
SELECT * FROM orders WHERE email IS NOT NULL;
```

### Filter with a pattern match

```sql
SELECT * FROM customers WHERE email LIKE '%@gmail.com';
```

### Filter within a range

```sql
SELECT * FROM orders WHERE amount BETWEEN 50 AND 200;
```

### Limit the number of returned rows

```sql
SELECT * FROM orders LIMIT 100;
```

### Sort results

```sql
SELECT * FROM orders ORDER BY created_at DESC;
```

### Sort by multiple columns

```sql
SELECT * FROM orders ORDER BY region ASC, amount DESC;
```

### Remove duplicate rows from results

```sql
SELECT DISTINCT customer_id FROM orders;
```

---

## Aggregation & Grouping

### Count all rows

```sql
SELECT COUNT(*) FROM orders;
```

### Count non-null values in a column

```sql
SELECT COUNT(email) FROM customers;
```

### Count distinct values

```sql
SELECT COUNT(DISTINCT customer_id) FROM orders;
```

### Sum, average, min, max

```sql
SELECT SUM(amount), AVG(amount), MIN(amount), MAX(amount) FROM orders;
```

### Group by and aggregate

```sql
SELECT region, COUNT(*) AS total_orders, SUM(amount) AS revenue
FROM orders
GROUP BY region;
```

### Filter aggregated results

```sql
SELECT customer_id, COUNT(*) AS order_count
FROM orders
GROUP BY customer_id
HAVING COUNT(*) > 5;
```

---

## Joins

### Inner join — only matching rows from both tables

```sql
SELECT o.order_id, c.name, o.amount
FROM orders o
INNER JOIN customers c ON o.customer_id = c.id;
```

### Left join — all rows from left, matched rows from right

```sql
SELECT o.order_id, c.name
FROM orders o
LEFT JOIN customers c ON o.customer_id = c.id;
```

### Right join — all rows from right, matched rows from left

```sql
SELECT o.order_id, c.name
FROM orders o
RIGHT JOIN customers c ON o.customer_id = c.id;
```

### Full outer join — all rows from both tables

```sql
SELECT o.order_id, c.name
FROM orders o
FULL OUTER JOIN customers c ON o.customer_id = c.id;
```

### Join on multiple keys

```sql
SELECT * FROM orders o
JOIN shipments s ON o.order_id = s.order_id AND o.region = s.region;
```

### Self join — join a table with itself

```sql
SELECT a.id, a.name, b.name AS manager
FROM employees a
JOIN employees b ON a.manager_id = b.id;
```

---

## Subqueries & CTEs

### Subquery in WHERE

```sql
SELECT * FROM orders
WHERE customer_id IN (SELECT id FROM customers WHERE country = 'Finland');
```

### Subquery in FROM

```sql
SELECT region, AVG(total) AS avg_revenue
FROM (SELECT region, SUM(amount) AS total FROM orders GROUP BY region) AS sub
GROUP BY region;
```

### CTE — Common Table Expression

```sql
WITH high_value AS (
    SELECT customer_id, SUM(amount) AS total
    FROM orders
    GROUP BY customer_id
    HAVING SUM(amount) > 1000
)
SELECT c.name, h.total
FROM customers c
JOIN high_value h ON c.id = h.customer_id;
```

### Chained CTEs

```sql
WITH monthly AS (
    SELECT DATE_TRUNC('month', created_at) AS month, SUM(amount) AS revenue
    FROM orders
    GROUP BY 1
),
ranked AS (
    SELECT *, RANK() OVER (ORDER BY revenue DESC) AS rnk
    FROM monthly
)
SELECT * FROM ranked WHERE rnk <= 3;
```

---

## Window Functions

### Row number within a partition

```sql
SELECT *, ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS rn
FROM employees;
```

### Rank within a partition (with gaps for ties)

```sql
SELECT *, RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS rnk
FROM employees;
```

### Dense rank (no gaps for ties)

```sql
SELECT *, DENSE_RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS drnk
FROM employees;
```

### Running total

```sql
SELECT order_id, amount,
    SUM(amount) OVER (PARTITION BY customer_id ORDER BY created_at) AS running_total
FROM orders;
```

### Moving average over the last 7 rows

```sql
SELECT order_id, amount,
    AVG(amount) OVER (ORDER BY created_at ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS moving_avg
FROM orders;
```

### Access the previous row's value

```sql
SELECT order_id, amount,
    LAG(amount, 1) OVER (PARTITION BY customer_id ORDER BY created_at) AS prev_amount
FROM orders;
```

### Access the next row's value

```sql
SELECT order_id, amount,
    LEAD(amount, 1) OVER (PARTITION BY customer_id ORDER BY created_at) AS next_amount
FROM orders;
```

### First and last value in a window

```sql
SELECT order_id,
    FIRST_VALUE(amount) OVER (PARTITION BY customer_id ORDER BY created_at) AS first_order,
    LAST_VALUE(amount) OVER (PARTITION BY customer_id ORDER BY created_at
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS last_order
FROM orders;
```

---

## Date & Time

### Get the current timestamp

```sql
SELECT NOW();
```

### Truncate a timestamp to the month

```sql
SELECT DATE_TRUNC('month', created_at) FROM orders;
```

### Extract part of a date

```sql
SELECT EXTRACT(YEAR FROM created_at), EXTRACT(MONTH FROM created_at) FROM orders;
```

### Add or subtract an interval

```sql
SELECT created_at + INTERVAL '7 days' FROM orders;
SELECT created_at - INTERVAL '1 month' FROM orders;
```

### Difference between two dates in days

```sql
SELECT DATEDIFF('day', start_date, end_date) FROM orders;
-- PostgreSQL style:
SELECT (end_date::date - start_date::date) AS days_diff FROM orders;
```

### Cast a string to a date

```sql
SELECT CAST('2024-06-01' AS DATE);
-- or
SELECT '2024-06-01'::DATE;
```

---

## String Functions

### Uppercase and lowercase

```sql
SELECT UPPER(name), LOWER(email) FROM customers;
```

### Trim whitespace

```sql
SELECT TRIM(name) FROM customers;
```

### Concatenate strings

```sql
SELECT first_name || ' ' || last_name AS full_name FROM customers;
-- or
SELECT CONCAT(first_name, ' ', last_name) AS full_name FROM customers;
```

### Extract a substring

```sql
SELECT SUBSTRING(email FROM 1 FOR 5) FROM customers;
```

### Replace part of a string

```sql
SELECT REPLACE(phone, '-', '') FROM customers;
```

### Get the length of a string

```sql
SELECT LENGTH(name) FROM customers;
```

### Split a string and get a part

```sql
SELECT SPLIT_PART(email, '@', 2) FROM customers;
```

---

## Conditional Logic

### CASE WHEN — conditional column

```sql
SELECT order_id, amount,
    CASE
        WHEN amount >= 500 THEN 'high'
        WHEN amount >= 100 THEN 'mid'
        ELSE 'low'
    END AS order_tier
FROM orders;
```

### COALESCE — return the first non-null value

```sql
SELECT COALESCE(phone, mobile, 'no contact') FROM customers;
```

### NULLIF — return null if two values are equal

```sql
SELECT NULLIF(status, 'unknown') FROM orders;
```

---

## Views & Materialized Views

### Create a view

```sql
CREATE VIEW active_orders AS
SELECT * FROM orders WHERE status = 'active';
```

### Drop a view

```sql
DROP VIEW IF EXISTS active_orders;
```

### Create a materialized view (stores the result physically)

```sql
CREATE MATERIALIZED VIEW monthly_revenue AS
SELECT DATE_TRUNC('month', created_at) AS month, SUM(amount) AS revenue
FROM orders
GROUP BY 1;
```

### Refresh a materialized view

```sql
REFRESH MATERIALIZED VIEW monthly_revenue;
```

---

## Indexes

### Create an index on a column

```sql
CREATE INDEX idx_orders_customer_id ON orders(customer_id);
```

### List indexes

```sql
SHOW INDEXES FROM orders;
```

### Create a unique index

```sql
CREATE UNIQUE INDEX idx_customers_email ON customers(email);
```

### Drop an index

```sql
DROP INDEX idx_orders_customer_id;
```

---

## ETL Patterns

### Export a table to a SQL dump file (MySQL)

```bash
mysqldump -u root -p database_name table_name > /path/to/output.sql
```

### Count value frequency in a column

```sql
SELECT status, COUNT(*) AS frequency
FROM orders
GROUP BY status
ORDER BY frequency DESC;
```

### Find duplicate rows based on a key

```sql
SELECT customer_id, COUNT(*) AS cnt
FROM customers
GROUP BY customer_id
HAVING COUNT(*) > 1;
```

### Deduplicate — keep only the latest record per key

```sql
WITH ranked AS (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY updated_at DESC) AS rn
    FROM customers
)
SELECT * FROM ranked WHERE rn = 1;
```

### Null audit — count nulls in every column

```sql
SELECT
    COUNT(*) - COUNT(order_id)    AS order_id_nulls,
    COUNT(*) - COUNT(customer_id) AS customer_id_nulls,
    COUNT(*) - COUNT(amount)      AS amount_nulls,
    COUNT(*) - COUNT(status)      AS status_nulls
FROM orders;
```

### Incremental load — select only new or changed rows

```sql
SELECT * FROM orders
WHERE updated_at > '2024-06-01 00:00:00';
```

### Enrich a fact table by joining a dimension table

```sql
SELECT
    o.order_id,
    o.amount,
    c.name AS customer_name,
    c.country,
    p.category AS product_category
FROM orders o
LEFT JOIN customers c ON o.customer_id = c.id
LEFT JOIN products p ON o.product_id = p.id;
```

### Detect changed rows between two snapshots (SCD check)

```sql
SELECT n.id, n.value AS new_value, o.value AS old_value
FROM orders_new n
JOIN orders_old o ON n.id = o.id
WHERE n.value <> o.value;
```

### Pivot — turn row values into columns

```sql
SELECT
    customer_id,
    SUM(CASE WHEN quarter = 'Q1' THEN revenue ELSE 0 END) AS Q1,
    SUM(CASE WHEN quarter = 'Q2' THEN revenue ELSE 0 END) AS Q2,
    SUM(CASE WHEN quarter = 'Q3' THEN revenue ELSE 0 END) AS Q3,
    SUM(CASE WHEN quarter = 'Q4' THEN revenue ELSE 0 END) AS Q4
FROM sales
GROUP BY customer_id;
```

### Running total per group

```sql
SELECT
    customer_id,
    order_id,
    amount,
    SUM(amount) OVER (PARTITION BY customer_id ORDER BY created_at) AS running_total
FROM orders;
```

### Top N rows per group

```sql
WITH ranked AS (
    SELECT *, RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS rnk
    FROM employees
)
SELECT * FROM ranked WHERE rnk <= 3;
```

### Flag bad rows for quarantine

```sql
SELECT *,
    CASE
        WHEN email IS NULL THEN 'missing_email'
        WHEN amount < 0 THEN 'negative_amount'
        ELSE 'ok'
    END AS data_quality_flag
FROM orders;
```

### Compute month-over-month change

```sql
WITH monthly AS (
    SELECT DATE_TRUNC('month', created_at) AS month, SUM(amount) AS revenue
    FROM orders
    GROUP BY 1
)
SELECT
    month,
    revenue,
    LAG(revenue) OVER (ORDER BY month) AS prev_revenue,
    ROUND(100.0 * (revenue - LAG(revenue) OVER (ORDER BY month)) / LAG(revenue) OVER (ORDER BY month), 2) AS pct_change
FROM monthly;
```

### Summarize a full table in one query

```sql
SELECT
    COUNT(*)                        AS total_rows,
    COUNT(DISTINCT customer_id)     AS unique_customers,
    SUM(amount)                     AS total_revenue,
    AVG(amount)                     AS avg_order_value,
    MIN(created_at)                 AS earliest_order,
    MAX(created_at)                 AS latest_order
FROM orders;
```

---

_Last updated: 2026_
