# Lesson 2.3: SQL Fundamentals

## Navigation
- [← Back to Lesson Plan](../2.3-sql-fundamentals.md)
- [← Back to Module Overview](../README.md)

## Overview
SQL (Structured Query Language) is the standard language for interacting with relational databases. As a data engineer, mastering SQL is essential because it allows you to extract, transform, and manipulate data stored in relational database systems. This lecture covers the core concepts of SQL, from basic queries to more advanced operations.

## Core Concepts

### Relational Database Fundamentals

A relational database organizes data into tables (relations) consisting of rows and columns. Each table represents an entity, with columns defining attributes and rows containing actual data. Key concepts include:

- **Tables**: Collections of related data organized in rows and columns
- **Columns**: Define the attributes and data types for each piece of information
- **Rows**: Individual records in a table
- **Primary Keys**: Uniquely identify each record in a table
- **Foreign Keys**: Create relationships between tables by referencing primary keys in other tables
- **Schemas**: Logical groupings of database objects

### Data Types

SQL supports various data types, which may vary slightly between database systems:

#### Numeric Types
- **INTEGER/INT**: Whole numbers without decimal places
- **SMALLINT/BIGINT**: Smaller or larger range integers
- **DECIMAL/NUMERIC(p,s)**: Exact numeric values with specified precision and scale
- **FLOAT/REAL/DOUBLE**: Approximate numeric values with floating decimal points

#### Character Types
- **CHAR(n)**: Fixed-length character strings
- **VARCHAR(n)**: Variable-length character strings with maximum length
- **TEXT**: Variable-length character strings with large maximum size

#### Date and Time Types
- **DATE**: Calendar date (year, month, day)
- **TIME**: Time of day
- **TIMESTAMP**: Date and time
- **INTERVAL**: Period of time

#### Other Common Types
- **BOOLEAN**: True/false values
- **BLOB/BINARY**: Binary data
- **JSON**: JSON-formatted data (in modern databases)
- **ARRAY**: Collections of elements (in some database systems)

## Basic SQL Operations

### CREATE TABLE Statement

The `CREATE TABLE` statement defines a new table structure:

```sql
CREATE TABLE employees (
    employee_id INTEGER PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    hire_date DATE,
    department_id INTEGER,
    salary DECIMAL(10,2),
    FOREIGN KEY (department_id) REFERENCES departments(department_id)
);
```

Key components:
- Column definitions with data types
- Constraints (PRIMARY KEY, NOT NULL, etc.)
- Relationships (FOREIGN KEY)

### SELECT Statement

The SELECT statement retrieves data from tables:

```sql
-- Basic SELECT
SELECT first_name, last_name, salary
FROM employees;

-- All columns
SELECT * FROM employees;

-- With filtering condition
SELECT first_name, last_name, salary
FROM employees
WHERE salary > 50000;

-- Sorting results
SELECT first_name, last_name, hire_date
FROM employees
ORDER BY hire_date DESC;

-- Limiting results
SELECT first_name, last_name, salary
FROM employees
ORDER BY salary DESC
LIMIT 10;
```

### Filtering with WHERE

The WHERE clause filters rows based on conditions:

```sql
-- Comparison operators
SELECT * FROM products WHERE price < 100;
SELECT * FROM employees WHERE department_id = 3;

-- Logical operators (AND, OR, NOT)
SELECT * FROM employees
WHERE (department_id = 3 OR department_id = 4)
AND salary > 60000;

-- Range checks
SELECT * FROM orders
WHERE order_date BETWEEN '2023-01-01' AND '2023-12-31';

-- Set membership
SELECT * FROM products
WHERE category IN ('Electronics', 'Computers', 'Accessories');

-- Pattern matching
SELECT * FROM customers
WHERE last_name LIKE 'Sm%';  -- Names starting with "Sm"
```

### Data Modification Statements

SQL provides statements for modifying data:

```sql
-- Inserting new records
INSERT INTO employees (employee_id, first_name, last_name, hire_date, department_id, salary)
VALUES (1001, 'John', 'Smith', '2023-02-15', 3, 75000);

-- Multiple inserts
INSERT INTO employees (employee_id, first_name, last_name, department_id)
VALUES
  (1002, 'Sarah', 'Johnson', 4),
  (1003, 'Michael', 'Williams', 3);

-- Updating records
UPDATE employees
SET salary = 80000
WHERE employee_id = 1001;

-- Updating multiple columns
UPDATE employees
SET 
    salary = salary * 1.05,
    last_updated = CURRENT_TIMESTAMP
WHERE department_id = 3;

-- Deleting records
DELETE FROM employees
WHERE employee_id = 1003;
```

