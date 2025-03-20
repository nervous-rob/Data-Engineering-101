# Lesson 4.5: Advanced Data Formats

## Navigation
- [← Back to Lesson Plan](../4.5-advanced-data-formats.md)
- [← Back to Module Overview](../README.md)

## Overview
Modern data engineering requires a deep understanding of various data formats and their optimal use cases. This lesson explores advanced data formats, their characteristics, and performance implications in big data processing. We'll examine how different formats handle schema evolution, compression, and query optimization, enabling data engineers to make informed decisions about data storage and processing strategies.

## Learning Objectives
- Master different data formats and their specific use cases
- Understand format-specific optimization techniques
- Implement efficient data serialization strategies
- Learn about schema evolution and compatibility
- Practice performance optimization techniques
- Evaluate format selection criteria

## Data Format Fundamentals

### Text-Based Formats

#### 1. CSV (Comma-Separated Values)
```python
def work_with_csv(spark):
    """Demonstrate CSV handling with optimization."""
    # Reading with options
    df = spark.read \
        .option("header", "true") \
        .option("inferSchema", "true") \
        .option("mode", "DROPMALFORMED") \
        .option("escape", "\"") \
        .csv("data.csv")
    
    # Writing with optimization
    df.write \
        .option("header", "true") \
        .option("compression", "gzip") \
        .option("sep", ",") \
        .mode("overwrite") \
        .csv("output.csv")

    return df
```

Advantages:
- Human-readable
- Universal compatibility
- Easy to edit manually
- Simple structure

Disadvantages:
- No schema enforcement
- Poor compression
- Slow parsing
- Limited data types

#### 2. JSON (JavaScript Object Notation)
```python
def optimize_json_processing(spark):
    """Optimize JSON data processing."""
    # Read JSON with optimization
    df = spark.read \
        .option("multiLine", "true") \
        .option("primitivesAsString", "false") \
        .option("prefersDecimal", "true") \
        .json("data.json")
    
    # Write optimized JSON
    df.write \
        .option("compression", "gzip") \
        .option("dateFormat", "yyyy-MM-dd") \
        .option("timestampFormat", "yyyy-MM-dd'T'HH:mm:ss.SSSZ") \
        .mode("overwrite") \
        .json("output.json")
    
    return df
```

Advantages:
- Schema flexibility
- Human-readable
- Wide support
- Nested structures

Disadvantages:
- Verbose format
- Space inefficient
- Slower parsing
- No schema enforcement

### Binary Formats

#### 1. Parquet
```python
def optimize_parquet_operations(spark):
    """Demonstrate Parquet optimization techniques."""
    # Reading with optimization
    df = spark.read \
        .option("mergeSchema", "true") \
        .option("vectorizedReader", "true") \
        .parquet("data.parquet")
    
    # Writing with optimization
    df.write \
        .option("compression", "snappy") \
        .option("maxRecordsPerFile", 1000000) \
        .partitionBy("year", "month") \
        .bucketBy(4, "user_id") \
        .sortBy("timestamp") \
        .mode("overwrite") \
        .parquet("optimized.parquet")
    
    # Demonstrate predicate pushdown
    filtered_df = spark.read.parquet("data.parquet") \
        .filter("date >= '2024-01-01'") \
        .select("user_id", "amount")  # Column pruning
    
    return df

def parquet_schema_evolution(spark):
    """Handle schema evolution in Parquet."""
    # Original schema
    original_df = spark.createDataFrame([
        (1, "John", 30)
    ], ["id", "name", "age"])
    
    original_df.write.parquet("data_v1.parquet")
    
    # Updated schema
    updated_df = spark.createDataFrame([
        (1, "John", 30, "john@example.com")
    ], ["id", "name", "age", "email"])
    
    # Merge schema during write
    updated_df.write \
        .option("mergeSchema", "true") \
        .mode("append") \
        .parquet("data_v1.parquet")
```

