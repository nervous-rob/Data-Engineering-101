# Lesson 2.2: Advanced Python

## Navigation
- [← Back to Lesson Plan](../2.2-advanced-python.md)
- [← Back to Module Overview](../README.md)

## Overview
Building on the fundamentals of Python covered in the previous lesson, this lesson explores advanced Python concepts and libraries essential for data engineering. As data volumes grow and processing requirements become more complex, data engineers need sophisticated tools and techniques to build efficient, scalable, and maintainable data pipelines.

## Learning Objectives
- Master advanced Python libraries for data engineering
- Learn data processing techniques for large datasets
- Understand performance optimization strategies
- Practice advanced data manipulation patterns

## Advanced Python Libraries

### Pandas in Depth

Pandas has become the backbone of data manipulation in Python, offering powerful capabilities for working with structured data. For data engineers, understanding Pandas at a deeper level is crucial for processing datasets efficiently.

#### DataFrame Internals

At its core, a Pandas DataFrame consists of:
- An index (row labels)
- Column labels
- The data itself (typically NumPy arrays)
- Metadata (types, etc.)

This structure enables efficient slicing, filtering, and computation:

```python
import pandas as pd
import numpy as np

# Creating a DataFrame
df = pd.DataFrame({
    'int_col': [1, 2, 3, 4, 5],
    'float_col': [1.1, 2.2, 3.3, 4.4, 5.5],
    'str_col': ['a', 'b', 'c', 'd', 'e']
})

# Examining DataFrame structure
print(f"Index: {df.index}")
print(f"Columns: {df.columns}")
print(f"Data types: {df.dtypes}")
print(f"Memory usage: {df.memory_usage(deep=True)} bytes")
```

#### Efficient DataFrame Operations

Vectorized operations in Pandas dramatically outperform iterative approaches:

```python
# Inefficient: Iterating through rows
def add_columns_loop(df):
    results = []
    for i, row in df.iterrows():
        results.append(row['col1'] + row['col2'])
    return results

# Efficient: Vectorized operation
def add_columns_vectorized(df):
    return df['col1'] + df['col2']
```

For a DataFrame with millions of rows, the vectorized approach can be orders of magnitude faster.

#### Advanced Indexing

Sophisticated indexing capabilities enable complex data selection:

```python
# Multi-level indexing
df = pd.DataFrame({
    'date': pd.date_range('2023-01-01', periods=100),
    'category': np.random.choice(['A', 'B', 'C'], size=100),
    'subcategory': np.random.choice(['X', 'Y', 'Z'], size=100),
    'value': np.random.rand(100) * 100
})

# Create a multi-index DataFrame
indexed_df = df.set_index(['date', 'category', 'subcategory'])

# Slicing with multi-index
january_data = indexed_df.loc['2023-01-01':'2023-01-31']
category_a = indexed_df.xs('A', level='category')
specific_slice = indexed_df.loc[('2023-01-15', 'B', 'Z'), :]
```

#### Time Series Handling

Time series data is common in data engineering, and Pandas offers specialized functions:

```python
# Create time series data
dates = pd.date_range('2023-01-01', periods=365, freq='D')
ts_data = pd.Series(np.random.normal(10, 2, 365), index=dates)

# Resampling (e.g., daily to monthly)
monthly_avg = ts_data.resample('M').mean()

# Rolling operations
rolling_avg = ts_data.rolling(window=7).mean()

# Time-based filtering
q1_data = ts_data['2023-01-01':'2023-03-31']

# Shifting and lagging
previous_day = ts_data.shift(1)
next_day = ts_data.shift(-1)
day_over_day_change = ts_data - ts_data.shift(1)
```

#### Group Operations

GroupBy operations are fundamental for aggregation and transformation:

```python
# Sample sales data
sales = pd.DataFrame({
    'date': pd.date_range('2023-01-01', periods=1000),
    'product': np.random.choice(['A', 'B', 'C', 'D'], size=1000),
    'store': np.random.choice(['North', 'South', 'East', 'West'], size=1000),
    'units': np.random.randint(1, 50, size=1000),
    'price': np.random.uniform(10, 100, size=1000)
})

# Calculate revenue
sales['revenue'] = sales['units'] * sales['price']

# Basic groupby aggregations
product_sales = sales.groupby('product')['revenue'].sum()
store_product_sales = sales.groupby(['store', 'product'])['revenue'].sum()

# Multiple aggregations
summary = sales.groupby(['store', 'product']).agg({
    'revenue': ['sum', 'mean', 'count'],
    'units': ['sum', 'mean'],
    'price': ['mean', 'min', 'max']
})

# Group transformation (maintaining the original DataFrame shape)
sales['store_avg_price'] = sales.groupby('store')['price'].transform('mean')
sales['product_total_revenue'] = sales.groupby('product')['revenue'].transform('sum')

# Filter groups based on aggregations
high_volume_products = sales.groupby('product').filter(lambda x: x['units'].sum() > 5000)
```

#### Advanced Merging and Joining

Data engineering often involves combining data from multiple sources:

```python
# Sample tables
orders = pd.DataFrame({
    'order_id': range(1000, 1020),
    'customer_id': np.random.randint(1, 100, size=20),
    'order_date': pd.date_range('2023-01-01', periods=20),
    'amount': np.random.uniform(10, 500, size=20)
})

customers = pd.DataFrame({
    'customer_id': range(1, 120),
    'name': [f"Customer {i}" for i in range(1, 120)],
    'segment': np.random.choice(['Retail', 'Corporate', 'Home Office'], size=119)
})

# Basic merge (inner join by default)
order_details = pd.merge(orders, customers, on='customer_id')

# Different join types
left_join = pd.merge(orders, customers, on='customer_id', how='left')
right_join = pd.merge(orders, customers, on='customer_id', how='right')
outer_join = pd.merge(orders, customers, on='customer_id', how='outer')

# Merging on different columns
renamed_customers = customers.rename(columns={'customer_id': 'cid'})
merged = pd.merge(orders, renamed_customers, left_on='customer_id', right_on='cid')

# Indicators and suffixes
merged_with_indicator = pd.merge(
    orders, customers, on='customer_id', how='outer', indicator=True,
    suffixes=('_order', '_customer')
)

# Self-joins
employees = pd.DataFrame({
    'emp_id': range(1, 11),
    'name': [f"Employee {i}" for i in range(1, 11)],
    'manager_id': [None, 1, 1, 2, 2, 3, 3, 4, 4, 5]
})

emp_hierarchy = pd.merge(
    employees, employees, left_on='emp_id', right_on='manager_id',
    suffixes=('', '_manager')
).dropna(subset=['manager_id'])
```

#### Reshaping Data

Changing data structure is a common requirement in data pipelines:

