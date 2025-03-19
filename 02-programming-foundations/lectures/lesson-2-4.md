# Lesson 2.4: Advanced SQL

## Navigation
- [← Back to Lesson Plan](../2.4-advanced-sql.md)
- [← Back to Module Overview](../README.md)

## Overview
While basic SQL knowledge forms the foundation for working with relational databases, data engineers frequently require more sophisticated SQL techniques. Advanced SQL enables you to solve complex analytical problems, optimize query performance, and manipulate data in more powerful ways. This lecture builds on fundamental SQL concepts to explore advanced features essential for professional data engineering.

## Advanced SQL Concepts

### Window Functions

Window functions are powerful SQL features that perform calculations across a set of rows related to the current row, unlike aggregation functions that group rows into a single output row.

#### OVER Clause
The OVER clause defines the "window" or set of rows that a window function operates on:

```sql
-- Basic window function with OVER()
SELECT 
    employee_id,
    first_name,
    last_name,
    salary,
    AVG(salary) OVER() as avg_company_salary
FROM employees;
```

The above query returns each employee's details along with the company-wide average salary on each row.

#### Partitioning
You can divide rows into groups using PARTITION BY:

```sql
-- Partitioning by department
SELECT 
    employee_id,
    first_name,
    last_name,
    department_id,
    salary,
    AVG(salary) OVER(PARTITION BY department_id) as avg_dept_salary
FROM employees;
```

This calculates the average salary separately for each department.

#### Ordering
The ORDER BY clause within OVER defines the logical order of rows within each partition:

```sql
-- Running total of sales by date
SELECT 
    order_date,
    order_id,
    amount,
    SUM(amount) OVER(ORDER BY order_date) as running_total
FROM orders;
```

#### Frame Specifications
Frame specifications determine which rows are included in the window function's calculation:

```sql
-- Moving average with 3 rows before, current row, and 3 rows after
SELECT 
    order_date,
    amount,
    AVG(amount) OVER(
        ORDER BY order_date
        ROWS BETWEEN 3 PRECEDING AND 3 FOLLOWING
    ) as moving_avg_7days
FROM orders;
```

Some common frame specifications include:
- `ROWS BETWEEN n PRECEDING AND CURRENT ROW`
- `ROWS BETWEEN CURRENT ROW AND n FOLLOWING`
- `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`
- `ROWS BETWEEN n PRECEDING AND n FOLLOWING`

#### Common Window Functions

1. **Ranking Functions**

```sql
-- Different ranking functions
SELECT 
    first_name,
    last_name,
    department_id,
    salary,
    RANK() OVER(PARTITION BY department_id ORDER BY salary DESC) as dept_rank,
    DENSE_RANK() OVER(PARTITION BY department_id ORDER BY salary DESC) as dept_dense_rank,
    ROW_NUMBER() OVER(PARTITION BY department_id ORDER BY salary DESC) as dept_row_num,
    NTILE(4) OVER(PARTITION BY department_id ORDER BY salary DESC) as dept_quartile
FROM employees;
```

- `RANK()`: Assigns the same rank to ties, skipping subsequent ranks
- `DENSE_RANK()`: Assigns the same rank to ties without skipping ranks
- `ROW_NUMBER()`: Assigns unique numbers even to ties
- `NTILE(n)`: Divides the rows into n approximately equal groups

2. **Value Functions**

```sql
-- Value functions
SELECT 
    product_id,
    category_id,
    price,
    FIRST_VALUE(price) OVER(PARTITION BY category_id ORDER BY price) as cheapest_in_category,
    LAST_VALUE(price) OVER(
        PARTITION BY category_id ORDER BY price
        RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) as most_expensive_in_category,
    LAG(price, 1) OVER(PARTITION BY category_id ORDER BY price) as prev_price,
    LEAD(price, 1) OVER(PARTITION BY category_id ORDER BY price) as next_price
FROM products;
```

- `FIRST_VALUE()`: Returns the first value in the window
- `LAST_VALUE()`: Returns the last value in the window
- `LAG()`: Accesses a row that comes before the current row
- `LEAD()`: Accesses a row that comes after the current row

