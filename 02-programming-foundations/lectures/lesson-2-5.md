# Lesson 2.5: Java/Scala Basics

## Navigation
- [← Back to Lesson Plan](../2.5-java-scala-basics.md)
- [← Back to Module Overview](../README.md)

## Overview
Java and Scala are powerful programming languages that form the backbone of many data engineering technologies like Apache Spark, Kafka, and Hadoop. While Python offers simplicity and rapid development, Java and Scala provide performance benefits, strong typing, and superior concurrency support - critical advantages for large-scale data processing systems. This lecture introduces the fundamentals of both languages from a data engineering perspective.

## Java Fundamentals

### Introduction to Java

Java is a class-based, object-oriented language designed for portability and platform independence. Its "write once, run anywhere" philosophy comes from the Java Virtual Machine (JVM), which allows Java code to run on any device with a JVM installed.

Key Java characteristics important for data engineering:

- **Static typing**: Variables must be declared with specific types
- **JVM-based**: Code compiles to bytecode for the Java Virtual Machine
- **Mature ecosystem**: Extensive libraries and frameworks
- **Concurrency support**: Robust multithreading capabilities
- **Enterprise adoption**: Widely used in large-scale systems

### Basic Syntax and Structure

A simple Java program:

```java
// File: HelloWorld.java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, Data Engineering World!");
    }
}
```

Key elements:
- `public class HelloWorld`: Class declaration matching the filename
- `public static void main(String[] args)`: Main method entry point
- `System.out.println()`: Standard output method

To compile and run:
```
javac HelloWorld.java
java HelloWorld
```

### Data Types and Variables

Java has two categories of data types:

**Primitive Types**:
```java
// Integer types
byte b = 127;          // 8-bit, -128 to 127
short s = 32767;       // 16-bit, -32,768 to 32,767
int i = 2147483647;    // 32-bit, -2^31 to 2^31-1
long l = 9223372036854775807L; // 64-bit, -2^63 to 2^63-1

// Floating-point types
float f = 3.14f;       // 32-bit, 'f' suffix required
double d = 3.14159265; // 64-bit, more precision

// Other primitives
boolean bool = true;   // true or false
char c = 'A';          // 16-bit Unicode character
```

**Reference Types**:
```java
// Strings
String text = "Hello, Data Engineering!";

// Arrays
int[] numbers = {1, 2, 3, 4, 5};
String[] names = new String[10]; // Array with 10 elements

// Classes and objects
Date today = new Date();
```

### Control Structures

**Conditional Statements**:
```java
// If-else statement
if (temperature > 30) {
    System.out.println("Hot day!");
} else if (temperature > 20) {
    System.out.println("Pleasant day!");
} else {
    System.out.println("Cold day!");
}

// Switch statement
switch (dayOfWeek) {
    case "Monday":
        System.out.println("Start of work week");
        break;
    case "Friday":
        System.out.println("End of work week");
        break;
    case "Saturday":
    case "Sunday":
        System.out.println("Weekend!");
        break;
    default:
        System.out.println("Mid-week");
        break;
}
```

**Loops**:
```java
// For loop
for (int i = 0; i < 10; i++) {
    System.out.println("Iteration: " + i);
}

// Enhanced for loop (for-each)
String[] fruits = {"Apple", "Banana", "Cherry"};
for (String fruit : fruits) {
    System.out.println("Fruit: " + fruit);
}

// While loop
int count = 0;
while (count < 5) {
    System.out.println("Count: " + count);
    count++;
}

// Do-while loop
int i = 0;
do {
    System.out.println("Value: " + i);
    i++;
} while (i < 5);
```

### Object-Oriented Programming

**Classes and Objects**:
```java
// Class definition
public class Person {
    // Fields (attributes)
    private String name;
    private int age;
    
    // Constructor
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    // Methods
    public String getName() {
        return name;
    }
    
    public void setName(String name) {
        this.name = name;
    }
    
    public int getAge() {
        return age;
    }
    
    public void setAge(int age) {
        this.age = age;
    }
    
    public void introduce() {
        System.out.println("Hello, my name is " + name + 
                           " and I am " + age + " years old.");
    }
}

// Using the class
Person person = new Person("Alice", 30);
person.introduce();
person.setAge(31);
System.out.println(person.getName() + " is now " + person.getAge());
```

