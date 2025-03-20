# Lesson 5.4: Stream Processing with Spark Streaming

## Navigation
- [← Back to Lesson Plan](../5.4-stream-processing-with-spark-streaming.md)
- [← Back to Module Overview](../README.md)

## Overview
Apache Spark Streaming represents a powerful evolution in stream processing, offering both DStream-based and Structured Streaming APIs for real-time data processing. This lesson provides a deep dive into Spark's streaming capabilities, architectural patterns, and integration with modern streaming platforms like Apache Kafka.

## Learning Objectives
After completing this lesson, you'll be able to:
- Master Spark Structured Streaming's architecture and internal mechanics
- Implement sophisticated streaming patterns with windowing and watermarking
- Design fault-tolerant streaming applications with proper state management
- Build advanced Kafka integration patterns with security and scalability
- Optimize streaming performance and handle backpressure
- Deploy production-ready Spark Streaming applications
- Implement proper monitoring and debugging strategies

## Spark Structured Streaming Architecture

### 1. Core Components and Concepts

```python
from pyspark.sql import SparkSession
from pyspark.sql.types import *
from pyspark.sql.functions import *
from typing import Dict, Optional, List
import logging

class SparkStreamingContext:
    """Manages Spark Streaming context and configurations"""
    def __init__(
        self,
        app_name: str,
        master: str = "local[*]",
        log_level: str = "WARN"
    ):
        self.logger = logging.getLogger(__name__)
        self.conf = self._create_conf()
        self.spark = self._create_session(app_name, master)
        self.spark.sparkContext.setLogLevel(log_level)
        
    def _create_conf(self) -> Dict[str, str]:
        """Create Spark configuration with optimized settings"""
        return {
            # Streaming configurations
            "spark.sql.streaming.schemaInference": "true",
            "spark.sql.streaming.checkpointLocation": "/tmp/checkpoint",
            "spark.sql.streaming.minBatchesToRetain": "100",
            "spark.sql.streaming.pollingDelay": "10ms",
            
            # Memory configurations
            "spark.memory.offHeap.enabled": "true",
            "spark.memory.offHeap.size": "1g",
            "spark.memory.fraction": "0.8",
            "spark.memory.storageFraction": "0.3",
            
            # Shuffle configurations
            "spark.sql.shuffle.partitions": "200",
            "spark.default.parallelism": "200",
            
            # Network configurations
            "spark.network.timeout": "800s",
            "spark.executor.heartbeatInterval": "60s"
        }
        
    def _create_session(
        self,
        app_name: str,
        master: str
    ) -> SparkSession:
        """Create and configure Spark session"""
        builder = SparkSession.builder \
            .appName(app_name) \
            .master(master)
            
        # Apply configurations
        for key, value in self.conf.items():
            builder = builder.config(key, value)
            
        # Add necessary packages
        builder = builder.config(
            "spark.jars.packages",
            "org.apache.spark:spark-sql-kafka-0-10_2.12:3.5.0,"
            "org.apache.spark:spark-avro_2.12:3.5.0"
        )
        
        return builder.getOrCreate()
```

### 2. Stream Processing Engine

```python
class StreamProcessor:
    """Core stream processing engine"""
    def __init__(
        self,
        spark_session: SparkSession,
        input_schema: StructType,
        watermark_delay: str = "10 minutes",
        processing_time: str = "1 minute"
    ):
        self.spark = spark_session
        self.schema = input_schema
        self.watermark_delay = watermark_delay
        self.processing_time = processing_time
        self.logger = logging.getLogger(__name__)
        
    def create_streaming_df(
        self,
        source_config: Dict[str, str]
    ) -> DataFrame:
        """Create streaming DataFrame with optimized reader"""
        reader = self.spark.readStream
        
        # Apply source-specific configurations
        for key, value in source_config.items():
            reader = reader.option(key, value)
            
        # Create initial DataFrame
        df = reader.load()
        
        # Apply schema and parse data
        if "kafka" in source_config.get("format", ""):
            df = df.select(
                from_json(
                    col("value").cast("string"),
                    self.schema
                ).alias("data")
            ).select("data.*")
            
        return df
        
    def apply_transformations(
        self,
        df: DataFrame,
        timestamp_col: str,
        grouping_cols: List[str]
    ) -> DataFrame:
        """Apply streaming transformations with optimization"""
        return df \
            .withWatermark(timestamp_col, self.watermark_delay) \
            .groupBy(
                window(timestamp_col, self.processing_time),
                *grouping_cols
            ) \
            .agg(
                count("*").alias("event_count"),
                sum("value").alias("total_value"),
                avg("value").alias("avg_value"),
                max("value").alias("max_value"),
                min("value").alias("min_value")
            )
```