Advantages:
- Columnar storage
- Excellent compression
- Schema enforcement
- Predicate pushdown
- Column pruning

Disadvantages:
- Not human-readable
- Complex file format
- Limited streaming support

#### 2. Avro
```python
def implement_avro_processing(spark):
    """Implement Avro data processing with schema evolution."""
    # Define Avro schema
    schema = """
    {
        "type": "record",
        "name": "User",
        "fields": [
            {"name": "id", "type": "int"},
            {"name": "name", "type": "string"},
            {"name": "email", "type": ["null", "string"], "default": null},
            {"name": "age", "type": "int"}
        ]
    }
    """
    
    # Write with schema
    df.write \
        .format("avro") \
        .option("avroSchema", schema) \
        .option("compression", "snappy") \
        .save("users.avro")
    
    # Read with schema evolution
    evolved_schema = """
    {
        "type": "record",
        "name": "User",
        "fields": [
            {"name": "id", "type": "int"},
            {"name": "name", "type": "string"},
            {"name": "email", "type": ["null", "string"], "default": null},
            {"name": "age", "type": "int"},
            {"name": "address", "type": ["null", "string"], "default": null}
        ]
    }
    """
    
    df = spark.read \
        .format("avro") \
        .option("avroSchema", evolved_schema) \
        .load("users.avro")
    
    return df
```

Advantages:
- Rich data types
- Schema evolution
- Compact serialization
- Fast processing
- Row-based format

Disadvantages:
- No column pruning
- Larger storage size
- Complex schema definition

#### 3. ORC (Optimized Row Columnar)
```python
def optimize_orc_operations(spark):
    """Demonstrate ORC optimization techniques."""
    # Write with optimization
    df.write \
        .format("orc") \
        .option("compression", "zlib") \
        .option("orc.bloom.filter.columns", "id,category") \
        .option("orc.stripe.size", "67108864")  # 64MB
        .option("orc.row.index.stride", "10000") \
        .save("data.orc")
    
    # Read with predicate pushdown
    df = spark.read \
        .format("orc") \
        .option("orc.merge.schema", "true") \
        .load("data.orc") \
        .filter("category = 'A'")
    
    return df
```

Advantages:
- ACID transactions
- Built-in indexes
- Good compression
- Fast queries
- Column pruning

Disadvantages:
- Hive ecosystem dependency
- Complex format
- Limited tool support

### Delta Lake Format
```python
from delta.tables import DeltaTable

def implement_delta_lake(spark):
    """Implement Delta Lake features."""
    # Write as Delta table
    df.write \
        .format("delta") \
        .partitionBy("date") \
        .option("mergeSchema", "true") \
        .save("delta_table")
    
    # Time travel
    df_historical = spark.read \
        .format("delta") \
        .option("versionAsOf", "1") \
        .load("delta_table")
    
    # MERGE operation
    delta_table = DeltaTable.forPath(spark, "delta_table")
    
    delta_table.alias("target").merge(
        source_df.alias("source"),
        "target.id = source.id"
    ).whenMatchedUpdate(
        set={"value": "source.value"}
    ).whenNotMatchedInsert(
        values={"id": "source.id", "value": "source.value"}
    ).execute()
    
    # Optimize table
    delta_table.optimize().executeCompaction()
    
    # Vacuum old versions
    delta_table.vacuum(168)  # Retain 7 days of history
    
    return delta_table
```

Advantages:
- ACID transactions
- Time travel
- Schema evolution
- Audit history
- Unified batch/streaming

Disadvantages:
- Storage overhead
- Processing overhead
- Limited ecosystem support

## Performance Optimization Techniques

