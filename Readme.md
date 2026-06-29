# PostgreSQL & DBMS — Complete Study Notes

> Organized by topic · Easy to learn & revise · GitHub ready

---

## Table of Contents

1. [Data Types](#1-data-types)
2. [CREATE TABLE & Constraints](#2-create-table--constraints)
3. [ALTER TABLE](#3-alter-table)
4. [INSERT](#4-insert)
5. [SELECT & Filtering](#5-select--filtering)
6. [String & Scalar Functions](#6-string--scalar-functions)
7. [Aggregate Functions](#7-aggregate-functions)
8. [NULL Handling](#8-null-handling)
9. [LIMIT & OFFSET — Pagination](#9-limit--offset--pagination)
10. [UPDATE & DELETE](#10-update--delete)
11. [GROUP BY & HAVING](#11-group-by--having)
12. [Foreign Keys & Relationships](#12-foreign-keys--relationships)
13. [JOINs](#13-joins)
14. [Subqueries](#14-subqueries)
15. [EXTRACT — Date Functions](#15-extract--date-functions)
16. [Indexes](#16-indexes)
17. [Functions](#17-functions)
18. [Stored Procedures](#18-stored-procedures)
19. [Triggers](#19-triggers)
20. [Transactions & ACID](#20-transactions--acid)
21. [Views](#21-views)
22. [Quick Reference Cheatsheet](#22-quick-reference-cheatsheet)
23. [Real Project — Football Ticket Booking System](#23-real-project--football-ticket-booking-system)
24. [Theory Questions & Answers](#24-theory-questions--answers)

---

## 1. Data Types

| Type | Use Case |
|------|----------|
| `SERIAL` | Auto-increment integer (PK) |
| `INT` / `SMALLINT` | Whole numbers |
| `DECIMAL(p, s)` | Exact decimal — money, prices |
| `VARCHAR(n)` | Variable-length string, max n chars |
| `TEXT` | Unlimited-length string |
| `CHAR(n)` | Fixed-length string |
| `BOOLEAN` | `true` / `false` |
| `DATE` | `YYYY-MM-DD` |
| `TIMESTAMP` | Date + time |

---

## 2. CREATE TABLE & Constraints

```sql
CREATE TABLE users (
  id        SERIAL        PRIMARY KEY,
  username  VARCHAR(50)   NOT NULL UNIQUE,
  email     VARCHAR(100)  UNIQUE,
  age       SMALLINT      CHECK (age >= 18),
  is_active BOOLEAN       DEFAULT true
);
```

### Constraint Summary

| Constraint | Description |
|------------|-------------|
| `PRIMARY KEY` | Unique + NOT NULL. One per table. Auto-indexed. |
| `NOT NULL` | Column must always have a value. |
| `UNIQUE` | All values must be different (NULL allowed). |
| `CHECK (expr)` | Value must pass the condition. |
| `DEFAULT value` | Used when no value is given on INSERT. |
| `REFERENCES` | Foreign key — links to another table's PK. |

### Composite / Table-level Constraints
```sql
-- Composite primary key (both columns together must be unique)
PRIMARY KEY (id, email)

-- Table-level unique
UNIQUE (username)
```

### Drop a Table
```sql
DROP TABLE users;
```

### ✏️ Practice

1. Create a `products` table with: `id` (PK), `name` (NOT NULL), `price` (DECIMAL, must be > 0), `in_stock` (BOOLEAN, default true).
2. Create a `courses` table where `course_title` is NOT NULL and `price` has a CHECK that it cannot be negative.
3. What is the difference between `PRIMARY KEY` and `UNIQUE`? Can a UNIQUE column have NULLs?

---

## 3. ALTER TABLE

Use `ALTER TABLE` to modify an existing table's structure without dropping it.

```sql
-- Rename table
ALTER TABLE person RENAME TO users;

-- Add column
ALTER TABLE users ADD COLUMN email VARCHAR(100);

-- Drop column
ALTER TABLE users DROP COLUMN email;

-- Rename column
ALTER TABLE users RENAME COLUMN username TO user_name;

-- Change column type
ALTER TABLE users ALTER COLUMN email TYPE VARCHAR(200);

-- Add / drop NOT NULL
ALTER TABLE users ALTER COLUMN email SET NOT NULL;
ALTER TABLE users ALTER COLUMN email DROP NOT NULL;

-- Set / drop DEFAULT
ALTER TABLE users ALTER COLUMN email SET DEFAULT 'none@email.com';
ALTER TABLE users ALTER COLUMN email DROP DEFAULT;

-- Add named constraint
ALTER TABLE users ADD CONSTRAINT unique_email UNIQUE (email);
ALTER TABLE users ADD CONSTRAINT pk_user_id  PRIMARY KEY (id);

-- Drop named constraint
ALTER TABLE users DROP CONSTRAINT unique_email;
```

### ✏️ Practice

1. Add a `phone VARCHAR(20)` column to an existing `students` table.
2. Rename the column `phone` to `mobile_number`.
3. Add a table-level constraint to ensure `email` is unique in the `students` table.

---

## 4. INSERT

```sql
-- Single row
INSERT INTO users (username, age)
VALUES ('rafi', 24);

-- Multiple rows at once
INSERT INTO users (username, age)
VALUES
  ('jami', 35),
  ('asir', 26),
  ('gool', 29);
```

> Columns with `DEFAULT` or `SERIAL` can be omitted — they fill automatically.

### ✏️ Practice

1. Insert 3 students into a `students` table with `first_name`, `last_name`, `email`, `age`.
2. Insert a record without providing the `email` column (assuming it allows NULL).
3. What happens if you try to insert a duplicate value into a `UNIQUE` column?

---

## 5. SELECT & Filtering

### Basic SELECT
```sql
SELECT * FROM students;
SELECT first_name, age FROM students;

-- Column alias
SELECT first_name AS "First Name", email AS user_email FROM students;
```

### ORDER BY
```sql
SELECT * FROM students ORDER BY age ASC;    -- ascending (default)
SELECT * FROM students ORDER BY age DESC;   -- descending
SELECT * FROM students ORDER BY grade ASC, age DESC;  -- multiple columns
```

### DISTINCT
```sql
SELECT DISTINCT country FROM students;
SELECT DISTINCT course FROM students;
```

### WHERE — Filtering Rows

```sql
-- Equality
SELECT * FROM students WHERE country = 'USA';

-- AND / OR
SELECT * FROM students WHERE country = 'USA' OR country = 'Germany';
SELECT * FROM students WHERE grade = 'A' AND country = 'USA';

-- Comparison:  =  !=  <>  <  >  <=  >=
SELECT * FROM students WHERE age <= 18;
SELECT * FROM students WHERE country <> 'USA';   -- <> is same as !=

-- BETWEEN (inclusive on both ends)
SELECT * FROM students WHERE age BETWEEN 22 AND 24;

-- IN (shortcut for multiple OR conditions)
SELECT * FROM students WHERE grade IN ('A', 'A-', 'B+');

-- NOT IN
SELECT * FROM students WHERE country NOT IN ('USA', 'Canada');

-- NOT
SELECT * FROM students WHERE NOT country = 'USA';
```

### LIKE — Pattern Matching

| Pattern | Meaning |
|---------|---------|
| `'M%'` | Starts with M |
| `'%l'` | Ends with l |
| `'%ar%'` | Contains "ar" anywhere |
| `'P___'` | P followed by exactly 3 chars |
| `'%u_'` | "u" as second-to-last character |

```sql
SELECT * FROM students WHERE first_name LIKE 'M%';
SELECT * FROM students WHERE first_name LIKE '%u_';
SELECT * FROM students WHERE first_name ILIKE 'm%';  -- case-insensitive
```

### ✏️ Practice

1. Find all students from 'Bangladesh' or 'Pakistan', ordered by age descending.
2. Find all courses where the title contains the word 'Python'.
3. Find all students whose `first_name` starts with 'A' and age is between 20 and 25.

---

## 6. String & Scalar Functions

```sql
-- Case conversion
SELECT UPPER(first_name) FROM students;
SELECT LOWER(last_name)  FROM students;

-- Concatenation
SELECT CONCAT(first_name, ' ', last_name) AS full_name FROM students;

-- String length
SELECT LENGTH(username) FROM users;

-- Trim whitespace
SELECT TRIM(username) FROM users;

-- Substring
SELECT SUBSTRING(email FROM 1 FOR 5) FROM students;  -- first 5 chars

-- Replace
SELECT REPLACE(email, '@gmail.com', '') FROM students;
```

### ✏️ Practice

1. Show each student's full name (first + last) in UPPERCASE.
2. Find all students whose email length is greater than 20 characters.
3. Display the first 3 characters of every student's `first_name`.

---

## 7. Aggregate Functions

Aggregate functions collapse **many rows → one value**.

```sql
SELECT COUNT(*)   FROM students;          -- total rows
SELECT COUNT(age) FROM students;          -- rows where age is NOT NULL
SELECT AVG(age)   FROM students;          -- average
SELECT MAX(age)   FROM students;          -- maximum
SELECT MIN(age)   FROM students;          -- minimum
SELECT SUM(salary) FROM employees;        -- total

-- Round the result
SELECT ROUND(AVG(salary), 2) FROM employees;
```

> `COUNT(*)` counts all rows including NULLs.  
> `COUNT(column)` counts only non-NULL values in that column.

### ✏️ Practice

1. Find the average price of all courses.
2. Find the highest and lowest salary among all employees.
3. Count how many students have provided their phone number (phone IS NOT NULL).

---

## 8. NULL Handling

> `NULL` means "unknown" — it is NOT zero, NOT an empty string.  
> You **cannot** compare NULL with `=` or `!=`. Use `IS NULL` / `IS NOT NULL`.

```sql
-- ❌ This will NOT work
SELECT * FROM students WHERE email = NULL;

-- ✅ Correct
SELECT * FROM students WHERE email IS NULL;
SELECT * FROM students WHERE email IS NOT NULL;

-- COALESCE — returns first non-NULL value in the list
SELECT first_name, COALESCE(phone, 'Not Provided') AS phone FROM students;
SELECT first_name, COALESCE(email, phone, 'No Contact') FROM students;
```

### ✏️ Practice

1. Display all students. If their `phone` is NULL, show `'Not Provided'` using COALESCE.
2. Delete all enrollment records where `progress_percentage` IS NULL.
3. Count how many employees have no email address recorded.

---

## 9. LIMIT & OFFSET — Pagination

`LIMIT` = how many rows to return.  
`OFFSET` = how many rows to skip (start from).

```sql
SELECT * FROM students LIMIT 5;             -- first 5 rows
SELECT * FROM students LIMIT 5 OFFSET 5;   -- skip 5, return next 5

-- Pagination formula:
-- Page 1: OFFSET = 5 * 0 = 0
-- Page 2: OFFSET = 5 * 1 = 5
-- Page 3: OFFSET = 5 * 2 = 10
SELECT * FROM students ORDER BY id LIMIT 5 OFFSET 5 * 1;
```

> Always use `ORDER BY` with pagination — otherwise the order of results is unpredictable.

### ✏️ Practice

1. Show the top 3 most expensive courses (order by price descending, limit 3).
2. Show page 2 of a student list, with 4 students per page.
3. Get the 5th and 6th most recently hired employees.

---

## 10. UPDATE & DELETE

```sql
-- Update one field
UPDATE students SET email = 'default@email.com' WHERE email IS NULL;

-- Update multiple fields at once
UPDATE students
SET grade = 'B+', blood_group = 'O-'
WHERE id IN (1, 2, 3);

-- Increase price by 10% for a category
UPDATE courses SET price = price * 1.10 WHERE category = 'Programming';

-- Delete one row
DELETE FROM students WHERE id = 9;

-- Delete multiple rows
DELETE FROM students WHERE id IN (9, 11);

-- Delete with condition
DELETE FROM students WHERE age > 20 AND grade = 'B-';

-- ⚠️ Delete ALL rows (keeps the table structure)
DELETE FROM students;

-- ⚡ TRUNCATE — faster way to clear all rows
TRUNCATE TABLE students;
```

> **Difference:** `DELETE` logs each row removal (can use WHERE). `TRUNCATE` removes all rows instantly, no conditions.

### ✏️ Practice

1. Increase the salary of all employees in the 'HR' department by 15%.
2. Delete all students who enrolled before '2023-02-01'.
3. Update all courses with NULL price to set the price to 0.00.

---

## 11. GROUP BY & HAVING

`GROUP BY` groups rows with the same value so aggregates can be applied per group.  
`HAVING` filters those groups (like `WHERE` but runs *after* grouping).

```sql
-- Count per group
SELECT country, COUNT(*) AS total FROM students GROUP BY country;

-- Average per group
SELECT country, AVG(age) AS avg_age FROM students GROUP BY country;

-- Multiple aggregates at once
SELECT department, COUNT(*) AS emp_count, ROUND(AVG(salary)) AS avg_salary
FROM employees
GROUP BY department;

-- HAVING: only groups meeting a condition
SELECT country, AVG(age) FROM students
GROUP BY country
HAVING AVG(age) > 22;

SELECT country, COUNT(*) FROM students
GROUP BY country
HAVING COUNT(*) > 5;
```

### WHERE vs HAVING

| | WHERE | HAVING |
|-|-------|--------|
| Filters | Individual rows | Groups (after GROUP BY) |
| Used with aggregates? | ❌ No | ✅ Yes |
| Runs | Before grouping | After grouping |

```sql
-- WHERE filters rows BEFORE grouping
-- HAVING filters groups AFTER grouping
SELECT department, AVG(salary)
FROM employees
WHERE salary > 50000          -- filter individual rows first
GROUP BY department
HAVING AVG(salary) > 80000;  -- then filter groups
```

### ✏️ Practice

1. Find the total `paid_amount` per course category.
2. Show only those course categories where the average price is greater than $60.
3. Find departments where more than 5 employees were hired after 2020.

---

## 12. Foreign Keys & Relationships

A **Foreign Key (FK)** in one table points to the **Primary Key** in another, enforcing referential integrity — you can't insert a value that doesn't exist in the referenced table.

```sql
CREATE TABLE users (
  id       SERIAL PRIMARY KEY,
  username VARCHAR(25) NOT NULL
);

CREATE TABLE posts (
  id      SERIAL PRIMARY KEY,
  title   TEXT    NOT NULL,
  user_id INT     REFERENCES users(id)  -- FK
);
```

### ON DELETE Behavior (optional but important)

```sql
-- If the user is deleted, also delete their posts
user_id INT REFERENCES users(id) ON DELETE CASCADE;

-- If the user is deleted, set user_id to NULL in posts
user_id INT REFERENCES users(id) ON DELETE SET NULL;

-- (default) Reject deletion if referenced rows exist
user_id INT REFERENCES users(id) ON DELETE RESTRICT;
```

### Relationship Types (DBMS Theory)

| Type | Example |
|------|---------|
| One-to-One | One user → one profile |
| One-to-Many | One user → many posts |
| Many-to-Many | Students ↔ Courses (needs a junction table) |

### ✏️ Practice

1. What happens if you try to insert an enrollment with a `student_id` that doesn't exist in `students`?
2. Create an `orders` table with a FK referencing a `customers` table using `ON DELETE CASCADE`.
3. Design a Many-to-Many relationship between `students` and `courses`.

---

## 13. JOINs

JOINs combine rows from two or more tables based on a related column.

```sql
-- Sample tables used in examples:
-- users(id, username)
-- posts(id, title, user_id)
```

### INNER JOIN — only matching rows from BOTH tables
```sql
SELECT p.id, p.title, u.username
FROM posts AS p
INNER JOIN users AS u ON p.user_id = u.id;

-- USING shortcut (when column name is same in both tables)
SELECT * FROM employees
INNER JOIN departments USING (department_id);
```

### LEFT JOIN — all left table rows + matched right rows (NULL if no match)
```sql
SELECT p.id, p.title, u.username
FROM posts AS p
LEFT JOIN users AS u ON p.user_id = u.id;
-- Posts with no user → username will be NULL
```

### Find rows with NO match (anti-join pattern)
```sql
-- Students who have NOT enrolled in any course
SELECT s.first_name FROM students AS s
LEFT JOIN enrollments AS e ON s.student_id = e.student_id
WHERE e.student_id IS NULL;
```

### RIGHT JOIN — all right table rows + matched left rows
```sql
SELECT p.id, p.title, u.username
FROM posts AS p
RIGHT JOIN users AS u ON p.user_id = u.id;
-- Users with no posts → title will be NULL
```

### FULL JOIN — all rows from both, NULLs where no match
```sql
SELECT p.id, p.title, u.username
FROM posts AS p
FULL JOIN users AS u ON p.user_id = u.id;
```

### CROSS JOIN — every row × every row (n × m rows)
```sql
SELECT * FROM employees CROSS JOIN departments;
-- 4 employees × 3 departments = 12 rows
```

### Multi-table JOIN
```sql
-- 3-table join: enrollments + students + courses
SELECT
  CONCAT(s.first_name, ' ', s.last_name) AS student_name,
  c.course_title,
  e.paid_amount
FROM enrollments AS e
INNER JOIN students AS s ON e.student_id = s.student_id
INNER JOIN courses  AS c ON e.course_id  = c.course_id;
```

### Visual Summary

| Join Type | Left no match | Right no match |
|-----------|--------------|----------------|
| INNER | excluded | excluded |
| LEFT | included (NULL right) | excluded |
| RIGHT | excluded | included (NULL left) |
| FULL | included (NULL right) | included (NULL left) |

### ✏️ Practice

1. Show all courses and the number of students enrolled in each (include courses with 0 enrollments).
2. Find all students who have NOT enrolled in any course.
3. Show each department's name with the average salary of its employees, ordered highest to lowest.

---

## 14. Subqueries

A **subquery** is a query nested inside another query. The inner query runs first and its result is used by the outer query.

```sql
-- Scalar subquery (returns a single value)
SELECT * FROM employees
WHERE salary = (SELECT MAX(salary) FROM employees);

-- Compare to aggregate (employees earning above average)
SELECT * FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);

-- Highest salary in a specific department
SELECT * FROM employees
WHERE salary = (
  SELECT MAX(salary) FROM employees WHERE department = 'HR'
);

-- IN subquery (returns a list)
SELECT * FROM posts
WHERE user_id IN (SELECT id FROM users WHERE username = 'Rafi');

-- Subquery in FROM (derived table / inline view)
SELECT department, avg_sal
FROM (
  SELECT department, AVG(salary) AS avg_sal
  FROM employees
  GROUP BY department
) AS dept_avg
WHERE avg_sal > 80000;
```

### Subquery vs JOIN — when to use which?

| Use Subquery | Use JOIN |
|-------------|----------|
| Single value comparison (max, avg) | Need columns from both tables |
| Existence check (IN / NOT IN) | Better performance on large datasets |
| Derived/temporary result sets | Multiple aggregations needed |

### ✏️ Practice

1. Find the employee(s) with the highest salary in the 'Engineering' department.
2. Find all courses whose price is above the average course price.
3. Find students who are enrolled in at least one course using a subquery with `IN`.

---

## 15. EXTRACT — Date Functions

`EXTRACT` pulls out a specific part (year, month, day, etc.) from a date.

```sql
-- Syntax: EXTRACT(part FROM date_column)
SELECT EXTRACT(YEAR  FROM hire_date) FROM employees;
SELECT EXTRACT(MONTH FROM hire_date) FROM employees;
SELECT EXTRACT(DAY   FROM hire_date) FROM employees;

-- Cast a string to date
SELECT EXTRACT(YEAR FROM '2020-01-20'::DATE);

-- Count employees hired per year
SELECT EXTRACT(YEAR FROM hire_date) AS hire_year, COUNT(*) AS total
FROM employees
GROUP BY hire_year
ORDER BY hire_year;

-- Count enrollments per month
SELECT EXTRACT(MONTH FROM enrollment_date) AS month, COUNT(*)
FROM enrollments
GROUP BY month
ORDER BY month;
```

### Other Useful Date Functions

```sql
SELECT NOW();                          -- current date and time
SELECT CURRENT_DATE;                   -- today's date
SELECT AGE(dob) FROM students;         -- age from date of birth
SELECT DATE_PART('year', hire_date) FROM employees;  -- same as EXTRACT
```

### ✏️ Practice

1. Count how many employees were hired each year.
2. Find all enrollments that happened in the month of June (month = 6).
3. Find employees who have been with the company for more than 3 years.

---

## 16. Indexes

An **index** is a data structure that speeds up `SELECT` queries — at the cost of slightly slower writes and extra disk space.

```sql
-- Create index on a column (good for columns used in WHERE often)
CREATE INDEX idx_students_country ON students(country);

-- Unique index (also enforces uniqueness like UNIQUE constraint)
CREATE UNIQUE INDEX idx_users_email ON users(email);

-- Multi-column index
CREATE INDEX idx_emp_dept_salary ON employees(department, salary);

-- Drop index
DROP INDEX idx_students_country;

-- Check existing indexes on a table
SELECT indexname, indexdef FROM pg_indexes WHERE tablename = 'employees';

-- Analyze query performance (EXPLAIN ANALYZE)
EXPLAIN ANALYZE SELECT * FROM employees WHERE id = 3;
```

> PostgreSQL **automatically** creates indexes for `PRIMARY KEY` and `UNIQUE` columns.  
> Use `EXPLAIN ANALYZE` before and after adding an index to see the performance difference.

### When to use Indexes

| ✅ Good for | ❌ Avoid on |
|------------|------------|
| Columns in WHERE / JOIN | Small tables |
| Columns in ORDER BY | Columns rarely queried |
| Foreign key columns | Columns with very few distinct values |

### ✏️ Practice

1. Create an index on the `email` column of the `students` table. Use `EXPLAIN ANALYZE` to test the difference.
2. Create a multi-column index on `(department, salary)` in the `employees` table.
3. When would adding an index actually *hurt* performance?

---

## 17. Functions

A **function** is reusable SQL logic that returns a value. PostgreSQL supports two languages: `SQL` (simple) and `PL/pgSQL` (procedural, with variables/loops).

```sql
-- Simple function: return a scalar value
CREATE FUNCTION emp_count()
RETURNS INT
LANGUAGE SQL
AS $$
  SELECT COUNT(*) FROM employees;
$$;

-- Call the function
SELECT emp_count();

-- Function with parameter
CREATE FUNCTION get_emp_by_dept(dept_name VARCHAR)
RETURNS TABLE(id INT, name VARCHAR, salary DECIMAL)
LANGUAGE SQL
AS $$
  SELECT id, name, salary FROM employees WHERE department = dept_name;
$$;

SELECT * FROM get_emp_by_dept('Engineering');

-- Function that performs DML (delete)
CREATE FUNCTION delete_emp_by_id(emp_id INT)
RETURNS VOID
LANGUAGE SQL
AS $$
  DELETE FROM employees WHERE id = emp_id;
$$;

SELECT delete_emp_by_id(5);

-- Replace/update an existing function
CREATE OR REPLACE FUNCTION emp_count()
RETURNS INT LANGUAGE SQL
AS $$ SELECT COUNT(*) FROM employees; $$;
```

### ✏️ Practice

1. Create a function `get_avg_salary(dept VARCHAR)` that returns the average salary for a given department.
2. Create a function that returns the total number of enrollments for a given `course_id`.
3. What is the difference between a function and a stored procedure?

---

## 18. Stored Procedures

A **procedure** is like a function but:
- Does **not** return a value (use `RETURNS VOID` or omit).
- Called with `CALL` instead of `SELECT`.
- Can contain transaction control (`COMMIT`, `ROLLBACK`).
- Uses `PL/pgSQL` for complex logic.

```sql
-- Simple procedure: delete by ID
CREATE PROCEDURE delete_emp_by_id(emp_id INT)
LANGUAGE plpgsql
AS $$
BEGIN
  DELETE FROM employees WHERE id = emp_id;
END;
$$;

CALL delete_emp_by_id(6);

-- Procedure with variables and logic
CREATE OR REPLACE PROCEDURE increase_low_salary(dept_name VARCHAR)
LANGUAGE plpgsql
AS $$
DECLARE
  avg_sal DECIMAL(10, 2);
BEGIN
  -- Step 1: Calculate average salary into a variable
  SELECT AVG(salary) INTO avg_sal
  FROM employees
  WHERE department = dept_name;

  -- Step 2: Increase only those below average
  UPDATE employees
  SET salary = salary * 1.10
  WHERE department = dept_name
    AND salary < avg_sal;
END;
$$;

CALL increase_low_salary('IT');
```

### Function vs Procedure

| | Function | Procedure |
|-|----------|-----------|
| Returns a value | ✅ Yes | ❌ No (VOID) |
| Called with | `SELECT func()` | `CALL proc()` |
| Transaction control | ❌ No | ✅ Yes |
| Used in SQL query | ✅ Yes | ❌ No |

### ✏️ Practice

1. Write a procedure `reset_progress(course_id INT)` that sets `progress_percentage = 0` for all enrollments of that course.
2. Write a procedure that deletes all enrollments for a student when given their `student_id`.
3. What happens if an error occurs inside a procedure — does it auto-rollback?

---

## 19. Triggers

A **trigger** automatically runs a function BEFORE or AFTER an event (`INSERT`, `UPDATE`, `DELETE`) on a table.

**Use cases:** audit logs, data validation, auto-updating timestamps.

### Setup: Trigger = Log Table + Trigger Function + Trigger

```sql
-- Step 1: Create a log table
CREATE TABLE employees_log (
  id          SERIAL PRIMARY KEY,
  emp_id      INT,
  action      VARCHAR(25),
  action_time TIMESTAMP DEFAULT NOW()
);

-- Step 2: Create the trigger function (must return TRIGGER)
CREATE OR REPLACE FUNCTION log_employee_deletion()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
  INSERT INTO employees_log (emp_id, action, action_time)
  VALUES (OLD.id, 'DELETE', NOW());
  RETURN OLD;  -- required for AFTER DELETE triggers
END;
$$;

-- Step 3: Attach the trigger to the table
CREATE TRIGGER save_delete_logs
AFTER DELETE ON employees
FOR EACH ROW
EXECUTE FUNCTION log_employee_deletion();

-- Now any delete on employees auto-logs to employees_log
DELETE FROM employees WHERE id = 10;
SELECT * FROM employees_log;
```

### Trigger Timing & Events

| Timing | Event | `OLD` available | `NEW` available |
|--------|-------|-----------------|-----------------|
| BEFORE / AFTER | INSERT | ❌ | ✅ |
| BEFORE / AFTER | UPDATE | ✅ | ✅ |
| BEFORE / AFTER | DELETE | ✅ | ❌ |

- `OLD` = the row **before** the change (UPDATE/DELETE).
- `NEW` = the row **after** the change (INSERT/UPDATE).

```sql
-- Example: BEFORE INSERT trigger (auto-capitalize name)
CREATE OR REPLACE FUNCTION capitalize_name()
RETURNS TRIGGER LANGUAGE plpgsql AS $$
BEGIN
  NEW.name = UPPER(NEW.name);
  RETURN NEW;
END;
$$;

CREATE TRIGGER trg_capitalize
BEFORE INSERT ON employees
FOR EACH ROW EXECUTE FUNCTION capitalize_name();
```

### ✏️ Practice

1. Create a trigger that logs every UPDATE on the `employees` table (log old salary and new salary).
2. Create a trigger that prevents inserting an employee with salary < 0 (raise an error in BEFORE INSERT).
3. What is the difference between `BEFORE` and `AFTER` triggers?

---

## 20. Transactions & ACID

A **transaction** groups multiple SQL operations into one unit — either **all succeed** or **all fail**. This prevents partial updates (e.g., money deducted but not credited).

```sql
BEGIN;
  UPDATE accounts SET balance = balance - 500 WHERE id = 1;
  UPDATE accounts SET balance = balance + 500 WHERE id = 2;
COMMIT;     -- ✅ Save all changes

-- If something goes wrong:
ROLLBACK;   -- ❌ Undo everything since BEGIN
```

### SAVEPOINT — partial rollback

```sql
BEGIN;
  INSERT INTO orders VALUES (...);
  SAVEPOINT my_save;

  INSERT INTO payments VALUES (...);  -- if this fails:
  ROLLBACK TO SAVEPOINT my_save;     -- undo only from savepoint

COMMIT;
```

### ACID Properties

| Property | Meaning | Example |
|----------|---------|---------|
| **A**tomicity | All or nothing | Transfer either fully completes or fully fails |
| **C**onsistency | DB stays valid | Balance can't go negative if constraint exists |
| **I**solation | Concurrent transactions don't interfere | Two users can't read each other's uncommitted data |
| **D**urability | Committed data survives crashes | Written to disk after COMMIT |

### ✏️ Practice

1. Write a transaction that transfers $200 from account A to account B, with a ROLLBACK if any step fails.
2. Use a SAVEPOINT to insert two records — rollback only the second if it fails.
3. Explain: why is Atomicity important in a banking system?

---

## 21. Views

A **view** is a saved SELECT query that acts like a virtual table. The data is not stored — it's computed each time you query the view.

```sql
-- Create a view
CREATE VIEW active_students AS
SELECT id, first_name, last_name, email
FROM students
WHERE is_active = true;

-- Query the view like a normal table
SELECT * FROM active_students;
SELECT first_name FROM active_students WHERE country = 'Bangladesh';

-- Update a view definition
CREATE OR REPLACE VIEW active_students AS
SELECT id, first_name, last_name, email, country
FROM students
WHERE is_active = true;

-- Drop the view
DROP VIEW active_students;
```

### Why use Views?

| Benefit | Description |
|---------|-------------|
| Simplify complex queries | Hide JOINs behind a simple name |
| Security | Expose only certain columns to users |
| Reusability | Define once, query many times |
| Consistency | One source of truth for a report |

### ✏️ Practice

1. Create a view `enrolled_students` that shows `student_name`, `course_title`, and `paid_amount` using a JOIN of 3 tables.
2. Create a view `high_salary_employees` showing employees earning above the company average salary.
3. What is the difference between a VIEW and a subquery in FROM?

---

## 22. Quick Reference Cheatsheet

### DDL — Structure
```sql
CREATE TABLE ...
ALTER TABLE ... ADD COLUMN / DROP COLUMN / RENAME COLUMN / ALTER COLUMN
DROP TABLE ...
```

### DML — Data
```sql
INSERT INTO ... VALUES ...
UPDATE ... SET ... WHERE ...
DELETE FROM ... WHERE ...
TRUNCATE TABLE ...
```

### DQL — Query
```sql
SELECT ... FROM ... JOIN ... ON ...
WHERE ...
GROUP BY ...
HAVING ...
ORDER BY ...
LIMIT ... OFFSET ...
```

### SELECT Clause Order (must follow this order)
```
SELECT → FROM → JOIN → WHERE → GROUP BY → HAVING → ORDER BY → LIMIT/OFFSET
```

### Filtering Operators
```sql
WHERE col = 'value'         -- equality
WHERE col != 'value'        -- not equal (also: <>)
WHERE col IN ('a', 'b')     -- in list
WHERE col NOT IN ('a')      -- not in list
WHERE col BETWEEN 10 AND 20 -- range (inclusive)
WHERE col LIKE 'A%'         -- pattern match
WHERE col ILIKE 'a%'        -- case-insensitive pattern
WHERE col IS NULL           -- null check
WHERE col IS NOT NULL       -- not null check
```

### Aggregate Functions
```sql
COUNT(*) | COUNT(col) | SUM() | AVG() | MAX() | MIN() | ROUND(val, decimal)
```

### Joins
```sql
INNER JOIN  -- matching rows only
LEFT JOIN   -- all left + matched right (NULL if no match)
RIGHT JOIN  -- all right + matched left (NULL if no match)
FULL JOIN   -- all rows from both (NULL where no match)
CROSS JOIN  -- every row × every row
```

### NULL Helpers
```sql
COALESCE(col, 'fallback')   -- first non-NULL value
col IS NULL / IS NOT NULL   -- null check
```

### Date
```sql
EXTRACT(YEAR FROM date_col)
EXTRACT(MONTH FROM date_col)
NOW() | CURRENT_DATE
```

### Objects
```sql
CREATE FUNCTION name() RETURNS type LANGUAGE sql AS $$ ... $$;
CREATE PROCEDURE name() LANGUAGE plpgsql AS $$ BEGIN ... END; $$;
CREATE TRIGGER name AFTER DELETE ON table FOR EACH ROW EXECUTE FUNCTION fn();
CREATE VIEW name AS SELECT ...;
CREATE INDEX idx_name ON table(column);
```

---

*Notes by Rafi — PostgreSQL & DBMS · IIUC CSE*

---

## 23. Real Project — Football Ticket Booking System

> A complete mini-project using `Users`, `Matches`, and `Bookings` tables.  
> Great for revision — covers SELECT, ILIKE, NULL, JOIN, Subquery, LIMIT/OFFSET together.

### Schema

```sql
CREATE TABLE Users (
  user_id      INT          PRIMARY KEY,
  full_name    VARCHAR(100) NOT NULL,
  email        VARCHAR(100) UNIQUE NOT NULL,
  role         VARCHAR(50)  CHECK (role IN ('Ticket Manager', 'Football Fan')),
  phone_number VARCHAR(20)
);

CREATE TABLE Matches (
  match_id            INT           PRIMARY KEY,
  fixture             VARCHAR(100)  NOT NULL,
  tournament_category VARCHAR(50),
  base_ticket_price   DECIMAL(10,2) CHECK (base_ticket_price >= 0),
  match_status        VARCHAR(20)   CHECK (
    match_status IN ('Available', 'Selling Fast', 'Sold Out', 'Postponed')
  )
);

CREATE TABLE Bookings (
  booking_id     INT           PRIMARY KEY,
  user_id        INT           REFERENCES Users(user_id),
  match_id       INT           REFERENCES Matches(match_id),
  seat_number    VARCHAR(10),
  payment_status VARCHAR(20)   CHECK (
    payment_status IN ('Pending', 'Confirmed', 'Cancelled', 'Refunded')
  ),
  total_cost     DECIMAL(10,2) CHECK (total_cost >= 0)
);
```

### Relationships

| Relationship | Description |
|---|---|
| One → Many | One User → Many Bookings |
| Many → One | Many Bookings → One Match |
| One → One (logical) | Each booking maps exactly one user to one match + seat |

---

### Query 1 — Champions League Available Matches
> Get all Champions League matches where status is 'Available'.  
> **Concepts:** `WHERE`, `AND`, `CAST`

```sql
SELECT match_id, fixture,
       CAST(base_ticket_price AS DECIMAL(10)) AS base_ticket_price
FROM   Matches
WHERE  tournament_category = 'Champions League'
  AND  match_status = 'Available';
```

---

### Query 2 — Search Users by Name Pattern
> Find users whose name starts with 'Tanvir' OR contains 'Haque' (case-insensitive).  
> **Concepts:** `ILIKE`, `OR`

```sql
SELECT user_id, full_name, email
FROM   Users
WHERE  full_name ILIKE 'Tanvir%'
  OR   full_name ILIKE '%Haque%';
```

> 💡 `ILIKE` = case-insensitive version of `LIKE` (PostgreSQL only).

---

### Query 3 — Bookings with Missing Payment Status
> Show bookings where `payment_status` is NULL, replacing NULL with 'Action Required'.  
> **Concepts:** `IS NULL`, `COALESCE`

```sql
SELECT booking_id, user_id, match_id,
       COALESCE(payment_status, 'Action Required') AS systematic_status
FROM   Bookings
WHERE  payment_status IS NULL;
```

---

### Query 4 — Full Booking Details (3-Table JOIN)
> Show booking details with user name and match fixture combined.  
> **Concepts:** `INNER JOIN` × 2

```sql
SELECT b.booking_id, u.full_name, m.fixture,
       CAST(b.total_cost AS DECIMAL(10)) AS total_cost
FROM   Bookings b
INNER JOIN Users   u ON b.user_id  = u.user_id
INNER JOIN Matches m ON b.match_id = m.match_id;
```

---

### Query 5 — All Users Including Those with No Bookings
> Show every user even if they have never made a booking.  
> **Concepts:** `LEFT JOIN`, NULL in result

```sql
SELECT u.user_id, u.full_name, b.booking_id
FROM   Users u
LEFT JOIN Bookings b ON u.user_id = b.user_id;
-- Users with no bookings → booking_id will be NULL
```

> 💡 To find users with **zero** bookings: add `WHERE b.booking_id IS NULL`

---

### Query 6 — Bookings Above Average Total Cost
> Find bookings where total cost is higher than the average of all bookings.  
> **Concepts:** Subquery, `AVG`  
> AVG = (150+120+150+150+120) / 5 = **138**

```sql
SELECT booking_id, match_id,
       CAST(total_cost AS DECIMAL(10)) AS total_cost
FROM   Bookings
WHERE  total_cost > (SELECT AVG(total_cost) FROM Bookings);
```

---

### Query 7 — 2nd and 3rd Most Expensive Matches
> Skip the most expensive match, return the next two.  
> **Concepts:** `ORDER BY DESC`, `LIMIT`, `OFFSET`

```sql
SELECT match_id, fixture,
       CAST(base_ticket_price AS DECIMAL(10)) AS base_ticket_price
FROM   Matches
ORDER BY base_ticket_price DESC
LIMIT 2 OFFSET 1;
-- OFFSET 1 skips Real Madrid vs Barcelona (150) → gets 130 and 120
```

---

### ✏️ Practice — Project Queries

1. Find all 'Football Fan' users who have made at least one 'Confirmed' booking. *(JOIN + WHERE)*
2. Show each match fixture with the total number of bookings it has received, including matches with zero bookings. *(LEFT JOIN + GROUP BY + COUNT)*
3. Find all users who have never made any booking. *(LEFT JOIN + IS NULL)*
4. Show the total revenue collected per `tournament_category`. *(JOIN + GROUP BY + SUM)*
5. Find the fan who has spent the most money in total across all bookings. *(GROUP BY + SUM + ORDER BY + LIMIT 1)*

---

## 24. Theory Questions & Answers

> Key DBMS concepts — commonly asked in exams and interviews.

---

### Q1 — What role does a Foreign Key play, and how does it protect data integrity?

A **Foreign Key** enforces **Referential Integrity**. It ensures that a value in one table's column must already exist in another table's Primary Key column.

In the Bookings table, `user_id` and `match_id` are FKs. Before any booking is saved, PostgreSQL checks that the referenced `user_id` exists in `Users` and `match_id` exists in `Matches`. If not — the insert is rejected entirely.

```sql
-- ❌ This will fail if match_id = 999 doesn't exist in Matches
INSERT INTO Bookings (booking_id, user_id, match_id, total_cost)
VALUES (999, 1, 999, 150.00);
-- ERROR: insert or update on table "bookings" violates foreign key constraint
```

> **Summary:** FK = a promise that every reference points to something real.

---

### Q2 — Why can't you use aggregate functions inside WHERE? How does HAVING fix this?

SQL executes in this order: `FROM` → `WHERE` → `GROUP BY` → aggregates → `HAVING`.

`WHERE` runs **before** any grouping — so at that point, `COUNT()` or `AVG()` haven't been calculated yet. Using them inside `WHERE` throws an error.

`HAVING` runs **after** `GROUP BY` and the aggregates — so it can filter using their results.

```sql
-- ❌ Wrong — aggregates not available yet
SELECT match_id, COUNT(booking_id)
FROM Bookings
WHERE COUNT(booking_id) > 2
GROUP BY match_id;

-- ✅ Correct
SELECT match_id, COUNT(booking_id) AS total_bookings
FROM Bookings
GROUP BY match_id
HAVING COUNT(booking_id) > 2;
```

> **Rule:** `WHERE` filters rows. `HAVING` filters groups.

---

### Q3 — Why does a Primary Key forbid NULL values?

A Primary Key enforces **Entity Integrity** — it must uniquely and definitively identify every row.

`NULL` in SQL means *unknown*. The problem: `NULL ≠ NULL` in SQL logic. If two rows both had `NULL` as their PK, the database cannot tell whether they are the same row or different — breaking uniqueness entirely.

Think of it like a National ID card — a citizen cannot exist in the system with a blank ID. Without a definite identity, the row cannot be found, linked to, or reliably tracked.

> **Rule:** PK = UNIQUE + NOT NULL. Both are required — one without the other is incomplete.

---

### Q4 — What does a LEFT JOIN show for a user with no bookings?

`LEFT JOIN` guarantees **every row from the left table appears**, regardless of matches in the right table. For a user with zero bookings, their row still appears — but all columns from the right table (`Bookings`) are filled with `NULL`.

```
| user_id | full_name  | booking_id |
|---------|------------|------------|
| 4       | Jannat Ara | NULL       |
```

This pattern is very useful — it's the standard way to find records with no related data:

```sql
-- Find users who have NEVER booked a ticket
SELECT u.user_id, u.full_name
FROM Users u
LEFT JOIN Bookings b ON u.user_id = b.user_id
WHERE b.booking_id IS NULL;
```

---

### Q5 — What is the difference between a Subquery and a JOIN? When to use each?

| | Subquery | JOIN |
|-|----------|------|
| Purpose | Compute an intermediate value first | Combine columns from two tables |
| Result type | Usually a single value or list | Combined rows from both tables |
| Use when | Two-step problem (calculate → filter) | You need columns from both tables |
| Performance | Can be slower on large sets | Generally faster with indexes |

**Subquery example — two-step problem:**
```sql
-- Step 1 (inner): calculate average cost
-- Step 2 (outer): filter bookings using that average
SELECT booking_id, total_cost
FROM Bookings
WHERE total_cost > (SELECT AVG(total_cost) FROM Bookings);
```

A JOIN cannot solve this — `AVG(total_cost)` doesn't exist as a column anywhere. It must be computed first.

**JOIN example — need columns from both tables:**
```sql
-- Need full_name from Users AND fixture from Matches
SELECT u.full_name, m.fixture, b.total_cost
FROM Bookings b
INNER JOIN Users   u ON b.user_id  = u.user_id
INNER JOIN Matches m ON b.match_id = m.match_id;
```

---

### Q6 — What is Normalization? (DBMS Theory)

**Normalization** is the process of organizing a database to reduce data redundancy and improve data integrity by dividing data into related tables.

| Normal Form | Rule |
|-------------|------|
| **1NF** | Each column has atomic (single) values. No repeating groups. |
| **2NF** | 1NF + every non-key column depends on the **whole** primary key. |
| **3NF** | 2NF + no non-key column depends on **another non-key** column. |

```
❌ Unnormalized — storing courses in one column:
students(id, name, courses)
1, Rafi, "SQL, React, Python"

✅ Normalized — separate tables linked by FK:
students(id, name)
courses(id, course_name)
enrollments(student_id, course_id)
```

---

### Q7 — What is the difference between DELETE, TRUNCATE, and DROP?

| Command | What it does | Can use WHERE? | Rollback? | Keeps table? |
|---------|-------------|----------------|-----------|--------------|
| `DELETE` | Removes specific rows | ✅ Yes | ✅ Yes | ✅ Yes |
| `TRUNCATE` | Removes ALL rows instantly | ❌ No | ❌ No (usually) | ✅ Yes |
| `DROP` | Removes the entire table + data | ❌ No | ❌ No | ❌ No |

```sql
DELETE FROM students WHERE id = 5;     -- remove one row
TRUNCATE TABLE students;               -- clear all rows, keep structure
DROP TABLE students;                   -- destroy the table completely
```

---

### Q8 — What is an Index and when should you use one?

An **Index** is a separate data structure that PostgreSQL maintains to speed up lookups — like a book's index pointing to page numbers.

**Without index:** PostgreSQL reads every row (Sequential Scan) — slow on large tables.  
**With index:** PostgreSQL jumps directly to matching rows (Index Scan) — fast.

```sql
CREATE INDEX idx_country ON students(country);

-- Check if index is being used:
EXPLAIN ANALYZE SELECT * FROM students WHERE country = 'Bangladesh';
```

**Use index on:** columns in `WHERE`, `JOIN ON`, `ORDER BY`, FK columns.  
**Avoid on:** small tables, columns rarely queried, columns with very few distinct values.

> ⚠️ Indexes slow down `INSERT`/`UPDATE`/`DELETE` slightly — each write must also update the index.

---

### ✏️ Practice — Theory

1. Why is `NULL <> NULL` in SQL? What does this mean for UNIQUE constraints?
2. A table has `first_name` and `last_name` as a composite PK. Can two rows have the same `first_name`? Why?
3. Explain the difference between `ON DELETE CASCADE` and `ON DELETE SET NULL` with an example.
4. What is the difference between a clustered and non-clustered index? (PostgreSQL uses heap + B-tree)
5. If you run `TRUNCATE TABLE` inside a `BEGIN` transaction, can you `ROLLBACK`? Test and explain.

---

## 🔗 Related Notes

| Note | Topic | Link |
|------|-------|------|
| 📘 **PostgreSQL & DBMS** | This file — full SQL reference | `PostgreSQL_Notes.md` |
| 🎫 **Football Ticket Booking** | Mini-project: schema + 7 queries + theory | `Football_Ticket_Booking.md` |
| 🧠 **DSA Notes** | Data Structures & Algorithms concepts | `DSA_Notes.md` |

## 🔗 Related Notes
 
| Note | Topic | Link |
| :--- | :--- | :--- |
| 🎫 **Football Ticket Booking** | Mini-project: schema + 7 queries | [Football_Ticket_Booking.md](./Football_Ticket_Booking.md) |
| 📂 **DBMS Part 1** | Database Models,Relational,Table/Relation,Keys,,Set,Database Design(SDLC),Relationship Cardinality,ERD| [DBMS Part-1.pdf](./DBMS%20Part-1.pdf) |
| 📂 **DBMS Part 2** | Anomalies,Normalization,Functional Dependency,Resolving Many to Many | [DBMS Part-2.pdf](./DBMS%20Part-2.pdf) |
| 📂 **DBMS Part 3** | Data Types,Column Constraints| [DBMS Part-3.pdf](./DBMS%20Part-3.pdf) |
| 📂 **DBMS Part 4** | ALTER,SELECT,Scaler | [DBMS Part-4.pdf](./DBMS%20Part-4.pdf) |

---

*Notes by Rafi — PostgreSQL & DBMS .*