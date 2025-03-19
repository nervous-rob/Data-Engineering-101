# Lesson 2.1: Python Basics

## Navigation
- [← Back to Lesson Plan](../2.1-python-basics.md)
- [← Back to Module Overview](../README.md)

## Overview
Python has emerged as the de facto language for data engineering due to its readability, extensive library ecosystem, and versatility across various data processing tasks. This lesson introduces the foundational elements of Python programming that form the basis for data engineering work, from basic syntax to practical data manipulation techniques.

## Learning Objectives
- Understand Python syntax and data types
- Master control structures and functions
- Learn effective error handling and debugging strategies
- Practice fundamental data manipulation techniques

## Python Fundamentals

### Introduction to Python

Python is an interpreted, high-level, general-purpose programming language created by Guido van Rossum and first released in 1991. Its design philosophy emphasizes code readability with significant use of whitespace and a clean, pragmatic syntax. Python's features that make it particularly suitable for data engineering include:

- **Readability**: Clean syntax that resembles pseudocode, making code easier to write and maintain
- **Extensive libraries**: Rich ecosystem of packages for data manipulation, analysis, and processing
- **Versatility**: Works well for both small scripts and large-scale applications
- **Integration capabilities**: Easily interfaces with other languages and systems
- **Community support**: Large, active community providing resources and assistance

For data engineers, Python serves as both a "glue language" connecting various components of data systems and a powerful tool for implementing complex data transformations and pipelines.

### Python Setup and Environment

Before diving into coding, it's important to understand Python environments:

- **Python versions**: Python 2 vs. Python 3 (Python 2 reached end-of-life in 2020, so Python 3 is the standard)
- **Installation**: Standard installation vs. distributions like Anaconda (preferred for data work)
- **Virtual environments**: Tools like `venv`, `virtualenv`, and `conda` for isolated development
- **Package managers**: `pip` and `conda` for installing libraries
- **IDEs and editors**: Options like PyCharm, VS Code, Jupyter Notebooks, and Spyder

For data engineering work, a typical setup might include:
```bash
# Create a virtual environment
python -m venv data_eng_env

# Activate the environment (Windows)
data_eng_env\Scripts\activate

# Activate the environment (macOS/Linux)
source data_eng_env/bin/activate

# Install essential data packages
pip install pandas numpy scipy scikit-learn jupyter
```

### Data Types and Variables

Python's core data types provide the building blocks for representing and manipulating data.

#### Numeric Types
- **int**: Integers (e.g., `42`, `-7`, `0`)
- **float**: Floating-point numbers (e.g., `3.14`, `-0.001`, `2e10`)
- **complex**: Complex numbers (e.g., `3+4j`)

```python
# Basic numeric operations
x = 10
y = 3.14
z = x + y  # 13.14 (float)
```

#### Text Type
- **str**: Strings, immutable sequences of Unicode characters

```python
# String operations
name = "Python"
greeting = f"Hello, {name}!"  # "Hello, Python!" (f-string, Python 3.6+)
lowercase = name.lower()  # "python"
substring = name[0:2]  # "Py" (slicing)
```

Strings are particularly important in data engineering for parsing text data, generating reports, and creating queries.

#### Boolean Type
- **bool**: Truth values `True` and `False`

```python
# Boolean operations
is_valid = True
is_complete = False
result = is_valid and is_complete  # False
```

Boolean values control flow in conditional statements and are crucial for data filtering and validation.

#### Sequence Types
- **list**: Mutable ordered collections (e.g., `[1, 2, 3]`)
- **tuple**: Immutable ordered collections (e.g., `(1, 2, 3)`)
- **range**: Immutable sequence of numbers

```python
# List operations
numbers = [1, 2, 3, 4, 5]
numbers.append(6)  # [1, 2, 3, 4, 5, 6]
numbers[0] = 10   # [10, 2, 3, 4, 5, 6]
sublist = numbers[1:4]  # [2, 3, 4]

# Tuple operations
coordinates = (10, 20)
x, y = coordinates  # Unpacking
```

Lists and tuples are fundamental for storing collections of data, with lists being more flexible but tuples more memory-efficient.

#### Mapping Type
- **dict**: Key-value mappings (e.g., `{"name": "Python", "year": 1991}`)

```python
# Dictionary operations
config = {"host": "localhost", "port": 5432, "database": "analytics"}
config["username"] = "admin"  # Adding a new key-value pair
db_name = config.get("database", "default")  # Safe access with default
```

Dictionaries are extremely useful in data engineering for configuration management, representing structured data, and performing lookups.

