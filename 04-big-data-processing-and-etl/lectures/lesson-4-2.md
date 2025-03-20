# Lesson 4.2: Spark DataFrames and SQL

## Navigation
- [← Back to Lesson Plan](../4.2-spark-dataframes-sql.md)
- [← Back to Module Overview](../README.md)

## Overview
While RDDs provide a powerful foundation for distributed data processing, they require manual optimization and lack schema enforcement. Spark DataFrames and Spark SQL address these limitations by introducing a higher-level, structured data abstraction with advanced optimization. This lesson explores how DataFrames and SQL capabilities in Spark combine the flexibility of RDDs with the performance benefits of schema-aware processing, enabling more efficient and expressive data analysis.

## Learning Objectives
- Master Spark DataFrames API and operations
- Learn Spark SQL capabilities and syntax
- Understand DataFrame optimization techniques
- Practice common data transformation patterns
- Implement efficient joins and aggregations

## The Evolution from RDDs to DataFrames

The progression from RDDs to DataFrames represents a significant evolution in Spark's data processing model:

### Limitations of RDDs

While RDDs are powerful and flexible, they have several limitations:

1. **No Schema Awareness**: RDDs treat data as opaque objects, lacking understanding of the structure or schema of the data being processed.

2. **Limited Optimization**: Without schema information, the Spark engine cannot automatically optimize query execution.

3. **Verbose Code**: RDD operations often require more verbose code compared to SQL queries or DataFrame operations.

4. **Serialization Overhead**: Passing functions to RDD operations can introduce serialization overhead.

### DataFrames: A Structured Approach

DataFrames address these limitations by providing:

1. **Schema-Aware Processing**: DataFrames understand the structure of the data, allowing for more intelligent optimization.

2. **SQL-Like Operations**: Users can express transformations using familiar SQL-like syntax or a domain-specific language.

3. **Automatic Optimization**: The Catalyst optimizer automatically rewrites queries for optimal execution.

4. **Efficient Storage**: Column-based storage formats reduce memory usage and improve performance.

5. **Interoperability**: Easy conversion between RDDs, DataFrames, and Datasets.

## Spark DataFrame Fundamentals

A DataFrame in Spark is a distributed collection of data organized into named columns, conceptually equivalent to a table in a relational database or a data frame in R/Python.

### Creating DataFrames

DataFrames can be created from various sources:

#### 1. From Structured Files

```python
# From CSV
df_csv = spark.read.csv("data.csv", header=True, inferSchema=True)

# From JSON
df_json = spark.read.json("data.json")

# From Parquet
df_parquet = spark.read.parquet("data.parquet")
```

#### 2. From RDDs

```python
# From RDD with schema inference
rdd = spark.sparkContext.parallelize([(1, "Alice", 28), (2, "Bob", 35)])
df = spark.createDataFrame(rdd, ["id", "name", "age"])

# From RDD with explicit schema
from pyspark.sql.types import StructType, StructField, IntegerType, StringType

schema = StructType([
    StructField("id", IntegerType(), False),
    StructField("name", StringType(), False),
    StructField("age", IntegerType(), True)
])

df = spark.createDataFrame(rdd, schema)
```

#### 3. From Python Collections

```python
data = [
    {"id": 1, "name": "Alice", "age": 28},
    {"id": 2, "name": "Bob", "age": 35}
]
df = spark.createDataFrame(data)
```

#### 4. From External Databases

```python
# Using JDBC to connect to a database
df = spark.read \
    .format("jdbc") \
    .option("url", "jdbc:postgresql://localhost:5432/mydb") \
    .option("dbtable", "users") \
    .option("user", "username") \
    .option("password", "password") \
    .load()
```

### DataFrame Schema

A DataFrame's schema defines the column names, data types, and nullable status:

```python
# Print the schema
df.printSchema()

# Output:
# root
#  |-- id: integer (nullable = false)
#  |-- name: string (nullable = false)
#  |-- age: integer (nullable = true)
```

### Basic DataFrame Operations

DataFrames support a wide range of operations:

#### 1. Viewing Data

```python
# Show first 5 rows
df.show(5)

# Get summary statistics
df.describe().show()

# Display schema
df.printSchema()

# Count rows
df.count()
```

#### 2. Selecting and Filtering

```python
from pyspark.sql.functions import col

# Select specific columns
df.select("name", "age").show()

# Select with expressions
df.select(col("name"), col("age") + 1).show()

# Filter rows
df.filter(col("age") > 30).show()
df.filter("age > 30").show()  # SQL expression

# Distinct values
df.select("age").distinct().show()
```

