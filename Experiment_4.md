# Experiment 4

**AIM :** Using the Employee schema, write queries demonstrating:
- INNER JOIN 
- LEFT JOIN 
- SELF JOIN 
- 3-WAY JOIN 
- Correlated Subquery 
- EXISTS 
- Simulated INTERSECT 
- Simulated EXCEPT 
- Compare execution plans using EXPLAIN

**SOURCE CODE :** 

```sql
CREATE DATABASE IF NOT EXISTS companydb;
USE companydb;

DROP TABLE IF EXISTS Employee;
DROP TABLE IF EXISTS Department;
DROP TABLE IF EXISTS Project;

CREATE TABLE Department (
 dept_id INT PRIMARY KEY,
 dept_name VARCHAR(50) NOT NULL,
 location VARCHAR(50)
);

CREATE TABLE Project (
 project_id INT PRIMARY KEY,
 project_name VARCHAR(100) NOT NULL,
 dept_id INT,
 budget DECIMAL(12,2),
 FOREIGN KEY (dept_id) REFERENCES Department(dept_id)
);

CREATE TABLE Employee (
 emp_id INT PRIMARY KEY,
 emp_name VARCHAR(100) NOT NULL,
 salary DECIMAL(10,2),
 dept_id INT,
 manager_id INT,
 project_id INT,
 FOREIGN KEY (dept_id) REFERENCES Department(dept_id),
 FOREIGN KEY (project_id) REFERENCES Project(project_id),
 FOREIGN KEY (manager_id) REFERENCES Employee(emp_id)
);

INSERT INTO Department VALUES
(1, 'IT', 'Delhi'),
(2, 'HR', 'Mumbai'),
(3, 'Finance', 'Pune'),
(4, 'Marketing', 'Bangalore'),
(5, 'Operations', 'Chennai');

INSERT INTO Project VALUES
(101, 'AI Platform', 1, 500000),
(102, 'Cloud Migration', 1, 350000),
(103, 'Recruitment System', 2, 150000),
(104, 'Financial Analytics', 3, 400000),
(105, 'Digital Marketing', 4, 250000),
(106, 'Supply Chain', 5, 300000);

INSERT INTO Employee VALUES
(1, 'Rahul', 90000, 1, NULL, 101),
(2, 'Amit', 70000, 1, 1, 101),
(3, 'Priya', 65000, 1, 1, 102),
(4, 'Neha', 60000, 2, NULL, 103),
(5, 'Rohit', 50000, 2, 4, 103),
(6, 'Anjali', 85000, 3, NULL, 104),
(7, 'Vikas', 55000, 3, 6, 104),
(8, 'Sneha', 75000, 4, NULL, 105),
(9, 'Karan', 52000, 4, 8, 105),
(10, 'Arjun', 80000, 5, NULL, 106),
(11, 'Pooja', 48000, 5, 10, 106),
(12, 'Manish', 45000, 1, 2, 102);
```

### 1. INNER JOIN
Display employees along with their department names.
```sql
SELECT
 e.emp_id,
 e.emp_name,
 e.salary,
 d.dept_name
FROM Employee e
INNER JOIN Department d
 ON e.dept_id = d.dept_id;
```

**OUTPUT :**
```text
+--------+----------+----------+-----------+
| emp_id | emp_name | salary   | dept_name |
+--------+----------+----------+-----------+
|      1 | Rahul    | 90000.00 | IT        |
|      2 | Amit     | 70000.00 | IT        |
|      3 | Priya    | 65000.00 | IT        |
|     12 | Manish   | 45000.00 | IT        |
|      4 | Neha     | 60000.00 | HR        |
|      5 | Rohit    | 50000.00 | HR        |
|      6 | Anjali   | 85000.00 | Finance   |
|      7 | Vikas    | 55000.00 | Finance   |
|      8 | Sneha    | 75000.00 | Marketing |
|      9 | Karan    | 52000.00 | Marketing |
|     10 | Arjun    | 80000.00 | Operations|
|     11 | Pooja    | 48000.00 | Operations|
+--------+----------+----------+-----------+
```

### 2. LEFT JOIN
Display all departments, including departments that have no employees.
```sql
SELECT
 d.dept_id,
 d.dept_name,
 e.emp_name,
 e.salary
FROM Department d
LEFT JOIN Employee e
 ON d.dept_id = e.dept_id;
```

**OUTPUT :**
```text
+---------+-----------+----------+----------+
| dept_id | dept_name | emp_name | salary   |
+---------+-----------+----------+----------+
|       1 | IT        | Rahul    | 90000.00 |
|       1 | IT        | Amit     | 70000.00 |
|       1 | IT        | Priya    | 65000.00 |
|       1 | IT        | Manish   | 45000.00 |
|       2 | HR        | Neha     | 60000.00 |
|       2 | HR        | Rohit    | 50000.00 |
|       3 | Finance   | Anjali   | 85000.00 |
|       3 | Finance   | Vikas    | 55000.00 |
|       4 | Marketing | Sneha    | 75000.00 |
|       4 | Marketing | Karan    | 52000.00 |
|       5 | Operations| Arjun    | 80000.00 |
|       5 | Operations| Pooja    | 48000.00 |
+---------+-----------+----------+----------+
```

