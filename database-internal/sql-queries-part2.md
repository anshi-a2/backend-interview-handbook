# SQL Interview Queries - Part 2 (Advanced SQL)

# Table of Contents

1. Employee-Manager Queries
2. Consecutive Records
3. Gaps and Islands Problems
4. Latest Record Per Group
5. First and Last Record
6. Pivot and Unpivot
7. CTE (Common Table Expression)
8. Recursive CTE
9. Window Functions Deep Dive
10. Case Statements
11. EXISTS vs IN
12. DELETE vs TRUNCATE vs DROP
13. Union vs Union All
14. Stored Procedures vs Functions
15. Important SQL Interview Problems
16. Interview Cheat Sheet

---

# 1. Employee-Manager Queries

## Employee Earning More Than Manager

```sql
SELECT e.emp_id,
       e.emp_name,
       e.salary,
       m.emp_name AS manager_name,
       m.salary AS manager_salary
FROM Employee e
JOIN Employee m
ON e.manager_id = m.emp_id
WHERE e.salary > m.salary;
```

---

## Count Employees Under Each Manager

```sql
SELECT manager_id,
       COUNT(*) AS employee_count
FROM Employee
GROUP BY manager_id;
```

---

## Employees Without Managers

```sql
SELECT *
FROM Employee
WHERE manager_id IS NULL;
```

---

# 2. Consecutive Records

Sample Table

```sql
CREATE TABLE Logs (
    id INT,
    num INT
);
```

| id | num |
|----|----|
|1|1|
|2|1|
|3|1|
|4|2|
|5|3|

---

## Find Numbers Appearing 3 Consecutive Times

```sql
SELECT DISTINCT l1.num
FROM Logs l1
JOIN Logs l2
ON l1.id + 1 = l2.id
AND l1.num = l2.num
JOIN Logs l3
ON l2.id + 1 = l3.id
AND l2.num = l3.num;
```

---

Output

```text
1
```

---

# 3. Gaps and Islands Problem

Most Asked SQL Problem

---

Sample Data

| Date |
|--------|
|2025-01-01|
|2025-01-02|
|2025-01-03|
|2025-01-07|
|2025-01-08|

---

Goal

```text
Identify continuous date ranges.
```

---

Solution

```sql
SELECT date_value,
       ROW_NUMBER()
       OVER(ORDER BY date_value) AS rn
FROM Activity;
```

Create grouping key:

```sql
date_value - rn
```

Rows with same value belong to same island.

---

# 4. Latest Record Per Group

Orders

| order_id | customer_id | created_at |
|-----------|-------------|------------|
|1|101|2025-01-01|
|2|101|2025-02-01|
|3|102|2025-03-01|

---

## Latest Order Per Customer

```sql
SELECT *
FROM (
    SELECT *,
           ROW_NUMBER()
           OVER(
               PARTITION BY customer_id
               ORDER BY created_at DESC
           ) AS rn
    FROM Orders
) t
WHERE rn = 1;
```

---

# 5. First and Last Record

---

## First Order Per Customer

```sql
SELECT *
FROM (
    SELECT *,
           ROW_NUMBER()
           OVER(
              PARTITION BY customer_id
              ORDER BY created_at
           ) rn
    FROM Orders
) t
WHERE rn = 1;
```

---

## Last Order Per Customer

```sql
SELECT *
FROM (
    SELECT *,
           ROW_NUMBER()
           OVER(
              PARTITION BY customer_id
              ORDER BY created_at DESC
           ) rn
    FROM Orders
) t
WHERE rn = 1;
```

---

# 6. Pivot and Unpivot

Sample Data

| department | salary |
|------------|--------|
|Engineering|100|
|HR|90|
|Sales|80|

---

## Pivot

Convert rows into columns.

```sql
SELECT
SUM(CASE WHEN department='Engineering'
         THEN salary END) AS engineering,

SUM(CASE WHEN department='HR'
         THEN salary END) AS hr,

SUM(CASE WHEN department='Sales'
         THEN salary END) AS sales
FROM Employee;
```

---

Output

| engineering | hr | sales |
|-------------|----|------|
|100|90|80|

---

# 7. Common Table Expression (CTE)

CTE improves readability.

---

Without CTE

```sql
SELECT *
FROM (
    SELECT *
    FROM Employee
) t;
```

---

Using CTE

```sql
WITH EmployeeData AS (
    SELECT *
    FROM Employee
)
SELECT *
FROM EmployeeData;
```

---

Benefits

```text
Readable
Reusable
Modular
```

---

# 8. Recursive CTE

Used for:

```text
Tree Structure
Hierarchy
Org Charts
Category Trees
```

---

Employee Hierarchy

```sql
WITH RECURSIVE EmployeeTree AS (

    SELECT emp_id,
           emp_name,
           manager_id,
           1 AS level
    FROM Employee
    WHERE manager_id IS NULL

    UNION ALL

    SELECT e.emp_id,
           e.emp_name,
           e.manager_id,
           et.level + 1
    FROM Employee e
    JOIN EmployeeTree et
    ON e.manager_id = et.emp_id
)
SELECT *
FROM EmployeeTree;
```

---

# 9. Window Functions Deep Dive

---

## Running Total

```sql
SELECT amount,
       SUM(amount)
       OVER(
          ORDER BY transaction_date
       )
FROM Transactions;
```

---

## Moving Average

```sql
SELECT amount,
       AVG(amount)
       OVER(
          ORDER BY transaction_date
          ROWS BETWEEN 2 PRECEDING
          AND CURRENT ROW
       )
FROM Transactions;
```