3. **Analytical Functions**

```sql
-- Analytical functions for statistics
SELECT 
    product_id,
    category_id,
    price,
    AVG(price) OVER(PARTITION BY category_id) as avg_price,
    STDDEV(price) OVER(PARTITION BY category_id) as std_dev_price,
    PERCENT_RANK() OVER(PARTITION BY category_id ORDER BY price) as percentile
FROM products;
```

- `PERCENT_RANK()`: Relative rank of the current row (0 to 1)
- `CUME_DIST()`: Cumulative distribution of a value
- `PERCENTILE_CONT()` & `PERCENTILE_DISC()`: Find percentiles within groups

### Common Table Expressions (CTEs)

CTEs provide a way to write auxiliary statements for use in a larger query, improving readability and allowing for recursion.

#### Basic CTEs

```sql
-- Simple CTE for sales analysis
WITH monthly_sales AS (
    SELECT 
        EXTRACT(YEAR FROM order_date) AS year,
        EXTRACT(MONTH FROM order_date) AS month,
        SUM(amount) AS total_sales
    FROM orders
    GROUP BY 
        EXTRACT(YEAR FROM order_date),
        EXTRACT(MONTH FROM order_date)
)
SELECT 
    year,
    month,
    total_sales,
    LAG(total_sales, 1) OVER(ORDER BY year, month) AS prev_month_sales,
    (total_sales - LAG(total_sales, 1) OVER(ORDER BY year, month)) / 
        LAG(total_sales, 1) OVER(ORDER BY year, month) * 100 AS growth_percent
FROM monthly_sales
ORDER BY year, month;
```

#### Multiple CTEs

```sql
-- Multiple CTEs for complex analysis
WITH customer_orders AS (
    SELECT 
        customer_id,
        COUNT(*) AS order_count,
        SUM(amount) AS total_spent
    FROM orders
    GROUP BY customer_id
),
customer_segments AS (
    SELECT 
        customer_id,
        CASE 
            WHEN order_count > 10 AND total_spent > 1000 THEN 'Premium'
            WHEN order_count > 5 OR total_spent > 500 THEN 'Regular'
            ELSE 'Occasional'
        END AS segment
    FROM customer_orders
)
SELECT 
    cs.segment,
    COUNT(DISTINCT co.customer_id) AS customer_count,
    SUM(co.order_count) AS total_orders,
    SUM(co.total_spent) AS total_revenue,
    AVG(co.total_spent) AS avg_spend_per_customer
FROM customer_orders co
JOIN customer_segments cs ON co.customer_id = cs.customer_id
GROUP BY cs.segment;
```

#### Recursive CTEs

Recursive CTEs enable hierarchical or graph-traversal queries:

```sql
-- Employee hierarchy using recursive CTE
WITH RECURSIVE employee_hierarchy AS (
    -- Base case: CEO (no manager)
    SELECT 
        employee_id, 
        first_name,
        last_name,
        manager_id,
        0 AS level,
        first_name || ' ' || last_name AS path
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
        eh.path || ' > ' || e.first_name || ' ' || e.last_name
    FROM employees e
    JOIN employee_hierarchy eh ON e.manager_id = eh.employee_id
)
SELECT 
    employee_id,
    first_name,
    last_name,
    level,
    path
FROM employee_hierarchy
ORDER BY path;
```

#### CTE Optimization

CTEs can be used for optimization by materializing complex subqueries:

```sql
-- Using CTE for optimization
WITH filtered_sales AS (
    SELECT 
        product_id,
        SUM(quantity) AS total_quantity,
        SUM(quantity * unit_price) AS total_sales
    FROM order_items
    WHERE order_date BETWEEN '2023-01-01' AND '2023-12-31'
    GROUP BY product_id
)
SELECT 
    p.product_name,
    p.category_id,
    fs.total_quantity,
    fs.total_sales
FROM products p
JOIN filtered_sales fs ON p.product_id = fs.product_id
WHERE fs.total_sales > 10000
ORDER BY fs.total_sales DESC;
```

### Advanced Querying

#### Complex Joins

Beyond basic joins, advanced SQL allows for sophisticated join techniques:

1. **Self-joins** are useful for hierarchical or relationship data:

```sql
-- Self-join example for manager-employee relationship
SELECT 
    e.employee_id,
    e.first_name || ' ' || e.last_name AS employee_name,
    m.first_name || ' ' || m.last_name AS manager_name
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.employee_id;
```

2. **Cross joins** generate Cartesian products:

```sql
-- Cross join for generating date-product combinations
SELECT 
    d.date,
    p.product_id,
    p.product_name
FROM calendar_dates d
CROSS JOIN products p
WHERE d.date BETWEEN '2023-01-01' AND '2023-01-31';
```

3. **Lateral joins** allow subqueries to reference columns from preceding tables:

```sql
-- Lateral join for top products per category
SELECT 
    c.category_id,
    c.category_name,
    p.product_id,
    p.product_name,
    p.price
FROM categories c
CROSS JOIN LATERAL (
    SELECT product_id, product_name, price
    FROM products
    WHERE category_id = c.category_id
    ORDER BY price DESC
    LIMIT 3
) p;
```

#### Subqueries

Advanced subquery techniques enhance query flexibility:

1. **Correlated subqueries** reference the outer query:

```sql
-- Find employees who earn more than their department average
SELECT 
    e.employee_id,
    e.first_name,
    e.last_name,
    e.salary,
    e.department_id
FROM employees e
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
    WHERE department_id = e.department_id
);
```

2. **Nested subqueries** place one subquery inside another:

```sql
-- Find the most recent order for each customer who has made at least 3 orders
SELECT 
    customer_id,
    order_id,
    order_date,
    amount
FROM orders o1
WHERE order_date = (
    SELECT MAX(order_date)
    FROM orders o2
    WHERE o2.customer_id = o1.customer_id
)
AND customer_id IN (
    SELECT customer_id
    FROM orders
    GROUP BY customer_id
    HAVING COUNT(*) >= 3
);
```

3. **EXISTS and NOT EXISTS** check for the presence or absence of rows:

```sql
-- Find products that have never been ordered
SELECT 
    product_id,
    product_name
FROM products p
WHERE NOT EXISTS (
    SELECT 1
    FROM order_items
    WHERE product_id = p.product_id
);
```

4. **ANY and ALL** compare values against subquery results:

```sql
-- Find products priced higher than ALL products in category 5
SELECT 
    product_id,
    product_name,
    price
FROM products
WHERE price > ALL (
    SELECT price
    FROM products
    WHERE category_id = 5
);

-- Find products priced higher than ANY product in category 5
SELECT 
    product_id,
    product_name,
    price
FROM products
WHERE price > ANY (
    SELECT price
    FROM products
    WHERE category_id = 5
)
AND category_id != 5;
```

### Performance Tuning

Performance optimization is critical for SQL in data engineering.

#### Execution Plans

Understanding query execution plans helps identify performance bottlenecks:

```sql
-- PostgreSQL EXPLAIN ANALYZE
EXPLAIN ANALYZE SELECT 
    c.customer_id,
    c.name,
    COUNT(o.order_id) AS order_count,
    SUM(o.amount) AS total_spent
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_date >= '2023-01-01'
GROUP BY c.customer_id, c.name
HAVING COUNT(o.order_id) > 5
ORDER BY total_spent DESC;
```

Different databases have different syntax for execution plans:
- MySQL: `EXPLAIN`
- SQL Server: `SET STATISTICS IO, TIME ON; [query]`
- Oracle: `EXPLAIN PLAN FOR [query]; SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY);`

#### Statistics

Database statistics inform the query optimizer about data distribution:

```sql
-- PostgreSQL update statistics
ANALYZE customers;

-- SQL Server update statistics
UPDATE STATISTICS customers;

-- Oracle gather statistics
EXEC DBMS_STATS.GATHER_TABLE_STATS('schema', 'customers');
```

Key concepts in statistics:
- Histograms show value distribution
- Selectivity estimates help choose optimal access paths
- Cardinality estimates predict result set sizes

#### Indexes

Strategic indexing is fundamental to query performance:

```sql
-- B-tree index (default)
CREATE INDEX idx_customers_email ON customers(email);

-- Composite index for multi-column conditions
CREATE INDEX idx_orders_customer_date ON orders(customer_id, order_date);

-- Unique index
CREATE UNIQUE INDEX idx_products_sku ON products(sku);

-- Partial index
CREATE INDEX idx_orders_recent ON orders(order_date) 
WHERE order_date > CURRENT_DATE - INTERVAL '90 days';

-- Function-based index
CREATE INDEX idx_customers_lower_email ON customers(LOWER(email));
```

Index types vary by database:
- B-tree: General-purpose balanced tree structure
- Hash: Optimized for equality conditions
- GiST/GIN: Specialized for full-text search and complex types
- BRIN: Block Range INdexes for ordered data
- Spatial: For geographic data

#### Partitioning

Partitioning divides large tables into manageable chunks:

```sql
-- PostgreSQL table partitioning example
CREATE TABLE orders (
    order_id INT,
    customer_id INT,
    order_date DATE,
    amount DECIMAL(10,2)
) PARTITION BY RANGE (order_date);

-- Create partitions
CREATE TABLE orders_2021 PARTITION OF orders
    FOR VALUES FROM ('2021-01-01') TO ('2022-01-01');

CREATE TABLE orders_2022 PARTITION OF orders
    FOR VALUES FROM ('2022-01-01') TO ('2023-01-01');

CREATE TABLE orders_2023 PARTITION OF orders
    FOR VALUES FROM ('2023-01-01') TO ('2024-01-01');
```

Partitioning strategies include:
- Range Partitioning: Based on value ranges (dates, IDs)
- List Partitioning: Based on discrete values (regions, categories)
- Hash Partitioning: Based on hash values for more even distribution
- Composite Partitioning: Combines multiple strategies

#### Query rewriting

Rewriting queries can dramatically improve performance:

```sql
-- Inefficient query with unnecessary JOIN
SELECT 
    c.customer_id,
    COUNT(o.order_id) AS order_count
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id;

-- More efficient version
SELECT 
    customer_id,
    COUNT(order_id) AS order_count
FROM orders
GROUP BY customer_id;
```

Common query rewriting techniques:
- Simplify JOINs when possible
- Avoid SELECT * and retrieve only needed columns
- Prefer EXISTS over IN for uncorrelated subqueries
- Avoid user-defined functions in WHERE clauses
- Reduce the use of DISTINCT when possible

### Advanced Features

#### Materialized Views

Materialized views store query results physically, improving performance for complex queries:

```sql
-- PostgreSQL materialized view
CREATE MATERIALIZED VIEW product_sales_summary AS
SELECT 
    p.product_id,
    p.product_name,
    p.category_id,
    SUM(oi.quantity) AS total_quantity,
    SUM(oi.quantity * oi.unit_price) AS total_sales
FROM products p
JOIN order_items oi ON p.product_id = oi.product_id
JOIN orders o ON oi.order_id = o.order_id
GROUP BY p.product_id, p.product_name, p.category_id;

-- Refresh materialized view
REFRESH MATERIALIZED VIEW product_sales_summary;
```

Benefits:
- Faster query performance for complex aggregations
- Reduced load on source tables
- Practical for reporting dashboards with periodic updates

#### Stored Procedures

Stored procedures encapsulate business logic within the database:

```sql
-- PostgreSQL stored procedure
CREATE OR REPLACE PROCEDURE process_new_order(
    p_customer_id INT,
    p_order_date DATE,
    p_items JSONB
)
LANGUAGE plpgsql
AS $$
DECLARE
    v_order_id INT;
    v_item JSONB;
    v_product_id INT;
    v_quantity INT;
    v_unit_price DECIMAL(10,2);
BEGIN
    -- Create order
    INSERT INTO orders(customer_id, order_date, status)
    VALUES(p_customer_id, p_order_date, 'NEW')
    RETURNING order_id INTO v_order_id;
    
    -- Process items
    FOR v_item IN SELECT * FROM jsonb_array_elements(p_items)
    LOOP
        v_product_id := (v_item->>'product_id')::INT;
        v_quantity := (v_item->>'quantity')::INT;
        
        -- Get current price
        SELECT price INTO v_unit_price
        FROM products
        WHERE product_id = v_product_id;
        
        -- Insert order item
        INSERT INTO order_items(order_id, product_id, quantity, unit_price)
        VALUES(v_order_id, v_product_id, v_quantity, v_unit_price);
        
        -- Update inventory (simplified)
        UPDATE products
        SET stock_quantity = stock_quantity - v_quantity
        WHERE product_id = v_product_id;
    END LOOP;
    
    -- Finalize order
    UPDATE orders SET status = 'PROCESSED' WHERE order_id = v_order_id;
    
    COMMIT;
EXCEPTION
    WHEN OTHERS THEN
        ROLLBACK;
        RAISE;
END;
$$;

-- Call the procedure
CALL process_new_order(
    101, 
    CURRENT_DATE, 
    '[{"product_id": 1, "quantity": 5}, {"product_id": 2, "quantity": 3}]'::JSONB
);
```

Stored procedure syntax varies significantly across databases.

#### User-Defined Functions

User-defined functions extend SQL's capabilities:

```sql
-- PostgreSQL scalar function
CREATE OR REPLACE FUNCTION calculate_discount(
    price DECIMAL,
    discount_percent DECIMAL
) 
RETURNS DECIMAL
LANGUAGE SQL
AS $$
    SELECT ROUND(price * (1 - discount_percent / 100), 2);
$$;

-- Use the function
SELECT 
    product_id,
    product_name,
    price,
    calculate_discount(price, 15) AS discounted_price
FROM products;

-- Table-valued function
CREATE OR REPLACE FUNCTION get_customer_orders(
    p_customer_id INT
)
RETURNS TABLE (
    order_id INT,
    order_date DATE,
    total_amount DECIMAL
)
LANGUAGE SQL
AS $$
    SELECT
        o.order_id,
        o.order_date,
        SUM(oi.quantity * oi.unit_price) AS total_amount
    FROM orders o
    JOIN order_items oi ON o.order_id = oi.order_id
    WHERE o.customer_id = p_customer_id
    GROUP BY o.order_id, o.order_date;
$$;

-- Use the table-valued function
SELECT * FROM get_customer_orders(101);
```

#### Triggers

Triggers execute automatically in response to database events:

```sql
-- PostgreSQL trigger for audit log
CREATE OR REPLACE FUNCTION log_price_change()
RETURNS TRIGGER 
LANGUAGE plpgsql
AS $$
BEGIN
    IF NEW.price <> OLD.price THEN
        INSERT INTO price_change_log(
            product_id,
            old_price,
            new_price,
            change_date,
            change_user
        )
        VALUES(
            NEW.product_id,
            OLD.price,
            NEW.price,
            CURRENT_TIMESTAMP,
            CURRENT_USER
        );
    END IF;
    RETURN NEW;
END;
$$;

CREATE TRIGGER trg_product_price_change
AFTER UPDATE ON products
FOR EACH ROW
EXECUTE FUNCTION log_price_change();
```

Common trigger types:
- BEFORE or AFTER triggers
- Row-level (FOR EACH ROW) or statement-level triggers
- INSERT, UPDATE, DELETE triggers
- INSTEAD OF triggers (for views)

#### Dynamic SQL

Dynamic SQL generates and executes SQL statements at runtime:

```sql
-- PostgreSQL dynamic SQL with EXECUTE
CREATE OR REPLACE PROCEDURE dynamic_query(
    table_name TEXT,
    condition TEXT,
    out_record_count INT
)
LANGUAGE plpgsql
AS $$
DECLARE
    v_sql TEXT;
BEGIN
    v_sql := 'SELECT COUNT(*) FROM ' || quote_ident(table_name);
    
    IF condition IS NOT NULL AND condition <> '' THEN
        v_sql := v_sql || ' WHERE ' || condition;
    END IF;
    
    EXECUTE v_sql INTO out_record_count;
END;
$$;

-- Call with parameters
CALL dynamic_query('orders', 'order_date >= ''2023-01-01''', NULL);
```

Security warning: Dynamic SQL can be vulnerable to SQL injection. Always sanitize user input and use parameterized queries when possible.

