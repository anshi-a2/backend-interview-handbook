# SQL Interview Queries - Complete Study Guide

# Table of Contents

1. Sample Dataset
2. Second Highest Salary
3. Nth Highest Salary
4. Duplicate Records
5. Running Total
6. Rank vs Dense Rank vs Row Number
7. SQL Joins
8. Most Asked Follow-Up Questions
9. Interview Cheat Sheet

---

# 1. Sample Dataset

We'll use the following Employee table throughout the guide.

```sql
CREATE TABLE Employee (
    emp_id INT,
    emp_name VARCHAR(100),
    department VARCHAR(50),
    salary INT
);
```

Sample Data

| emp_id | emp_name | department | salary |
|---------|----------|------------|---------|
| 1 | Anshi | Engineering | 100000 |
| 2 | John | Engineering | 120000 |
| 3 | Alex | HR | 90000 |
| 4 | Sarah | HR | 120000 |
| 5 | Mike | Sales | 80000 |
| 6 | David | Sales | 100000 |

---

# 2. Second Highest Salary

---

## Approach 1 (Most Common)

```sql
SELECT MAX(salary)
FROM Employee
WHERE salary < (
    SELECT MAX(salary)
    FROM Employee
);
```

---

### Dry Run

Highest Salary:

```text
120000
```

Query becomes:

```sql
SELECT MAX(salary)
FROM Employee
WHERE salary < 120000;
```

Result:

```text
100000
```

---

## Approach 2 (Using DISTINCT)

```sql
SELECT DISTINCT salary
FROM Employee
ORDER BY salary DESC
LIMIT 1 OFFSET 1;
```

MySQL

---

Result

```text
100000
```

---

## Approach 3 (Using Dense Rank)

```sql
SELECT salary
FROM (
    SELECT salary,
           DENSE_RANK() OVER(
                ORDER BY salary DESC
           ) AS rnk
    FROM Employee
) t
WHERE rnk = 2;
```

---

# Interview Follow-Up

What if highest salary appears multiple times?

```text
120000
120000
100000
```

Use:

```sql
DENSE_RANK()
```

not

```sql
ROW_NUMBER()
```

---

# 3. Nth Highest Salary

---

## Generic Solution

Find 3rd highest salary

```sql
SELECT salary
FROM (
    SELECT salary,
           DENSE_RANK() OVER(
               ORDER BY salary DESC
           ) AS rnk
    FROM Employee
) t
WHERE rnk = 3;
```

---

### Output

```text
90000
```

---

## Dynamic Version

```sql
WITH SalaryRank AS (
    SELECT salary,
           DENSE_RANK() OVER(
                ORDER BY salary DESC
           ) AS rnk
    FROM Employee
)
SELECT salary
FROM SalaryRank
WHERE rnk = N;
```

Replace:

```text
N = 2
N = 3
N = 4
```

---

## MySQL Alternative

```sql
SELECT DISTINCT salary
FROM Employee
ORDER BY salary DESC
LIMIT 1 OFFSET N-1;
```

Example

```text
N = 3
```

```sql
LIMIT 1 OFFSET 2
```

---

# 4. Duplicate Records

Consider:

```sql
CREATE TABLE Users(
    id INT,
    email VARCHAR(100)
);
```

Data

| id | email |
|----|---------|
| 1 | a@gmail.com |
| 2 | b@gmail.com |
| 3 | a@gmail.com |
| 4 | c@gmail.com |
| 5 | b@gmail.com |

---

## Find Duplicate Emails

```sql
SELECT email,
       COUNT(*) AS cnt
FROM Users
GROUP BY email
HAVING COUNT(*) > 1;
```

---

Result

```text
a@gmail.com
b@gmail.com
```

---

## Find Complete Duplicate Rows

```sql
SELECT *,
       COUNT(*)
FROM Users
GROUP BY id,email
HAVING COUNT(*) > 1;
```

---

## Delete Duplicate Records

Keep smallest ID

```sql
DELETE u1
FROM Users u1
JOIN Users u2
ON u1.email = u2.email
AND u1.id > u2.id;
```

---

# Interview Question

Difference between:

```sql
WHERE
```

and

```sql
HAVING
```

### WHERE

Filters rows before grouping.

### HAVING

Filters groups after aggregation.

---

# 5. Running Total

---

# Sample Sales Table

```sql
CREATE TABLE Sales(
    sale_date DATE,
    amount INT
);
```

Data

| sale_date | amount |
|------------|---------|
| 2025-01-01 | 100 |
| 2025-01-02 | 200 |
| 2025-01-03 | 300 |
| 2025-01-04 | 400 |

---

## Running Total Using Window Function

```sql
SELECT sale_date,
       amount,
       SUM(amount)
       OVER(
          ORDER BY sale_date
       ) AS running_total
FROM Sales;
```

---

Result

| Date | Amount | Running Total |
|--------|---------|---------------|
| 1 Jan | 100 | 100 |
| 2 Jan | 200 | 300 |
| 3 Jan | 300 | 600 |
| 4 Jan | 400 | 1000 |

---

## Partitioned Running Total

Department-wise running salary.

```sql
SELECT department,
       emp_name,
       salary,
       SUM(salary)
       OVER(
          PARTITION BY department
          ORDER BY emp_id
       ) AS running_salary
FROM Employee;
```

---

# Interview Insight

```sql
SUM() OVER()
```

does not collapse rows.

Unlike:

```sql
GROUP BY
```

---

# 6. Rank vs Dense Rank vs Row Number

