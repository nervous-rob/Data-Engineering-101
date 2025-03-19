# Lesson 2.6: Functional Programming

## Navigation
- [← Back to Lesson Plan](../2.6-functional-programming.md)
- [← Back to Module Overview](../README.md)

## Overview
Functional programming represents a paradigm shift in how we think about building software. While object-oriented programming focuses on modeling state and behavior through objects, functional programming emphasizes immutable data and pure functions. This approach is particularly valuable in data engineering, where we often process large datasets through a series of transformations. This lesson explores functional programming principles and their application in Java, Scala, and Python.

## Learning Objectives
- Understand the core concepts of functional programming
- Learn how to apply functional patterns in data processing tasks
- Master functional programming features in Java, Scala, and Python
- Practice building data processing workflows using functional techniques

## Functional Programming Fundamentals

### What is Functional Programming?

Functional programming is a programming paradigm that treats computation as the evaluation of mathematical functions and avoids changing state and mutable data. It emphasizes:

1. **Pure functions**: Functions that always produce the same output for the same input and have no side effects
2. **Immutable data**: Once created, data cannot be changed
3. **First-class functions**: Functions can be assigned to variables, passed as arguments, and returned from other functions
4. **Higher-order functions**: Functions that take other functions as parameters or return functions
5. **Declarative style**: Focusing on what to solve rather than how to solve it

This approach contrasts with imperative programming, where programs are structured as sequences of statements that change program state.

### Why Functional Programming for Data Engineering?

Data engineering often involves building pipelines that transform data through multiple stages. Functional programming offers several advantages for these tasks:

1. **Parallelization**: Pure functions with no shared state are easier to parallelize
2. **Testability**: Functions with no side effects are easier to test
3. **Reasoning**: Immutable data and pure functions simplify reasoning about code
4. **Composability**: Smaller functions can be combined to build complex transformations
5. **Data flow clarity**: Function composition models data flow explicitly

Let's explore these concepts in more detail.

### Pure Functions

A pure function has two key characteristics:
1. It returns the same result given the same arguments
2. It has no side effects (doesn't modify external state)

```python
# Impure function
total = 0
def add_to_total(value):
    global total
    total += value  # Side effect: modifies global state
    return total

# Pure function
def add(a, b):
    return a + b  # Always returns same output for same input, no side effects
```

Pure functions make code more predictable, testable, and easier to reason about. They're particularly valuable in data pipelines, where predictability is crucial.

### Immutability

Immutability means that once created, data cannot be changed. Instead of modifying existing data, functional programming creates new data structures with the desired changes.

```python
# Mutable approach
def add_item_mutable(shopping_list, item):
    shopping_list.append(item)  # Modifies original list
    return shopping_list

# Immutable approach
def add_item_immutable(shopping_list, item):
    return shopping_list + [item]  # Returns new list, original unchanged
```

In data engineering, immutability helps ensure data consistency, especially in distributed or concurrent environments.

### First-Class and Higher-Order Functions

When functions are "first-class citizens," they can be:
- Assigned to variables
- Passed as arguments
- Returned from functions

Higher-order functions either take functions as arguments or return them:

```python
# Function assigned to variable
square = lambda x: x * x

# Higher-order function that takes a function as argument
def apply_twice(f, x):
    return f(f(x))

# Using the higher-order function
apply_twice(square, 3)  # Returns 81 (3 squared = 9, then 9 squared = 81)

# Higher-order function that returns a function
def create_multiplier(factor):
    def multiplier(x):
        return x * factor
    return multiplier

double = create_multiplier(2)
triple = create_multiplier(3)
double(5)  # Returns 10
triple(5)  # Returns 15
```

These capabilities enable powerful abstractions for data processing.

### Function Composition

Function composition combines simple functions to build more complex ones:

```python
def add_one(x):
    return x + 1

def double(x):
    return x * 2

# Manual composition
def add_one_then_double(x):
    return double(add_one(x))

# Using tools for composition (Python 3.8+)
from functools import reduce
def compose(*functions):
    def compose_two(f, g):
        return lambda x: f(g(x))
    return reduce(compose_two, functions, lambda x: x)

add_one_then_double = compose(double, add_one)
add_one_then_double(3)  # Returns 8
```

Function composition mirrors the flow of data through transformations, making pipelines more intuitive.

## Functional Programming in Java

Java has introduced several features to support functional programming, especially since Java 8.

### Lambda Expressions

Lambda expressions provide a concise way to represent anonymous functions:

```java
// Traditional anonymous class
Runnable runnable1 = new Runnable() {
    @Override
    public void run() {
        System.out.println("Hello, world");
    }
};

// Lambda expression
Runnable runnable2 = () -> System.out.println("Hello, world");

// Lambda with parameters
Comparator<String> comparator = (s1, s2) -> s1.length() - s2.length();

// Lambda with block body
Function<Integer, Integer> factorial = n -> {
    int result = 1;
    for (int i = 1; i <= n; i++) {
        result *= i;
    }
    return result;
};
```

### Functional Interfaces

Java's functional interfaces are interfaces with a single abstract method, designed to be used with lambda expressions:

```java
// Common functional interfaces in java.util.function
Function<String, Integer> length = s -> s.length();
Predicate<Integer> isEven = n -> n % 2 == 0;
Consumer<String> printer = s -> System.out.println(s);
Supplier<Double> random = () -> Math.random();
BinaryOperator<Integer> sum = (a, b) -> a + b;

// Using these interfaces
length.apply("hello");  // Returns 5
isEven.test(4);  // Returns true
printer.accept("hello");  // Prints "hello"
random.get();  // Returns a random double
sum.apply(2, 3);  // Returns 5
```

### Stream API

The Stream API enables functional-style operations on streams of elements:

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "Dave", "Eve");