**Inheritance**:
```java
// Base class
public class Employee {
    private String name;
    private String id;
    
    public Employee(String name, String id) {
        this.name = name;
        this.id = id;
    }
    
    public void work() {
        System.out.println(name + " is working");
    }
}

// Derived class
public class DataEngineer extends Employee {
    private String[] skills;
    
    public DataEngineer(String name, String id, String[] skills) {
        super(name, id);  // Call parent constructor
        this.skills = skills;
    }
    
    // Override parent method
    @Override
    public void work() {
        super.work();  // Call parent method
        System.out.println("Building data pipelines");
    }
    
    public void code() {
        System.out.println("Writing Java code");
    }
}
```

**Interfaces**:
```java
// Interface definition
public interface Processor {
    void process(String data);
    int getProcessedCount();
}

// Implementation
public class DataProcessor implements Processor {
    private int count = 0;
    
    @Override
    public void process(String data) {
        System.out.println("Processing: " + data);
        count++;
    }
    
    @Override
    public int getProcessedCount() {
        return count;
    }
}
```

### Exception Handling

```java
// Try-catch-finally
try {
    File file = new File("data.txt");
    Scanner scanner = new Scanner(file);
    while (scanner.hasNextLine()) {
        String line = scanner.nextLine();
        processLine(line);
    }
    scanner.close();
} catch (FileNotFoundException e) {
    System.err.println("Error: File not found");
    e.printStackTrace();
} catch (Exception e) {
    System.err.println("Error processing file");
    e.printStackTrace();
} finally {
    System.out.println("File processing completed");
}

// Try-with-resources (Java 7+)
try (Scanner scanner = new Scanner(new File("data.txt"))) {
    while (scanner.hasNextLine()) {
        String line = scanner.nextLine();
        processLine(line);
    }
} catch (Exception e) {
    System.err.println("Error: " + e.getMessage());
}
```

### Collections Framework

Java provides powerful data structures through its Collections Framework:

```java
// Lists - ordered collections
List<String> names = new ArrayList<>();
names.add("Alice");
names.add("Bob");
names.add("Charlie");
System.out.println("First name: " + names.get(0));

// Sets - unique elements
Set<String> uniqueNames = new HashSet<>();
uniqueNames.add("Alice");
uniqueNames.add("Bob");
uniqueNames.add("Alice");  // Duplicate, won't be added
System.out.println("Set size: " + uniqueNames.size());  // 2

// Maps - key-value pairs
Map<String, Integer> ages = new HashMap<>();
ages.put("Alice", 30);
ages.put("Bob", 25);
ages.put("Charlie", 35);
System.out.println("Bob's age: " + ages.get("Bob"));

// Iterating through collections
for (String name : names) {
    System.out.println(name);
}

// Stream API (Java 8+)
names.stream()
     .filter(name -> name.startsWith("A"))
     .map(String::toUpperCase)
     .forEach(System.out::println);
```

### Java for Data Engineering

**File I/O**:
```java
// Reading a file line by line
Path filePath = Paths.get("data.csv");
try (BufferedReader reader = Files.newBufferedReader(filePath)) {
    String line;
    while ((line = reader.readLine()) != null) {
        // Process each line
        String[] parts = line.split(",");
        // ...
    }
} catch (IOException e) {
    e.printStackTrace();
}

// Writing to a file
List<String> lines = Arrays.asList("Line 1", "Line 2", "Line 3");
try {
    Files.write(Paths.get("output.txt"), lines);
} catch (IOException e) {
    e.printStackTrace();
}
```

**Database Connectivity (JDBC)**:
```java
// Connecting to a database
String url = "jdbc:postgresql://localhost:5432/mydatabase";
String user = "username";
String password = "password";

try (Connection conn = DriverManager.getConnection(url, user, password)) {
    // Query execution
    String sql = "SELECT * FROM employees WHERE department = ?";
    PreparedStatement pstmt = conn.prepareStatement(sql);
    pstmt.setString(1, "Engineering");
    
    ResultSet rs = pstmt.executeQuery();
    while (rs.next()) {
        int id = rs.getInt("employee_id");
        String name = rs.getString("name");
        System.out.println("Employee: " + id + " - " + name);
    }
} catch (SQLException e) {
    e.printStackTrace();
}
```