#### Set Types
- **set**: Mutable unordered collections of unique elements
- **frozenset**: Immutable unordered collections of unique elements

```python
# Set operations
tags = {"python", "data", "engineering"}
tags.add("etl")  # {"python", "data", "engineering", "etl"}
is_present = "data" in tags  # True (membership test)
```

Sets are invaluable for removing duplicates, performing set operations (union, intersection), and efficient membership testing.

#### None Type
- **NoneType**: The `None` object, representing absence of value

```python
# None usage
result = None
if result is None:
    print("No result available")
```

`None` is commonly used to represent missing values, initialization states, and optional parameters.

### Variables and Assignment

Python uses dynamic typing, meaning variables can change type during execution:

```python
# Dynamic typing
x = 10        # x is an integer
x = "hello"   # x is now a string
x = [1, 2, 3]  # x is now a list
```

While flexible, it's generally good practice to maintain consistent types for variables to avoid confusion.

Some variable naming conventions in Python:
- Use lowercase with underscores for variable names (`file_name`, `user_id`)
- Constants are typically uppercase (`MAX_SIZE`, `DEFAULT_TIMEOUT`)
- Avoid using reserved words (`if`, `for`, `class`, etc.)

## Control Structures

Control structures direct the flow of execution in Python programs, enabling conditional logic and repetition.

### Conditional Statements

The `if`, `elif`, and `else` statements allow execution based on conditions:

```python
# Basic if statement
if temperature > 90:
    print("It's hot outside")
elif temperature > 70:
    print("It's warm outside")
else:
    print("It's cool outside")
```

Python also supports inline conditionals (ternary operators):

```python
# Ternary operator
message = "High" if value > threshold else "Low"
```

### Loops

#### For Loops

The `for` loop iterates over a sequence:

```python
# Iterating over a list
fruits = ["apple", "banana", "cherry"]
for fruit in fruits:
    print(fruit)

# Iterating with range
for i in range(5):
    print(i)  # Prints 0, 1, 2, 3, 4

# Iterating with enumerate for index and value
for index, fruit in enumerate(fruits):
    print(f"{index}: {fruit}")
```

#### While Loops

The `while` loop executes as long as a condition is true:

```python
# Basic while loop
count = 0
while count < 5:
    print(count)
    count += 1

# With break statement
while True:
    user_input = input("Enter command (quit to exit): ")
    if user_input == "quit":
        break
    # Process command
```

#### Loop Control

Python provides several ways to control loop execution:

- **break**: Exits the loop entirely
- **continue**: Skips to the next iteration
- **else**: Executes when the loop condition becomes false (not after a break)

```python
# Loop with break, continue, and else
for num in range(1, 11):
    if num == 3:
        continue  # Skip 3
    if num == 8:
        break     # Stop at 8
    print(num)
else:
    print("Loop completed normally")  # Won't execute due to break
```

### List Comprehensions

List comprehensions provide a concise way to create lists:

```python
# Basic list comprehension
squares = [x**2 for x in range(10)]  # [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]

# With condition
even_squares = [x**2 for x in range(10) if x % 2 == 0]  # [0, 4, 16, 36, 64]

# Nested comprehension
matrix = [[i*j for j in range(1, 4)] for i in range(1, 4)]
# [[1, 2, 3], [2, 4, 6], [3, 6, 9]]
```

List comprehensions are often more readable and faster than equivalent `for` loops, making them valuable for data transformation tasks.

### Dictionary Comprehensions

Similar to list comprehensions, but for creating dictionaries:

```python
# Basic dictionary comprehension
squares_dict = {x: x**2 for x in range(5)}  # {0: 0, 1: 1, 2: 4, 3: 9, 4: 16}

# With condition
even_squares = {x: x**2 for x in range(10) if x % 2 == 0}
```

Dictionary comprehensions are particularly useful in data engineering for transforming data structures and creating lookup tables.

## Functions

Functions enable code reuse, modularization, and abstraction—critical for maintainable data engineering code.

### Function Definition

Basic function syntax in Python:

```python
# Basic function
def greet(name):
    """Return a greeting message for the given name."""
    return f"Hello, {name}!"

# Function with default parameter
def connect_database(host="localhost", port=5432, user="admin"):
    """Establish a database connection."""
    # Connection logic here
    return connection

# Function with multiple return values (actually returns a tuple)
def get_statistics(numbers):
    """Calculate basic statistics for a list of numbers."""
    return min(numbers), max(numbers), sum(numbers) / len(numbers)
```

