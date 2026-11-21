# EXPERIMENT 10

**AIM:**
Generate a dataset containing at least 10,000 rows, write five slow SQL queries, analyze them using `EXPLAIN`, optimize them using indexing and query rewriting, and compare execution time before and after optimization.

### 1. Create the Sales Dataset

```sql
DROP TABLE IF EXISTS Sales_Analytics;

CREATE TABLE Sales_Analytics (
    order_id INT PRIMARY KEY,
    customer_id INT,
    product_id INT,
    city VARCHAR(50),
    order_date DATE,
    amount DECIMAL(10,2),
    quantity INT,
    status VARCHAR(20)
);

SET SESSION cte_max_recursion_depth = 11000;

INSERT INTO Sales_Analytics
(order_id, customer_id, product_id, city,
 order_date, amount, quantity, status)
WITH RECURSIVE numbers AS (
    SELECT 1 AS n
    UNION ALL
    SELECT n + 1 FROM numbers WHERE n < 10000
)
SELECT
    n,
    MOD(n, 1000) + 1,
    MOD(n, 100) + 1,
    CASE MOD(n, 4)
        WHEN 0 THEN 'Delhi'
        WHEN 1 THEN 'Mumbai'
        WHEN 2 THEN 'Pune'
        ELSE 'Jaipur'
    END,
    DATE_ADD('2025-01-01', INTERVAL MOD(n, 365) DAY),
    100 + MOD(n * 17, 5000),
    MOD(n, 10) + 1,
    CASE MOD(n, 3)
        WHEN 0 THEN 'Completed'
        WHEN 1 THEN 'Pending'
        ELSE 'Cancelled'
    END
FROM numbers;

SELECT COUNT(*) AS total_records FROM Sales_Analytics;
```

**Output:**

| total_records |
|---|
| 10000 |

### 2. Query 1 — Date Filtering

**Before Optimization**

```sql
EXPLAIN
SELECT *
FROM Sales_Analytics
WHERE YEAR(order_date) = 2025;
```

Applying `YEAR()` prevents direct use of an index on `order_date`.

**After Optimization**

```sql
CREATE INDEX idx_order_date
ON Sales_Analytics(order_date);

EXPLAIN
SELECT *
FROM Sales_Analytics
WHERE order_date >= '2025-01-01'
  AND order_date < '2026-01-01';
```

### 3. Query 2 — City Filtering

**Before Optimization**

```sql
EXPLAIN
SELECT *
FROM Sales_Analytics
WHERE LOWER(city) = 'delhi';
```

**After Optimization**

```sql
CREATE INDEX idx_city
ON Sales_Analytics(city);

EXPLAIN
SELECT *
FROM Sales_Analytics
WHERE city = 'Delhi';
```

Avoiding `LOWER()` allows MySQL to use the city index.

### 4. Query 3 — Status and Date Filtering

**Before Optimization**

```sql
EXPLAIN
SELECT *
FROM Sales_Analytics
WHERE status = 'Completed'
  AND order_date >= '2025-06-01';
```

**After Optimization**

```sql
CREATE INDEX idx_status_date
ON Sales_Analytics(status, order_date);

EXPLAIN
SELECT *
FROM Sales_Analytics
WHERE status = 'Completed'
  AND order_date >= '2025-06-01';
```

### 5. Query 4 — Repeated Correlated Subquery

**Before Optimization**

```sql
EXPLAIN
SELECT DISTINCT s.customer_id
FROM Sales_Analytics s
WHERE (
    SELECT SUM(s2.amount)
    FROM Sales_Analytics s2
    WHERE s2.customer_id = s.customer_id
) > 30000;
```

**After Optimization**

```sql
CREATE INDEX idx_customer_amount
ON Sales_Analytics(customer_id, amount);

EXPLAIN
SELECT customer_id
FROM Sales_Analytics
GROUP BY customer_id
HAVING SUM(amount) > 30000;
```

The aggregate query avoids repeated correlated subqueries and uses the composite index efficiently.

### 6. Query 5 — Sorting by Amount

**Before Optimization**

```sql
EXPLAIN
SELECT order_id, amount
FROM Sales_Analytics
ORDER BY amount DESC
LIMIT 10;
```

**After Optimization**

```sql
CREATE INDEX idx_amount
ON Sales_Analytics(amount);

EXPLAIN
SELECT order_id, amount
FROM Sales_Analytics
ORDER BY amount DESC
LIMIT 10;
```

The amount index allows the database to avoid sorting all rows.

### 7. Measure Optimized Execution Time

```sql
SET @start_time = NOW(6);

SELECT *
FROM Sales_Analytics
WHERE order_date >= '2025-01-01'
  AND order_date < '2026-01-01';

SELECT
    TIMESTAMPDIFF(MICROSECOND, @start_time, NOW(6)) / 1000
    AS execution_time_ms;
```