### Aggregate Functions

Aggregate functions perform calculations on sets of values:

```sql
-- Basic aggregates
SELECT 
    COUNT(*) AS total_employees,
    AVG(salary) AS average_salary,
    MIN(salary) AS lowest_salary,
    MAX(salary) AS highest_salary,
    SUM(salary) AS salary_budget
FROM employees;

-- Grouping data
SELECT 
    department_id,
    COUNT(*) AS employee_count,
    AVG(salary) AS average_salary
FROM employees
GROUP BY department_id;

-- Filtering groups
SELECT 
    department_id,
    COUNT(*) AS employee_count,
    AVG(salary) AS average_salary
FROM employees
GROUP BY department_id
HAVING COUNT(*) > 5;
```

Common aggregate functions:
- **COUNT()**: Counts rows or non-NULL values
- **SUM()**: Calculates the sum of numeric values
- **AVG()**: Calculates the average of numeric values
- **MIN()**: Returns the minimum value
- **MAX()**: Returns the maximum value
- **STDDEV()**: Standard deviation (statistical measure)

## Joins and Relationships

### Types of Joins

Joins combine rows from multiple tables based on related columns:

```sql
-- INNER JOIN: Returns matching rows from both tables
SELECT e.employee_id, e.first_name, e.last_name, d.department_name
FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id;

-- LEFT JOIN: Returns all rows from left table and matching rows from right table
SELECT e.employee_id, e.first_name, e.last_name, d.department_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.department_id;

-- RIGHT JOIN: Returns all rows from right table and matching rows from left table
SELECT e.employee_id, e.first_name, e.last_name, d.department_name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.department_id;

-- FULL OUTER JOIN: Returns all rows when there's a match in either table
SELECT e.employee_id, e.first_name, d.department_name
FROM employees e
FULL OUTER JOIN departments d ON e.department_id = d.department_id;

-- CROSS JOIN: Cartesian product (all possible combinations)
SELECT e.first_name, l.location_name
FROM employees e
CROSS JOIN locations l;
```

### Self Joins

A self join combines a table with itself:

```sql
-- Finding employee and their manager
SELECT 
    e.employee_id, 
    e.first_name || ' ' || e.last_name AS employee,
    m.first_name || ' ' || m.last_name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.employee_id;
```

### Multi-Table Joins

Complex queries often join multiple tables:

```sql
-- Join three tables
SELECT 
    e.employee_id,
    e.first_name,
    e.last_name,
    d.department_name,
    l.city,
    l.country
FROM employees e
JOIN departments d ON e.department_id = d.department_id
JOIN locations l ON d.location_id = l.location_id
WHERE l.country = 'USA';
```

## Subqueries

Subqueries (nested queries) are queries embedded within another query:

### Subqueries in WHERE Clauses

```sql
-- Find employees with higher than average salary
SELECT employee_id, first_name, last_name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);

-- Find employees in departments with more than 10 employees
SELECT employee_id, first_name, last_name, department_id
FROM employees
WHERE department_id IN (
    SELECT department_id
    FROM employees
    GROUP BY department_id
    HAVING COUNT(*) > 10
);
```

### Subqueries in FROM Clauses

```sql
-- Use a subquery as a derived table
SELECT dept_name, avg_salary
FROM (
    SELECT 
        d.department_name AS dept_name,
        AVG(e.salary) AS avg_salary
    FROM employees e
    JOIN departments d ON e.department_id = d.department_id
    GROUP BY d.department_name
) AS dept_salaries
WHERE avg_salary > 60000;
```

### Correlated Subqueries

A correlated subquery references columns from the outer query:

```sql
-- Find employees who earn more than the average in their department
SELECT e.employee_id, e.first_name, e.last_name, e.salary, e.department_id
FROM employees e
WHERE e.salary > (
    SELECT AVG(salary)
    FROM employees
    WHERE department_id = e.department_id
);
```

## Working with NULL Values

NULL represents missing or unknown values and requires special handling:

```sql
-- Finding records with NULL values
SELECT employee_id, first_name, last_name
FROM employees
WHERE manager_id IS NULL;

-- Finding records with non-NULL values
SELECT employee_id, first_name, last_name
FROM employees
WHERE phone_number IS NOT NULL;

-- COALESCE function returns the first non-NULL value
SELECT 
    employee_id,
    first_name,
    COALESCE(commission_pct, 0) AS commission_pct
FROM employees;

-- NULLIF function returns NULL if two values are equal
SELECT 
    employee_id,
    NULLIF(division_count, 0) AS safe_divisor
FROM department_stats;
```

## SQL Functions

SQL provides various built-in functions to transform and manipulate data:

### String Functions

```sql
-- String concatenation
SELECT first_name || ' ' || last_name AS full_name FROM employees;
-- Or in some systems: CONCAT(first_name, ' ', last_name)

-- Case conversion
SELECT UPPER(last_name), LOWER(email) FROM employees;

-- Substring
SELECT SUBSTRING(phone_number, 1, 3) AS area_code FROM employees;
-- Or: SUBSTR(phone_number, 1, 3)

-- String length
SELECT LENGTH(last_name) AS name_length FROM employees;

-- Replacing text
SELECT REPLACE(phone_number, '.', '-') FROM employees;

-- Trimming whitespace
SELECT TRIM(' John Smith ') AS trimmed_name;
```

### Date Functions

```sql
-- Current date and time
SELECT CURRENT_DATE, CURRENT_TIMESTAMP;

-- Extract parts of dates
SELECT 
    EXTRACT(YEAR FROM hire_date) AS hire_year,
    EXTRACT(MONTH FROM hire_date) AS hire_month
FROM employees;

-- Date arithmetic
SELECT 
    employee_id,
    hire_date,
    hire_date + INTERVAL '90 days' AS probation_end_date
FROM employees;

-- Date difference
SELECT 
    DATEDIFF('day', hire_date, CURRENT_DATE) AS days_employed
FROM employees;
-- Or: CURRENT_DATE - hire_date (in some systems)
```

### Numeric Functions

```sql
-- Rounding
SELECT ROUND(salary/12, 2) AS monthly_salary FROM employees;

-- Ceiling and floor
SELECT CEILING(amount), FLOOR(amount) FROM payments;
-- Also: CEIL(amount)

-- Absolute value
SELECT ABS(balance_change) FROM transactions;

-- Power and square root
SELECT POWER(base, 2), SQRT(value) FROM measurements;
```

### Conditional Functions

```sql
-- CASE expression
SELECT 
    employee_id,
    first_name,
    last_name,
    CASE
        WHEN salary < 50000 THEN 'Low'
        WHEN salary BETWEEN 50000 AND 100000 THEN 'Medium'
        ELSE 'High'
    END AS salary_category
FROM employees;

-- Simple CASE (switch-like)
SELECT 
    order_id,
    CASE status
        WHEN 'PENDING' THEN 'In Process'
        WHEN 'SHIPPED' THEN 'On the Way'
        WHEN 'DELIVERED' THEN 'Completed'
        ELSE 'Unknown'
    END AS status_description
FROM orders;
```

## Indexing and Performance

### Creating Indexes

Indexes improve query performance by providing faster access paths to data:

```sql
-- Creating a basic index
CREATE INDEX idx_employees_last_name ON employees(last_name);

-- Creating a unique index
CREATE UNIQUE INDEX idx_departments_name ON departments(department_name);

-- Creating a composite index (multiple columns)
CREATE INDEX idx_employees_dept_hire ON employees(department_id, hire_date);
```

### Index Types

Common index types include:
- **B-tree**: Default balanced tree structure, good for equality and range conditions
- **Hash**: Optimized for equality operations
- **Bitmap**: Efficient for low-cardinality columns
- **GiST/GIN**: Specialized indexes for full-text search and complex data types

### Analyzing Query Performance

Most database systems provide tools to analyze query execution:

```sql
-- PostgreSQL EXPLAIN
EXPLAIN SELECT * FROM employees WHERE department_id = 3;

-- MySQL EXPLAIN
EXPLAIN SELECT * FROM employees WHERE department_id = 3;

-- SQL Server
SET STATISTICS IO, TIME ON;
SELECT * FROM employees WHERE department_id = 3;
```

### Query Optimization Tips

1. Use specific columns in SELECT instead of SELECT *
2. Add appropriate indexes for frequently queried columns
3. Be cautious with wildcard searches (LIKE '%text%')
4. Limit result sets when possible
5. Use JOINs instead of subqueries when feasible
6. Avoid functions on indexed columns in WHERE clauses

## Practical SQL Examples