// Imperative approach
List<String> filteredNames = new ArrayList<>();
for (String name : names) {
    if (name.length() > 3) {
        filteredNames.add(name.toUpperCase());
    }
}

// Functional approach with streams
List<String> filteredNames = names.stream()
    .filter(name -> name.length() > 3)
    .map(String::toUpperCase)
    .collect(Collectors.toList());

// More complex example
int sumOfSquares = numbers.stream()
    .filter(n -> n % 2 == 0)  // Keep only even numbers
    .map(n -> n * n)          // Square each number
    .reduce(0, Integer::sum); // Sum the squares
```

The Stream API provides many operations for data transformation:
- **Intermediate operations**: filter, map, flatMap, distinct, sorted, peek, limit, skip
- **Terminal operations**: forEach, collect, reduce, min, max, count, anyMatch, allMatch, noneMatch, findFirst, findAny

### Optional

Java's Optional type helps avoid null pointer exceptions by explicitly representing the presence or absence of a value:

```java
// Without Optional
String name = getName();  // Might return null
int length = name.length();  // Potential NullPointerException

// With Optional
Optional<String> name = getOptionalName();
int length = name.map(String::length).orElse(0);

// More Optional operations
Optional<User> user = findUser(id);
String displayName = user
    .map(User::getName)
    .filter(name -> !name.isEmpty())
    .orElse("Anonymous");
```

### Method References

Method references provide a shorthand notation for lambda expressions that call a single method:

```java
// Lambda expression
Function<String, Integer> length = s -> s.length();

// Equivalent method reference
Function<String, Integer> length = String::length;

// Different types of method references
Consumer<String> printer = System.out::println;  // Reference to static method
Function<String, String> toUpper = String::toUpperCase;  // Reference to instance method
BiPredicate<String, String> contains = String::contains;  // Reference with parameters
Supplier<ArrayList<String>> listCreator = ArrayList::new;  // Reference to constructor
```

## Functional Programming in Scala

Scala was designed with functional programming in mind and offers more comprehensive support compared to Java.

### Functions as First-Class Citizens

Scala treats functions as first-class citizens with concise syntax:

```scala
// Function literals (lambda expressions)
val add = (a: Int, b: Int) => a + b
val square = (x: Int) => x * x

// Placeholder syntax
val isEven = (_: Int) % 2 == 0