```python
# Sample data
monthly_sales = pd.DataFrame({
    'year': [2022, 2022, 2022, 2023, 2023, 2023],
    'month': ['Jan', 'Feb', 'Mar', 'Jan', 'Feb', 'Mar'],
    'product_A': [100, 110, 120, 130, 140, 150],
    'product_B': [200, 210, 220, 230, 240, 250],
    'product_C': [300, 310, 320, 330, 340, 350]
})

# Wide to long format
sales_long = pd.melt(
    monthly_sales, 
    id_vars=['year', 'month'],
    value_vars=['product_A', 'product_B', 'product_C'],
    var_name='product',
    value_name='sales'
)

# Long to wide format
sales_wide = sales_long.pivot(
    index=['year', 'month'],
    columns='product',
    values='sales'
).reset_index()

# Cross-tabulation
cross_tab = pd.crosstab(sales_long['year'], sales_long['product'])

# Stacking and unstacking
stacked = sales_wide.set_index(['year', 'month']).stack()
unstacked = stacked.unstack()
```

#### Custom Aggregations with apply() and applymap()

For operations beyond built-in functions, custom functions can be applied:

```python
# Apply function to each column
def custom_summary(column):
    return pd.Series({
        'mean': column.mean(),
        'median': column.median(),
        'skew': column.skew(),
        'range': column.max() - column.min()
    })

column_stats = df.apply(custom_summary)

# Apply function elementwise
df_cleaned = df.applymap(lambda x: x.strip() if isinstance(x, str) else x)

# Apply function to specific axis with custom args
def percentile_diff(x, lower=25, upper=75):
    """Return difference between upper and lower percentiles."""
    return np.percentile(x, upper) - np.percentile(x, lower)

interquartile_ranges = df.apply(percentile_diff, axis=0)
group_iqr = df.groupby('category').apply(lambda x: percentile_diff(x['value']))
```

### NumPy Advanced Features

NumPy underpins much of the scientific Python ecosystem, including Pandas. Understanding its advanced features enables more efficient numeric computation.

#### Vectorization and Broadcasting

NumPy operations are optimized for entire arrays, eliminating the need for explicit loops:

```python
import numpy as np

# Create large arrays
a = np.random.rand(1000000)
b = np.random.rand(1000000)

# Vectorized operations (much faster than loops)
c = a + b
d = a * b
e = np.sin(a)
f = np.sqrt(b)

# Broadcasting (operating on arrays of different shapes)
matrix = np.random.rand(5, 3)
row_vector = np.array([1, 2, 3])
# Broadcast row_vector to each row of matrix
result = matrix * row_vector

column_vector = np.array([1, 2, 3, 4, 5]).reshape(5, 1)
# Broadcast column_vector to each column of matrix
result2 = matrix * column_vector
```

Broadcasting allows operations between arrays of different dimensions, avoiding unnecessary copies.

#### Advanced Indexing and Slicing

NumPy offers sophisticated ways to access array elements:

```python
# Boolean indexing
arr = np.random.randint(0, 100, size=20)
high_values = arr[arr > 50]

# Fancy indexing (using arrays of indices)
indices = [0, 2, 5, 7]
selected = arr[indices]

# Combined indexing
mask = arr > 30
indices = [1, 3, 5]
complex_selection = arr[mask][indices]

# Slicing with steps
every_second = arr[::2]
reversed_array = arr[::-1]

# Multi-dimensional indexing
matrix = np.random.rand(10, 10)
block = matrix[2:5, 3:7]
diagonal = np.diag(matrix)
```

#### Structured Arrays and Record Arrays

For complex data types, structured arrays provide efficient storage:

```python
# Define structured array
dtype = np.dtype([
    ('name', 'U20'),
    ('age', 'i4'),
    ('weight', 'f4'),
    ('active', 'bool')
])

people = np.array([
    ('Alice', 25, 65.5, True),
    ('Bob', 32, 75.2, True),
    ('Charlie', 45, 80.1, False)
], dtype=dtype)

# Access fields
names = people['name']
active_people = people[people['active']]

# Record arrays (attribute-style access)
people_records = people.view(np.recarray)
ages = people_records.age
```

#### Universal Functions (ufuncs)

Ufuncs operate element-wise on arrays with optimized performance:

```python
# Basic ufuncs
a = np.random.rand(1000000)
b = np.random.rand(1000000)

c = np.add(a, b)
d = np.multiply(a, b)
e = np.sin(a)
f = np.exp(b)

# Aggregation ufuncs
total = np.sum(a)
maximum = np.maximum(a, b)  # Element-wise maximum

# Custom ufuncs
from numba import vectorize

@vectorize
def custom_operation(x, y):
    return x * x + y * y

result = custom_operation(a, b)
```

#### Linear Algebra Operations

NumPy provides a comprehensive set of linear algebra functions:

```python
# Matrix operations
A = np.random.rand(3, 3)
B = np.random.rand(3, 3)

# Matrix multiplication
C = np.matmul(A, B)  # or A @ B in Python 3.5+

# Matrix inversion
A_inv = np.linalg.inv(A)

# Determinant
det_A = np.linalg.det(A)

# Eigenvalues and eigenvectors
eigenvalues, eigenvectors = np.linalg.eig(A)

# Solving linear equations Ax = b
b = np.random.rand(3)
x = np.linalg.solve(A, b)

# SVD decomposition
U, S, Vh = np.linalg.svd(A)
```

### PySpark Basics

For processing truly large datasets, Apache Spark is the industry standard. PySpark brings Spark's power to Python users.

#### Spark Context and Session

The entry point to Spark functionality:

```python
from pyspark.sql import SparkSession

# Create a SparkSession
spark = SparkSession \
    .builder \
    .appName("PySpark Example") \
    .config("spark.some.config.option", "some-value") \
    .getOrCreate()

# SparkContext is available as
sc = spark.sparkContext
```

#### Resilient Distributed Datasets (RDDs)

The foundational data structure in Spark:

```python
# Create an RDD from a list
data = [1, 2, 3, 4, 5]
rdd = sc.parallelize(data)

# Create an RDD from a file
text_file = sc.textFile("hdfs://input.txt")

# Basic transformations
mapped = rdd.map(lambda x: x * x)
filtered = rdd.filter(lambda x: x % 2 == 0)

# Actions
sum_result = rdd.reduce(lambda a, b: a + b)
collected = rdd.collect()
count = rdd.count()
```

#### Spark DataFrames

A higher-level API similar to Pandas but distributed:

