# Lesson 5.5: Real-Time Analytics

## Navigation
- [← Back to Lesson Plan](../5.5-real-time-analytics.md)
- [← Back to Module Overview](../README.md)

## Overview
Real-time analytics involves processing data as it's generated and delivering insights for immediate decision-making. This lesson explores the architectures, patterns, and implementations for building robust real-time analytics systems that can handle high-throughput data processing while maintaining low latency.

## Learning Objectives
After completing this lesson, you'll be able to:
- Design and implement real-time analytics architectures
- Build scalable streaming analytics pipelines
- Implement time-series processing and analysis
- Create real-time visualization and monitoring systems
- Handle complex event processing and pattern detection
- Optimize analytics performance and reliability
- Deploy production-ready analytics solutions

## Real-Time Analytics Architecture

### 1. Core Components

```python
from typing import Dict, List, Optional, Any
from datetime import datetime, timedelta
import logging
import json

class AnalyticsArchitecture:
    """Core analytics architecture components"""
    def __init__(
        self,
        streaming_platform: str,
        analytics_db: str,
        retention_period: timedelta = timedelta(days=7)
    ):
        self.streaming_platform = streaming_platform
        self.analytics_db = analytics_db
        self.retention_period = retention_period
        self.logger = logging.getLogger(__name__)
        
    def configure_ingestion(
        self,
        source_config: Dict[str, Any]
    ) -> Dict[str, Any]:
        """Configure data ingestion layer"""
        config = {
            "source": {
                "type": source_config.get("type", "kafka"),
                "bootstrap.servers": source_config.get("servers"),
                "topics": source_config.get("topics", []),
                "group.id": "analytics-consumer",
                "auto.offset.reset": "latest",
                "enable.auto.commit": False
            },
            "processing": {
                "window_size": "5 minutes",
                "watermark_delay": "10 minutes",
                "checkpoint_location": "/checkpoints/analytics",
                "max_offsets_per_trigger": 10000
            },
            "sink": {
                "type": self.analytics_db,
                "write_mode": "append",
                "batch_size": 1000,
                "retry_attempts": 3
            }
        }
        return config
        
    def setup_monitoring(
        self,
        metrics_config: Dict[str, Any]
    ) -> None:
        """Setup analytics monitoring"""
        self.metrics = {
            "latency": {
                "threshold": metrics_config.get("latency_threshold_ms", 1000),
                "window": metrics_config.get("window_size", "1 minute")
            },
            "throughput": {
                "min_rate": metrics_config.get("min_rate", 1000),
                "max_rate": metrics_config.get("max_rate", 10000)
            },
            "error_rate": {
                "max_percentage": metrics_config.get("max_error_rate", 0.1)
            }
        }
```

### 2. Time Series Processing

```python
class TimeSeriesProcessor:
    """Handles time series data processing and analysis"""
    def __init__(
        self,
        window_size: str = "5 minutes",
        slide_duration: str = "1 minute"
    ):
        self.window_size = window_size
        self.slide_duration = slide_duration
        self.metrics_store = {}
        
    def process_window(
        self,
        df: DataFrame,
        timestamp_col: str,
        value_col: str,
        grouping_cols: List[str]
    ) -> DataFrame:
        """Process time series data with windowing"""
        return df \
            .withWatermark(timestamp_col, "10 minutes") \
            .groupBy(
                window(timestamp_col, self.window_size, self.slide_duration),
                *grouping_cols
            ) \
            .agg(
                avg(value_col).alias("avg_value"),
                stddev(value_col).alias("stddev_value"),
                min(value_col).alias("min_value"),
                max(value_col).alias("max_value"),
                count("*").alias("count")
            )
            
    def detect_anomalies(
        self,
        df: DataFrame,
        value_col: str,
        threshold: float = 2.0
    ) -> DataFrame:
        """Detect anomalies using statistical methods"""
        # Calculate rolling statistics
        window_spec = Window \
            .orderBy("timestamp") \
            .rowsBetween(-10, 0)
            
        return df \
            .withColumn("rolling_avg",
                       avg(value_col).over(window_spec)) \
            .withColumn("rolling_stddev",
                       stddev(value_col).over(window_spec)) \
            .withColumn("zscore",
                       (col(value_col) - col("rolling_avg")) /
                       col("rolling_stddev")) \
            .withColumn("is_anomaly",
                       abs(col("zscore")) > threshold)
                       
    def forecast_values(
        self,
        df: DataFrame,
        timestamp_col: str,
        value_col: str,
        horizon: int = 10
    ) -> DataFrame:
        """Implement time series forecasting"""
        # Apply exponential smoothing
        window_spec = Window \
            .orderBy(timestamp_col) \
            .rowsBetween(Window.unboundedPreceding, 0)
            
        alpha = lit(0.3)  # Smoothing factor
        
        return df \
            .withColumn("level",
                       when(row_number().over(window_spec) == 1,
                            col(value_col))
                       .otherwise(
                           alpha * col(value_col) +
                           (1 - alpha) * lag("level", 1)
                           .over(window_spec))) \
            .withColumn("trend",
                       when(row_number().over(window_spec) <= 2,
                            lit(0.0))
                       .otherwise(
                           alpha * (col("level") -
                                  lag("level", 1).over(window_spec)) +
                           (1 - alpha) * lag("trend", 1)
                           .over(window_spec))) \
            .withColumn("forecast",
                       col("level") + col("trend"))
```

