# EXPERIMENT 6

**AIM:**
Create a stored procedure transfer_employee(emp_id, new_dept_id) with validation and error handling. Implement triggers for salary validation and audit logging. Test edge cases.

### SQL QUERIES

### 1. Create Audit Table
First, create a table to store salary changes.
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
```sql
CALL transfer_employee(1, 2);

-- Edge Case — Employee Does Not Exist
CALL transfer_employee(999, 2);
-- Error: Employee does not exist

-- Edge Case — Employee Already in Department
CALL transfer_employee(1, 2);
-- Error: Employee is already in this department
```

### 15-21. Test Salary Validation
```sql
-- Negative Salary
UPDATE Employee SET Salary = -5000 WHERE Emp_ID = 2;
-- Error: Salary must be greater than zero

-- Salary Below Minimum
UPDATE Employee SET Salary = 5000 WHERE Emp_ID = 2;
-- Error: Salary cannot be less than 10000

-- Salary Above Maximum
UPDATE Employee SET Salary = 1500000 WHERE Emp_ID = 2;
-- Error: Salary cannot exceed 1000000

-- NULL
UPDATE Employee SET Salary = NULL WHERE Emp_ID = 2;
-- Error: Salary cannot be NULL

-- Valid Salary Update
UPDATE Employee SET Salary = 75000 WHERE Emp_ID = 2;
```