Note the use of docstrings (the triple-quoted strings) which document what the function does.

### Parameters and Arguments

Python offers flexible ways to define and pass function parameters:

```python
# Positional arguments
def process_data(source, destination, format):
    # Processing logic
    pass

process_data("input.csv", "output.json", "utf-8")

# Keyword arguments
process_data(source="input.csv", format="utf-8", destination="output.json")

# Mix of positional and keyword arguments
process_data("input.csv", destination="output.json", format="utf-8")
```

#### Variable-length Arguments

Python allows functions to accept variable numbers of arguments:

```python
# Variable positional arguments (*args)
def calculate_total(*values):
    """Sum any number of values."""
    return sum(values)

total = calculate_total(1, 2, 3, 4, 5)  # 15

# Variable keyword arguments (**kwargs)
def configure_connection(**options):
    """Configure a connection with arbitrary options."""
    # Process options
    connection_string = f"host={options.get('host', 'localhost')}"
    # More processing
    return connection_string

conn = configure_connection(host="db.example.com", user="admin", use_ssl=True)
```

These features are particularly useful when designing flexible APIs for data processing functions.

### Return Values

Functions can return single values, multiple values (as tuples), or nothing (`None`):

```python
# Single return value
def square(x):
    return x * x

# Multiple return values
def split_name(full_name):
    return full_name.split(" ", 1)  # Returns first_name, last_name

# No return value (implicitly returns None)
def log_event(event_type, message):
    print(f"[{event_type}] {message}")
    # No return statement
```

### Lambda Functions

Lambda functions (anonymous functions) provide a concise way to create small, one-line functions:

```python
# Basic lambda function
square = lambda x: x * x
print(square(5))  # 25

# Lambda with multiple parameters
calculate = lambda x, y, operation: operation(x, y)
print(calculate(5, 3, lambda a, b: a + b))  # 8
print(calculate(5, 3, lambda a, b: a * b))  # 15
```

Lambdas are particularly useful with higher-order functions like `map`, `filter`, and `sorted`:

```python
# Using lambda with map
numbers = [1, 2, 3, 4, 5]
squares = list(map(lambda x: x**2, numbers))  # [1, 4, 9, 16, 25]

# Using lambda with filter
even_numbers = list(filter(lambda x: x % 2 == 0, numbers))  # [2, 4]

# Using lambda with sorted
people = [{"name": "Alice", "age": 30}, {"name": "Bob", "age": 25}]
sorted_people = sorted(people, key=lambda x: x["age"])  # Sort by age
```

### Decorators

Decorators modify or enhance the behavior of functions:

```python
# Basic decorator
def log_execution(func):
    def wrapper(*args, **kwargs):
        print(f"Executing {func.__name__}")
        result = func(*args, **kwargs)
        print(f"Finished {func.__name__}")
        return result
    return wrapper

# Using the decorator
@log_execution
def process_file(filename):
    # Processing logic
    return f"Processed {filename}"

# This is equivalent to:
# process_file = log_execution(process_file)

result = process_file("data.csv")
# Outputs:
# Executing process_file
# Finished process_file
```

Decorators are widely used in data engineering frameworks for adding functionality like caching, retrying failed operations, timing execution, and enforcing access controls.

## Error Handling and Debugging

Robust error handling is crucial for data engineering applications, which often process large volumes of data and interact with multiple external systems.

### Exception Types

Python has a rich hierarchy of built-in exceptions:

- **SyntaxError**: Code that violates Python's syntax rules
- **TypeError**: Operations on inappropriate types
- **ValueError**: Operations with values of correct type but inappropriate value
- **NameError**: Accessing undefined variables
- **IndexError**: Accessing invalid indices in sequences
- **KeyError**: Accessing non-existent keys in dictionaries
- **FileNotFoundError**: Attempting to access non-existent files
- **ZeroDivisionError**: Division by zero
- **IOError**: Input/output operation failures
- **ImportError**: Issues importing modules
- **AttributeError**: Accessing non-existent attributes
- **RuntimeError**: Generic runtime errors

### Try-Except Blocks

The `try-except` statement handles exceptions:

```python
# Basic exception handling
try:
    data = open("data.csv").read()
    processed_data = process_data(data)
    save_results(processed_data)
except FileNotFoundError:
    print("Error: data.csv not found")
except Exception as e:
    print(f"An error occurred: {e}")
```

#### Multiple Except Blocks

You can handle different exceptions differently:

```python
try:
    value = int(input("Enter a number: "))
    result = 100 / value
    print(f"Result: {result}")
except ValueError:
    print("You must enter a valid number")
except ZeroDivisionError:
    print("You cannot divide by zero")
except Exception as e:
    print(f"An unexpected error occurred: {e}")
```

#### Finally Clause

The `finally` clause executes regardless of whether an exception occurred:

```python
try:
    file = open("data.log", "w")
    file.write("Processing started\n")
    # Processing logic
except Exception as e:
    file.write(f"Error: {e}\n")
finally:
    file.write("Processing finished\n")
    file.close()  # This ensures the file is closed even if an exception occurs
```

#### Else Clause

The `else` clause executes when no exception is raised:

```python
try:
    data = process_input(user_input)
except ValueError:
    print("Invalid input format")
else:
    # This only executes if no exception occurred
    save_data(data)
    print("Data processed successfully")
```

### Custom Exceptions

Creating custom exceptions improves code clarity and error handling specificity:

```python
# Custom exception
class DataValidationError(Exception):
    """Raised when data fails validation checks."""
    pass

class ConfigurationError(Exception):
    """Raised when configuration is invalid."""
    pass

# Using custom exceptions
def validate_config(config):
    if "database" not in config:
        raise ConfigurationError("Missing database configuration")
    if "credentials" not in config:
        raise ConfigurationError("Missing credentials")
```

### With Statement (Context Managers)

The `with` statement simplifies resource management:

```python
# File handling with context manager
with open("data.txt", "r") as file:
    data = file.read()
    # Process data
# File is automatically closed when the block exits

# Multiple context managers
with open("input.txt", "r") as infile, open("output.txt", "w") as outfile:
    for line in infile:
        outfile.write(line.upper())
```

Context managers automatically handle cleanup operations, making code more robust and less prone to resource leaks.

### Debugging Techniques

Several approaches can help identify and fix issues in Python code:

#### Print Debugging

The simplest form of debugging involves strategically placed print statements:

```python
def complex_function(data):
    print(f"Input data: {data}")
    
    result = initial_processing(data)
    print(f"After initial processing: {result}")
    
    final_result = final_processing(result)
    print(f"Final result: {final_result}")
    
    return final_result
```

#### Logging

The `logging` module provides more sophisticated output control than print statements:

```python
import logging

# Configure logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    filename='pipeline.log'
)

def process_data(data):
    logging.info(f"Processing {len(data)} records")
    try:
        result = transform_data(data)
        logging.info("Data transformation successful")
        return result
    except Exception as e:
        logging.error(f"Error processing data: {e}")
        raise
```

Logging provides advantages like:
- Level-based filtering (DEBUG, INFO, WARNING, ERROR, CRITICAL)
- Timestamp and context information
- Output to different destinations (files, console, network)
- Configuration without code changes

#### Debugger Usage

Python's built-in `pdb` module provides interactive debugging:

```python
import pdb

def problematic_function():
    x = 10
    y = 0
    pdb.set_trace()  # Debugger will start here
    result = x / y  # This will cause a ZeroDivisionError
    return result
```

When the debugger activates, you can:
- Inspect variable values
- Execute code step by step
- Evaluate expressions
- Set breakpoints

Modern IDEs like PyCharm and VS Code provide graphical debugging interfaces that make this process more intuitive.

## File Operations

Processing files is a fundamental task in data engineering, from reading configuration files to processing large datasets.

### Basic File I/O

Python makes file operations straightforward:

```python
# Writing to a file
with open("output.txt", "w") as file:
    file.write("Hello, world!\n")
    file.write("This is a test file.")

# Reading from a file
with open("output.txt", "r") as file:
    content = file.read()
    print(content)

# Reading line by line
with open("output.txt", "r") as file:
    for line in file:
        print(line.strip())  # strip() removes trailing newline
```

### File Modes

Python supports various file modes:
- `"r"`: Read (default)
- `"w"`: Write (creates new file or truncates existing file)
- `"a"`: Append (creates new file or appends to existing file)
- `"x"`: Exclusive creation (fails if file exists)
- `"b"`: Binary mode (add to other modes, e.g., `"rb"` for binary reading)
- `"t"`: Text mode (default)

```python
# Binary mode for images or other non-text data
with open("image.jpg", "rb") as file:
    binary_data = file.read()

# Appending to a log file
with open("app.log", "a") as log:
    log.write(f"{datetime.now()} - Application started\n")
```

### CSV Processing

CSV (Comma-Separated Values) is a common format in data engineering:

```python
import csv

# Reading CSV
with open("data.csv", "r", newline="") as csvfile:
    reader = csv.reader(csvfile)
    for row in reader:
        print(row)  # row is a list of strings

# Reading CSV with headers
with open("data.csv", "r", newline="") as csvfile:
    reader = csv.DictReader(csvfile)
    for row in reader:
        print(row)  # row is a dictionary

# Writing CSV
data = [
    ["Name", "Age", "City"],
    ["Alice", 30, "New York"],
    ["Bob", 25, "San Francisco"]
]
with open("output.csv", "w", newline="") as csvfile:
    writer = csv.writer(csvfile)
    writer.writerows(data)

# Writing CSV from dictionaries
data = [
    {"Name": "Alice", "Age": 30, "City": "New York"},
    {"Name": "Bob", "Age": 25, "City": "San Francisco"}
]
with open("output.csv", "w", newline="") as csvfile:
    fieldnames = ["Name", "Age", "City"]
    writer = csv.DictWriter(csvfile, fieldnames=fieldnames)
    writer.writeheader()
    writer.writerows(data)
```

### JSON Processing

JSON (JavaScript Object Notation) is widely used for configuration files and APIs:

```python
import json

# Reading JSON
with open("config.json", "r") as jsonfile:
    config = json.load(jsonfile)
    print(config["database"]["host"])

# Writing JSON
data = {
    "name": "Pipeline Configuration",
    "version": 1.0,
    "settings": {
        "max_threads": 4,
        "retry_count": 3
    }
}
with open("config.json", "w") as jsonfile:
    json.dump(data, jsonfile, indent=4)

# Pretty-printing JSON
print(json.dumps(data, indent=4))
```

## Data Manipulation Basics

For data engineering tasks, Python offers powerful libraries for data manipulation.

### Introduction to NumPy

NumPy provides efficient numerical operations on arrays:

```python
import numpy as np

# Creating arrays
arr = np.array([1, 2, 3, 4, 5])
matrix = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])

# Array operations
print(arr * 2)  # Element-wise multiplication: [2, 4, 6, 8, 10]
print(arr.mean())  # Average: 3.0
print(matrix.shape)  # Dimensions: (3, 3)

# Array manipulation
transposed = matrix.T  # Transpose
flat = matrix.flatten()  # Flatten to 1D array
reshaped = matrix.reshape(1, 9)  # Reshape to 1x9 matrix
```

NumPy's advantages include:
- Efficient memory usage
- Vectorized operations (faster than loops)
- Broadcasting capabilities
- Rich mathematical functions

### Introduction to Pandas

Pandas is the primary library for data manipulation:

```python
import pandas as pd

# Creating a DataFrame
data = {
    "Name": ["Alice", "Bob", "Charlie"],
    "Age": [25, 30, 35],
    "City": ["New York", "San Francisco", "Seattle"]
}
df = pd.DataFrame(data)

# Reading from files
df_csv = pd.read_csv("data.csv")
df_excel = pd.read_excel("data.xlsx")

# Basic operations
print(df.head())  # First few rows
print(df.describe())  # Statistical summary
print(df["Age"].mean())  # Column average

# Filtering
adults = df[df["Age"] >= 30]
new_yorkers = df[df["City"] == "New York"]

# Grouping and aggregation
age_by_city = df.groupby("City")["Age"].mean()

# Joining data
employees = pd.DataFrame({
    "ID": [1, 2, 3],
    "Name": ["Alice", "Bob", "Charlie"]
})
departments = pd.DataFrame({
    "ID": [1, 2, 3],
    "Department": ["HR", "Engineering", "Marketing"]
})
merged = pd.merge(employees, departments, on="ID")
```

Pandas is essential for data engineering tasks like:
- Data cleaning and preparation
- Feature engineering
- ETL processes
- Data analysis
- Report generation

## Best Practices in Python for Data Engineering

Following best practices ensures that your Python code is maintainable, efficient, and robust.

### Code Style

- **PEP 8**: Follow the Python style guide for consistent formatting
- **Meaningful names**: Use descriptive variable, function, and class names
- **Comments**: Add comments to explain why, not what (the code should be self-explanatory)
- **Docstrings**: Document functions, classes, and modules
- **Line length**: Keep lines under 79-88 characters for readability

```python
def calculate_statistics(data, metrics=None):
    """
    Calculate statistical metrics for the given data.
    
    Parameters:
    -----------
    data : list or array-like
        The data to analyze
    metrics : list, optional
        List of metrics to calculate. If None, calculates all available metrics.
        
    Returns:
    --------
    dict
        Dictionary of calculated metrics
    """
    # Implementation here
    pass
```