```python
# Create a DataFrame from a list
data = [("Alice", 25), ("Bob", 30), ("Charlie", 35)]
df = spark.createDataFrame(data, ["name", "age"])

# Create a DataFrame from a file
csv_df = spark.read.csv("data.csv", header=True, inferSchema=True)
json_df = spark.read.json("data.json")
parquet_df = spark.read.parquet("data.parquet")

# Basic operations
df.show()
df.printSchema()
df.select("name").show()
df.filter(df["age"] > 30).show()
df.groupBy("age").count().show()

# Registering a temporary view for SQL
df.createOrReplaceTempView("people")
spark.sql("SELECT * FROM people WHERE age > 30").show()
```

#### Spark SQL

SQL interface for Spark:

```python
# Direct SQL queries
result = spark.sql("""
    SELECT 
        department,
        AVG(salary) as avg_salary,
        COUNT(*) as employee_count
    FROM employees
    WHERE status = 'Active'
    GROUP BY department
    HAVING COUNT(*) > 10
    ORDER BY avg_salary DESC
""")

# Complex queries with multiple tables
spark.sql("""
    SELECT 
        e.name,
        d.department_name,
        e.salary
    FROM employees e
    JOIN departments d ON e.department_id = d.id
    WHERE e.hire_date > '2020-01-01'
    ORDER BY e.salary DESC
    LIMIT 10
""").show()
```

#### DataFrame API Operations

The DataFrame API offers a rich set of transformations:

```python
from pyspark.sql import functions as F
from pyspark.sql.window import Window

# Column operations
df = df.withColumn("age_years", df["age"])
df = df.withColumn("is_adult", F.when(df["age"] >= 18, True).otherwise(False))

# String operations
df = df.withColumn("name_upper", F.upper(df["name"]))
df = df.withColumn("name_length", F.length(df["name"]))

# Date operations
df = df.withColumn("current_date", F.current_date())
df = df.withColumn("days_since_birth", F.datediff(F.current_date(), df["birthdate"]))

# Aggregations
summary = df.groupBy("department").agg(
    F.count("*").alias("employee_count"),
    F.avg("salary").alias("avg_salary"),
    F.min("salary").alias("min_salary"),
    F.max("salary").alias("max_salary")
)

# Window functions
window_spec = Window.partitionBy("department").orderBy(F.desc("salary"))
df = df.withColumn("dept_salary_rank", F.rank().over(window_spec))

# Joining DataFrames
employees = spark.table("employees")
departments = spark.table("departments")
joined = employees.join(departments, employees["dept_id"] == departments["id"])

# Writing data
df.write.mode("overwrite").parquet("output/data.parquet")
df.write.mode("append").partitionBy("year", "month").parquet("data/partitioned")
```

## Data Processing Techniques

Beyond individual libraries, data engineers need to master broader techniques for effective data processing.

### Data Cleaning

Messy data is a universal challenge in data engineering. Python offers robust tools for cleaning data.

#### Handling Missing Values

Strategies for missing data depend on context:

```python
import pandas as pd
import numpy as np

df = pd.read_csv("raw_data.csv")

# Identifying missing values
missing_count = df.isnull().sum()
missing_percentage = df.isnull().mean() * 100

# Handling strategies
# 1. Dropping missing values
df_dropped = df.dropna()  # Drop rows with any NaN
df_dropped_cols = df.dropna(axis=1)  # Drop columns with any NaN
df_thresh = df.dropna(thresh=5)  # Drop rows with < 5 non-NA values

# 2. Filling missing values
df_fill = df.fillna(0)  # Fill all NaN with 0
df_fill_dict = df.fillna({
    'numeric_col': 0,
    'string_col': 'unknown',
    'date_col': pd.Timestamp('2020-01-01')
})

# 3. Fill with statistics
df['numeric_col'] = df['numeric_col'].fillna(df['numeric_col'].mean())
df['category_col'] = df['category_col'].fillna(df['category_col'].mode()[0])

# 4. Fill using method
df['time_series'] = df['time_series'].fillna(method='ffill')  # Forward fill
df['time_series'] = df['time_series'].fillna(method='bfill')  # Backward fill

# 5. Interpolation
df['smooth_series'] = df['smooth_series'].interpolate(method='linear')
```

#### Data Type Conversion

Ensuring correct data types is a common cleaning task:

```python
# Checking data types
dtypes = df.dtypes

# Conversion to numeric
df['string_number'] = pd.to_numeric(df['string_number'], errors='coerce')

# Conversion to datetime
df['string_date'] = pd.to_datetime(df['string_date'], errors='coerce')

# Conversion to category (more efficient for repeated strings)
df['category_col'] = df['category_col'].astype('category')

# Complex conversions with apply
def convert_currency(value):
    if pd.isna(value):
        return np.nan
    # Remove currency symbol and commas, then convert to float
    if isinstance(value, str):
        return float(value.replace(', '').replace(',', ''))
    return value

df['price'] = df['price'].apply(convert_currency)
```

#### Duplicate Handling

Identifying and managing duplicates:

```python
# Find duplicates
duplicates = df.duplicated()
duplicate_count = duplicates.sum()

# Show duplicate rows
duplicate_rows = df[df.duplicated(keep=False)]

# Drop duplicates
df_unique = df.drop_duplicates()

# Drop duplicates based on specific columns
df_unique_subset = df.drop_duplicates(subset=['id', 'date'])

# Keep first or last occurrence
df_first = df.drop_duplicates(keep='first')  # Default
df_last = df.drop_duplicates(keep='last')
```

#### String Cleaning

Text data often requires specialized cleaning:

```python
# String methods in pandas
df['name'] = df['name'].str.strip()  # Remove leading/trailing whitespace
df['name'] = df['name'].str.lower()  # Convert to lowercase
df['name'] = df['name'].str.replace('[^a-zA-Z0-9]', '', regex=True)  # Remove special chars

# More complex string operations
df['full_name'] = df['first_name'].str.cat(df['last_name'], sep=' ')
df['domain'] = df['email'].str.extract(r'@([^.]+)')  # Extract domain part

# Standardizing categories
def standardize_category(category):
    category = category.lower().strip()
    if 'electronics' in category:
        return 'Electronics'
    elif 'clothing' in category or 'apparel' in category:
        return 'Clothing'
    # More mappings...
    else:
        return 'Other'

df['category_std'] = df['category'].apply(standardize_category)
```

#### Outlier Detection and Treatment

Outliers can significantly skew analysis:

```python
# Z-score method
from scipy import stats
z_scores = stats.zscore(df['value'])
abs_z_scores = np.abs(z_scores)
filtered_entries = (abs_z_scores < 3)  # Keep only those within 3 std dev
df_no_outliers = df[filtered_entries]

# IQR method
Q1 = df['value'].quantile(0.25)
Q3 = df['value'].quantile(0.75)
IQR = Q3 - Q1
df_iqr = df[~((df['value'] < (Q1 - 1.5 * IQR)) | (df['value'] > (Q3 + 1.5 * IQR)))]

# Capping outliers rather than removing
df['value_capped'] = np.where(
    df['value'] > Q3 + 1.5 * IQR,
    Q3 + 1.5 * IQR,
    np.where(
        df['value'] < Q1 - 1.5 * IQR,
        Q1 - 1.5 * IQR,
        df['value']
    )
)
```