### 3. Advanced Windowing Operations

```python
class WindowOperations:
    """Advanced windowing patterns for stream processing"""
    def __init__(
        self,
        window_duration: str,
        slide_duration: str,
        watermark_delay: str
    ):
        self.window_duration = window_duration
        self.slide_duration = slide_duration
        self.watermark_delay = watermark_delay
        
    def apply_sliding_window(
        self,
        df: DataFrame,
        timestamp_col: str,
        grouping_cols: List[str]
    ) -> DataFrame:
        """Apply sliding window aggregations"""
        return df \
            .withWatermark(timestamp_col, self.watermark_delay) \
            .groupBy(
                window(
                    timestamp_col,
                    self.window_duration,
                    self.slide_duration
                ),
                *grouping_cols
            )
            
    def apply_session_window(
        self,
        df: DataFrame,
        timestamp_col: str,
        grouping_cols: List[str],
        session_gap: str
    ) -> DataFrame:
        """Apply session window aggregations"""
        return df \
            .withWatermark(timestamp_col, self.watermark_delay) \
            .groupBy(
                session_window(
                    timestamp_col,
                    session_gap
                ),
                *grouping_cols
            )
            
    def handle_late_data(
        self,
        df: DataFrame,
        timestamp_col: str,
        max_late_threshold: str
    ) -> DataFrame:
        """Handle late-arriving data with custom logic"""
        # Add late data marker
        df = df.withColumn(
            "is_late",
            col(timestamp_col) < current_timestamp() - expr(max_late_threshold)
        )
        
        # Split stream for different handling
        main_stream = df.filter(~col("is_late"))
        late_stream = df.filter(col("is_late"))
        
        # Process late data separately
        processed_late = self._process_late_data(late_stream)
        
        return main_stream.union(processed_late)
        
    def _process_late_data(self, df: DataFrame) -> DataFrame:
        """Custom processing for late data"""
        return df \
            .withColumn(
                "processing_time",
                current_timestamp()
            ) \
            .withColumn(
                "late_by",
                col("processing_time") - col("timestamp")
            )
```

## Kafka Integration Patterns

### 1. Secure Kafka Connection

```python
class SecureKafkaConnector:
    """Implements secure Kafka connection patterns"""
    def __init__(
        self,
        bootstrap_servers: str,
        security_protocol: str = "SASL_SSL",
        sasl_mechanism: str = "PLAIN",
        client_id: str = "spark-streaming-client"
    ):
        self.bootstrap_servers = bootstrap_servers
        self.security_protocol = security_protocol
        self.sasl_mechanism = sasl_mechanism
        self.client_id = client_id
        
    def create_source(
        self,
        spark: SparkSession,
        topics: List[str],
        starting_offsets: str = "latest",
        credentials: Optional[Dict[str, str]] = None
    ) -> DataFrame:
        """Create secure Kafka source"""
        options = {
            "kafka.bootstrap.servers": self.bootstrap_servers,
            "subscribe": ",".join(topics),
            "startingOffsets": starting_offsets,
            "kafka.security.protocol": self.security_protocol,
            "kafka.sasl.mechanism": self.sasl_mechanism,
            "kafka.client.id": self.client_id,
            "failOnDataLoss": "false",
            "maxOffsetsPerTrigger": "10000"
        }
        
        # Add security credentials if provided
        if credentials:
            jaas_config = (
                f"org.apache.kafka.common.security.plain.PlainLoginModule "
                f"required username='{credentials.get('username')}' "
                f"password='{credentials.get('password')}'"
            )
            options["kafka.sasl.jaas.config"] = jaas_config
            
        return spark.readStream \
            .format("kafka") \
            .options(**options) \
            .load()
            
    def create_sink(
        self,
        df: DataFrame,
        topic: str,
        checkpoint_location: str,
        trigger_interval: str = "1 minute",
        credentials: Optional[Dict[str, str]] = None
    ) -> StreamingQuery:
        """Create secure Kafka sink"""
        options = {
            "kafka.bootstrap.servers": self.bootstrap_servers,
            "topic": topic,
            "checkpointLocation": checkpoint_location,
            "kafka.security.protocol": self.security_protocol,
            "kafka.sasl.mechanism": self.sasl_mechanism,
            "kafka.client.id": f"{self.client_id}-sink"
        }
        
        # Add security credentials if provided
        if credentials:
            jaas_config = (
                f"org.apache.kafka.common.security.plain.PlainLoginModule "
                f"required username='{credentials.get('username')}' "
                f"password='{credentials.get('password')}'"
            )
            options["kafka.sasl.jaas.config"] = jaas_config
            
        return df.writeStream \
            .format("kafka") \
            .options(**options) \
            .trigger(processingTime=trigger_interval) \
            .start()
```