### Code Organization

- **Modularity**: Break code into logical, reusable components
- **Single responsibility**: Each function or class should do one thing well
- **Imports**: Organize imports (standard library, third-party, local) and import only what you need
- **Constants**: Define constants at the module level
- **Configuration**: Separate configuration from code

### Performance Considerations

- **Vectorization**: Use NumPy/Pandas vectorized operations instead of loops when possible
- **Generators**: Use generators for large datasets to reduce memory usage
- **Appropriate data structures**: Choose the right data structure for the task (e.g., sets for unique values)
- **Profiling**: Use tools like `cProfile` to identify bottlenecks
- **Caching**: Cache expensive operations

```python
# Bad: Using loops
total = 0
for value in large_array:
    total += value

# Good: Using vectorization
total = np.sum(large_array)

# Bad: Loading entire file into memory
with open("large_file.txt", "r") as file:
    lines = file.readlines()  # Loads everything into memory
    for line in lines:
        process(line)

# Good: Processing line by line
with open("large_file.txt", "r") as file:
    for line in file:  # Iterates without loading everything
        process(line)
```

### Error Handling

- **Specific exceptions**: Catch specific exceptions rather than generic ones
- **Fail fast**: Validate inputs early to avoid processing invalid data
- **Graceful degradation**: Recover from errors when possible
- **Logging**: Log errors with context information for debugging
- **User feedback**: Provide clear error messages

### Testing and Debugging

- **Unit tests**: Write tests for individual functions and components
- **Integration tests**: Test how components work together
- **Assertions**: Use assertions to catch programming errors
- **Logging**: Implement comprehensive logging
- **Debugger**: Know how to use the debugger effectively

## Practical Examples for Data Engineering

Let's look at some practical examples of Python code for data engineering tasks.

### ETL Process Example

```python
import pandas as pd
import logging
from datetime import datetime

# Configure logging
logging.basicConfig(
    filename=f"etl_{datetime.now().strftime('%Y%m%d')}.log",
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s'
)

def extract(file_path):
    """Extract data from source file."""
    try:
        logging.info(f"Extracting data from {file_path}")
        if file_path.endswith('.csv'):
            return pd.read_csv(file_path)
        elif file_path.endswith('.json'):
            return pd.read_json(file_path)
        else:
            raise ValueError(f"Unsupported file format: {file_path}")
    except Exception as e:
        logging.error(f"Extraction error: {e}")
        raise

def transform(data):
    """Apply transformations to the data."""
    logging.info("Transforming data")
    try:
        # Example transformations
        # 1. Convert date strings to datetime objects
        if "date" in data.columns:
            data["date"] = pd.to_datetime(data["date"])
        
        # 2. Handle missing values
        data = data.fillna({
            "numeric_column": 0,
            "text_column": "Unknown"
        })
        
        # 3. Create derived columns
        if "first_name" in data.columns and "last_name" in data.columns:
            data["full_name"] = data["first_name"] + " " + data["last_name"]
        
        # 4. Filter out invalid records
        data = data[data["value"] > 0]
        
        logging.info(f"Transformation complete. {len(data)} records processed.")
        return data
    except Exception as e:
        logging.error(f"Transformation error: {e}")
        raise

def load(data, output_path):
    """Load transformed data to destination."""
    try:
        logging.info(f"Loading data to {output_path}")
        if output_path.endswith('.csv'):
            data.to_csv(output_path, index=False)
        elif output_path.endswith('.parquet'):
            data.to_parquet(output_path, index=False)
        else:
            raise ValueError(f"Unsupported output format: {output_path}")
        logging.info(f"Data loaded successfully. {len(data)} records written.")
    except Exception as e:
        logging.error(f"Load error: {e}")
        raise

def run_etl_process(source_path, destination_path):
    """Run the complete ETL process."""
    try:
        logging.info("Starting ETL process")
        raw_data = extract(source_path)
        transformed_data = transform(raw_data)
        load(transformed_data, destination_path)
        logging.info("ETL process completed successfully")
    except Exception as e:
        logging.error(f"ETL process failed: {e}")
        raise

if __name__ == "__main__":
    try:
        run_etl_process("sales_data.csv", "processed_sales.parquet")
    except Exception as e:
        print(f"ETL process failed: {e}")

### Data Validation Example

```python
import pandas as pd
import numpy as np
import logging

logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')