#### Validation and Constraints

Enforcing data quality rules:

```python
# Define validation function
def validate_row(row):
    """Return True if row passes all validation rules."""
    if pd.isna(row['id']):
        return False
    if row['age'] < 0 or row['age'] > 120:
        return False
    if not pd.isna(row['email']) and '@' not in row['email']:
        return False
    return True

# Apply validation and filter
valid_mask = df.apply(validate_row, axis=1)
valid_df = df[valid_mask]
invalid_df = df[~valid_mask]

# Log invalid records
invalid_df.to_csv('invalid_records.csv', index=False)
```

### Data Transformation

Transforming data from its raw form to analysis-ready state is a core data engineering task.

#### Feature Engineering

Creating new features from existing data:

```python
# Date-based features
df['year'] = df['date'].dt.year
df['month'] = df['date'].dt.month
df['day_of_week'] = df['date'].dt.dayofweek
df['is_weekend'] = df['day_of_week'].isin([5, 6])
df['quarter'] = df['date'].dt.quarter

# Text-based features
df['name_length'] = df['name'].str.len()
df['word_count'] = df['description'].str.split().str.len()
df['contains_important'] = df['text'].str.contains('important', case=False)

# Numeric transformations
df['log_value'] = np.log1p(df['value'])  # log(1+x) to handle zeros
df['normalized'] = (df['value'] - df['value'].mean()) / df['value'].std()
df['binned'] = pd.cut(df['value'], bins=5, labels=False)

# Interaction features
df['area'] = df['length'] * df['width']
df['density'] = df['weight'] / df['volume']
df['ratio'] = df['feature1'] / df['feature2'].replace(0, 0.001)  # Avoid division by zero

# Aggregation features
df['rolling_mean'] = df.groupby('id')['value'].transform(
    lambda x: x.rolling(window=3, min_periods=1).mean()
)
```

#### Normalization and Scaling

Standardizing numeric features:

```python
from sklearn.preprocessing import MinMaxScaler, StandardScaler, RobustScaler

# Min-Max Scaling (to [0,1] range)
min_max_scaler = MinMaxScaler()
df['scaled_minmax'] = min_max_scaler.fit_transform(df[['value']])

# Z-score standardization (mean=0, std=1)
standard_scaler = StandardScaler()
df['scaled_standard'] = standard_scaler.fit_transform(df[['value']])

# Robust scaling (uses median and IQR, less sensitive to outliers)
robust_scaler = RobustScaler()
df['scaled_robust'] = robust_scaler.fit_transform(df[['value']])

# Log transformation (handles skewed distributions)
df['log_transformed'] = np.log1p(df['value'])

# Box-Cox transformation (requires positive values)
from scipy import stats
df['boxcox_transformed'], lambda_param = stats.boxcox(df['positive_value'])
```

#### Encoding Categorical Variables

Converting categories to numeric form:

```python
# One-hot encoding
dummies = pd.get_dummies(df['category'], prefix='category')
df = pd.concat([df, dummies], axis=1)

# Label encoding
from sklearn.preprocessing import LabelEncoder
label_encoder = LabelEncoder()
df['category_encoded'] = label_encoder.fit_transform(df['category'])

# Ordinal encoding (when categories have a natural order)
order_mapping = {'low': 0, 'medium': 1, 'high': 2}
df['priority_encoded'] = df['priority'].map(order_mapping)

# Target encoding (mean of target per category)
target_means = df.groupby('category')['target'].mean()
df['category_target_encoded'] = df['category'].map(target_means)

# Count encoding
count_encoding = df['category'].value_counts()
df['category_count'] = df['category'].map(count_encoding)
```

#### Time Series Processing

Working with temporal data:

```python
# Convert to datetime
df['date'] = pd.to_datetime(df['date'])
df = df.set_index('date')

# Resampling
daily = df.resample('D').sum()
monthly = df.resample('M').mean()
quarterly = df.resample('Q').mean()

# Rolling calculations
df['rolling_mean_7d'] = df['value'].rolling('7D').mean()
df['rolling_max_30d'] = df['value'].rolling('30D').max()
df['expanding_mean'] = df['value'].expanding().mean()

# Lagged features
for lag in [1, 7, 28]:
    df[f'lag_{lag}d'] = df['value'].shift(lag)

# Percent changes
df['daily_change'] = df['value'].pct_change()
df['weekly_change'] = df['value'].pct_change(periods=7)

# Calendar features extraction
df['year'] = df.index.year
df['month'] = df.index.month
df['day'] = df.index.day
df['day_of_week'] = df.index.dayofweek
df['is_month_end'] = df.index.is_month_end
```

### Performance Optimization

As data volumes grow, optimization becomes increasingly important.

#### Vectorization

Preferring vectorized operations over loops:

```python
# Slow: Loop-based approach
def calculate_metrics_loop(df):
    results = []
    for i, row in df.iterrows():
        result = (row['value1'] * row['value2']) / (row['value3'] + 0.001)
        results.append(result)
    return results

# Fast: Vectorized approach
def calculate_metrics_vectorized(df):
    return (df['value1'] * df['value2']) / (df['value3'] + 0.001)

# Example with complex condition
# Slow:
result = []
for i, row in df.iterrows():
    if row['condition1'] > 10 and row['condition2'] == 'active':
        result.append(row['value1'] * 2)
    else:
        result.append(row['value1'])

# Fast:
mask = (df['condition1'] > 10) & (df['condition2'] == 'active')
df['result'] = df['value1']
df.loc[mask, 'result'] *= 2
```

#### Efficient Memory Usage

Managing memory for large datasets:

```python
# Check memory usage
memory_usage = df.memory_usage(deep=True)
total_memory_mb = memory_usage.sum() / 1024 / 1024
print(f"Total memory usage: {total_memory_mb:.2f} MB")

# Optimize data types
# Convert float64 to float32
float_columns = df.select_dtypes(include=['float64']).columns
df[float_columns] = df[float_columns].astype('float32')

# Convert int64 to smaller integers where possible
int_columns = df.select_dtypes(include=['int64']).columns
for col in int_columns:
    max_val = df[col].max()
    min_val = df[col].min()
    
    if min_val >= 0:
        if max_val < 2**8:
            df[col] = df[col].astype('uint8')
        elif max_val < 2**16:
            df[col] = df[col].astype('uint16')
        elif max_val < 2**32:
            df[col] = df[col].astype('uint32')
    else:
        if min_val > -2**7 and max_val < 2**7:
            df[col] = df[col].astype('int8')
        elif min_val > -2**15 and max_val < 2**15:
            df[col] = df[col].astype('int16')
        elif min_val > -2**31 and max_val < 2**31:
            df[col] = df[col].astype('int32')

# Use category for string columns with repeated values
for col in df.select_dtypes(include=['object']):
    num_unique = df[col].nunique()
    if num_unique < len(df) * 0.5:  # If less than 50% unique values
        df[col] = df[col].astype('category')

# Process data in chunks for large files
chunk_iter = pd.read_csv('large_file.csv', chunksize=100000)
processed_chunks = []

for chunk in chunk_iter:
    # Process each chunk
    processed_chunk = process_function(chunk)
    processed_chunks.append(processed_chunk)

# Combine results if needed
result = pd.concat(processed_chunks)
```