#### 3. Adding and Modifying Columns

```python
from pyspark.sql.functions import lit, expr

# Add a constant column
df = df.withColumn("country", lit("USA"))

# Add a computed column
df = df.withColumn("age_in_months", col("age") * 12)

# Rename columns
df = df.withColumnRenamed("age", "years")

# Drop columns
df = df.drop("country")
```

## Advanced DataFrame Operations

Beyond basic operations, DataFrames offer powerful capabilities for complex data processing:

### Aggregations

```python
from pyspark.sql.functions import avg, count, sum, min, max

# Simple aggregation
df.groupBy("department").count().show()

# Multiple aggregations
df.groupBy("department").agg(
    count("*").alias("employee_count"),
    avg("salary").alias("avg_salary"),
    sum("salary").alias("total_salary"),
    min("age").alias("youngest"),
    max("age").alias("oldest")
).show()

# Aggregation with filter
df.groupBy("department").agg(
    count(when(col("gender") == "F", 1)).alias("female_count"),
    count(when(col("gender") == "M", 1)).alias("male_count")
).show()
```

### Joins

```python
# Inner join
joined_df = employees.join(departments, "department_id")

# Left outer join
joined_df = employees.join(
    departments, 
    "department_id", 
    "left_outer"
)

# Advanced join conditions
joined_df = employees.join(
    departments,
    (employees.department_id == departments.id) & 
    (employees.status == "Active"),
    "inner"
)
```

### Window Functions

Window functions allow calculations across rows related to the current row:

```python
from pyspark.sql.window import Window
from pyspark.sql.functions import rank, dense_rank, row_number, lead, lag

# Define window specification
window_spec = Window.partitionBy("department").orderBy("salary")

# Rank employees by salary within each department
df = df.withColumn("rank", rank().over(window_spec))
df = df.withColumn("dense_rank", dense_rank().over(window_spec))
df = df.withColumn("row_number", row_number().over(window_spec))

# Access preceding or following rows
df = df.withColumn("previous_salary", lag("salary", 1).over(window_spec))
df = df.withColumn("next_salary", lead("salary", 1).over(window_spec))

# Calculate running totals
df = df.withColumn(
    "running_total", 
    sum("salary").over(window_spec.rowsBetween(Window.unboundedPreceding, 0))
)
```

### Pivot and Unpivot

```python
# Pivot example: convert rows to columns
pivot_df = df.groupBy("department").pivot("year").agg(sum("revenue"))

# Unpivot example: convert columns to rows
from pyspark.sql.functions import expr

unpivot_expr = "stack(3, '2020', `2020`, '2021', `2021`, '2022', `2022`) as (year, revenue)"
unpivot_df = pivot_df.select("department", expr(unpivot_expr))
```

## Spark SQL

Spark SQL allows executing SQL queries directly against DataFrames:

### Temporary Views

To use SQL, you first register DataFrames as temporary views:

```python
# Register as temporary view
df.createOrReplaceTempView("employees")

# Register as global temporary view (available across sessions)
df.createOrReplaceGlobalTempView("global_employees")
```

### Running SQL Queries

```python
# Simple query
result = spark.sql("""
    SELECT department, AVG(salary) as avg_salary
    FROM employees
    GROUP BY department
    HAVING AVG(salary) > 50000
    ORDER BY avg_salary DESC
""")

# Join in SQL
result = spark.sql("""
    SELECT e.name, e.salary, d.department_name
    FROM employees e
    JOIN departments d ON e.department_id = d.id
    WHERE e.salary > 60000
""")
```

### SQL Subqueries

```python
result = spark.sql("""
    SELECT e.name, e.salary, e.department
    FROM employees e
    WHERE e.salary > (
        SELECT AVG(salary) * 1.5
        FROM employees
        WHERE department = e.department
    )
""")
```

### SQL Functions

```python
result = spark.sql("""
    SELECT
        name,
        salary,
        RANK() OVER (PARTITION BY department ORDER BY salary DESC) as rank,
        CASE
            WHEN salary > 100000 THEN 'High'
            WHEN salary > 70000 THEN 'Medium'
            ELSE 'Low'
        END as salary_category
    FROM employees
""")
```

## User-Defined Functions (UDFs)

UDFs allow extending Spark's functionality with custom logic:

### Registering Python UDFs