### 3. Real-Time Pattern Detection

```python
class PatternDetector:
    """Implements real-time pattern detection"""
    def __init__(
        self,
        pattern_configs: Dict[str, Any]
    ):
        self.patterns = pattern_configs
        self.state = {}
        
    def detect_sequence_pattern(
        self,
        df: DataFrame,
        event_col: str,
        timestamp_col: str,
        window_duration: str = "5 minutes"
    ) -> DataFrame:
        """Detect sequence patterns in event streams"""
        # Create pattern matching expressions
        pattern_expr = self.patterns.get("sequence", [])
        
        return df \
            .withWatermark(timestamp_col, "10 minutes") \
            .groupBy(
                window(timestamp_col, window_duration)
            ) \
            .agg(
                collect_list(
                    struct(event_col, timestamp_col)
                ).alias("events")
            ) \
            .select(
                "window",
                expr(f"sequence_match({pattern_expr}, events)").alias("matches")
            )
            
    def detect_threshold_patterns(
        self,
        df: DataFrame,
        value_col: str,
        threshold_config: Dict[str, float]
    ) -> DataFrame:
        """Detect threshold-based patterns"""
        conditions = []
        
        for pattern, config in threshold_config.items():
            if "upper" in config:
                conditions.append(
                    when(col(value_col) > lit(config["upper"]),
                         lit(f"{pattern}_exceeded"))
                )
            if "lower" in config:
                conditions.append(
                    when(col(value_col) < lit(config["lower"]),
                         lit(f"{pattern}_below"))
                )
                
        return df.withColumn(
            "detected_patterns",
            array_remove(array(*conditions), null)
        )
```

### 4. Analytics Pipeline

```python
class AnalyticsPipeline:
    """Implements end-to-end analytics pipeline"""
    def __init__(
        self,
        spark_session: SparkSession,
        config: Dict[str, Any]
    ):
        self.spark = spark_session
        self.config = config
        self.processor = TimeSeriesProcessor(
            config.get("window_size", "5 minutes"),
            config.get("slide_duration", "1 minute")
        )
        self.pattern_detector = PatternDetector(
            config.get("patterns", {})
        )
        
    def create_pipeline(self) -> StreamingQuery:
        """Create and configure analytics pipeline"""
        # Read streaming data
        df = self.spark.readStream \
            .format(self.config["source"]["type"]) \
            .options(**self.config["source"]) \
            .load()
            
        # Process time series
        processed_df = self.processor.process_window(
            df,
            self.config["timestamp_column"],
            self.config["value_column"],
            self.config.get("grouping_columns", [])
        )
        
        # Detect anomalies
        anomalies_df = self.processor.detect_anomalies(
            processed_df,
            "avg_value",
            self.config.get("anomaly_threshold", 2.0)
        )
        
        # Detect patterns
        patterns_df = self.pattern_detector.detect_threshold_patterns(
            anomalies_df,
            "avg_value",
            self.config.get("threshold_patterns", {})
        )
        
        # Write results
        return patterns_df.writeStream \
            .format(self.config["sink"]["type"]) \
            .outputMode("append") \
            .options(**self.config["sink"]) \
            .start()
```

## Real-Time Visualization

### 1. Dashboard Implementation

