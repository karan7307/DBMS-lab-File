# EXPERIMENT 5

**AIM:**
Create SQL views for department salary summary and employee hierarchy. Test updatability of views. Implement a recursive CTE to display reporting chains.

### 1. Add Manager Information to Employee Table
*Explanation: Adds a Manager_ID column and a self-referencing foreign key, then updates the table to assign hierarchical managers to employees.*
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
**Output:**
| Emp_ID | Emp_Name | Manager_ID |
|---|---|---|
| 1 | Aarav Sharma | null |
| 2 | Riya Gupta | null |
| 3 | Arjun Verma | null |
| 4 | Ananya Singh | null |
| 5 | Karan Mehta | null |
| 6 | Priya Sharma | null |
| 7 | Neha Kapoor | 8 |
| 8 | Rahul Jain | 8 |
| 9 | Simran Kaur | 8 |
| 10 | Aditya Roy | 8 |
| 11 | Megha Joshi | 8 |
| 12 | Vivek Malhotra | 8 |
| 13 | Ishita Rao | 18 |
| 14 | Manish Kumar | 18 |
| 15 | Pooja Agarwal | 18 |
| 16 | Rohit Bansal | 18 |
| 17 | Kavya Nair | 18 |
| 18 | Sahil Gupta | 18 |
| 19 | Tanya Singh | 21 |
| 20 | Yash Verma | 21 |
| 21 | Nisha Patel | 21 |
| 22 | Aman Khan | 21 |
| 23 | Shreya Das | 21 |
| 24 | Dev Sharma | 21 |
| 25 | Akash Yadav | 27 |
| 26 | Sneha Roy | 27 |
| 27 | Varun Singh | 27 |
| 28 | Komal Gupta | 27 |
| 29 | Nitin Sharma | 27 |
| 30 | Aditi Mehta | 27 |


### 2. View for Department Salary Summary
*Explanation: Creates a virtual table (view) that automatically calculates and stores aggregate salary statistics for each department.*
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
**Output:**
| Dept_ID | Dept_Name | Total_Employees | Total_Salary | Average_Salary | Minimum_Salary | Maximum_Salary |
|---|---|---|---|---|---|---|
| 1 | IT | 6 | 338000.00 | 56333.333333 | 45000.00 | 70000.00 |
| 2 | HR | 6 | 330000.00 | 55000.000000 | 42000.00 | 72000.00 |
| 3 | Finance | 6 | 393000.00 | 65500.000000 | 52000.00 | 80000.00 |
| 4 | Marketing | 6 | 319000.00 | 53166.666667 | 44000.00 | 62000.00 |
| 5 | Operations | 6 | 338000.00 | 56333.333333 | 46000.00 | 69000.00 |


### 3. Employee Hierarchy View
*Explanation: Creates a view displaying employees alongside their direct managers by using a LEFT JOIN on the Employee table with itself.*
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
**Output:**
| Emp_ID | Employee_Name | Job_Title | Dept_ID | Manager_ID | Manager_Name |
|---|---|---|---|---|---|
| 1 | Aarav Sharma | Developer | 1 | null | null |
| 2 | Riya Gupta | Developer | 1 | null | null |
| 3 | Arjun Verma | Tester | 1 | null | null |
| 4 | Ananya Singh | Team Lead | 1 | null | null |
| 5 | Karan Mehta | Support Engineer | 1 | null | null |
| 6 | Priya Sharma | Developer | 1 | null | null |
| 7 | Neha Kapoor | HR Executive | 2 | 8 | Rahul Jain |
| 8 | Rahul Jain | HR Manager | 2 | 8 | Rahul Jain |
| 9 | Simran Kaur | Recruiter | 2 | 8 | Rahul Jain |
| 10 | Aditya Roy | HR Executive | 2 | 8 | Rahul Jain |
| 11 | Megha Joshi | Recruiter | 2 | 8 | Rahul Jain |
| 12 | Vivek Malhotra | HR Manager | 2 | 8 | Rahul Jain |
| 13 | Ishita Rao | Accountant | 3 | 18 | Sahil Gupta |
| 14 | Manish Kumar | Financial Analyst | 3 | 18 | Sahil Gupta |
| 15 | Pooja Agarwal | Accountant | 3 | 18 | Sahil Gupta |
| 16 | Rohit Bansal | Financial Analyst | 3 | 18 | Sahil Gupta |
| 17 | Kavya Nair | Accountant | 3 | 18 | Sahil Gupta |
| 18 | Sahil Gupta | Finance Manager | 3 | 18 | Sahil Gupta |
| 19 | Tanya Singh | Marketing Executive | 4 | 21 | Nisha Patel |
| 20 | Yash Verma | Marketing Analyst | 4 | 21 | Nisha Patel |
| 21 | Nisha Patel | Marketing Manager | 4 | 21 | Nisha Patel |
| 22 | Aman Khan | Sales Executive | 4 | 21 | Nisha Patel |
| 23 | Shreya Das | Marketing Executive | 4 | 21 | Nisha Patel |
| 24 | Dev Sharma | Sales Executive | 4 | 21 | Nisha Patel |
| 25 | Akash Yadav | Operations Executive | 5 | 27 | Varun Singh |
| 26 | Sneha Roy | Operations Analyst | 5 | 27 | Varun Singh |
| 27 | Varun Singh | Operations Manager | 5 | 27 | Varun Singh |
| 28 | Komal Gupta | Operations Executive | 5 | 27 | Varun Singh |
| 29 | Nitin Sharma | Process Analyst | 5 | 27 | Varun Singh |
| 30 | Aditi Mehta | Operations Manager | 5 | 27 | Varun Singh |