#### Parallel Processing

Leveraging multiple cores for faster computation:

```python
import concurrent.futures
import multiprocessing

# Parallel processing with concurrent.futures
def process_file(filename):
    # Process a single file
    data = pd.read_csv(filename)
    result = complex_processing(data)
    return result

file_list = ['file1.csv', 'file2.csv', 'file3.csv', 'file4.csv']

# Process files in parallel
with concurrent.futures.ProcessPoolExecutor(max_workers=4) as executor:
    results = list(executor.map(process_file, file_list))

# Parallel processing with joblib
from joblib import Parallel, delayed

def process_chunk(chunk_df):
    # Process a chunk of data
    result = complex_calculation(chunk_df)
    return result

# Split DataFrame into chunks
chunks = np.array_split(df, multiprocessing.cpu_count())

# Process in parallel
results = Parallel(n_jobs=multiprocessing.cpu_count())(
    delayed(process_chunk)(chunk) for chunk in chunks
)

# Combine results
final_result = pd.concat(results)
```

#### Caching

Saving intermediate results to avoid redundant computation:

```python
import functools
from joblib import Memory

# Using Python's built-in lru_cache
@functools.lru_cache(maxsize=128)
def expensive_computation(x):
    # Simulating expensive operation
    import time
    time.sleep(1)
    return x * x

# Test caching
expensive_computation(10)  # Takes 1 second
expensive_computation(10)  # Instant, returns cached result

# Using joblib for disk-based caching
memory = Memory('cache_dir', verbose=0)

@memory.cache
def expensive_data_operation(filename):
    # Expensive data loading and processing
    data = pd.read_csv(filename)
    result = complex_processing(data)
    return result

# First call: computes and caches
result1 = expensive_data_operation('large_file.csv')
# Second call: loads from cache
result2 = expensive_data_operation('large_file.csv')
```

### Best Practices

Establishing good practices ensures code is maintainable, efficient, and robust.

#### Code Style and Organization

Following Python conventions:

```python
# Module-level imports (standard ordering)
import os
import sys
from typing import Dict, List, Optional, Union

import numpy as np
import pandas as pd
from sklearn.preprocessing import StandardScaler

# Constants in UPPERCASE
MAX_RETRIES = 3
DEFAULT_TIMEOUT = 60
DATA_PATH = os.path.join('data', 'raw')

# Class definition with docstring
class DataProcessor:
    """
    Process data files for analysis.
    
    This class handles loading, preprocessing, and transformation
    of data files into analysis-ready DataFrames.
    """
    
    def __init__(self, config_path: str):
        """Initialize the processor with configuration."""
        self.config = self._load_config(config_path)
        self.data = None
    
    def _load_config(self, path: str) -> Dict:
        """
        Load configuration from file.
        
        Parameters
        ----------
        path : str
            Path to configuration file
            
        Returns
        -------
        Dict
            Configuration parameters
        """
        # Implementation...
        pass
    
    def load_data(self, file_path: str) -> pd.DataFrame:
        """Load data from file."""
        # Implementation...
        pass
    
    def preprocess(self) -> None:
        """Apply preprocessing steps to loaded data."""
        # Implementation...
        pass
```

#### Error Handling

Robust error handling patterns:

```python
def process_file(filepath):
    try:
        # Attempt to process file
        with open(filepath, 'r') as f:
            data = json.load(f)
        
        # Process data
        result = transform_data(data)
        return result
    
    except FileNotFoundError:
        logging.error(f"File not found: {filepath}")
        # Handle missing file (e.g., use default data)
        return None
    
    except json.JSONDecodeError as e:
        logging.error(f"Invalid JSON in {filepath}: {e}")
        # Handle invalid file format
        raise ValueError(f"Invalid file format: {filepath}") from e
    
    except Exception as e:
        logging.exception(f"Unexpected error processing {filepath}")
        # Re-raise with context
        raise ProcessingError(f"Failed to process {filepath}") from e
    
    finally:
        # Cleanup code that always runs
        clean_temporary_files()
```

#### Modular Design

Designing reusable components:

```python
# utils.py - Utility functions
def read_data(file_path, file_type=None):
    """Read data from various file formats."""
    if file_type is None:
        file_type = file_path.split('.')[-1].lower()
    
    if file_type == 'csv':
        return pd.read_csv(file_path)
    elif file_type == 'json':
        return pd.read_json(file_path)
    elif file_type == 'parquet':
        return pd.read_parquet(file_path)
    else:
        raise ValueError(f"Unsupported file type: {file_type}")

# transformers.py - Data transformation components
class DateTransformer:
    """Process date fields in data."""
    
    def __init__(self, date_format=None):
        self.date_format = date_format
    
    def transform(self, df, date_columns):
        """Convert date columns to datetime."""
        for col in date_columns:
            if col in df.columns:
                if self.date_format:
                    df[col] = pd.to_datetime(df[col], format=self.date_format)
                else:
                    df[col] = pd.to_datetime(df[col])
        return df

# pipeline.py - Pipeline orchestration
class DataPipeline:
    """Orchestrate data processing steps."""
    
    def __init__(self, transformers=None):
        self.transformers = transformers or []
    
    def add_transformer(self, transformer):
        self.transformers.append(transformer)
    
    def run(self, data):
        """Run the pipeline on input data."""
        result = data.copy()
        for transformer in self.transformers:
            result = transformer.transform(result)
        return result

# Example usage
from utils import read_data
from transformers import DateTransformer
from pipeline import DataPipeline

# Create pipeline
pipeline = DataPipeline()
pipeline.add_transformer(DateTransformer())

# Process data
data = read_data('input.csv')
processed_data = pipeline.run(data)
```

## Practical Applications in Data Engineering

Let's explore some common data engineering scenarios using advanced Python.

### ETL Pipeline with Pandas and SQLAlchemy