// Using functions
add(2, 3)  // Returns 5
square(4)  // Returns 16
isEven(6)  // Returns true
```

### Higher-Order Functions

Scala makes higher-order functions feel natural:

```scala
// Function that takes a function parameter
def transform(list: List[Int], f: Int => Int): List[Int] = list.map(f)

// Using the function
val numbers = List(1, 2, 3, 4, 5)
transform(numbers, x => x * x)  // Returns List(1, 4, 9, 16, 25)
transform(numbers, _ + 10)      // Returns List(11, 12, 13, 14, 15)

// Function returning a function
def createMultiplier(factor: Int): Int => Int = x => x * factor
val double = createMultiplier(2)
val triple = createMultiplier(3)
double(5)  // Returns 10
triple(5)  // Returns 15
```

### Immutability

Scala encourages immutability through `val` declarations and immutable collections:

```scala
// Immutable variable
val name = "Alice"  // Cannot be reassigned

// Mutable variable (discouraged in functional style)
var counter = 0     // Can be reassigned

// Immutable collections
val numbers = List(1, 2, 3, 4, 5)
val doubled = numbers.map(_ * 2)  // Creates new list without modifying original

// Mutable collections (available but discouraged)
import scala.collection.mutable
val mutableList = mutable.ListBuffer(1, 2, 3)
mutableList += 4  // Modifies original list
```

### Pattern Matching

Pattern matching provides a powerful mechanism for conditional logic:

```scala
// Basic pattern matching
def describe(x: Any): String = x match {
  case i: Int if i > 0 => "Positive integer"
  case 0 => "Zero"
  case s: String => s"String: $s"
  case _ => "Something else"
}

// Pattern matching with case classes
case class Person(name: String, age: Int)

def greet(person: Person): String = person match {
  case Person("Alice", _) => "Hello, Alice!"
  case Person(name, age) if age < 18 => s"Hi, young $name!"
  case Person(name, _) => s"Hello, $name!"
}
```

### Function Composition

Scala offers elegant ways to compose functions:

```scala
val addOne = (x: Int) => x + 1
val double = (x: Int) => x * 2

// Composition using andThen (applies functions left to right)
val addOneThenDouble = addOne andThen double
addOneThenDouble(3)  // Returns 8

// Composition using compose (applies functions right to left)
val doubleThenAddOne = double compose addOne
doubleThenAddOne(3)  // Returns 7
```

### Immutable Data Structures

Scala provides efficient immutable data structures:

```scala
// Immutable List
val numbers = List(1, 2, 3, 4, 5)
val moreNumbers = 0 :: numbers  // Creates new list with 0 at front
val evenMoreNumbers = numbers :+ 6  // Creates new list with 6 at end

// Immutable Map
val scores = Map("Alice" -> 10, "Bob" -> 8, "Charlie" -> 9)
val updatedScores = scores + ("Dave" -> 7)  // Creates new map with added entry

// Vector (efficient random access)
val vector = Vector(1, 2, 3, 4, 5)
val updated = vector.updated(2, 30)  // Creates new vector with element 2 changed to 30
```

### Options, Try, and Either

Scala provides monadic types for dealing with optional values and errors:

```scala
// Option for optional values
val maybeValue: Option[String] = Some("Hello")
val noValue: Option[String] = None

// Using Option
maybeValue.map(_.toUpperCase).getOrElse("No value")

// Try for operations that might fail
import scala.util.{Try, Success, Failure}
val result = Try {
  "42".toInt
}

result match {
  case Success(value) => s"The value is $value"
  case Failure(exception) => s"Failed with: ${exception.getMessage}"
}

// Either for operations with two possible outcomes
def divide(a: Int, b: Int): Either[String, Int] = {
  if (b == 0) Left("Division by zero")
  else Right(a / b)
}

divide(10, 2) match {
  case Right(result) => s"Result: $result"
  case Left(error) => s"Error: $error"
}
```

## Functional Programming in Python

Python is a multi-paradigm language that supports functional programming concepts.

### Lambda Functions

Python's lambda functions provide anonymous function capabilities:

```python
# Lambda function
square = lambda x: x * x
add = lambda a, b: a + b

# Using lambda functions
square(5)  # Returns 25
add(2, 3)  # Returns 5

