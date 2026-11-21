# EXPERIMENT 8

**AIM:**
Implement banking-style transactions using `BEGIN`, `COMMIT`, `ROLLBACK`, and `SAVEPOINT`. Simulate partial failures and demonstrate isolation levels and concurrency anomalies using two database sessions.

### 1. Create the Bank Table

```sql
CREATE TABLE Bank (
    account_id INT PRIMARY KEY,
    account_name VARCHAR(50),
    balance DECIMAL(10,2)
);

INSERT INTO Bank VALUES
(1, 'Yusuf', 10000),
(2, 'Amit', 5000);
```

### 2. BEGIN and COMMIT

Transfer ₹1,000 from Yusuf to Amit.

```sql
START TRANSACTION;

UPDATE Bank
SET balance = balance - 1000
WHERE account_id = 1;

UPDATE Bank
SET balance = balance + 1000
WHERE account_id = 2;

COMMIT;

SELECT * FROM Bank;
```

**Output:**

| account_id | account_name | balance |
|---|---|---|
| 1 | Yusuf | 9000.00 |
| 2 | Amit | 6000.00 |

### 3. ROLLBACK

```sql
START TRANSACTION;

UPDATE Bank
SET balance = balance - 2000
WHERE account_id = 1;

ROLLBACK;

SELECT * FROM Bank;
```

The rollback cancels the update, so Yusuf's balance remains `9000.00`.

### 4. SAVEPOINT

```sql
START TRANSACTION;

UPDATE Bank
SET balance = balance - 500
WHERE account_id = 1;

SAVEPOINT sp1;

UPDATE Bank
SET balance = balance + 500
WHERE account_id = 2;

ROLLBACK TO sp1;
COMMIT;

SELECT * FROM Bank;
```

The credit to Amit is undone. Yusuf remains unchanged after the savepoint rollback.

### 5. Insufficient-Balance Failure

```sql
START TRANSACTION;

UPDATE Bank
SET balance = balance - 20000
WHERE account_id = 1
  AND balance >= 20000;

SELECT ROW_COUNT() AS rows_updated;

ROLLBACK;
```

**Output:**

| rows_updated |
|---|
| 0 |

No row is updated because Yusuf does not have sufficient funds. `ROLLBACK` ends the transaction without saving changes.

### 6. Demonstrate Isolation Levels

```sql
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
START TRANSACTION;
SELECT * FROM Bank;
COMMIT;
```

Other supported MySQL isolation levels are:

```sql
SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SET SESSION TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

Run one isolation-level setting at a time before starting the transaction.

### 7. Concurrency Using Two Sessions

**Session A**

```sql
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
START TRANSACTION;
SELECT balance FROM Bank WHERE account_id = 1;

UPDATE Bank
SET balance = balance - 500
WHERE account_id = 1;

COMMIT;
```

**Session B**

```sql
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
START TRANSACTION;
SELECT balance FROM Bank WHERE account_id = 1;
COMMIT;
```

Under `READ COMMITTED`, Session B can see only committed versions of the balance.

### 8. Demonstrate REPEATABLE READ

**Session B**

```sql
SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;
START TRANSACTION;

SELECT balance FROM Bank WHERE account_id = 1;
SELECT balance FROM Bank WHERE account_id = 1;

COMMIT;
```

Both reads in the transaction see a consistent snapshot.

### 9. Demonstrate a Dirty Read

**Session B**

```sql
SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
START TRANSACTION;

SELECT balance FROM Bank WHERE account_id = 1;

COMMIT;
```

`READ UNCOMMITTED` may allow a transaction to read data that another transaction has changed but not committed.