Most asked SQL interview topic.

---

# Sample Data

| Name | Salary |
|--------|--------|
| John | 100 |
| Alex | 100 |
| Sarah | 90 |
| Mike | 80 |

---

## ROW_NUMBER()

```sql
SELECT name,
       salary,
       ROW_NUMBER()
       OVER(
          ORDER BY salary DESC
       ) AS rn
FROM Employee;
```

Result

| Salary | Row Number |
|----------|----------|
| 100 | 1 |
| 100 | 2 |
| 90 | 3 |
| 80 | 4 |

---

### Properties

```text
Always Unique
No Ties
```

---

## RANK()

```sql
SELECT name,
       salary,
       RANK()
       OVER(
           ORDER BY salary DESC
       ) AS rnk
FROM Employee;
```

Result

| Salary | Rank |
|----------|------|
| 100 | 1 |
| 100 | 1 |
| 90 | 3 |
| 80 | 4 |

Notice:

```text
Rank 2 skipped
```

---

## DENSE_RANK()

```sql
SELECT name,
       salary,
       DENSE_RANK()
       OVER(
          ORDER BY salary DESC
       ) AS drnk
FROM Employee;
```

Result

| Salary | Dense Rank |
|----------|------------|
| 100 | 1 |
| 100 | 1 |
| 90 | 2 |
| 80 | 3 |

No gaps.

---

## Interview Comparison

| Function | Duplicate Values | Gap |
|------------|------------|------|
| ROW_NUMBER | Unique Rank | No |
| RANK | Same Rank | Yes |
| DENSE_RANK | Same Rank | No |

---

## Which One For Nth Highest Salary?

Preferred:

```sql
DENSE_RANK()
```

Reason:

```text
Handles duplicates correctly.
```

---

# 7. SQL Joins

Assume:

Employee

| emp_id | emp_name | dept_id |
|---------|-----------|----------|
| 1 | Anshi | 10 |
| 2 | John | 20 |
| 3 | Alex | 30 |

Department

| dept_id | dept_name |
|----------|-----------|
| 10 | Engineering |
| 20 | HR |
| 40 | Finance |

---

# INNER JOIN

Returns matching rows only.

```sql
SELECT *
FROM Employee e
INNER JOIN Department d
ON e.dept_id = d.dept_id;
```

Result

```text
Anshi
John
```

Alex removed.

Finance removed.

---

# LEFT JOIN

Returns:

```text
All rows from Left Table
```

```sql
SELECT *
FROM Employee e
LEFT JOIN Department d
ON e.dept_id = d.dept_id;
```

Result

```text
Anshi
John
Alex(NULL)
```

---

# RIGHT JOIN

Returns:

```text
All rows from Right Table
```

```sql
SELECT *
FROM Employee e
RIGHT JOIN Department d
ON e.dept_id = d.dept_id;
```

Result

```text
Anshi
John
Finance(NULL)
```

---

# FULL OUTER JOIN

Returns:

```text
Matching
+
Left Unmatched
+
Right Unmatched
```

```sql
SELECT *
FROM Employee e
FULL OUTER JOIN Department d
ON e.dept_id = d.dept_id;
```

Result

```text
Anshi
John
Alex
Finance
```

---

# SELF JOIN

Employees and Managers

```sql
CREATE TABLE Employee(
   emp_id INT,
   emp_name VARCHAR(100),
   manager_id INT
);
```

---

Find Employee + Manager

```sql
SELECT e.emp_name,
       m.emp_name AS manager_name
FROM Employee e
LEFT JOIN Employee m
ON e.manager_id = m.emp_id;
```

---

# CROSS JOIN

Cartesian Product

```sql
SELECT *
FROM Employee
CROSS JOIN Department;
```

If:

```text
Employee = 3 rows
Department = 4 rows
```

Output:

```text
12 rows
```

---

# 8. Most Asked Follow-Up Questions

---

## Find Employees Earning More Than Their Manager

```sql
SELECT e.emp_name
FROM Employee e
JOIN Employee m
ON e.manager_id = m.emp_id
WHERE e.salary > m.salary;
```

---

## Find Department Wise Highest Salary

```sql
SELECT department,
       MAX(salary)
FROM Employee
GROUP BY department;
```

---

## Find Top 3 Salaries

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
WHERE rnk <= 3;
```

---

## Find Employees With No Department

```sql
SELECT *
FROM Employee e
LEFT JOIN Department d
ON e.dept_id = d.dept_id
WHERE d.dept_id IS NULL;
```

---

# 9. Interview Cheat Sheet

## Second Highest Salary

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

## Nth Highest Salary

```sql
DENSE_RANK()
```

---

## Duplicate Records

```sql
GROUP BY
HAVING COUNT(*) > 1
```

---

## Running Total

```sql
SUM()
OVER(
 ORDER BY
)
```

---

## Rank Functions

```text
ROW_NUMBER -> Unique Rank
RANK       -> Gaps Allowed
DENSE_RANK -> No Gaps
```

---

## Joins

```text
INNER -> Matching Only
LEFT  -> All Left Rows
RIGHT -> All Right Rows
FULL  -> Everything
SELF  -> Same Table
CROSS -> Cartesian Product
```

---

# Golden Interview Answer

> Window Functions (`ROW_NUMBER`, `RANK`, `DENSE_RANK`, `SUM OVER`) are generally the preferred modern SQL approach for solving ranking, running totals, and Nth highest salary problems because they are more readable, scalable, and optimizer-friendly than nested subqueries.