### 1. Compression Strategies
```python
def implement_compression_strategies(spark):
    """Implement various compression strategies."""
    # Parquet with different compression codecs
    df.write \
        .format("parquet") \
        .option("compression", "snappy")  # Fast compression/decompression
        .save("snappy_compressed")
    
    df.write \
        .format("parquet") \
        .option("compression", "gzip")    # Better compression ratio
        .save("gzip_compressed")
    
    df.write \
        .format("parquet") \
        .option("compression", "zstd")    # Best compression ratio
        .save("zstd_compressed")
    
    # Compare sizes and read performance
    def compare_formats(path):
        start_time = time.time()
        count = spark.read.parquet(path).count()
        end_time = time.time()
        return {
            'count': count,
            'read_time': end_time - start_time,
            'size': get_directory_size(path)
        }
    
    return {
        'snappy': compare_formats("snappy_compressed"),
        'gzip': compare_formats("gzip_compressed"),
        'zstd': compare_formats("zstd_compressed")
    }
```

### 2. Partitioning and Bucketing
```python
def optimize_data_layout(spark):
    """Optimize data layout with partitioning and bucketing."""
    # Partitioning strategy
    df.write \
        .partitionBy("year", "month", "day") \
        .bucketBy(4, "user_id") \
        .sortBy("timestamp") \
        .format("parquet") \
        .save("optimized_layout")
    
    # Read with partition pruning
    df = spark.read \
        .format("parquet") \
        .load("optimized_layout") \
        .filter("year = 2024 AND month = 3")
    
    return df

def analyze_partition_strategy(spark, df, partition_cols):
    """Analyze partition strategy effectiveness."""
    # Calculate partition statistics
    stats = df.groupBy(partition_cols).count()
    
    # Analyze partition skew
    skew_analysis = stats.select([
        mean('count').alias('mean_records'),
        stddev('count').alias('stddev_records'),
        min('count').alias('min_records'),
        max('count').alias('max_records')
    ]).collect()[0]
    
    # Calculate partition size recommendations
    total_records = df.count()
    recommended_partitions = max(1, total_records // 1000000)  # Aim for ~1M records per partition
    
    return {
        'statistics': skew_analysis,
        'recommended_partitions': recommended_partitions,
        'current_partitions': stats.count()
    }
```

### 3. Query Optimization
```python
def optimize_queries(spark):
    """Implement query optimization techniques."""
    # Enable adaptive query execution
    spark.conf.set("spark.sql.adaptive.enabled", "true")
    
    # Optimize join strategies
    df1 = spark.read.parquet("large_table.parquet")
    df2 = spark.read.parquet("small_table.parquet")
    
    # Broadcast join for small tables
    from pyspark.sql.functions import broadcast
    
    optimized_join = df1.join(
        broadcast(df2),
        "join_key"
    )
    
    # Repartition for better parallelism
    optimized_df = df1.repartition(
        spark.conf.get("spark.sql.shuffle.partitions")
    )
    
    return optimized_df
```

## Format Selection Guidelines

### 1. Use Case Analysis
```python
def analyze_format_suitability(requirements):
    """Analyze format suitability based on requirements."""
    format_scores = {
        'parquet': {
            'query_performance': 5,
            'compression': 5,
            'schema_evolution': 4,
            'streaming_support': 3,
            'tool_compatibility': 5
        },
        'avro': {
            'query_performance': 4,
            'compression': 4,
            'schema_evolution': 5,
            'streaming_support': 5,
            'tool_compatibility': 4
        },
        'orc': {
            'query_performance': 5,
            'compression': 5,
            'schema_evolution': 3,
            'streaming_support': 3,
            'tool_compatibility': 3
        },
        'delta': {
            'query_performance': 4,
            'compression': 5,
            'schema_evolution': 5,
            'streaming_support': 5,
            'tool_compatibility': 3
        }
    }
    
    # Calculate weighted scores based on requirements
    weighted_scores = {}
    for format_name, scores in format_scores.items():
        weighted_score = sum(
            scores[metric] * requirements[metric]
            for metric in requirements
        )
        weighted_scores[format_name] = weighted_score
    
    return weighted_scores
```