# Lambda with sorting
names = ["Alice", "Bob", "Charlie", "Dave"]
sorted(names, key=lambda name: len(name))  # Sorts by length
```

### Map, Filter, and Reduce

Python provides higher-order functions for common operations:

```python
# Map applies a function to each element
numbers = [1, 2, 3, 4, 5]
squared = list(map(lambda x: x * x, numbers))  # [1, 4, 9, 16, 25]

# Filter selects elements that satisfy a predicate
evens = list(filter(lambda x: x % 2 == 0, numbers))  # [2, 4]

# Reduce combines elements
from functools import reduce
sum_all = reduce(lambda a, b: a + b, numbers)  # 15
product = reduce(lambda a, b: a * b, numbers)  # 120
```

### List Comprehensions

List comprehensions provide a concise way to create lists:

```python
# Simple list comprehension
squares = [x * x for x in range(1, 6)]  # [1, 4, 9, 16, 25]

# With condition
evens = [x for x in range(1, 11) if x % 2 == 0]  # [2, 4, 6, 8, 10]

# Nested comprehension
matrix = [[i * j for j in range(1, 4)] for i in range(1, 4)]
# [[1, 2, 3], [2, 4, 6], [3, 6, 9]]

# Dictionary comprehension
square_dict = {x: x * x for x in range(1, 6)}
# {1: 1, 2: 4, 3: 9, 4: 16, 5: 25}
```

### Function Decorators

Decorators modify the behavior of functions:

```python
# Basic decorator
def log_function(func):
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__} with args: {args}, kwargs: {kwargs}")
        result = func(*args, **kwargs)
        print(f"{func.__name__} returned: {result}")
        return result
    return wrapper

@log_function
def add(a, b):
    return a + b

# Using the decorated function
add(2, 3)  # Logs function call and return value

# Multiple decorators can be stacked
def memoize(func):
    cache = {}
    def wrapper(*args):
        if args not in cache:
            cache[args] = func(*args)
        return cache[args]
    return wrapper

@memoize
@log_function
def fibonacci(n):
    if n <= 1:
        return n
    return fibonacci(n-1) + fibonacci(n-2)
```

### Itertools and Functools

Python provides modules with functional programming utilities:

```python
import itertools
import functools

# Itertools for working with iterators
# Infinite iterators
counter = itertools.count(start=1, step=2)  # 1, 3, 5, ...
cycle = itertools.cycle(['A', 'B', 'C'])  # A, B, C, A, B, C, ...
repeated = itertools.repeat('X', 3)  # X, X, X

# Combinatoric iterators
combinations = list(itertools.combinations('ABC', 2))  # [('A', 'B'), ('A', 'C'), ('B', 'C')]
permutations = list(itertools.permutations('ABC', 2))  # [('A', 'B'), ('A', 'C'), ('B', 'A'), ('B', 'C'), ('C', 'A'), ('C', 'B')]

# Functools for higher-order functions
# Partial application
multiply = lambda x, y: x * y
double = functools.partial(multiply, 2)
double(4)  # Returns 8

# Function composition
def compose(*functions):
    def compose_two(f, g):
        return lambda x: f(g(x))
    return functools.reduce(compose_two, functions, lambda x: x)

add_one = lambda x: x + 1
double = lambda x: x * 2
add_one_then_double = compose(double, add_one)
add_one_then_double(3)  # Returns 8
```

## Functional Patterns in Data Engineering

Let's explore how functional programming applies to data engineering tasks.

### Transformation Pipelines

Data pipelines can be modeled as a series of pure transformations:

```python
# Instead of:
def process_data(data):
    data = clean_data(data)
    data = transform_data(data)
    data = validate_data(data)
    return data

# Functional approach:
def process_data(data):
    return validate_data(transform_data(clean_data(data)))

# Or using composition:
process_data = compose(validate_data, transform_data, clean_data)
```

### Higher-Order Functions for Processing

Higher-order functions can encapsulate common data processing patterns:

```python
def map_columns(df, column_mapping):
    """Apply functions to columns in a DataFrame."""
    result = df.copy()
    for col, func in column_mapping.items():
        result[col] = result[col].apply(func)
    return result