```python
import pandas as pd
from sqlalchemy import create_engine
import logging

# Configure logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s',
    filename='etl_pipeline.log'
)

def extract_from_csv(file_path):
    """Extract data from CSV file."""
    try:
        logging.info(f"Extracting data from {file_path}")
        return pd.read_csv(file_path)
    except Exception as e:
        logging.error(f"Error extracting data from {file_path}: {e}")
        raise

def extract_from_database(connection_string, query):
    """Extract data from database."""
    try:
        logging.info("Extracting data from database")
        engine = create_engine(connection_string)
        return pd.read_sql(query, engine)
    except Exception as e:
        logging.error(f"Error extracting data from database: {e}")
        raise

def transform_data(df):
    """Apply transformations to data."""
    logging.info("Transforming data")
    
    try:
        # 1. Clean data
        # Handle missing values
        df['numeric_col'] = df['numeric_col'].fillna(df['numeric_col'].median())
        df['category_col'] = df['category_col'].fillna('unknown')
        
        # Convert types
        df['date_col'] = pd.to_datetime(df['date_col'])
        
        # 2. Feature engineering
        # Extract date components
        df['year'] = df['date_col'].dt.year
        df['month'] = df['date_col'].dt.month
        df['day_of_week'] = df['date_col'].dt.dayofweek
        
        # Create new features
        df['amount_category'] = pd.cut(
            df['amount'], 
            bins=[0, 100, 1000, float('inf')],
            labels=['small', 'medium', 'large']
        )
        
        # 3. Aggregations
        # Group by category and calculate statistics
        agg_data = df.groupby('category').agg({
            'amount': ['sum', 'mean', 'count'],
            'numeric_col': ['mean', 'median']
        })
        
        # Flatten column names
        agg_data.columns = ['_'.join(col).strip() for col in agg_data.columns.values]
        agg_data = agg_data.reset_index()
        
        logging.info(f"Transformation complete. Output shape: {df.shape}")
        
        return df, agg_data
    
    except Exception as e:
        logging.error(f"Error transforming data: {e}")
        raise

def load_to_database(df, table_name, connection_string, if_exists='replace'):
    """Load data to database."""
    try:
        logging.info(f"Loading data to {table_name}")
        engine = create_engine(connection_string)
        df.to_sql(table_name, engine, if_exists=if_exists, index=False)
        logging.info(f"Successfully loaded {len(df)} rows to {table_name}")
    except Exception as e:
        logging.error(f"Error loading data to {table_name}: {e}")
        raise

def etl_pipeline():
    """Run the complete ETL pipeline."""
    try:
        # Define parameters
        input_csv = 'data/transactions.csv'
        db_connection = 'postgresql://username:password@localhost:5432/mydatabase'
        
        # Extract data
        data = extract_from_csv(input_csv)
        
        # Transform data
        transformed_data, aggregated_data = transform_data(data)
        
        # Load data
        load_to_database(transformed_data, 'transactions_processed', db_connection)
        load_to_database(aggregated_data, 'transactions_summary', db_connection)
        
        logging.info("ETL pipeline completed successfully")
        return True
    
    except Exception as e:
        logging.error(f"ETL pipeline failed: {e}")
        return False

if __name__ == "__main__":
    success = etl_pipeline()
    print(f"Pipeline completed {'successfully' if success else 'with errors'}")
```

### Data Quality Monitoring System

