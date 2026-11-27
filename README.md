# 🗄️ MySQL Assignment 2 – Querying the Employee Database

![MySQL](https://img.shields.io/badge/Database-MySQL-4479A1?logo=mysql&logoColor=white)
![SQL](https://img.shields.io/badge/Language-SQL-orange)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)

## 📌 Project Overview

As part of my journey to become a **Data Analyst**, this project explores how to retrieve, filter, aggregate and analyse data using SQL. It builds on the **Employee Database** created in Assignment 1, which already contains the tables, relationships and constraints.

In this assignment I:
- Populated the tables using DML `INSERT` statements
- Wrote queries using **clauses & operators**, **sorting & grouping**, **aggregate functions**, **joins** and **window functions**
- Interpreted the results to answer business-style questions about employees, departments and locations

---

## 🧱 Database Schema

| Table | Column | Description |
|-------|--------|-------------|
| **Departments** | `department_id` (PK) | Unique department ID |
| | `department_name` | Name of the department |
| **Location** | `location_id` (PK) | Unique location ID |
| | `location_name` | Name of the location |
| **Employees** | `employee_id` (PK) | Unique employee ID |
| | `employee_name` | Employee's name |
| | `gender` | Gender |
| | `age` | Age |
| | `hire_date` | Date of joining |
| | `designation` | Job title |
| | `salary` | Salary (₹) |
| | `department_id` (FK) | References `Departments(department_id)` |
| | `location_id` (FK) | References `Location(location_id)` |

### Entity Relationship

```
Departments (1) ────< (Many) Employees (Many) >──── (1) Location
```

---

## 📂 Repository Structure

```
├── README.md                 # Project documentation
├── schema.sql                # DDL – table creation (from Assignment 1)
├── data_insertion.sql        # DML – inserting employee data
└── queries.sql               # All assignment queries
```

---

## 🛠️ Tasks & SQL Queries

### 1️⃣ Clauses & Operators

**DISTINCT – Unique salaries**
```sql
SELECT DISTINCT salary
FROM Employees;
```

**ALIAS – Rename columns**
```sql
SELECT age    AS Employee_Age,
       salary AS Employee_Salary
FROM Employees;
```

**WHERE – Salary > ₹50,000 and hired before 2016-01-01**
```sql
SELECT *
FROM Employees
WHERE salary > 50000
  AND hire_date < '2016-01-01';
```

**WHERE – Fill missing designation with "Data Scientist"**
```sql
-- Find the employee
SELECT * FROM Employees
WHERE designation IS NULL;

-- Update the missing value
UPDATE Employees
SET designation = 'Data Scientist'
WHERE designation IS NULL;
```

---

### 2️⃣ Sorting & Grouping Data

**ORDER BY – Department ascending, salary descending**
```sql
SELECT *
FROM Employees
ORDER BY department_id ASC, salary DESC;
```

**LIMIT – First 5 employees hired in 2018**
```sql
SELECT *
FROM Employees
WHERE YEAR(hire_date) = 2018
ORDER BY hire_date
LIMIT 5;
```

**AGGREGATE – Total salary in the Finance department**
```sql
SELECT SUM(e.salary) AS total_finance_salary
FROM Employees e
JOIN Departments d ON e.department_id = d.department_id
WHERE d.department_name = 'Finance';
```

**AGGREGATE – Minimum age**
```sql
SELECT MIN(age) AS minimum_age
FROM Employees;
```

**GROUP BY – Maximum salary per location**
```sql
SELECT l.location_name,
       MAX(e.salary) AS max_salary
FROM Employees e
JOIN Location l ON e.location_id = l.location_id
GROUP BY l.location_name;
```

**GROUP BY – Average salary for 'Analyst' designations**
```sql
SELECT designation,
       AVG(salary) AS avg_salary
FROM Employees
WHERE designation LIKE '%Analyst%'
GROUP BY designation;
```

**HAVING – Departments with fewer than 3 employees**
```sql
SELECT d.department_name,
       COUNT(e.employee_id) AS employee_count
FROM Departments d
JOIN Employees e ON d.department_id = e.department_id
GROUP BY d.department_name
HAVING COUNT(e.employee_id) < 3;
```

**HAVING – Locations where female employees' average age is below 30**
```sql
SELECT l.location_name,
       AVG(e.age) AS avg_female_age
FROM Employees e
JOIN Location l ON e.location_id = l.location_id
WHERE e.gender = 'Female'
GROUP BY l.location_name
HAVING AVG(e.age) < 30;
```

---

### 3️⃣ Joins

**INNER JOIN – Employees with their department**
```sql
SELECT e.employee_name,
       e.designation,
       d.department_name
FROM Employees e
INNER JOIN Departments d
        ON e.department_id = d.department_id;
```

**LEFT JOIN – Employee count per department (including empty ones)**
```sql
SELECT d.department_name,
       COUNT(e.employee_id) AS total_employees
FROM Departments d
LEFT JOIN Employees e
       ON d.department_id = e.department_id
GROUP BY d.department_name;
```

**RIGHT JOIN – All locations with assigned employees**
```sql
SELECT l.location_name,
       e.employee_name
FROM Employees e
RIGHT JOIN Location l
        ON e.location_id = l.location_id;
```

**CROSS JOIN – All department–location combinations**
```sql
SELECT d.department_name,
       l.location_name
FROM Departments d
CROSS JOIN Location l;
```

**SELF JOIN – Pairs of employees in the same department**
```sql
SELECT e1.employee_name AS employee_1,
       e2.employee_name AS employee_2,
       e1.department_id
FROM Employees e1
JOIN Employees e2
  ON e1.department_id = e2.department_id
 AND e1.employee_id < e2.employee_id;   -- excludes self-pairs and duplicate pairs
```

---

### 4️⃣ Window Functions

**RANK() – Rank employees by salary**
```sql
SELECT employee_name,
       salary,
       RANK() OVER (ORDER BY salary DESC) AS salary_rank
FROM Employees;
```

**DENSE_RANK() – Rank within each department**
```sql
SELECT employee_name,
       department_id,
       salary,
       DENSE_RANK() OVER (PARTITION BY department_id
                          ORDER BY salary DESC) AS dept_salary_rank
FROM Employees;
```

**Running total of salary by department**
```sql
SELECT employee_name,
       department_id,
       salary,
       SUM(salary) OVER (PARTITION BY department_id
                         ORDER BY employee_id) AS running_total_salary
FROM Employees;
```

---

## 💡 Key Learnings

- **DISTINCT** removes duplicate values; **aliases** make output readable.
- **WHERE** filters rows *before* grouping, while **HAVING** filters groups *after* aggregation.
- **IS NULL** (not `= NULL`) is the correct way to detect missing values.
- **INNER JOIN** returns only matching rows; **LEFT/RIGHT JOIN** keep unmatched rows from one side; **CROSS JOIN** returns every combination.
- In a **SELF JOIN**, using `e1.employee_id < e2.employee_id` removes both self-pairs and mirrored duplicates.
- **RANK()** leaves gaps after ties, while **DENSE_RANK()** does not.
- **PARTITION BY** in window functions calculates results per group without collapsing rows like `GROUP BY`.

---

## ▶️ How to Run

1. Install **MySQL Server** and **MySQL Workbench** (or any SQL client).
2. Run `schema.sql` to create the database and tables.
3. Run `data_insertion.sql` to populate the tables.
4. Run the queries in `queries.sql` one by one and review the output.

---

## 🧰 Tools Used

- MySQL 8.0
- MySQL Workbench
- Git & GitHub

---

## 👤 About Me

Aspiring **Data Analyst** building a portfolio of SQL, Excel, Python and data-visualisation projects.

📫 Connect with me: [LinkedIn](#) | [GitHub](#) | [Email](#)

⭐ If you found this project helpful, feel free to star the repository!