# Usage
transformed_df = map_columns(df, {
    'price': lambda x: x * 0.9,  # 10% discount
    'name': str.upper,
    'date': lambda d: d.strftime('%Y-%m-%d')
})
```

### Handling Optional Values

Functional approaches help with missing or invalid data:

```python
# Using Optional-like pattern
def safe_division(a, b):
    return None if b == 0 else a / b

# Using a monad-like chain
def get_discount_percentage(user_id):
    return (get_user(user_id)
            .and_then(get_user_subscription)
            .and_then(get_subscription_discount)
            .or_else(0))  # Default value
```

### Error Handling

Functional error handling focuses on returning errors rather than throwing exceptions:

```python
# Result type pattern
def parse_json(data):
    try:
        return {'success': True, 'data': json.loads(data)}
    except json.JSONDecodeError as e:
        return {'success': False, 'error': str(e)}

# Usage
result = parse_json(input_data)
if result['success']:
    process_data(result['data'])
else:
    log_error(result['error'])
```

### Working with Streams and Lazy Evaluation

Functional programming works well with streaming data and lazy evaluation:

```python
# Processing a large file lazily
def process_large_file(filename):
    with open(filename, 'r') as file:
        # Generate processed lines without loading entire file
        for line in file:
            yield process_line(line)

# In Java with Stream API:
try (Stream<String> lines = Files.lines(Paths.get(filename))) {
    lines.map(this::processLine)
         .filter(Objects::nonNull)
         .forEach(this::saveLine);
}

# In Scala:
Source.fromFile(filename).getLines()
      .map(processLine)
      .filter(_ != null)
      .foreach(saveLine)
```

## Advanced Functional Concepts

### Monads

Monads are a design pattern that allows composition of functions with context:

```scala
// Option monad in Scala
val result = for {
  user <- findUser(userId)
  account <- findAccount(user.accountId)
  balance <- getBalance(account.id)
} yield balance

// Equivalent to:
findUser(userId).flatMap(user =>
  findAccount(user.accountId).flatMap(account =>
    getBalance(account.id)
  )
)
```

### Functors

Functors are structures that can be mapped over:

```scala
// List is a functor
val numbers = List(1, 2, 3, 4, 5)
val doubled = numbers.map(_ * 2)

// Option is a functor
val maybeValue: Option[Int] = Some(42)
val maybeDoubled = maybeValue.map(_ * 2)  // Some(84)
```

### Currying and Partial Application

Currying transforms a function with multiple arguments into a series of functions with single arguments:

```scala
// Curried function
def add(a: Int)(b: Int): Int = a + b

// Partial application
val add5 = add(5) _
add5(3)  // Returns 8

// In Python
from functools import partial
def add(a, b):
    return a + b
add5 = partial(add, 5)
add5(3)  # Returns 8
```

### Function Memoization

Memoization caches function results to avoid redundant computations:

```python
from functools import lru_cache

@lru_cache(maxsize=100)
def fibonacci(n):
    if n <= 1:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

# Without memoization, this would be very inefficient
fibonacci(100)  # Fast with memoization
```

## Practical Activities

### Activity 1: Implementing Data Transformations

Let's implement a data transformation pipeline using functional programming techniques:

```python
# Sample data
data = [
    {"name": "Alice", "age": 30, "salary": 75000},
    {"name": "Bob", "age": 25, "salary": 65000},
    {"name": "Charlie", "age": 35, "salary": 85000},
    {"name": "Dave", "age": 40, "salary": 95000}
]

# Step 1: Define transformation functions
def filter_by_age(min_age):
    return lambda person: person["age"] >= min_age

def adjust_salary(factor):
    return lambda person: {**person, "salary": person["salary"] * factor}

def format_person(person):
    return {**person, "description": f"{person['name']}, {person['age']} - ${person['salary']}"}

# Step 2: Define pipeline composition
def pipeline(*functions):
    def apply(data):
        result = data
        for func in functions:
            result = map(func, result)
        return list(result)
    return apply

# Step 3: Create specific pipelines
senior_pipeline = pipeline(
    filter_by_age(30),
    adjust_salary(1.1),
    format_person
)