```python
import pandas as pd
import numpy as np
import logging
from datetime import datetime
from sqlalchemy import create_engine
import json
import smtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart

# Configure logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s',
    filename=f'data_quality_{datetime.now().strftime("%Y%m%d")}.log'
)

class DataQualityCheck:
    """Base class for data quality checks."""
    
    def __init__(self, name, description, severity='medium'):
        self.name = name
        self.description = description
        self.severity = severity  # 'low', 'medium', 'high', 'critical'
    
    def run(self, df):
        """Run the check on the given DataFrame."""
        raise NotImplementedError("Subclasses must implement run()")
    
    def create_result(self, success, details=None):
        """Create a result dictionary."""
        return {
            'check_name': self.name,
            'description': self.description,
            'severity': self.severity,
            'success': success,
            'details': details or {},
            'timestamp': datetime.now().isoformat()
        }

class NullCheck(DataQualityCheck):
    """Check percentage of null values in columns."""
    
    def __init__(self, name, columns, threshold=0.05, severity='medium'):
        super().__init__(name, f"Check null values < {threshold*100}% in {columns}", severity)
        self.columns = columns if isinstance(columns, list) else [columns]
        self.threshold = threshold
    
    def run(self, df):
        results = {}
        overall_success = True
        
        for col in self.columns:
            if col not in df.columns:
                results[col] = {'present': False}
                overall_success = False
                continue
                
            null_pct = df[col].isnull().mean()
            success = null_pct <= self.threshold
            if not success:
                overall_success = False
                
            results[col] = {
                'null_percentage': null_pct,
                'success': success
            }
        
        return self.create_result(overall_success, results)

class UniqueValueCheck(DataQualityCheck):
    """Check uniqueness of values in columns."""
    
    def __init__(self, name, columns, must_be_unique=True, severity='high'):
        super().__init__(name, f"Check uniqueness in {columns}", severity)
        self.columns = columns if isinstance(columns, list) else [columns]
        self.must_be_unique = must_be_unique
    
    def run(self, df):
        results = {}
        overall_success = True
        
        for col in self.columns:
            if col not in df.columns:
                results[col] = {'present': False}
                overall_success = False
                continue
                
            total_rows = len(df)
            unique_values = df[col].nunique()
            duplicate_count = total_rows - unique_values
            success = (duplicate_count == 0) if self.must_be_unique else True
            
            if not success:
                overall_success = False
                
            results[col] = {
                'total_rows': total_rows,
                'unique_values': unique_values,
                'duplicate_count': duplicate_count,
                'success': success
            }
        
        return self.create_result(overall_success, results)

class ValueRangeCheck(DataQualityCheck):
    """Check if values fall within acceptable range."""
    
    def __init__(self, name, column, min_value=None, max_value=None, severity='medium'):
        desc = f"Check {column} values"
        if min_value is not None:
            desc += f" >= {min_value}"
        if min_value is not None and max_value is not None:
            desc += " and"
        if max_value is not None:
            desc += f" <= {max_value}"
            
        super().__init__(name, desc, severity)
        self.column = column
        self.min_value = min_value
        self.max_value = max_value
    
    def run(self, df):
        if self.column not in df.columns:
            return self.create_result(False, {'present': False})
            
        column_data = df[self.column]
        
        results = {
            'count': len(column_data),
            'min': column_data.min(),
            'max': column_data.max(),
            'mean': column_data.mean() if pd.api.types.is_numeric_dtype(column_data) else None
        }
        
        min_check = True if self.min_value is None else (column_data >= self.min_value).all()
        max_check = True if self.max_value is None else (column_data <= self.max_value).all()
        success = min_check and max_check
        
        if not min_check:
            count_below = (column_data < self.min_value).sum()
            results['below_min'] = {
                'count': count_below,
                'percentage': count_below / len(column_data) * 100
            }
            
        if not max_check:
            count_above = (column_data > self.max_value).sum()
            results['above_max'] = {
                'count': count_above,
                'percentage': count_above / len(column_data) * 100
            }
        
        results['success'] = success
        return self.create_result(success, results)

class DataQualityMonitor:
    """Monitor data quality by running multiple checks."""
    
    def __init__(self):
        self.checks = []
        self.notification_handlers = []
    
    def add_check(self, check):
        """Add a data quality check."""
        self.checks.append(check)
    
    def add_notification_handler(self, handler):
        """Add a notification handler."""
        self.notification_handlers.append(handler)
    
    def run_checks(self, df):
        """Run all checks on the given DataFrame."""
        logging.info(f"Running {len(self.checks)} data quality checks on DataFrame with shape {df.shape}")
        results = []
        
        for check in self.checks:
            try:
                logging.info(f"Running check: {check.name}")
                result = check.run(df)
                results.append(result)
                
                # Log results
                if result['success']:
                    logging.info(f"Check '{check.name}' passed")
                else:
                    logging.warning(f"Check '{check.name}' failed: {result['details']}")
                
            except Exception as e:
                logging.error(f"Error running check '{check.name}': {e}")
                results.append({
                    'check_name': check.name,
                    'description': check.description,
                    'severity': check.severity,
                    'success': False,
                    'details': {'error': str(e)},
                    'timestamp': datetime.now().isoformat()
                })
        
        # Send notifications for failed checks
        failed_checks = [r for r in results if not r['success']]
        if failed_checks and self.notification_handlers:
            for handler in self.notification_handlers:
                try:
                    handler(failed_checks)
                except Exception as e:
                    logging.error(f"Error sending notification: {e}")
        
        return results
    
    def save_results(self, results, output_path):
        """Save results to a JSON file."""
        try:
            with open(output_path, 'w') as f:
                json.dump(results, f, indent=2)
            logging.info(f"Results saved to {output_path}")
        except Exception as e:
            logging.error(f"Error saving results to {output_path}: {e}")

def email_notification(failed_checks):
    """Send email notification for failed checks."""
    # Email configuration
    smtp_server = 'smtp.example.com'
    smtp_port = 587
    sender_email = 'monitoring@example.com'
    receiver_email = 'data-team@example.com'
    password = 'your-password'  # In production, use secure methods to store credentials
    
    # Create message
    msg = MIMEMultipart()
    msg['Subject'] = f'Data Quality Alert: {len(failed_checks)} checks failed'
    msg['From'] = sender_email
    msg['To'] = receiver_email
    
    # Create message body
    body = f"<h2>Data Quality Checks Failed</h2>"
    body += f"<p>{len(failed_checks)} checks failed. Details below:</p>"
    body += "<table border='1'>"
    body += "<tr><th>Check</th><th>Description</th><th>Severity</th><th>Details</th></tr>"
    
    for check in failed_checks:
        body += f"<tr>"
        body += f"<td>{check['check_name']}</td>"
        body += f"<td>{check['description']}</td>"
        body += f"<td>{check['severity']}</td>"
        body += f"<td>{json.dumps(check['details'], indent=2)}</td>"
        body += f"</tr>"
    
    body += "</table>"
    
    msg.attach(MIMEText(body, 'html'))
    
    # Send email
    try:
        server = smtplib.SMTP(smtp_server, smtp_port)
        server.starttls()
        server.login(sender_email, password)
        server.send_message(msg)
        server.quit()
        logging.info("Email notification sent successfully")
    except Exception as e:
        logging.error(f"Error sending email notification: {e}")

# Example usage
def monitor_database_quality():
    """Example of monitoring database table quality."""
    try:
        # Create database connection
        connection_string = 'postgresql://username:password@localhost:5432/mydatabase'
        engine = create_engine(connection_string)
        
        # Load data
        query = "SELECT * FROM transactions WHERE created_at >= CURRENT_DATE - INTERVAL '1 day'"
        df = pd.read_sql(query, engine)
        
        # Initialize monitor
        monitor = DataQualityMonitor()
        
        # Add checks
        monitor.add_check(NullCheck("transaction_id_null", "transaction_id", threshold=0, severity="critical"))
        monitor.add_check(NullCheck("amount_null", "amount", threshold=0.01, severity="high"))
        monitor.add_check(UniqueValueCheck("transaction_id_unique", "transaction_id", must_be_unique=True, severity="critical"))
        monitor.add_check(ValueRangeCheck("amount_range", "amount", min_value=0, severity="high"))
        monitor.add_check(ValueRangeCheck("item_count_range", "item_count", min_value=1, max_value=1000, severity="medium"))
        
        # Add notification handler
        monitor.add_notification_handler(email_notification)
        
        # Run checks
        results = monitor.run_checks(df)
        
        # Save results
        output_path = f"quality_results_{datetime.now().strftime('%Y%m%d_%H%M%S')}.json"
        monitor.save_results(results, output_path)
        
        return results
    
    except Exception as e:
        logging.error(f"Error in quality monitoring: {e}")
        raise

if __name__ == "__main__":
    results = monitor_database_quality()
    print(f"Monitoring complete. {sum(1 for r in results if r['success'])} checks passed, "
          f"{sum(1 for r in results if not r['success'])} checks failed.")
```

### Streaming Data Processing with Pandas and PySpark