class DataValidator:
    """Validates data quality according to defined rules."""
    
    def __init__(self, data):
        """Initialize with a pandas DataFrame."""
        self.data = data
        self.validation_results = []
    
    def validate_not_null(self, column_name):
        """Check if column has no null values."""
        null_count = self.data[column_name].isnull().sum()
        is_valid = null_count == 0
        self.validation_results.append({
            "check": "not_null",
            "column": column_name,
            "is_valid": is_valid,
            "details": f"Found {null_count} null values"
        })
        return is_valid
    
    def validate_unique(self, column_name):
        """Check if column values are unique."""
        unique_count = self.data[column_name].nunique()
        total_count = len(self.data)
        is_valid = unique_count == total_count
        self.validation_results.append({
            "check": "unique",
            "column": column_name,
            "is_valid": is_valid,
            "details": f"Found {total_count - unique_count} duplicate values"
        })
        return is_valid
    
    def validate_range(self, column_name, min_value, max_value):
        """Check if column values fall within range."""
        out_of_range = self.data[
            (self.data[column_name] < min_value) | 
            (self.data[column_name] > max_value)
        ]
        is_valid = len(out_of_range) == 0
        self.validation_results.append({
            "check": "range",
            "column": column_name,
            "is_valid": is_valid,
            "details": f"Found {len(out_of_range)} values outside range [{min_value}, {max_value}]"
        })
        return is_valid
    
    def validate_format(self, column_name, pattern):
        """Check if column values match regex pattern."""
        import re
        invalid_format = self.data[~self.data[column_name].str.match(pattern)]
        is_valid = len(invalid_format) == 0
        self.validation_results.append({
            "check": "format",
            "column": column_name,
            "is_valid": is_valid,
            "details": f"Found {len(invalid_format)} values with invalid format"
        })
        return is_valid
    
    def get_validation_summary(self):
        """Return summary of all validation checks."""
        return pd.DataFrame(self.validation_results)
    
    def is_valid(self):
        """Return True if all validations passed."""
        return all(result["is_valid"] for result in self.validation_results)

# Example usage
def validate_customer_data(file_path):
    try:
        # Load data
        customers = pd.read_csv(file_path)
        logging.info(f"Loaded {len(customers)} customer records for validation")
        
        # Initialize validator
        validator = DataValidator(customers)
        
        # Apply validation rules
        validator.validate_not_null("customer_id")
        validator.validate_unique("customer_id")
        validator.validate_not_null("email")
        validator.validate_format("email", r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,})
        validator.validate_range("age", 18, 120)
        
        # Get results
        summary = validator.get_validation_summary()
        logging.info(f"Validation complete. {summary['is_valid'].sum()} of {len(summary)} checks passed.")
        
        # Take action based on results
        if validator.is_valid():
            logging.info("All validation checks passed!")
            return True
        else:
            failed_checks = summary[~summary["is_valid"]]
            logging.warning(f"Data validation failed: {len(failed_checks)} checks failed")
            for _, check in failed_checks.iterrows():
                logging.warning(f"Failed {check['check']} on {check['column']}: {check['details']}")
            return False
    
    except Exception as e:
        logging.error(f"Validation error: {e}")
        raise

# Example data processing pipeline with validation
def process_customer_data(input_file, output_file):
    try:
        # Validate data first
        is_valid = validate_customer_data(input_file)
        
        if not is_valid:
            logging.error("Data processing aborted due to validation failure")
            return False
        
        # Proceed with processing if valid
        data = pd.read_csv(input_file)
        # ... processing steps ...
        data.to_csv(output_file, index=False)
        logging.info(f"Data processed successfully and saved to {output_file}")
        return True
    
    except Exception as e:
        logging.error(f"Processing error: {e}")
        return False
```

### Configuration Management Example

```python
import os
import json
import yaml
import logging
from dataclasses import dataclass
from typing import Dict, List, Optional, Any

logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')

@dataclass
class DatabaseConfig:
    """Database connection configuration."""
    host: str
    port: int
    database: str
    username: str
    password: str
    ssl_enabled: bool = False
    
    def get_connection_string(self) -> str:
        """Generate connection string from config."""
        ssl = "?ssl=true" if self.ssl_enabled else ""
        return f"postgresql://{self.username}:{self.password}@{self.host}:{self.port}/{self.database}{ssl}"

@dataclass
class PipelineConfig:
    """Data pipeline configuration."""
    name: str
    source_path: str
    destination_path: str
    batch_size: int = 1000
    timeout_seconds: int = 3600
    retry_count: int = 3