# Step 4: Apply pipeline
result = senior_pipeline(data)
```

### Activity 2: Implementing Functional Utilities

Let's create some functional programming utilities:

```scala
// In Scala

// Either implementation for error handling
sealed trait Result[+E, +A]
case class Success[A](value: A) extends Result[Nothing, A]
case class Failure[E](error: E) extends Result[E, Nothing]

// Monadic operations
implicit class ResultOps[E, A](result: Result[E, A]) {
  def flatMap[B](f: A => Result[E, B]): Result[E, B] = result match {
    case Success(a) => f(a)
    case Failure(e) => Failure(e)
  }
  
  def map[B](f: A => B): Result[E, B] = result match {
    case Success(a) => Success(f(a))
    case Failure(e) => Failure(e)
  }
  
  def orElse[B >: A](default: => B): B = result match {
    case Success(a) => a
    case Failure(_) => default
  }
}

// Using our Result type
def divide(a: Int, b: Int): Result[String, Int] =
  if (b == 0) Failure("Division by zero")
  else Success(a / b)

def sqrt(x: Int): Result[String, Double] =
  if (x < 0) Failure("Negative input")
  else Success(Math.sqrt(x))

// Composing operations
def divideAndSqrt(a: Int, b: Int): Result[String, Double] =
  for {
    div <- divide(a, b)
    result <- sqrt(div)
  } yield result
```

### Activity 3: Functional Data Processing with Spark

Apache Spark provides a functional API for distributed data processing:

```scala
// Using Spark's DataFrame API with functional operations
import org.apache.spark.sql.functions._

val sales = spark.read.parquet("sales.parquet")

val processedSales = sales
  .filter(col("date") >= "2023-01-01")
  .groupBy("product_id")
  .agg(
    sum("quantity").as("total_quantity"),
    sum("amount").as("total_amount")
  )
  .withColumn("average_price", col("total_amount") / col("total_quantity"))
  .orderBy(col("total_amount").desc)
  .limit(10)

processedSales.write.parquet("top_products.parquet")
```

## Best Practices

### Code Design

1. **Favor immutability**: Use immutable data structures when possible
2. **Write pure functions**: Minimize side effects
3. **Use function composition**: Build complex operations from simple functions
4. **Apply type safety**: Leverage strong typing to prevent errors
5. **Embrace higher-order functions**: Use functions that operate on other functions

### Performance Considerations

1. **Be mindful of recursion**: Ensure tail recursion or use iteration for deep recursion
2. **Use memoization**: Cache results of pure functions for performance
3. **Consider memory usage**: Immutable data structures can use more memory
4. **Leverage lazy evaluation**: Process data only when needed
5. **Understand the cost of functional operations**: Some operations may have hidden overhead

### Testing

1. **Test pure functions individually**: They're easy to test in isolation
2. **Use property-based testing**: Test function properties rather than specific cases
3. **Mock dependencies**: For functions that interact with external systems
4. **Test composition**: Ensure composed functions work correctly together
5. **Verify error handling**: Test both success and failure paths

## Conclusion

Functional programming offers powerful tools for data engineering tasks. By emphasizing pure functions, immutable data, and declarative code, it leads to more maintainable, testable, and often more efficient systems. While you don't need to adopt a purely functional style, integrating functional concepts into your data engineering work can significantly improve code quality and developer productivity.

The functional paradigm is especially valuable in data pipelines, where data flows through a series of transformations. By thinking of each stage as a pure function, you can build complex pipelines from simple, reusable components.

As you continue your data engineering journey, look for opportunities to apply functional techniques in your code. Start with small steps—using higher-order functions like map and filter, writing pure functions, and avoiding mutable state—and gradually incorporate more advanced functional patterns as you become comfortable with the paradigm.

## References and Further Reading

1. "Functional Programming in Scala" by Paul Chiusano and Rúnar Bjarnason
2. "Java 8 in Action" by Raoul-Gabriel Urma
3. "Functional Python Programming" by Steven Lott
4. "Domain Modeling Made Functional" by Scott Wlaschin
5. "Professor Frisby's Mostly Adequate Guide to Functional Programming" (free online book)