```python
class AnalyticsDashboard:
    """Implements real-time analytics dashboard"""
    def __init__(
        self,
        update_interval: int = 1000,
        max_points: int = 1000
    ):
        self.update_interval = update_interval
        self.max_points = max_points
        self.metrics = {}
        
    def update_metrics(
        self,
        new_data: Dict[str, Any]
    ) -> None:
        """Update dashboard metrics"""
        timestamp = datetime.now()
        
        for metric, value in new_data.items():
            if metric not in self.metrics:
                self.metrics[metric] = []
                
            self.metrics[metric].append({
                "timestamp": timestamp,
                "value": value
            })
            
            # Maintain sliding window
            if len(self.metrics[metric]) > self.max_points:
                self.metrics[metric].pop(0)
                
    def generate_visualization(
        self,
        metric_name: str,
        chart_type: str = "line"
    ) -> Dict[str, Any]:
        """Generate visualization for metric"""
        data = self.metrics.get(metric_name, [])
        
        return {
            "type": chart_type,
            "data": {
                "timestamps": [point["timestamp"] for point in data],
                "values": [point["value"] for point in data]
            },
            "config": {
                "title": f"{metric_name} Over Time",
                "x_axis": "Time",
                "y_axis": metric_name
            }
        }
```

## Practical Exercise: Building a Real-Time Analytics System

Build a complete real-time analytics system that processes streaming data, detects patterns, and visualizes results:

```python
def create_analytics_system():
    """Create end-to-end analytics system"""
    # Configuration
    config = {
        "source": {
            "type": "kafka",
            "servers": "localhost:9092",
            "topics": ["events"],
            "schema": {
                "timestamp": "timestamp",
                "metric": "string",
                "value": "double"
            }
        },
        "processing": {
            "window_size": "5 minutes",
            "slide_duration": "1 minute",
            "anomaly_threshold": 2.0,
            "patterns": {
                "sequence": ["A -> B -> C"],
                "threshold": {
                    "critical": {"upper": 100.0},
                    "warning": {"upper": 80.0, "lower": 20.0}
                }
            }
        },
        "sink": {
            "type": "memory",
            "queryName": "analytics_results"
        }
    }
    
    try:
        # Initialize components
        spark = SparkSession.builder \
            .appName("RealTimeAnalytics") \
            .getOrCreate()
            
        architecture = AnalyticsArchitecture(
            "kafka",
            "memory",
            timedelta(days=7)
        )
        
        pipeline = AnalyticsPipeline(
            spark,
            config
        )
        
        dashboard = AnalyticsDashboard(
            update_interval=1000,
            max_points=1000
        )
        
        # Start pipeline
        query = pipeline.create_pipeline()
        
        # Update dashboard
        def process_batch(df, epoch_id):
            results = df.collect()
            metrics = {
                "avg_value": results[0]["avg_value"],
                "anomaly_count": len([r for r in results if r["is_anomaly"]]),
                "pattern_count": len([r for r in results if r["detected_patterns"]])
            }
            dashboard.update_metrics(metrics)
            
        # Attach dashboard update
        query.foreachBatch(process_batch)
        
        # Monitor system health
        while query.isActive:
            metrics = query.lastProgress
            
            if metrics:
                logging.info(f"Input rate: {metrics['inputRate']}")
                logging.info(f"Processing rate: {metrics['processedRowsPerSecond']}")
                
            time.sleep(10)
            
    except Exception as e:
        logging.error(f"Analytics system error: {str(e)}")
        raise
    finally:
        if query:
            query.stop()

if __name__ == "__main__":
    create_analytics_system()
```

## Best Practices

1. **Architecture Design**
   - Use streaming platforms optimized for high throughput
   - Implement proper data retention policies
   - Design for scalability and fault tolerance
   - Choose appropriate analytics databases

2. **Data Processing**
   - Configure appropriate window sizes
   - Implement effective watermarking
   - Handle late-arriving data
   - Optimize aggregation operations

3. **Pattern Detection**
   - Use statistical methods for anomaly detection
   - Implement multiple detection algorithms
   - Handle false positives/negatives
   - Validate pattern detection accuracy

4. **Visualization**
   - Update dashboards efficiently
   - Implement proper data retention
   - Handle real-time updates smoothly
   - Optimize visualization performance

## Additional Resources
- [Real-Time Analytics Guide](https://www.redpanda.com/guides/fundamentals-of-data-engineering-real-time-data-analytics)
- [Architectural Patterns for Real-Time Analytics](https://aws.amazon.com/blogs/big-data/architectural-patterns-for-real-time-analytics-using-amazon-kinesis-data-streams-part-1/)
- [Stream Processing Documentation](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html)
- [Time Series Analysis Guide](https://spark.apache.org/docs/latest/ml-classification-regression.html#time-series-analysis)

## Next Steps
- Explore advanced analytics patterns
- Study visualization techniques
- Practice with real-world datasets
- Learn about analytics security 