### 2. Advanced Error Handling

```python
class StreamingErrorHandler:
    """Advanced error handling for streaming applications"""
    def __init__(
        self,
        checkpoint_dir: str,
        dead_letter_topic: str,
        max_retries: int = 3
    ):
        self.checkpoint_dir = checkpoint_dir
        self.dead_letter_topic = dead_letter_topic
        self.max_retries = max_retries
        self.logger = logging.getLogger(__name__)
        
    def handle_corrupt_data(
        self,
        df: DataFrame,
        corruption_handler: Optional[Callable] = None
    ) -> DataFrame:
        """Handle corrupt data with custom handler"""
        if corruption_handler:
            return corruption_handler(df)
            
        return df.select(
            when(
                col("value").isNull() |
                col("value") == "" |
                col("value").rlike("^\\s*$"),
                lit("corrupt_data")
            ).otherwise(col("value")).alias("cleaned_value")
        )
        
    def configure_recovery(
        self,
        query: StreamingQuery,
        retry_interval: str = "1 minute"
    ) -> StreamingQuery:
        """Configure recovery mechanisms"""
        return query \
            .option("checkpointLocation", self.checkpoint_dir) \
            .option("failOnDataLoss", "false") \
            .option("maxOffsetsPerTrigger", 10000) \
            .option("maxRetries", self.max_retries) \
            .option("retryInterval", retry_interval)
            
    def handle_processing_error(
        self,
        error: Exception,
        record: Row,
        retry_count: int = 0
    ) -> None:
        """Handle processing errors with retry logic"""
        if retry_count < self.max_retries:
            self.logger.warning(
                f"Retrying processing. Attempt {retry_count + 1}"
            )
            # Implement retry logic
        else:
            self._send_to_dead_letter_topic(record, error)
            
    def _send_to_dead_letter_topic(
        self,
        record: Row,
        error: Exception
    ) -> None:
        """Send failed records to dead letter topic"""
        error_record = {
            "original_data": record.asDict(),
            "error_message": str(error),
            "timestamp": current_timestamp()
        }
        # Implement dead letter queue logic
```

## State Management and Checkpointing

### 1. State Store Management

```python
class StateManager:
    """Manages stateful operations in streaming"""
    def __init__(
        self,
        checkpoint_dir: str,
        state_store_type: str = "rocksdb"
    ):
        self.checkpoint_dir = checkpoint_dir
        self.state_store_type = state_store_type
        
    def configure_state_store(
        self,
        query: StreamingQuery
    ) -> StreamingQuery:
        """Configure state store settings"""
        return query \
            .option("spark.sql.streaming.stateStore.providerClass",
                   f"org.apache.spark.sql.execution.streaming.state.{self.state_store_type.capitalize()}StateStoreProvider") \
            .option("spark.sql.streaming.stateStore.compression.codec", "lz4") \
            .option("spark.sql.streaming.stateStore.maintenanceInterval", "1 min")
            
    def manage_state_cleanup(
        self,
        df: DataFrame,
        timestamp_col: str,
        retention_threshold: str
    ) -> DataFrame:
        """Implement state cleanup logic"""
        return df \
            .withWatermark(timestamp_col, retention_threshold) \
            .dropDuplicates(["id", "timestamp"]) \
            .filter(
                col(timestamp_col) > current_timestamp() - expr(retention_threshold)
            )
```

## Performance Optimization

### 1. Backpressure Handling

```python
class BackpressureHandler:
    """Handles backpressure in streaming applications"""
    def __init__(
        self,
        max_rate_per_partition: int = 1000,
        min_batches_to_retain: int = 100
    ):
        self.max_rate_per_partition = max_rate_per_partition
        self.min_batches_to_retain = min_batches_to_retain
        
    def configure_backpressure(
        self,
        query: StreamingQuery
    ) -> StreamingQuery:
        """Configure backpressure mechanisms"""
        return query \
            .option("maxOffsetsPerTrigger", self.max_rate_per_partition) \
            .option("spark.sql.streaming.minBatchesToRetain",
                   self.min_batches_to_retain) \
            .option("spark.sql.streaming.backpressure.enabled", "true")
```

### 2. Performance Monitoring

