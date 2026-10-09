# EXPERIMENT 5

**AIM:**
Create SQL views for department salary summary and employee hierarchy. Test updatability of views. Implement a recursive CTE to display reporting chains.

### 1. Add Manager Information to Employee Table
Our original Employee table does not have a reporting/manager relationship. Add a Manager_ID column.
```sql
ALTER TABLE Employee 
ADD Manager_ID INT NULL;

ALTER TABLE Employee
ADD CONSTRAINT fk_manager
FOREIGN KEY (Manager_ID) REFERENCES Employee(Emp_ID);

UPDATE Employee
SET Manager_ID = CASE
WHEN Emp_ID IN (1, 2, 3, 4, 5, 6) THEN NULL
WHEN Emp_ID IN (7, 8, 9, 10, 11, 12) THEN 8
WHEN Emp_ID IN (13, 14, 15, 16, 17, 18) THEN 18
WHEN Emp_ID IN (19, 20, 21, 22, 23, 24) THEN 21
WHEN Emp_ID IN (25, 26, 27, 28, 29, 30) THEN 27 
END;
```

You can check the hierarchy using:
```sql
SELECT Emp_ID, Emp_Name, Manager_ID 
FROM Employee
ORDER BY Emp_ID;
```

### 2. View for Department Salary Summary
Create a view showing the number of employees, total salary, average salary, minimum salary, and maximum salary for every department.
```sql
CREATE VIEW Department_Salary_Summary AS 
SELECT
d.Dept_ID, d.Dept_Name,
COUNT(e.Emp_ID) AS Total_Employees, 
SUM(e.Salary) AS Total_Salary, 
AVG(e.Salary) AS Average_Salary, 
MIN(e.Salary) AS Minimum_Salary, 
MAX(e.Salary) AS Maximum_Salary
FROM Department d 
LEFT JOIN Employee e
ON d.Dept_ID = e.Dept_ID
GROUP BY d.Dept_ID, d.Dept_Name;

SELECT * FROM Department_Salary_Summary;
```

### 3. Employee Hierarchy View
Create a view showing each employee and their immediate manager.
```sql
CREATE VIEW Employee_Hierarchy AS 
SELECT
e.Emp_ID,
e.Emp_Name AS Employee_Name, 
e.Job_Title, e.Dept_ID, e.Manager_ID, 
m.Emp_Name AS Manager_Name
FROM Employee e 
LEFT JOIN Employee m
ON e.Manager_ID = m.Emp_ID;

SELECT * FROM Employee_Hierarchy;
```

### 4. Test View Updatability
A view is updatable when modifications made through the view can be translated unambiguously to the underlying table.
```sql
UPDATE Employee_Hierarchy
SET Employee_Name = 'Aarav Kumar' 
WHERE Emp_ID = 1;
-- Error: The target table Employee_Hierarchy of the UPDATE is not updatable
```

### 5. Test Department Salary Summary View
```sql
UPDATE Department_Salary_Summary 
SET Average_Salary = 60000
WHERE Dept_ID = 1;
-- Error: The target table Department_Salary_Summary of the UPDATE is not updatable
```

### 6. Create a Simple Updatable View
```sql
CREATE VIEW Employee_Salary_View AS 
SELECT Emp_ID, Emp_Name, Salary, Dept_ID 
FROM Employee;

UPDATE Employee_Salary_View 
SET Salary = 65000
WHERE Emp_ID = 1;
-- Success
```

### 7-9. Recursive CTE
A recursive Common Table Expression (CTE) is useful for hierarchical data such as reporting chains.
```sql
WITH RECURSIVE EmployeeChain AS 
(
-- Anchor member 
SELECT
Emp_ID, Emp_Name, Manager_ID, 0 AS Level, 
CAST(Emp_Name AS CHAR(1000)) AS Reporting_Chain
FROM Employee
WHERE Manager_ID IS NULL 
UNION ALL
-- Recursive member 
SELECT
e.Emp_ID, e.Emp_Name, e.Manager_ID, ec.Level + 1, 
CONCAT(ec.Reporting_Chain, ' -> ', e.Emp_Name)
FROM Employee e
INNER JOIN EmployeeChain ec 
ON e.Manager_ID = ec.Emp_ID
)
SELECT
Emp_ID, Emp_Name, Manager_ID, Level, 
Reporting_Chain
FROM EmployeeChain
ORDER BY Reporting_Chain;
```