### 2. Performance Testing Framework
```python
def benchmark_formats(spark, test_df):
    """Benchmark different format performances."""
    formats = ['parquet', 'avro', 'orc', 'delta']
    operations = ['write', 'read', 'filter', 'aggregate']
    
    results = {}
    for format_name in formats:
        format_results = {}
        
        # Write test
        start_time = time.time()
        test_df.write \
            .format(format_name) \
            .save(f"benchmark_{format_name}")
        format_results['write'] = time.time() - start_time
        
        # Read test
        start_time = time.time()
        df = spark.read \
            .format(format_name) \
            .load(f"benchmark_{format_name}")
        format_results['read'] = time.time() - start_time
        
        # Filter test
        start_time = time.time()
        df.filter("id > 1000").count()
        format_results['filter'] = time.time() - start_time
        
        # Aggregate test
        start_time = time.time()
        df.groupBy("category").count().collect()
        format_results['aggregate'] = time.time() - start_time
        
        results[format_name] = format_results
    
    return results
```

## Best Practices

### 1. Format-Specific Optimization
- Use appropriate compression codec
- Implement proper partitioning
- Enable statistics collection
- Optimize file sizes
- Monitor performance metrics

### 2. Schema Management
- Plan for evolution
- Document changes
- Test compatibility
- Monitor schema drift
- Handle defaults properly

### 3. Performance Monitoring
- Track query performance
- Monitor storage efficiency
- Analyze access patterns
- Measure compression ratios
- Profile resource usage

### 4. Data Layout
- Choose optimal partitioning
- Consider data skew
- Plan file sizes
- Implement bucketing
- Optimize sorting

## Common Pitfalls and Anti-Patterns

### 1. Poor Format Selection
Problem:
- Mismatched format for use case
- Inefficient storage usage
- Poor query performance

Solution:
- Analyze requirements thoroughly
- Test with representative data
- Consider future needs
- Monitor performance metrics

### 2. Inefficient Partitioning
Problem:
- Too many partitions
- Data skew
- Partition explosion
- Poor query performance

Solution:
- Analyze data distribution
- Choose appropriate keys
- Monitor partition sizes
- Implement bucketing

### 3. Compression Issues
Problem:
- Wrong compression codec
- Excessive compression
- Poor read performance
- Resource wastage

Solution:
- Balance compression ratio
- Consider access patterns
- Test different codecs
- Monitor resource usage

### 4. Schema Evolution Problems
Problem:
- Breaking changes
- Data corruption
- Compatibility issues
- Processing failures

Solution:
- Plan schema changes
- Test evolution paths
- Maintain compatibility
- Document changes

## Conclusion

Understanding and effectively utilizing advanced data formats is crucial for building efficient data processing systems. By mastering format-specific optimizations, implementing proper data layout strategies, and following best practices, data engineers can significantly improve storage efficiency and query performance. The choice of data format should be driven by specific use case requirements, considering factors such as query patterns, storage constraints, and processing needs.

## Additional Resources

1. **Documentation and Guides**
   - [Apache Parquet Documentation](https://parquet.apache.org/docs/)
   - [Apache Avro Documentation](https://avro.apache.org/docs/current/)
   - [Delta Lake Documentation](https://docs.delta.io/latest/index.html)
   - [Apache ORC Documentation](https://orc.apache.org/docs/)

2. **Performance Optimization**
   - [Spark SQL Performance Tuning](https://spark.apache.org/docs/latest/sql-performance-tuning.html)
   - [Delta Lake Performance Guide](https://docs.delta.io/latest/optimizations-oss.html)
   - [Parquet Performance Tuning](https://parquet.apache.org/docs/file-format/performance/)

3. **Further Reading**
   - "High Performance Spark" by Holden Karau and Rachel Warren
   - "Learning Spark" by Jules Damji, Brooke Wenig, Tathagata Das, and Denny Lee
   - "Designing Data-Intensive Applications" by Martin Kleppmann 