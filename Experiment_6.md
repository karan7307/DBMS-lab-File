# EXPERIMENT 6

**AIM:**
Create a stored procedure transfer_employee(emp_id, new_dept_id) with validation and error handling. Implement triggers for salary validation and audit logging. Test edge cases.

### SQL QUERIES

### 1. Create Audit Table
First, create a table to store salary changes.
*Explanation: Sets up a separate tracking table to keep an immutable history of whenever an employee's salary is modified.*
```sql
CREATE TABLE Salary_Audit (
Audit_ID INT AUTO_INCREMENT PRIMARY KEY, 
Emp_ID INT NOT NULL,
Old_Salary DECIMAL(10,2), 
New_Salary DECIMAL(10,2),
Changed_At TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
Action_Type VARCHAR(20)
);
```

### 2. Trigger for Salary Validation
Create a BEFORE INSERT trigger:
*Explanation: Creates a BEFORE INSERT trigger that intercepts new records and throws an error if the salary provided is invalid (NULL, negative, too low, or too high).*
```sql
CREATE TRIGGER validate_salary_insert 
BEFORE INSERT ON Employee
FOR EACH ROW 
BEGIN
IF NEW.Salary IS NULL THEN 
SIGNAL SQLSTATE '45000'
SET MESSAGE_TEXT = 'Salary cannot be NULL';
ELSEIF NEW.Salary <= 0 THEN 
SIGNAL SQLSTATE '45000'
SET MESSAGE_TEXT = 'Salary must be greater than zero';
ELSEIF NEW.Salary < 10000 THEN 
SIGNAL SQLSTATE '45000'
SET MESSAGE_TEXT = 'Salary cannot be less than 10000';
ELSEIF NEW.Salary > 1000000 THEN 
SIGNAL SQLSTATE '45000'
SET MESSAGE_TEXT = 'Salary cannot exceed 1000000'; 
END IF;
END;
```

### 9. Salary Audit Trigger
This trigger automatically records every successful salary change.
*Explanation: Creates an AFTER UPDATE trigger that automatically saves a record into the Salary_Audit table containing the old and new salaries immediately after an update occurs.*
```sql
CREATE TRIGGER salary_audit 
AFTER UPDATE ON Employee 
FOR EACH ROW
BEGIN
IF OLD.Salary <> NEW.Salary THEN 
INSERT INTO Salary_Audit
(Emp_ID, Old_Salary, New_Salary, Action_Type)
VALUES 
(NEW.Emp_ID, OLD.Salary, NEW.Salary, 'SALARY UPDATE');
END IF;
END;
```

### 10. Create Stored Procedure
The procedure transfer_employee() transfers an employee to another department.
*Explanation: Defines a reusable stored procedure with a transaction and error handling to safely transfer an employee to a new department, ensuring both the employee and department exist first.*
```sql
CREATE PROCEDURE transfer_employee( 
IN p_emp_id INT,
IN p_new_dept_id INT
)
BEGIN
DECLARE v_emp_count INT DEFAULT 0; 
DECLARE v_dept_count INT DEFAULT 0; 
DECLARE v_current_dept INT DEFAULT 0;
DECLARE EXIT HANDLER FOR SQLEXCEPTION 
BEGIN
ROLLBACK; 
RESIGNAL;
END;
START TRANSACTION;
SELECT COUNT(*) INTO v_emp_count FROM Employee WHERE Emp_ID = p_emp_id;
IF v_emp_count = 0 THEN 
SIGNAL SQLSTATE '45000'
SET MESSAGE_TEXT = 'Employee does not exist'; 
END IF;
SELECT COUNT(*) INTO v_dept_count FROM Department WHERE Dept_ID = p_new_dept_id;
IF v_dept_count = 0 THEN 
SIGNAL SQLSTATE '45000'
SET MESSAGE_TEXT = 'Department does not exist'; 
END IF;
SELECT Dept_ID INTO v_current_dept FROM Employee WHERE Emp_ID = p_emp_id;
IF v_current_dept = p_new_dept_id THEN 
SIGNAL SQLSTATE '45000'
SET MESSAGE_TEXT = 'Employee is already in this department'; 
END IF;
UPDATE Employee SET Dept_ID = p_new_dept_id WHERE Emp_ID = p_emp_id;
COMMIT; 
END;
```

### 12-14. Test the Stored Procedure — Valid Case
*Explanation: Executes the transfer_employee procedure with both valid parameters to test success, and invalid parameters to ensure our custom error conditions work correctly.*
```sql
-- Valid Case
CALL transfer_employee(1, 2);
SELECT Emp_ID, Emp_Name, Dept_ID 
FROM Employee
WHERE Emp_ID = 1;
```
**Output:**
| Emp_ID | Emp_Name | Dept_ID |
|---|---|---|
| 1 | Aarav Sharma | 2 |

```sql
-- Edge Case — Employee Does Not Exist
CALL transfer_employee(999, 2);
-- Output: Error: Employee does not exist

-- Edge Case — Employee Already in Department
CALL transfer_employee(1, 2);
-- Output: Error: Employee is already in this department
```

### 15-21. Test Salary Validation
*Explanation: Deliberately attempts to break the rules to verify that the validation trigger successfully prevents bad data, then does a valid update to verify the audit trigger logs the change correctly.*
```sql
-- Negative Salary
UPDATE Employee SET Salary = -5000 WHERE Emp_ID = 2;
-- Output: Error: Salary must be greater than zero

-- Salary Below Minimum
UPDATE Employee SET Salary = 5000 WHERE Emp_ID = 2;
-- Output: Error: Salary cannot be less than 10000

-- Salary Above Maximum
UPDATE Employee SET Salary = 1500000 WHERE Emp_ID = 2;
-- Output: Error: Salary cannot exceed 1000000

-- NULL
UPDATE Employee SET Salary = NULL WHERE Emp_ID = 2;
-- Output: Error: Salary cannot be NULL

-- Valid Salary Update
UPDATE Employee SET Salary = 75000 WHERE Emp_ID = 2;
```

### 20. Verify Salary Audit
```sql
SELECT *
FROM Salary_Audit
ORDER BY Audit_ID DESC;
```
**Output:**
| Audit_ID | Emp_ID | Old_Salary | New_Salary | Changed_At | Action_Type |
|---|---|---|---|---|---|
| 1 | 2 | 62000.00 | 75000.00 | 2026-09-27T07:48:23.000Z | SALARY UPDATE |

### 21. Test INSERT Salary Validation
```sql
INSERT INTO Employee
(Emp_ID, Emp_Name, Salary, Job_Title, Dept_ID, Project_ID) 
VALUES
(31, 'Test Employee', -5000, 'Developer', 1, 101);
-- Output: Error: Salary must be greater than zero

INSERT INTO Employee
(Emp_ID, Emp_Name, Salary, Job_Title, Dept_ID, Project_ID) 
VALUES
(31, 'Test Employee', 45000, 'Developer', 1, 101);
-- Success
```

### 22. Final Verification
```sql
SELECT *
FROM Employee 
WHERE Emp_ID = 1;
```
**Output:**
| Emp_ID | Emp_Name | Salary | Job_Title | Dept_ID | Project_ID | Manager_ID |
|---|---|---|---|---|---|---|
| 1 | Aarav Sharma | 65000.00 | Developer | 2 | 101 | null |