@dataclass
class AppConfig:
    """Application configuration container."""
    environment: str
    database: DatabaseConfig
    pipeline: PipelineConfig
    log_level: str = "INFO"
    enable_monitoring: bool = True
    max_workers: int = 4

class ConfigLoader:
    """Configuration loader that supports multiple formats."""
    
    @staticmethod
    def from_file(file_path: str) -> AppConfig:
        """Load configuration from file based on extension."""
        if not os.path.exists(file_path):
            raise FileNotFoundError(f"Configuration file not found: {file_path}")
        
        logging.info(f"Loading configuration from {file_path}")
        
        # Determine file type and load accordingly
        if file_path.endswith(".json"):
            return ConfigLoader._load_from_json(file_path)
        elif file_path.endswith((".yaml", ".yml")):
            return ConfigLoader._load_from_yaml(file_path)
        else:
            raise ValueError(f"Unsupported configuration file format: {file_path}")
    
    @staticmethod
    def _load_from_json(file_path: str) -> AppConfig:
        """Load configuration from JSON file."""
        with open(file_path, 'r') as f:
            config_data = json.load(f)
        return ConfigLoader._create_config_from_dict(config_data)
    
    @staticmethod
    def _load_from_yaml(file_path: str) -> AppConfig:
        """Load configuration from YAML file."""
        with open(file_path, 'r') as f:
            config_data = yaml.safe_load(f)
        return ConfigLoader._create_config_from_dict(config_data)
    
    @staticmethod
    def _create_config_from_dict(config_data: Dict[str, Any]) -> AppConfig:
        """Create AppConfig instance from dictionary data."""
        try:
            # Create nested configuration objects
            db_config = DatabaseConfig(**config_data["database"])
            pipeline_config = PipelineConfig(**config_data["pipeline"])
            
            # Create main config with nested objects
            app_config = AppConfig(
                environment=config_data["environment"],
                database=db_config,
                pipeline=pipeline_config,
                log_level=config_data.get("log_level", "INFO"),
                enable_monitoring=config_data.get("enable_monitoring", True),
                max_workers=config_data.get("max_workers", 4)
            )
            
            return app_config
        except KeyError as e:
            raise ValueError(f"Missing required configuration key: {e}")

# Example usage
def run_data_pipeline():
    try:
        # Load configuration
        config = ConfigLoader.from_file("config.yaml")
        
        # Configure logging based on config
        logging.getLogger().setLevel(config.log_level)
        
        # Use configuration in application
        logging.info(f"Starting pipeline '{config.pipeline.name}' in {config.environment} environment")
        logging.info(f"Connecting to database at {config.database.host}")
        
        # Database connection
        connection_string = config.database.get_connection_string()
        # ... connect to database using connection_string ...
        
        # Execute pipeline
        logging.info(f"Processing data from {config.pipeline.source_path}")
        # ... pipeline implementation using config parameters ...
        
        logging.info("Pipeline completed successfully")
    
    except Exception as e:
        logging.error(f"Pipeline execution failed: {e}")
        raise

# Example YAML configuration file (config.yaml):
"""
environment: development
log_level: INFO
enable_monitoring: true
max_workers: 4

database:
  host: localhost
  port: 5432
  database: analytics
  username: dbuser
  password: dbpass
  ssl_enabled: false

pipeline:
  name: customer_data_import
  source_path: /data/customers/
  destination_path: /processed/customers/
  batch_size: 5000
  timeout_seconds: 7200
  retry_count: 5
"""
```

## Conclusion

Python's combination of readability, flexibility, and extensive libraries makes it an ideal language for data engineering tasks. The fundamentals covered in this lesson—data types, control structures, functions, error handling, and file operations—form the foundation for building robust data pipelines and processing systems.

As you progress in your data engineering journey, you'll build on these basics to work with more advanced libraries like Pandas and NumPy, and eventually integrate with big data frameworks like Spark. The principles of writing clean, maintainable, and efficient code will remain constant regardless of the specific tools you use.

Remember that effective data engineering is not just about coding—it's about solving data problems in a sustainable way. This means thinking about error handling, performance, maintainability, and documentation from the start, rather than as afterthoughts.

## Next Steps

- Practice these Python fundamentals with real-world data engineering problems
- Explore standard library modules relevant to data processing (e.g., `csv`, `json`, `datetime`)
- Begin working with Pandas for data manipulation and analysis
- Learn about unit testing with the `unittest` or `pytest` frameworks
- Familiarize yourself with virtual environments and package management