---

## Previous Row (LAG)

```sql
SELECT amount,
       LAG(amount)
       OVER(
         ORDER BY transaction_date
       ) AS previous_amount
FROM Transactions;
```

---

## Next Row (LEAD)

```sql
SELECT amount,
       LEAD(amount)
       OVER(
          ORDER BY transaction_date
       ) AS next_amount
FROM Transactions;
```

---

# 10. CASE Statements

---

## Salary Classification

```sql
SELECT emp_name,
       salary,
       CASE
           WHEN salary >= 100000
           THEN 'HIGH'

           WHEN salary >= 50000
           THEN 'MEDIUM'

           ELSE 'LOW'
       END AS category
FROM Employee;
```

---

Output

```text
HIGH
MEDIUM
LOW
```

---

# 11. EXISTS vs IN

---

## IN

```sql
SELECT *
FROM Employee
WHERE dept_id IN (
    SELECT dept_id
    FROM Department
);
```

---

## EXISTS

```sql
SELECT *
FROM Employee e
WHERE EXISTS (
    SELECT 1
    FROM Department d
    WHERE d.dept_id = e.dept_id
);
```

---

## Interview Difference

### IN

```text
Good for small datasets.
```

---

### EXISTS

```text
Stops after first match.
Usually better for large datasets.
```

---

# 12. DELETE vs TRUNCATE vs DROP

| Feature | DELETE | TRUNCATE | DROP |
|----------|----------|----------|----------|
| Removes Rows | Yes | Yes | Yes |
| Removes Table | No | No | Yes |
| WHERE Allowed | Yes | No | No |
| Rollback | Yes* | Usually No | No |
| Faster | No | Yes | Fastest |

---

## DELETE

```sql
DELETE
FROM Employee
WHERE salary < 50000;
```

---

## TRUNCATE

```sql
TRUNCATE TABLE Employee;
```

---

## DROP

```sql
DROP TABLE Employee;
```

---

# 13. UNION vs UNION ALL

---

## UNION

Removes duplicates.

```sql
SELECT city
FROM Customers

UNION

SELECT city
FROM Suppliers;
```

---

## UNION ALL

Keeps duplicates.

```sql
SELECT city
FROM Customers

UNION ALL

SELECT city
FROM Suppliers;
```

---

## Interview Answer

```text
UNION performs DISTINCT.
UNION ALL does not.
UNION ALL is faster.
```

---

# 14. Stored Procedures vs Functions

| Feature | Procedure | Function |
|-----------|-----------|-----------|
| Returns Value | Optional | Mandatory |
| Used in SELECT | No | Yes |
| DML Allowed | Yes | Limited |
| Multiple Outputs | Yes | No |

---

## Procedure

```sql
CREATE PROCEDURE GetEmployees()
BEGIN
    SELECT *
    FROM Employee;
END;
```

---

## Function

```sql
CREATE FUNCTION GetBonus(
    salary INT
)
RETURNS INT
RETURN salary * 10 / 100;
```

---

Usage

```sql
SELECT GetBonus(100000);
```

---

# 15. Important SQL Interview Problems

---

## Find Duplicate Emails

```sql
SELECT email,
       COUNT(*)
FROM Users
GROUP BY email
HAVING COUNT(*) > 1;
```

---

## Find Second Highest Salary

```sql
SELECT MAX(salary)
FROM Employee
WHERE salary <
(
    SELECT MAX(salary)
    FROM Employee
);
```

---

## Find Nth Highest Salary

```sql
SELECT salary
FROM (
    SELECT salary,
           DENSE_RANK()
           OVER(
              ORDER BY salary DESC
           ) rnk
    FROM Employee
) t
WHERE rnk = N;
```

---

## Department Wise Highest Salary

```sql
SELECT department,
       MAX(salary)
FROM Employee
GROUP BY department;
```

---

## Top 3 Salaries

```sql
SELECT *
FROM (
    SELECT salary,
           DENSE_RANK()
           OVER(
              ORDER BY salary DESC
           ) rnk
    FROM Employee
) t
WHERE rnk <= 3;
```

---

## Find Employees Joined This Month

```sql
SELECT *
FROM Employee
WHERE MONTH(join_date) = MONTH(CURRENT_DATE)
AND YEAR(join_date) = YEAR(CURRENT_DATE);
```

---

## Count Employees Per Department

```sql
SELECT department,
       COUNT(*)
FROM Employee
GROUP BY department;
```

---

# 16. Interview Cheat Sheet

## Hierarchy Problems

```sql
Recursive CTE
```

---

## Latest Record Per Group

```sql
ROW_NUMBER()
PARTITION BY
ORDER BY DESC
```

---

## Running Total

```sql
SUM() OVER()
```

---

## Previous Row

```sql
LAG()
```

---

## Next Row

```sql
LEAD()
```

---

## Ranking

```text
ROW_NUMBER
RANK
DENSE_RANK
```

---

## Duplicate Records

```sql
GROUP BY
HAVING COUNT(*) > 1
```

---

## Large Dataset Existence Check

```sql
EXISTS
```

---

## Row to Column Conversion

```sql
PIVOT
or
CASE + GROUP BY
```

---

# Golden Interview Statement

> Modern SQL interviews heavily focus on Window Functions (`ROW_NUMBER`, `RANK`, `DENSE_RANK`, `LAG`, `LEAD`, `SUM OVER`), Recursive CTEs, latest-record-per-group problems, ranking problems, and join optimization because these topics demonstrate both SQL proficiency and database internals understanding.