**Multithreading**:
```java
// Creating a thread
Thread thread = new Thread(() -> {
    for (int i = 0; i < 5; i++) {
        System.out.println("Thread running: " + i);
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
});
thread.start();

// ExecutorService for thread pool
ExecutorService executor = Executors.newFixedThreadPool(5);
for (int i = 0; i < 10; i++) {
    final int taskId = i;
    executor.submit(() -> {
        System.out.println("Task " + taskId + " executed by " + 
                          Thread.currentThread().getName());
    });
}
executor.shutdown();
```

## Scala Fundamentals

### Introduction to Scala

Scala (Scalable Language) combines object-oriented and functional programming paradigms, running on the JVM and interoperating seamlessly with Java. It's particularly popular for big data processing with Apache Spark.

Key Scala characteristics for data engineering:

- **Conciseness**: More expressive than Java, less boilerplate
- **Functional programming**: First-class functions, immutability
- **Static typing**: Type inference reduces type declarations
- **JVM compatibility**: Can use Java libraries directly
- **Apache Spark**: Spark's native language

### Basic Syntax and Structure

A simple Scala program:

```scala
// File: HelloWorld.scala
object HelloWorld {
  def main(args: Array[String]): Unit = {
    println("Hello, Data Engineering World!")
  }
}
```

Alternative syntax using app trait:

```scala
// File: HelloWorld.scala
object HelloWorld extends App {
  println("Hello, Data Engineering World!")
}
```

To compile and run:
```
scalac HelloWorld.scala
scala HelloWorld
```

### Variables and Data Types

Scala handles variables differently from Java:

```scala
// Immutable (val) - cannot be reassigned
val name: String = "Alice"
val age = 30  // Type inference - Scala infers 'Int'

// Mutable (var) - can be reassigned
var score = 100
score = 110  // Valid

// Basic types
val anInt: Int = 42
val aLong: Long = 42L
val aDouble: Double = 3.14
val aFloat: Float = 3.14f
val aChar: Char = 'A'
val aString: String = "Hello"
val aBoolean: Boolean = true

// Collections
val numbers = List(1, 2, 3, 4, 5)
val nameToAge = Map("Alice" -> 30, "Bob" -> 25)
val uniqueNumbers = Set(1, 2, 3)
```

### Control Structures

**Conditional Expressions**:
```scala
// If-else (returns a value)
val status = if (temperature > 30) {
  "Hot day!"
} else if (temperature > 20) {
  "Pleasant day!"
} else {
  "Cold day!"
}

// Pattern matching (similar to switch but more powerful)
val dayType = dayOfWeek match {
  case "Monday" => "Start of work week"
  case "Friday" => "End of work week"
  case "Saturday" | "Sunday" => "Weekend!"
  case _ => "Mid-week"  // Default case
}
```

**Loops**:
```scala
// For loop (with yield for creating collections)
val squares = for (i <- 1 to 5) yield i * i
// Results in: Vector(1, 4, 9, 16, 25)

// For loop with conditions
for {
  i <- 1 to 10
  if i % 2 == 0
} {
  println(s"Even number: $i")
}

// While loop
var i = 0
while (i < 5) {
  println(s"Count: $i")
  i += 1
}
```

### Functional Programming

**Functions**:
```scala
// Function definition
def add(a: Int, b: Int): Int = {
  a + b
}

// Simplified one-liner
def multiply(a: Int, b: Int): Int = a * b

// Function with default parameter
def greet(name: String = "World"): Unit = {
  println(s"Hello, $name!")
}

// Calling functions
val sum = add(5, 3)
greet()
greet("Scala")
```

**Higher-Order Functions**:
```scala
// Functions that take functions as parameters
def applyTwice(f: Int => Int, x: Int): Int = {
  f(f(x))
}

val addOne = (x: Int) => x + 1
val result = applyTwice(addOne, 3)  // Result: 5

// Common higher-order functions on collections
val numbers = List(1, 2, 3, 4, 5)

// Map - transform each element
val doubled = numbers.map(x => x * 2)  // List(2, 4, 6, 8, 10)

// Filter - keep elements that satisfy a predicate
val evens = numbers.filter(x => x % 2 == 0)  // List(2, 4)

// Reduce - combine elements
val sum = numbers.reduce((a, b) => a + b)  // 15

// FoldLeft - combine with initial value
val sumPlusOne = numbers.foldLeft(1)((acc, x) => acc + x)  // 16
```