### Sales Analysis

```sql
-- Total sales by product category and month
SELECT 
    c.category_name,
    EXTRACT(YEAR FROM o.order_date) AS year,
    EXTRACT(MONTH FROM o.order_date) AS month,
    SUM(oi.quantity * oi.unit_price) AS total_sales
FROM order_items oi
JOIN products p ON oi.product_id = p.product_id
JOIN categories c ON p.category_id = c.category_id
JOIN orders o ON oi.order_id = o.order_id
WHERE o.order_date BETWEEN '2023-01-01' AND '2023-12-31'
GROUP BY c.category_name, EXTRACT(YEAR FROM o.order_date), EXTRACT(MONTH FROM o.order_date)
ORDER BY c.category_name, year, month;
```

### Customer Segmentation

```sql
-- Segment customers by purchase behavior
SELECT 
    customer_id,
    COUNT(DISTINCT order_id) AS order_count,
    SUM(total_amount) AS total_spent,
    CASE
        WHEN COUNT(DISTINCT order_id) >= 10 THEN 'Frequent Buyer'
        WHEN SUM(total_amount) >= 10000 THEN 'High Value'
        WHEN MAX(order_date) < CURRENT_DATE - INTERVAL '1 year' THEN 'Inactive'
        ELSE 'Regular'
    END AS customer_segment
FROM orders
GROUP BY customer_id;
```

### Employee Hierarchy Report

```sql
-- Generate a report showing employee hierarchies
WITH RECURSIVE employee_hierarchy AS (
    -- Base case: Top-level employees (no manager)
    SELECT 
        employee_id, 
        first_name,
        last_name,
        manager_id,
        1 AS level,
        first_name || ' ' || last_name AS hierarchy_path
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive case: employees with managers
    SELECT 
        e.employee_id,
        e.first_name,
        e.last_name,
        e.manager_id,
        eh.level + 1,
        eh.hierarchy_path || ' > ' || e.first_name || ' ' || e.last_name
    FROM employees e
    JOIN employee_hierarchy eh ON e.manager_id = eh.employee_id
)
SELECT 
    employee_id,
    first_name,
    last_name,
    level,
    hierarchy_path
FROM employee_hierarchy
ORDER BY hierarchy_path;
```

## Best Practices for SQL in Data Engineering

1. **Write readable code**
   - Use consistent indentation and capitalization
   - Add comments for complex queries
   - Break long queries into logical parts
   - Use meaningful aliases

2. **Optimize performance**
   - Create appropriate indexes
   - Avoid SELECT * in production code
   - Use JOINs with proper conditions
   - Test query performance with realistic data volumes

3. **Ensure data integrity**
   - Use appropriate constraints (NOT NULL, UNIQUE, etc.)
   - Implement referential integrity with FOREIGN KEYs
   - Validate data before insertion
   - Use transactions for multi-step operations

4. **Security considerations**
   - Avoid SQL injection by using parameterized queries
   - Grant minimal necessary permissions
   - Audit sensitive data access
   - Encrypt sensitive data

5. **Maintainability**
   - Use views for complex, frequently used queries
   - Create stored procedures for common operations
   - Document database schema and relationships
   - Keep queries version-controlled

## Common Pitfalls to Avoid

1. **Inefficient joins** - Forgetting join conditions, using cartesian products
2. **Mishandling NULL values** - Using = NULL instead of IS NULL
3. **Overusing subqueries** - When simpler joins would be more efficient
4. **Ignoring indexes** - Or making them ineffective with functions in WHERE clauses
5. **Not considering data volumes** - Queries that work well on small datasets may fail on production data
6. **Poor transaction management** - Not properly committing or rolling back transactions
7. **Hardcoding values** - Instead of using parameters or configuration tables

## Conclusion

SQL remains the fundamental language for working with relational data, and proficiency in SQL is critical for data engineers. By mastering these concepts, you'll be able to efficiently query, transform, and manipulate data across various database systems, creating the foundation for effective data pipelines and analytics.

As you continue your data engineering journey, you'll build on these SQL fundamentals to work with more advanced concepts like window functions, recursive queries, stored procedures, and database-specific optimizations.

## Next Steps

- Practice writing complex SQL queries across different scenarios
- Learn how SQL integrates with data engineering tools like Airflow, Spark, and dbt
- Explore database-specific SQL features in systems you'll be working with
- Study query optimization techniques for large-scale data
- Learn about SQL in ETL/ELT processes