```python
from pyspark.sql.functions import udf
from pyspark.sql.types import StringType

# Define a Python function
def salary_category(salary):
    if salary > 100000:
        return "High"
    elif salary > 70000:
        return "Medium"
    else:
        return "Low"

# Register as UDF
salary_category_udf = udf(salary_category, StringType())

# Use in DataFrame API
df = df.withColumn("category", salary_category_udf(df.salary))

# Register for SQL
spark.udf.register("salary_category_sql", salary_category, StringType())

# Use in SQL
result = spark.sql("""
    SELECT name, salary, salary_category_sql(salary) as category
    FROM employees
""")
```

### Pandas UDFs for Performance

Pandas UDFs leverage vectorized operations for better performance:

```python
from pyspark.sql.functions import pandas_udf
from pyspark.sql.types import StringType
import pandas as pd

# Define Pandas UDF
@pandas_udf(StringType())
def salary_category_pandas(salary_series: pd.Series) -> pd.Series:
    return salary_series.apply(lambda x: 
        "High" if x > 100000 else "Medium" if x > 70000 else "Low"
    )

# Use in DataFrame API
df = df.withColumn("category", salary_category_pandas(df.salary))
```

## Optimization Techniques

Spark's Catalyst optimizer automatically optimizes queries, but understanding these optimizations helps write more efficient code:

### Catalyst Optimizer

The Catalyst optimizer transforms queries through several phases:
1. Analysis: Resolves column references and types
2. Logical optimization: Applies rule-based optimizations
3. Physical planning: Converts logical plan to physical execution plans
4. Code generation: Generates efficient Java bytecode

### Key Optimizations

#### 1. Predicate Pushdown

Predicates (filters) are pushed down to data sources, reducing the amount of data read:

```python
# Predicate pushdown will push the filter to the data source
df = spark.read.parquet("data.parquet").filter(col("age") > 30)
```

#### 2. Column Pruning

Only necessary columns are read from the data source:

```python
# Column pruning will only read 'name' and 'age' columns
df = spark.read.parquet("data.parquet").select("name", "age")
```

#### 3. Partition Pruning

When data is partitioned, Spark only reads relevant partitions:

```python
# Assuming data is partitioned by 'year'
df = spark.read.parquet("data.parquet").filter(col("year") == 2022)
```

#### 4. Join Optimization

Spark automatically selects the most efficient join strategy:

```python
# Broadcast join will be used if departments is small
joined_df = employees.join(broadcast(departments), "department_id")

# Sort-merge join for larger tables
joined_df = employees.join(customers, "customer_id")
```

## Best Practices for Performance

### 1. Choose Appropriate Data Formats

Columnar formats like Parquet or ORC significantly improve performance:

```python
# Write as Parquet for future use
df.write.parquet("data.parquet")

# Partition by frequently filtered columns
df.write.partitionBy("year", "month").parquet("data_partitioned.parquet")
```

### 2. Optimize Join Operations

```python
from pyspark.sql.functions import broadcast

# Use broadcast hint for small tables (< 10MB)
df_joined = df_large.join(broadcast(df_small), "join_key")

# Ensure join keys have the same data type
# Prefer equi-joins over more complex join conditions
```

### 3. Avoid Data Skew

```python
# Identify skewed keys
key_counts = df.groupBy("join_key").count().orderBy("count", ascending=False)

# Handle skew by salting and exploding
from pyspark.sql.functions import explode, lit, array
salt_factor = 8

# For skewed keys in joining large tables
df_large_salted = df_large.withColumn("salt", 
    when(col("join_key").isin(skewed_keys), 
         explode(array([lit(i) for i in range(salt_factor)]))).otherwise(0))
df_small_salted = df_small.withColumn("salt", 
    when(col("join_key").isin(skewed_keys),
         explode(array([lit(i) for i in range(salt_factor)]))).otherwise(0))

# Join on both the key and salt
df_joined = df_large_salted.join(df_small_salted, ["join_key", "salt"])
```

### 4. Manage Resources Appropriately

```python
# Set appropriate configurations
spark.conf.set("spark.sql.shuffle.partitions", 200)  # Default is 200
spark.conf.set("spark.executor.memory", "4g")
spark.conf.set("spark.driver.memory", "2g")
```

### 5. Cache Intermediary Results

```python
# Cache frequently accessed DataFrames
df_processed = df.filter(...).select(...).dropDuplicates()
df_processed.cache()

# Use the cached DataFrame in multiple operations
result1 = df_processed.groupBy(...).agg(...)
result2 = df_processed.join(other_df, ...)

# Unpersist when no longer needed
df_processed.unpersist()
```

## Common Pitfalls and Anti-Patterns

### 1. Collecting Large DataFrames to Driver

```python
# Bad: Collecting large datasets to driver
all_data = df.collect()  # May cause OOM if data is large

# Good: Process data in a distributed manner
summary = df.groupBy("department").count()
top_n = df.orderBy("salary", ascending=False).limit(10)
```