**Anonymous Functions (Lambdas)**:
```scala
// Full syntax
val add = (a: Int, b: Int) => a + b

// With type inference
val numbers = List(1, 2, 3, 4, 5)
val evens = numbers.filter(_ % 2 == 0)  // Shorthand for x => x % 2 == 0

// Multi-line lambda
val process = (data: String) => {
  val processed = data.trim.toLowerCase
  processed.replaceAll("\\s+", "_")
}
```

### Object-Oriented Programming in Scala

**Classes and Objects**:
```scala
// Class definition
class Person(val name: String, var age: Int) {
  // Constructor body
  println(s"Creating a person named $name")
  
  // Method
  def introduce(): Unit = {
    println(s"Hello, my name is $name and I am $age years old.")
  }
  
  // Additional constructor
  def this(name: String) = {
    this(name, 0)
  }
}

// Creating instances
val alice = new Person("Alice", 30)
alice.introduce()
alice.age = 31  // Mutable field
println(alice.name)  // Immutable field

// Singleton object
object MathUtils {
  def square(x: Int): Int = x * x
  def cube(x: Int): Int = x * x * x
}

// Using singleton object
val squared = MathUtils.square(4)  // 16
```

**Case Classes**:
```scala
// Case class - immutable data containers with built-in functionality
case class Point(x: Int, y: Int)

// Creating instances
val p1 = Point(10, 20)
val p2 = Point(10, 20)

// Built-in toString
println(p1)  // Point(10,20)

// Built-in equality
println(p1 == p2)  // true

// Copy with modifications
val p3 = p1.copy(y = 30)  // Point(10,30)

// Pattern matching
p1 match {
  case Point(0, 0) => println("Origin")
  case Point(x, 0) => println(s"X-axis at $x")
  case Point(0, y) => println(s"Y-axis at $y")
  case Point(x, y) => println(s"Point at ($x, $y)")
}
```

**Traits**:
```scala
// Trait - similar to interfaces but can contain implementations
trait Logger {
  def log(message: String): Unit
  
  // Default implementation
  def warn(message: String): Unit = {
    log(s"WARNING: $message")
  }
}

// Class implementing a trait
class ConsoleLogger extends Logger {
  def log(message: String): Unit = {
    println(s"LOG: $message")
  }
  
  // We can use warn() as is or override it
}

// Using traits for mixin composition
trait Timestamping {
  def timestamp(): String = {
    val now = java.time.LocalDateTime.now()
    now.toString
  }
}

// Mixing in multiple traits
class TimestampedLogger extends Logger with Timestamping {
  def log(message: String): Unit = {
    println(s"${timestamp()}: $message")
  }
}
```

### Collections and Functional Operations

Scala has a rich collections library with functional operations:

```scala
// List operations
val numbers = List(1, 2, 3, 4, 5)

// Mapping
val squared = numbers.map(x => x * x)

// Filtering
val evens = numbers.filter(_ % 2 == 0)

// Flatmap - map + flatten
val nestedLists = List(List(1, 2), List(3, 4))
val flattened = nestedLists.flatMap(identity)  // List(1, 2, 3, 4)

// GroupBy
val people = List(("Alice", 30), ("Bob", 25), ("Charlie", 30))
val groupedByAge = people.groupBy(_._2)
// Map(30 -> List((Alice,30), (Charlie,30)), 25 -> List((Bob,25)))

// Sorting
val sorted = numbers.sorted
val customSorted = people.sortBy(_._2)  // Sort by age

// Collections operations chaining
val result = numbers
  .filter(_ > 2)
  .map(_ * 2)
  .sorted
  .mkString(", ")  // "6, 8, 10"
```

### Scala for Data Engineering