## Activities

### Activity 1: Advanced Query Writing

Practice writing complex queries:

1. Write a query to find the top 3 customers by total purchase amount in each region, including their purchase history trend over the last 6 months.

2. Develop a query to identify products that consistently underperform (below average sales) across all regions.

3. Create a query that analyzes customer buying patterns, identifying which products are frequently purchased together.

4. Write a query to detect potential fraudulent transactions based on unusual buying patterns or transaction amounts.

Solution approach for task 1:

```sql
WITH monthly_purchases AS (
    -- Calculate monthly purchase totals per customer
    SELECT
        c.customer_id,
        c.name,
        c.region,
        DATE_TRUNC('month', o.order_date) AS month,
        SUM(o.amount) AS monthly_amount
    FROM customers c
    JOIN orders o ON c.customer_id = o.customer_id
    WHERE o.order_date >= CURRENT_DATE - INTERVAL '6 months'
    GROUP BY c.customer_id, c.name, c.region, DATE_TRUNC('month', o.order_date)
),
customer_totals AS (
    -- Calculate total purchase amount per customer
    SELECT
        customer_id,
        name,
        region,
        SUM(monthly_amount) AS total_amount
    FROM monthly_purchases
    GROUP BY customer_id, name, region
),
ranked_customers AS (
    -- Rank customers within each region
    SELECT
        customer_id,
        name,
        region,
        total_amount,
        RANK() OVER(PARTITION BY region ORDER BY total_amount DESC) AS region_rank
    FROM customer_totals
)
-- Select top 3 customers per region with monthly breakdown
SELECT
    r.customer_id,
    r.name,
    r.region,
    r.total_amount,
    mp.month,
    mp.monthly_amount,
    mp.monthly_amount / r.total_amount * 100 AS percentage_of_total
FROM ranked_customers r
JOIN monthly_purchases mp ON r.customer_id = mp.customer_id
WHERE r.region_rank <= 3
ORDER BY r.region, r.region_rank, mp.month;
```

### Activity 2: Performance Optimization

Optimize complex queries:

1. Analyze the following query and identify performance issues:

```sql
SELECT 
    c.name AS customer_name,
    COUNT(DISTINCT o.order_id) AS order_count,
    SUM(oi.quantity * oi.unit_price) AS total_amount,
    MAX(o.order_date) AS last_order_date,
    (SELECT COUNT(*) FROM order_items WHERE product_id IN 
        (SELECT product_id FROM products WHERE category = 'Electronics')
        AND order_id IN (SELECT order_id FROM orders WHERE customer_id = c.customer_id)
    ) AS electronics_count
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
LEFT JOIN order_items oi ON o.order_id = oi.order_id
WHERE o.order_date >= '2023-01-01'
GROUP BY c.customer_id, c.name
HAVING COUNT(DISTINCT o.order_id) > 0
ORDER BY total_amount DESC;
```

2. Rewrite the query to improve performance

3. Suggest appropriate indexes to support the query

4. Document your optimization approach and explain why it's more efficient

Optimized solution:

```sql
-- Create indexes first
CREATE INDEX idx_orders_customer_date ON orders(customer_id, order_date);
CREATE INDEX idx_order_items_order ON order_items(order_id);
CREATE INDEX idx_order_items_product ON order_items(product_id);
CREATE INDEX idx_products_category ON products(category);

-- Optimized query
WITH electronics_products AS (
    -- Materialize this once
    SELECT product_id
    FROM products
    WHERE category = 'Electronics'
),
customer_orders AS (
    -- Aggregate at the order level first
    SELECT
        o.customer_id,
        o.order_id,
        o.order_date,
        SUM(oi.quantity * oi.unit_price) AS order_amount
    FROM orders o
    JOIN order_items oi ON o.order_id = oi.order_id
    WHERE o.order_date >= '2023-01-01'
    GROUP BY o.customer_id, o.order_id, o.order_date
),
electronics_orders AS (
    -- Pre-calculate electronics items per order
    SELECT
        o.customer_id,
        o.order_id,
        COUNT(*) AS electronics_items
    FROM orders o
    JOIN order_items oi ON o.order_id = oi.order_id
    JOIN electronics_products ep ON oi.product_id = ep.product_id
    WHERE o.order_date >= '2023-01-01'
    GROUP BY o.customer_id, o.order_id
)
-- Final aggregation per customer
SELECT
    c.name AS customer_name,
    COUNT(co.order_id) AS order_count,
    SUM(co.order_amount) AS total_amount,
    MAX(co.order_date) AS last_order_date,
    COALESCE(SUM(eo.electronics_items), 0) AS electronics_count
FROM customers c
JOIN customer_orders co ON c.customer_id = co.customer_id
LEFT JOIN electronics_orders eo ON co.order_id = eo.order_id
GROUP BY c.customer_id, c.name
ORDER BY total_amount DESC;
```