### 2. Row-at-a-Time Processing

```python
# Bad: Row-at-a-time processing with Python UDFs
def slow_function(x):
    # Process one row at a time
    return x + 1

# Good: Vectorized operations
from pyspark.sql.functions import pandas_udf
import pandas as pd

@pandas_udf("integer")
def fast_function(s: pd.Series) -> pd.Series:
    # Process entire series at once
    return s + 1
```

### 3. Excessive Shuffling

```python
# Bad: Multiple shuffles
df = df.repartition("key1")
df = df.groupBy("key2").agg(...)
df = df.join(other_df, "key3")

# Good: Plan operations to minimize shuffles
df = df.groupBy("key2").agg(...).join(other_df, "key3")
```

### 4. Over-caching

```python
# Bad: Caching everything
df1.cache()
df2.cache()
df3.cache()  # May lead to memory pressure

# Good: Cache strategically
df_expensive = df.groupBy(...).agg(...)  # Expensive computation
df_expensive.cache()  # Cache only this result
```

## Real-World Applications

### 1. Customer Analytics

```python
# Load customer data
customers = spark.read.parquet("customers.parquet")
transactions = spark.read.parquet("transactions.parquet")

# Join datasets
customer_transactions = customers.join(
    transactions, 
    customers.customer_id == transactions.customer_id
)

# Create customer metrics
customer_metrics = customer_transactions.groupBy("customer_id").agg(
    count("transaction_id").alias("transaction_count"),
    sum("amount").alias("total_spend"),
    avg("amount").alias("avg_transaction_value"),
    min("transaction_date").alias("first_purchase"),
    max("transaction_date").alias("latest_purchase")
)

# Segment customers
from pyspark.sql.functions import datediff, current_date

customer_segments = customer_metrics.withColumn(
    "days_since_last_purchase",
    datediff(current_date(), "latest_purchase")
).withColumn(
    "segment",
    when(col("transaction_count") > 10, "Frequent")
    .when((col("transaction_count") <= 10) & (col("transaction_count") > 5), "Regular")
    .when((col("transaction_count") <= 5) & (col("days_since_last_purchase") <= 90), "New")
    .otherwise("Churned")
)
```

### 2. Log Analysis

```python
# Parse log data
logs = spark.read.text("logs.txt")

# Extract fields using regular expressions
from pyspark.sql.functions import regexp_extract

parsed_logs = logs.select(
    regexp_extract("value", r"(\S+)", 1).alias("ip"),
    regexp_extract("value", r"\[([^\]]+)\]", 1).alias("timestamp"),
    regexp_extract("value", r"\"(\S+) (\S+) (\S+)\"", 1).alias("method"),
    regexp_extract("value", r"\"(\S+) (\S+) (\S+)\"", 2).alias("endpoint"),
    regexp_extract("value", r" (\d+) ", 1).cast("integer").alias("status"),
    regexp_extract("value", r" (\d+)$", 1).cast("integer").alias("bytes")
)

# Analyze errors
error_counts = parsed_logs.filter(col("status") >= 400).groupBy(
    "status", "endpoint"
).count().orderBy("count", ascending=False)

# Analyze traffic patterns
traffic_by_hour = parsed_logs.withColumn(
    "hour", date_format(to_timestamp("timestamp", "dd/MMM/yyyy:HH:mm:ss Z"), "HH")
).groupBy("hour", "endpoint").count().orderBy("hour", "count" desc)
```

## Conclusion

Spark DataFrames and SQL provide a powerful, efficient framework for processing structured data at scale. By leveraging schema information, the Catalyst optimizer, and declarative APIs, data engineers can write expressive code that automatically benefits from advanced optimization. These higher-level abstractions strike a balance between development productivity and execution performance, enabling complex data transformations with less code and better efficiency than RDD-based approaches.

In the next lesson, we'll explore how to build complete ETL pipelines using Spark, focusing on end-to-end data processing workflows from ingestion to delivery.

## Additional Resources

- [Spark SQL Programming Guide](https://spark.apache.org/docs/latest/sql-programming-guide.html)
- [DataFrame API Documentation](https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql.html)
- "Spark: The Definitive Guide" by Bill Chambers and Matei Zaharia
- [Databricks Blog: Catalyst Optimizer](https://databricks.com/blog/2015/04/13/deep-dive-into-spark-sqls-catalyst-optimizer.html)
- [Performance Tuning Guide](https://spark.apache.org/docs/latest/sql-performance-tuning.html) 