```python
import pandas as pd
import json
import time
from datetime import datetime
import logging
from pyspark.sql import SparkSession
from pyspark.sql.functions import window, count, avg, col, from_json, to_timestamp
from pyspark.sql.types import StructType, StructField, StringType, DoubleType, TimestampType

# Configure logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s',
    filename=f'streaming_{datetime.now().strftime("%Y%m%d")}.log'
)

def process_batch_with_pandas(data_batch):
    """Process a batch of data using Pandas."""
    try:
        # Convert to DataFrame
        df = pd.DataFrame(data_batch)
        
        # Parse timestamps
        df['timestamp'] = pd.to_datetime(df['timestamp'])
        
        # Apply transformations
        df['value_normalized'] = (df['value'] - df['value'].mean()) / df['value'].std()
        
        # Calculate metrics
        metrics = {
            'count': len(df),
            'avg_value': df['value'].mean(),
            'min_value': df['value'].min(),
            'max_value': df['value'].max(),
            'std_value': df['value'].std(),
            'timestamp': datetime.now().isoformat()
        }
        
        # Aggregate by category
        category_aggs = df.groupby('category').agg({
            'value': ['count', 'mean', 'min', 'max']
        })
        
        # Flatten column names
        category_aggs.columns = ['_'.join(col).strip() for col in category_aggs.columns.values]
        category_aggs = category_aggs.reset_index()
        
        # Return processed data and metrics
        return df, metrics, category_aggs
    
    except Exception as e:
        logging.error(f"Error processing batch with Pandas: {e}")
        raise

def simulate_data_stream(n_batches=10, batch_size=100, delay=1):
    """Simulate a streaming data source."""
    categories = ['A', 'B', 'C', 'D']
    
    for batch in range(n_batches):
        batch_data = []
        
        for i in range(batch_size):
            timestamp = datetime.now().isoformat()
            category = categories[i % len(categories)]
            value = np.random.normal(10, 2) + (i % len(categories)) * 5
            
            record = {
                'id': f"{batch}-{i}",
                'timestamp': timestamp,
                'category': category,
                'value': value
            }
            
            batch_data.append(record)
        
        logging.info(f"Generated batch {batch+1}/{n_batches} with {len(batch_data)} records")
        yield batch_data
        time.sleep(delay)

def process_stream_with_pandas():
    """Process a stream of data using Pandas."""
    try:
        logging.info("Starting Pandas streaming process")
        
        # Metrics storage
        all_metrics = []
        processed_count = 0
        
        # Process each batch
        for batch_idx, batch_data in enumerate(simulate_data_stream()):
            logging.info(f"Processing batch {batch_idx+1}")
            
            # Process batch
            df, metrics, category_aggs = process_batch_with_pandas(batch_data)
            
            # Store metrics
            all_metrics.append(metrics)
            processed_count += len(df)
            
            # Example: Output aggregated data
            print(f"Batch {batch_idx+1} metrics:")
            print(f"Records: {len(df)}")
            print(f"Average value: {metrics['avg_value']:.2f}")
            print("Category aggregations:")
            print(category_aggs)
            print("-" * 50)
        
        logging.info(f"Pandas streaming process complete. Processed {processed_count} records.")
        
        # Compute overall metrics
        all_metrics_df = pd.DataFrame(all_metrics)
        overall_metrics = {
            'total_records': all_metrics_df['count'].sum(),
            'avg_value': all_metrics_df['avg_value'].mean(),
            'min_value': all_metrics_df['min_value'].min(),
            'max_value': all_metrics_df['max_value'].max(),
        }
        
        return overall_metrics
    
    except Exception as e:
        logging.error(f"Error in Pandas stream processing: {e}")
        raise

def process_stream_with_spark():
    """Process a stream of data using Spark Structured Streaming."""
    try:
        # Initialize Spark session
        spark = SparkSession.builder \
            .appName("StreamProcessing") \
            .config("spark.sql.shuffle.partitions", "2") \
            .getOrCreate()
        
        logging.info("Starting Spark structured streaming process")
        
        # Define schema for JSON data
        schema = StructType([
            StructField("id", StringType(), True),
            StructField("timestamp", StringType(), True),
            StructField("category", StringType(), True),
            StructField("value", DoubleType(), True)
        ])
        
        # Read from socket source (for example)
        # In a real scenario, this might be Kafka, files, etc.
        stream_df = spark.readStream \
            .format("socket") \
            .option("host", "localhost") \
            .option("port", 9999) \
            .load()
        
        # Parse JSON data
        parsed_df = stream_df.select(
            from_json(col("value").cast("string"), schema).alias("data")
        ).select("data.*")
        
        # Convert timestamp
        df_with_timestamp = parsed_df \
            .withColumn("event_time", to_timestamp(col("timestamp")))
        
        # Basic stream processing
        # 1. Windowed aggregations
        windowed_counts = df_with_timestamp \
            .withWatermark("event_time", "10 seconds") \
            .groupBy(
                window(col("event_time"), "1 minute"),
                col("category")
            ) \
            .agg(
                count("*").alias("count"),
                avg("value").alias("avg_value")
            )
        
        # 2. Global running aggregates
        global_stats = df_with_timestamp \
            .groupBy() \
            .agg(
                count("*").alias("total_count"),
                avg("value").alias("overall_avg")
            )
        
        # 3. Category-wise aggregates
        category_stats = df_with_timestamp \
            .groupBy("category") \
            .agg(
                count("*").alias("count"),
                avg("value").alias("avg_value")
            )
        
        # Start the query to display windowed counts
        query1 = windowed_counts.writeStream \
            .outputMode("complete") \
            .format("console") \
            .option("truncate", "false") \
            .start()
        
        # Start the query for category stats
        query2 = category_stats.writeStream \
            .outputMode("complete") \
            .format("console") \
            .option("truncate", "false") \
            .start()
        
        # Wait for termination
        query1.awaitTermination()
        query2.awaitTermination()
        
    except Exception as e:
        logging.error(f"Error in Spark stream processing: {e}")
        raise

if __name__ == "__main__":
    # Choose which processing method to use
    use_spark = False
    
    if use_spark:
        process_stream_with_spark()
    else:
        metrics = process_stream_with_pandas()
        print("Final metrics:")
        for key, value in metrics.items():
            print(f"{key}: {value}")
```

## Conclusion

Advanced Python skills are essential for data engineers who need to build efficient, maintainable, and scalable data processing systems. This lesson has covered a range of advanced topics:

1. **Advanced Pandas and NumPy**: These libraries form the foundation for data manipulation, offering powerful capabilities for transforming, cleaning, and analyzing data efficiently.

2. **PySpark**: For truly large-scale data processing, PySpark combines Python's accessibility with Spark's distributed computing power.

3. **Data Processing Techniques**: From cleaning messy data to feature engineering and normalization, these techniques prepare raw data for analysis and modeling.

4. **Performance Optimization**: As data volumes grow, techniques like vectorization, efficient memory usage, parallel processing, and caching become increasingly important.

5. **Best Practices**: Writing clean, modular, and well-documented code ensures that data pipelines remain maintainable and robust as they evolve.

The practical examples demonstrate how these concepts come together in real-world data engineering scenarios like ETL pipelines, data quality monitoring, and stream processing.

As you continue your data engineering journey, remember that advanced Python skills are not just about knowing syntax but about applying these tools effectively to solve complex data problems. The most successful data engineers combine technical prowess with thoughtful design and an understanding of the broader data ecosystem.

## Next Steps

- Dive deeper into specific libraries like Pandas, PySpark, or specialized tools for your particular domain
- Practice building end-to-end data pipelines that incorporate these advanced concepts
- Explore integration with data orchestration tools like Airflow for scheduling and monitoring pipelines
- Learn about distributed computing concepts that underpin technologies like Spark
- Stay updated with the Python data ecosystem, which continues to evolve rapidly

By mastering these advanced Python concepts, you'll be well-equipped to tackle the diverse challenges of modern data engineering.