**File I/O**:
```scala
import scala.io.Source
import java.io.{FileWriter, BufferedWriter}

// Reading files
def readFile(filename: String): List[String] = {
  val source = Source.fromFile(filename)
  try {
    source.getLines().toList
  } finally {
    source.close()
  }
}

// Writing files
def writeFile(filename: String, lines: List[String]): Unit = {
  val writer = new BufferedWriter(new FileWriter(filename))
  try {
    lines.foreach { line =>
      writer.write(line)
      writer.newLine()
    }
  } finally {
    writer.close()
  }
}
```

**Error Handling**:
```scala
// Try-Success-Failure pattern
import scala.util.{Try, Success, Failure}

def divide(a: Int, b: Int): Try[Int] = {
  Try(a / b)
}

divide(10, 2) match {
  case Success(result) => println(s"Result: $result")
  case Failure(exception) => println(s"Error: ${exception.getMessage}")
}

// Option for handling nullable values
def findUser(id: Int): Option[String] = {
  val users = Map(1 -> "Alice", 2 -> "Bob")
  users.get(id)
}

val userName = findUser(3) match {
  case Some(name) => name
  case None => "Unknown user"
}

// Either for error handling
def computeValue(input: String): Either[String, Int] = {
  try {
    Right(input.toInt * 2)
  } catch {
    case _: NumberFormatException => Left("Invalid input: not a number")
  }
}

computeValue("42") match {
  case Right(value) => println(s"Computed: $value")
  case Left(error) => println(s"Error: $error")
}
```

## Functional Programming in Java & Scala

### Java 8+ Functional Features

```java
// Lambda expressions
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// Using lambda with forEach
numbers.forEach(n -> System.out.println(n));

// Using method reference
numbers.forEach(System.out::println);

// Stream API for functional operations
int sum = numbers.stream()
                 .filter(n -> n % 2 == 0)
                 .mapToInt(n -> n * n)
                 .sum();

// Collectors for accumulating results
Map<Boolean, List<Integer>> evenOddMap = numbers.stream()
    .collect(Collectors.partitioningBy(n -> n % 2 == 0));

// Optional for handling nullable values
Optional<String> optionalName = Optional.ofNullable(getName());
String name = optionalName.orElse("Unknown");
```

### Scala's Functional Approach

```scala
// Function composition
val addOne = (x: Int) => x + 1
val double = (x: Int) => x * 2
val addOneThenDouble = double compose addOne  // f(g(x))
val doubleTheAddOne = addOne andThen double   // g(f(x))

// Partial functions
val sqrt = new PartialFunction[Double, Double] {
  def apply(x: Double) = Math.sqrt(x)
  def isDefinedAt(x: Double) = x >= 0
}

// Function currying
def multiply(a: Int)(b: Int): Int = a * b
val multiplyByTwo = multiply(2) _  // Partially applied function

// Pattern matching in functions
def describe(x: Any): String = x match {
  case i: Int if i > 0 => "Positive integer"
  case 0 => "Zero"
  case s: String => s"String: $s"
  case _ => "Something else"
}
```

## Java/Scala Interoperability

One of Scala's strengths is its seamless interoperability with Java:

```scala
// Using Java libraries in Scala
import java.util.{ArrayList, HashMap}
import java.text.SimpleDateFormat
import java.util.Date

// Java collection in Scala
val javaList = new ArrayList[String]()
javaList.add("Item 1")
javaList.add("Item 2")

// Java date handling
val formatter = new SimpleDateFormat("yyyy-MM-dd")
val date = formatter.format(new Date())
```

```java
// Using Scala from Java
import scala.collection.immutable.List;
import scala.jdk.CollectionConverters;
import scala.Option;

// Scala 2.13+ provides conversion utilities
List<String> scalaList = List.apply("One", "Two", "Three");
java.util.List<String> javaList = CollectionConverters.ListHasAsJava(scalaList).asJava();

// Handling Scala Option in Java
Option<String> scalaOption = Option.apply("Value");
String value = scalaOption.isDefined() ? scalaOption.get() : "Default";
```

## Apache Spark with Scala/Java

### Spark with Scala