```python
class StreamingMonitor:
    """Monitors streaming application performance"""
    def __init__(self, query: StreamingQuery):
        self.query = query
        self.metrics = {}
        
    def collect_metrics(self) -> Dict[str, Any]:
        """Collect streaming metrics"""
        status = self.query.status
        
        self.metrics.update({
            "input_rate": status.inputRate,
            "processing_rate": status.processingRate,
            "batch_duration": status.batchDuration,
            "num_active_batches": len(status.activeBatches),
            "num_completed_batches": len(status.completedBatches),
            "num_retained_completed_batches": len(status.retainedCompletedBatches),
            "memory_used": status.memoryUsed,
            "num_total_rows": status.numTotalRows
        })
        
        return self.metrics
        
    def check_health(self) -> bool:
        """Check streaming application health"""
        metrics = self.collect_metrics()
        
        # Define health checks
        is_healthy = (
            metrics["processing_rate"] > 0.7 * metrics["input_rate"] and
            metrics["num_active_batches"] < 5 and
            metrics["memory_used"] < 0.9 * self.query.sparkSession.conf.get(
                "spark.memory.fraction"
            )
        )
        
        return is_healthy
```

## Practical Exercise: Advanced Analytics Pipeline

Build a sophisticated real-time analytics pipeline with multiple processing stages:

```python
def create_advanced_analytics_pipeline():
    """Create advanced streaming analytics pipeline"""
    # Initialize components
    context = SparkStreamingContext("AdvancedAnalytics")
    kafka_connector = SecureKafkaConnector("kafka:9092")
    error_handler = StreamingErrorHandler("/checkpoint/analytics", "dead_letter")
    state_manager = StateManager("/checkpoint/state")
    monitor = None
    
    try:
        # Define schema
        schema = StructType([
            StructField("event_id", StringType(), True),
            StructField("timestamp", TimestampType(), True),
            StructField("user_id", StringType(), True),
            StructField("event_type", StringType(), True),
            StructField("data", MapType(StringType(), StringType()), True)
        ])
        
        # Create streaming DataFrame
        stream_df = kafka_connector.create_source(
            context.spark,
            ["input_topic"],
            credentials={
                "username": "admin",
                "password": "secret"
            }
        )
        
        # Process stream
        processed_df = stream_df \
            .select(from_json(col("value").cast("string"), schema).alias("data")) \
            .select("data.*") \
            .transform(lambda df: error_handler.handle_corrupt_data(df)) \
            .transform(lambda df: state_manager.manage_state_cleanup(
                df, "timestamp", "24 hours"
            ))
            
        # Apply windowing
        windowed_df = WindowOperations(
            "5 minutes",
            "1 minute",
            "10 minutes"
        ).apply_sliding_window(
            processed_df,
            "timestamp",
            ["event_type", "user_id"]
        ).agg(
            count("*").alias("event_count"),
            collect_list("data").alias("events")
        )
        
        # Write to multiple sinks
        query = kafka_connector.create_sink(
            windowed_df,
            "output_topic",
            "/checkpoint/output"
        )
        
        # Setup monitoring
        monitor = StreamingMonitor(query)
        
        # Start monitoring loop
        while query.isActive:
            if not monitor.check_health():
                logging.warning("Unhealthy streaming state detected")
            time.sleep(60)
            
    except Exception as e:
        logging.error(f"Pipeline error: {str(e)}")
        raise
    finally:
        if monitor:
            final_metrics = monitor.collect_metrics()
            logging.info(f"Final metrics: {final_metrics}")

if __name__ == "__main__":
    create_advanced_analytics_pipeline()
```

## Best Practices
1. **State Management**
   - Use checkpointing for fault tolerance
   - Implement proper state cleanup
   - Monitor state store performance
   - Handle state store failures

2. **Performance Optimization**
   - Configure appropriate batch sizes
   - Enable backpressure handling
   - Monitor processing rates
   - Optimize resource usage

3. **Error Handling**
   - Implement comprehensive error handling
   - Use dead letter queues
   - Monitor error rates
   - Implement retry mechanisms

4. **Monitoring and Debugging**
   - Track key metrics
   - Set up alerting
   - Implement proper logging
   - Use debugging tools

## Additional Resources
- [Apache Spark Structured Streaming Documentation](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html)
- [Spark Streaming + Kafka Integration Guide](https://spark.apache.org/docs/latest/structured-streaming-kafka-integration.html)
- [Performance Tuning Guide](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html#performance-tuning)
- [Monitoring and Debugging Guide](https://spark.apache.org/docs/latest/monitoring.html)

## Next Steps
- Explore advanced streaming patterns
- Study monitoring and debugging techniques
- Practice with real-world scenarios
- Learn about streaming security patterns 