### 4. Test View Updatability
*Explanation: Demonstrates that complex views (with joins or aggregates) cannot be directly updated, but simple 1-to-1 views can be successfully updated.*
```sql
UPDATE Employee_Hierarchy
SET Employee_Name = 'Aarav Kumar' 
WHERE Emp_ID = 1;
-- Error: The target table Employee_Hierarchy of the UPDATE is not updatable
```
Therefore, use:
```sql
SELECT *
FROM Employee_Hierarchy 
WHERE Emp_ID = 1;
```
**Output:**
| Emp_ID | Employee_Name | Job_Title | Dept_ID | Manager_ID | Manager_Name |
|---|---|---|---|---|---|
| 1 | Aarav Sharma | Developer | 1 | null | null |


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

SELECT * FROM Employee_Salary_View;
```
**Output:**
| Emp_ID | Emp_Name | Salary | Dept_ID |
|---|---|---|---|
| 1 | Aarav Sharma | 55000.00 | 1 |
| 2 | Riya Gupta | 62000.00 | 1 |
| 3 | Arjun Verma | 48000.00 | 1 |
| 4 | Ananya Singh | 70000.00 | 1 |
| 5 | Karan Mehta | 45000.00 | 1 |
| 6 | Priya Sharma | 58000.00 | 1 |
| 7 | Neha Kapoor | 50000.00 | 2 |
| 8 | Rahul Jain | 65000.00 | 2 |
| 9 | Simran Kaur | 42000.00 | 2 |
| 10 | Aditya Roy | 46000.00 | 2 |
| ... | ... | ... | ... |

Now update an employee through the view:
```sql
UPDATE Employee_Salary_View 
SET Salary = 65000
WHERE Emp_ID = 1;
-- Success
```
Check the original table:
```sql
SELECT Emp_ID, Emp_Name, Salary 
FROM Employee
WHERE Emp_ID = 1;
```
**Output:**
| Emp_ID | Emp_Name | Salary |
|---|---|---|
| 1 | Aarav Sharma | 65000.00 |


### 7-8. Recursive CTE (Display Complete Reporting Chains)
*Explanation: Uses a Recursive Common Table Expression (CTE) to traverse the management hierarchy, tracking the levels and mapping out complete reporting chains for each employee.*
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
**Output:**
| Emp_ID | Emp_Name | Manager_ID | Level | Reporting_Chain |
|---|---|---|---|---|
| 1 | Aarav Sharma | null | 0 | Aarav Sharma |
| 4 | Ananya Singh | null | 0 | Ananya Singh |
| 3 | Arjun Verma | null | 0 | Arjun Verma |
| 5 | Karan Mehta | null | 0 | Karan Mehta |
| 6 | Priya Sharma | null | 0 | Priya Sharma |
| 2 | Riya Gupta | null | 0 | Riya Gupta |


### 9. Display Hierarchy Level by Level
If you only want to see the employee's hierarchy level:
```sql
WITH RECURSIVE EmployeeHierarchy AS 
(
SELECT
Emp_ID, Emp_Name, Manager_ID, 0 AS Hierarchy_Level
FROM Employee
WHERE Manager_ID IS NULL
UNION ALL 
SELECT
e.Emp_ID, e.Emp_Name, e.Manager_ID, eh.Hierarchy_Level + 1 
FROM Employee e
JOIN EmployeeHierarchy eh
ON e.Manager_ID = eh.Emp_ID
)
SELECT
Emp_ID, Emp_Name, Manager_ID, Hierarchy_Level 
FROM EmployeeHierarchy
ORDER BY Hierarchy_Level, Emp_ID;
```
**Output:**
| Emp_ID | Emp_Name | Manager_ID | Hierarchy_Level |
|---|---|---|---|
| 1 | Aarav Sharma | null | 0 |
| 2 | Riya Gupta | null | 0 |
| 3 | Arjun Verma | null | 0 |
| 4 | Ananya Singh | null | 0 |
| 5 | Karan Mehta | null | 0 |
| 6 | Priya Sharma | null | 0 |


### 10. Find All Employees Reporting to One Manager
For example, find the complete hierarchy below employee 4:
```sql
WITH RECURSIVE Subordinates AS 
(
SELECT
Emp_ID, Emp_Name, Manager_ID, 0 AS Level 
FROM Employee
WHERE Emp_ID = 4
UNION ALL 
SELECT
e.Emp_ID, e.Emp_Name, e.Manager_ID, s.Level + 1 
FROM Employee e
JOIN Subordinates s
ON e.Manager_ID = s.Emp_ID
)
SELECT *
FROM Subordinates;
```
**Output:**
| Emp_ID | Emp_Name | Manager_ID | Level |
|---|---|---|---|
| 4 | Ananya Singh | null | 0 |