## Best Practices

### Query Development
- **Readability**:
  - Use consistent indentation and capitalization
  - Break complex queries into CTEs
  - Add comments to explain complex logic
  - Use meaningful aliases

- **Maintainability**:
  - Prefer standard SQL features when possible
  - Document database-specific syntax
  - Create views for commonly used logic
  - Use stored procedures for complex business logic

- **Testability**:
  - Test with representative data volumes
  - Verify edge cases (NULL values, empty tables)
  - Compare results before and after optimization
  - Create automated tests for critical queries

### Performance
- **Indexing Strategy**:
  - Index columns used in JOIN, WHERE, and ORDER BY
  - Consider covering indexes for high-volume queries
  - Balance index benefits against write performance
  - Regularly review and tune indexes

- **Query Design**:
  - Filter early to reduce data volume
  - Use EXISTS instead of IN for better performance
  - Avoid functions on indexed columns in WHERE clauses
  - Limit result sets to what's actually needed

- **Statistics Management**:
  - Keep statistics up-to-date
  - Analyze tables after significant data changes
  - Monitor query plans for unexpected changes
  - Adjust database parameters based on workload

### Security

- **Access Control**:
  - Implement row-level security where needed
  - Use roles and privileges to enforce least privilege
  - Create views to limit data access
  - Audit sensitive data access

- **Query Safety**:
  - Use parameterized queries to prevent SQL injection
  - Validate and sanitize input for dynamic SQL
  - Set appropriate query timeout limits
  - Implement resource controls to prevent DoS

- **Data Protection**:
  - Encrypt sensitive data
  - Mask or redact sensitive data in query results
  - Implement data retention policies
  - Have a plan for secure data disposal

## Resources

- **Books**:
  - "High Performance SQL" by Baron Schwartz
  - "SQL Performance Explained" by Markus Winand
  - "SQL Antipatterns" by Bill Karwin
  - "SQL Cookbook" by Anthony Molinaro

- **Online Resources**:
  - [Use the Index, Luke](https://use-the-index-luke.com/) - Index optimization guide
  - [PostgreSQL Documentation](https://www.postgresql.org/docs/) - Comprehensive SQL reference
  - [SQL Performance Insights](https://sqlperformance.com/) - Advanced SQL performance articles
  - [Modern SQL](https://modern-sql.com/) - Examples of advanced SQL features

- **Tools**:
  - Execution plan analyzers (PostgreSQL's EXPLAIN ANALYZE, MySQL's EXPLAIN)
  - Index analyzers (pg_stat_statements, sys.dm_db_index_usage_stats)
  - Query repositories (version control for SQL)
  - Performance monitoring tools (Prometheus, Grafana)

## Conclusion

Advanced SQL skills are essential for data engineers who need to extract, transform, and optimize data efficiently. By mastering window functions, CTEs, complex joins, and performance tuning techniques, you'll be equipped to handle sophisticated data problems and build high-performance data pipelines.

Remember that SQL expertise is not just about knowing syntax—it's about understanding how databases work under the hood and making informed decisions about query design, indexing, and optimization. Continuously monitoring and tuning your SQL code is a crucial part of maintaining healthy data engineering systems.

As you practice these advanced techniques, focus on writing SQL that is not only correct and efficient but also maintainable and secure. The best SQL code balances performance with readability, ensuring that both computers and humans can understand and work with your queries effectively.