### 3. SELF JOIN
Display each employee with their manager.
```sql
SELECT
 e.emp_name AS Employee,
 m.emp_name AS Manager
FROM Employee e
LEFT JOIN Employee m
 ON e.manager_id = m.emp_id;
```

**OUTPUT :**
```text
+----------+---------+
| Employee | Manager |
+----------+---------+
| Rahul    | NULL    |
| Amit     | Rahul   |
| Priya    | Rahul   |
| Neha     | NULL    |
| Rohit    | Neha    |
| Anjali   | NULL    |
| Vikas    | Anjali  |
| Sneha    | NULL    |
| Karan    | Sneha   |
| Arjun    | NULL    |
| Pooja    | Arjun   |
| Manish   | Amit    |
+----------+---------+
```

### 4. THREE-WAY JOIN
Display employee name, department name and project name.
```sql
SELECT
 e.emp_name,
 d.dept_name,
 p.project_name,
 p.budget
FROM Employee e
INNER JOIN Department d
 ON e.dept_id = d.dept_id
INNER JOIN Project p
 ON e.project_id = p.project_id;
```

**OUTPUT :**
```text
+----------+-----------+---------------------+-----------+
| emp_name | dept_name | project_name        | budget    |
+----------+-----------+---------------------+-----------+
| Rahul    | IT        | AI Platform         | 500000.00 |
| Amit     | IT        | AI Platform         | 500000.00 |
| Priya    | IT        | Cloud Migration     | 350000.00 |
| Neha     | HR        | Recruitment System  | 150000.00 |
| Rohit    | HR        | Recruitment System  | 150000.00 |
| Anjali   | Finance   | Financial Analytics | 400000.00 |
| Vikas    | Finance   | Financial Analytics | 400000.00 |
| Sneha    | Marketing | Digital Marketing   | 250000.00 |
| Karan    | Marketing | Digital Marketing   | 250000.00 |
| Arjun    | Operations| Supply Chain        | 300000.00 |
| Pooja    | Operations| Supply Chain        | 300000.00 |
| Manish   | IT        | Cloud Migration     | 350000.00 |
+----------+-----------+---------------------+-----------+
```

### 5. CORRELATED SUBQUERY
Find employees whose salary is greater than the average salary of their own department.
```sql
SELECT
 e.emp_id,
 e.emp_name,
 e.salary,
 e.dept_id
FROM Employee e
WHERE e.salary > (
 SELECT AVG(e2.salary)
 FROM Employee e2
 WHERE e2.dept_id = e.dept_id
);
```
Here the inner query depends on the current row of the outer query, so it is a correlated subquery.

**OUTPUT :**
```text
+--------+----------+----------+---------+
| emp_id | emp_name | salary   | dept_id |
+--------+----------+----------+---------+
|      1 | Rahul    | 90000.00 |       1 |
|      2 | Amit     | 70000.00 |       1 |
|      4 | Neha     | 60000.00 |       2 |
|      6 | Anjali   | 85000.00 |       3 |
|      8 | Sneha    | 75000.00 |       4 |
|     10 | Arjun    | 80000.00 |       5 |
+--------+----------+----------+---------+
```

### 6. EXISTS
Find departments that have at least one employee
```sql
SELECT
 d.dept_id,
 d.dept_name
FROM Department d
WHERE EXISTS (
 SELECT 1
 FROM Employee e
 WHERE e.dept_id = d.dept_id
);
```

**OUTPUT :**
```text
+---------+-----------+
| dept_id | dept_name |
+---------+-----------+
|       1 | IT        |
|       2 | HR        |
|       3 | Finance   |
|       4 | Marketing |
|       5 | Operations|
+---------+-----------+
```

### 7. Simulated INTERSECT
Find employees who are working on projects with a budget greater than 300000 and whose salary is greater than 60000.

Using EXISTS to simulate an intersection:
```sql
SELECT e.emp_id, e.emp_name
FROM Employee e
WHERE e.salary > 60000
AND EXISTS (
 SELECT 1
 FROM Project p
 WHERE p.project_id = e.project_id
 AND p.budget > 300000
);
```
Another INTERSECT simulation using INNER JOIN
```sql
SELECT e.emp_id, e.emp_name
FROM Employee e
INNER JOIN Project p
 ON e.project_id = p.project_id
WHERE e.salary > 60000
AND p.budget > 300000;
```

**OUTPUT :**
```text
+--------+----------+
| emp_id | emp_name |
+--------+----------+
|      1 | Rahul    |
|      2 | Amit     |
|      3 | Priya    |
|      6 | Anjali   |
+--------+----------+
```

### 8. Simulated EXCEPT
Find employees who are not working in the IT department.
```sql
SELECT e.emp_id, e.emp_name
FROM Employee e
WHERE NOT EXISTS (
 SELECT 1
 FROM Department d
 WHERE d.dept_id = e.dept_id
 AND d.dept_name = 'IT'
);
```

**OUTPUT :**
```text
+--------+----------+
| emp_id | emp_name |
+--------+----------+
|      4 | Neha     |
|      5 | Rohit    |
|      6 | Anjali   |
|      7 | Vikas    |
|      8 | Sneha    |
|      9 | Karan    |
|     10 | Arjun    |
|     11 | Pooja    |
+--------+----------+
```

### 9-13. EXPLAIN Statements
EXPLAIN shows how MySQL plans to execute a query. (See EXPLAIN queries in the lab document)