```scala
import org.apache.spark.sql.{SparkSession, DataFrame}

// Create SparkSession
val spark = SparkSession.builder()
  .appName("ScalaSparkExample")
  .master("local[*]")
  .getOrCreate()

// Read CSV file
val df = spark.read
  .option("header", "true")
  .option("inferSchema", "true")
  .csv("data.csv")

// DataFrame transformations
val processedDf = df.filter($"age" > 25)
  .select($"name", $"age", $"salary")
  .groupBy($"age")
  .agg(
    avg($"salary").as("avg_salary"),
    count("*").as("count")
  )
  .orderBy($"avg_salary".desc)

// RDD operations
val rdd = spark.sparkContext.parallelize(1 to 100)
val filtered = rdd.filter(_ % 2 == 0)
val doubled = filtered.map(_ * 2)
val sum = doubled.reduce(_ + _)

// Write results
processedDf.write.mode("overwrite").parquet("output/processed_data")
```

### Spark with Java

```java
import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.SparkSession;
import static org.apache.spark.sql.functions.*;

// Create SparkSession
SparkSession spark = SparkSession.builder()
    .appName("JavaSparkExample")
    .master("local[*]")
    .getOrCreate();

// Read CSV file
Dataset<Row> df = spark.read()
    .option("header", "true")
    .option("inferSchema", "true")
    .csv("data.csv");

// DataFrame transformations
Dataset<Row> processedDf = df.filter(col("age").gt(25))
    .select(col("name"), col("age"), col("salary"))
    .groupBy(col("age"))
    .agg(
        avg(col("salary")).as("avg_salary"),
        count("*").as("count")
    )
    .orderBy(col("avg_salary").desc());

// Write results
processedDf.write().mode("overwrite").parquet("output/processed_data");
```

## Best Practices

### Java Best Practices

1. **Coding Style**
   - Follow Java naming conventions (camelCase for methods/variables, PascalCase for classes)
   - Prefer composition over inheritance
   - Use interfaces for flexibility
   - Make classes immutable when possible

2. **Performance**
   - Use appropriate collections for your needs
   - Avoid premature optimization
   - Use StringBuilder for string concatenation in loops
   - Be careful with autoboxing and unboxing

3. **Error Handling**
   - Catch specific exceptions rather than general Exception
   - Use try-with-resources for auto-closeable resources
   - Document exceptions in method signatures
   - Create custom exceptions for business logic

### Scala Best Practices

1. **Coding Style**
   - Prefer immutability (use val instead of var)
   - Use Option instead of null
   - Leverage case classes for data models
   - Use pattern matching instead of if-else chains

2. **Functional Programming**
   - Favor pure functions (no side effects)
   - Use higher-order functions on collections
   - Leverage function composition
   - Consider tail recursion for recursive algorithms

3. **Error Handling**
   - Use Option for values that might be absent
   - Use Either or Try for operations that might fail
   - Prefer pattern matching for error handling
   - Consider the "railway-oriented programming" approach

### Data Engineering Specific

1. **Processing Large Datasets**
   - Use lazy evaluation when possible
   - Prefer streaming over loading entire datasets
   - Understand memory implications of your operations
   - Consider partitioning strategies

2. **Performance Optimization**
   - Profile before optimizing
   - Understand serialization implications
   - Consider data locality
   - Use appropriate data structures

3. **Concurrency**
   - Prefer immutable data structures for concurrent code
   - Use higher-level concurrency abstractions when possible
   - Be careful with shared mutable state
   - Understand thread safety

## Conclusion

Java and Scala both provide powerful tools for data engineering tasks, with different strengths and approaches. Java offers stability, wide adoption, and a mature ecosystem, while Scala provides conciseness, functional programming paradigms, and excellent integration with big data frameworks like Spark.

As a data engineer, becoming proficient in these JVM languages opens up many opportunities to work with enterprise-grade systems and high-performance big data frameworks. While you don't need to master every aspect of these languages, understanding their fundamentals and how they relate to data engineering tasks will significantly enhance your toolbox.

Remember that the goal isn't to replace Python in your workflow, but to complement it. Each language has its place in the data engineering ecosystem, and knowing when to use each one is a valuable skill.

## Next Steps

- Practice basic Java and Scala syntax with small programs
- Experiment with Spark's Java and Scala APIs
- Compare implementations of the same data processing tasks in Python, Java, and Scala
- Learn more about functional programming concepts
- Explore real-world data engineering projects that use Java and Scala