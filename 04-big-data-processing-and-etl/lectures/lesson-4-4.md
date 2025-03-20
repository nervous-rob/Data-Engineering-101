# Lesson 4.4: Data Quality and Testing

## Navigation
- [← Back to Lesson Plan](../4.4-data-quality-testing.md)
- [← Back to Module Overview](../README.md)

## Overview
Data quality and testing are fundamental aspects of building reliable data pipelines. This lesson explores comprehensive approaches to ensuring data quality through automated testing, validation frameworks, and monitoring systems. We'll examine various tools and frameworks that help maintain data integrity throughout the ETL process, implement robust testing strategies, and establish quality gates that ensure data reliability. By mastering data quality practices, data engineers can deliver trustworthy data that meets business requirements while maintaining system reliability.

## Learning Objectives
- Implement comprehensive data quality checks and testing frameworks
- Master testing strategies for data pipelines and quality gates
- Learn about modern data validation tools and frameworks
- Understand monitoring and alerting systems for quality metrics
- Design automated quality control processes
- Implement statistical quality measures and thresholds

## Data Quality Fundamentals

Data quality is a multi-dimensional concept that requires a systematic approach to measurement and control:

### 1. Data Completeness
- Ensuring all required data is present
- Identifying and handling missing values
- Validating record counts and data coverage
- Checking for data gaps in time series
- Monitoring completeness trends over time

```python
def check_completeness(df, required_columns):
    """Check completeness of required columns."""
    completeness_metrics = {}
    total_records = df.count()
    
    for column in required_columns:
        non_null_count = df.filter(col(column).isNotNull()).count()
        completeness_ratio = non_null_count / total_records
        completeness_metrics[column] = {
            'total_records': total_records,
            'non_null_records': non_null_count,
            'completeness_ratio': completeness_ratio,
            'status': 'PASS' if completeness_ratio >= 0.95 else 'FAIL'
        }
    
    return completeness_metrics
```

### 2. Data Accuracy
- Verifying data correctness against known sources
- Validating calculations and transformations
- Checking for outliers and anomalies
- Ensuring precision in numerical values
- Statistical validation methods

```python
def validate_accuracy(df, validation_rules):
    """Validate data accuracy using statistical and rule-based methods."""
    from pyspark.sql.functions import stddev, mean, min, max
    
    accuracy_results = {}
    
    for rule in validation_rules:
        if rule['type'] == 'range_check':
            column = rule['column']
            stats = df.select(
                mean(column).alias('mean'),
                stddev(column).alias('stddev'),
                min(column).alias('min'),
                max(column).alias('max')
            ).collect()[0]
            
            # Check for outliers (3 sigma rule)
            outliers = df.filter(
                (col(column) > stats['mean'] + 3 * stats['stddev']) |
                (col(column) < stats['mean'] - 3 * stats['stddev'])
            ).count()
            
            accuracy_results[column] = {
                'stats': stats,
                'outliers_count': outliers,
                'status': 'FAIL' if outliers > rule['outlier_threshold'] else 'PASS'
            }
            
    return accuracy_results
```

### 3. Data Consistency
- Maintaining uniform data representation
- Ensuring referential integrity
- Validating cross-system data synchronization
- Checking for duplicate records
- Temporal consistency validation

```python
def check_consistency(df, consistency_rules):
    """Check data consistency across multiple dimensions."""
    consistency_results = {}
    
    for rule in consistency_rules:
        if rule['type'] == 'referential_integrity':
            source_df = df
            target_df = rule['target_df']
            join_keys = rule['join_keys']
            
            # Check for orphaned records
            orphaned = source_df.join(
                target_df,
                join_keys,
                'left_anti'
            ).count()
            
            consistency_results[f"ref_integrity_{join_keys}"] = {
                'orphaned_records': orphaned,
                'status': 'FAIL' if orphaned > 0 else 'PASS'
            }
            
        elif rule['type'] == 'temporal_consistency':
            # Check for temporal sequence validity
            temporal_issues = validate_temporal_sequence(
                df,
                rule['timestamp_column'],
                rule['sequence_key']
            )
            
            consistency_results[f"temporal_{rule['sequence_key']}"] = temporal_issues
            
    return consistency_results

def validate_temporal_sequence(df, timestamp_col, sequence_key):
    """Validate temporal sequence of records."""
    from pyspark.sql.window import Window
    from pyspark.sql.functions import lag
    
    window_spec = Window.partitionBy(sequence_key).orderBy(timestamp_col)
    
    # Check for gaps and overlaps
    sequence_issues = df.withColumn(
        'prev_timestamp',
        lag(timestamp_col).over(window_spec)
    ).filter(
        (col('prev_timestamp').isNotNull()) &
        (col(timestamp_col) <= col('prev_timestamp'))
    ).count()
    
    return {
        'sequence_issues': sequence_issues,
        'status': 'FAIL' if sequence_issues > 0 else 'PASS'
    }
```

### 4. Data Timeliness
- Monitoring data freshness
- Validating timestamp accuracy
- Checking processing delays
- Ensuring SLA compliance
- Latency tracking and alerting

```python
def monitor_timeliness(df, timeliness_config):
    """Monitor data timeliness and freshness."""
    from datetime import datetime, timedelta
    
    timeliness_metrics = {}
    current_time = datetime.now()
    
    # Check data freshness
    max_timestamp = df.select(max(timeliness_config['timestamp_column'])).collect()[0][0]
    freshness_delay = current_time - max_timestamp
    
    timeliness_metrics['freshness'] = {
        'max_timestamp': max_timestamp,
        'delay_seconds': freshness_delay.total_seconds(),
        'status': 'FAIL' if freshness_delay > timedelta(minutes=timeliness_config['sla_minutes']) else 'PASS'
    }
    
    # Check processing latency
    if 'processing_start_time' in df.columns:
        avg_latency = df.select(
            mean(
                unix_timestamp(timeliness_config['timestamp_column']) -
                unix_timestamp('processing_start_time')
            )
        ).collect()[0][0]
        
        timeliness_metrics['processing_latency'] = {
            'avg_latency_seconds': avg_latency,
            'status': 'FAIL' if avg_latency > timeliness_config['max_latency_seconds'] else 'PASS'
        }
    
    return timeliness_metrics
```

### 5. Data Validity
- Enforcing data type constraints
- Validating format specifications
- Checking value ranges
- Ensuring business rule compliance
- Complex validation rules

```python
def validate_data_rules(df, validation_rules):
    """Validate data against complex business rules."""
    validation_results = {}
    
    for rule in validation_rules:
        if rule['type'] == 'format_check':
            # Validate format using regex patterns
            invalid_format = df.filter(
                ~col(rule['column']).rlike(rule['pattern'])
            ).count()
            
            validation_results[f"format_{rule['column']}"] = {
                'invalid_count': invalid_format,
                'status': 'FAIL' if invalid_format > 0 else 'PASS'
            }
            
        elif rule['type'] == 'business_rule':
            # Validate complex business rules
            rule_violations = df.filter(
                expr(rule['condition'])
            ).count()
            
            validation_results[f"rule_{rule['name']}"] = {
                'violations': rule_violations,
                'status': 'FAIL' if rule_violations > rule['threshold'] else 'PASS'
            }
    
    return validation_results
```

## Advanced Testing Strategies

### Unit Testing for Data Quality

```python
import pytest
from pyspark.sql import SparkSession
from pyspark.sql.types import StructType, StructField, StringType, IntegerType

class TestDataQuality:
    @pytest.fixture(scope="session")
    def spark(self):
        return SparkSession.builder \
            .appName("DataQualityTests") \
            .master("local[*]") \
            .getOrCreate()
    
    @pytest.fixture
    def sample_data(self, spark):
        schema = StructType([
            StructField("id", StringType(), False),
            StructField("value", IntegerType(), True),
            StructField("category", StringType(), True)
        ])
        
        data = [
            ("1", 100, "A"),
            ("2", None, "B"),
            ("3", 300, None)
        ]
        
        return spark.createDataFrame(data, schema)
    
    def test_completeness(self, sample_data):
        completeness_results = check_completeness(
            sample_data,
            ['id', 'value', 'category']
        )
        
        assert completeness_results['id']['completeness_ratio'] == 1.0
        assert completeness_results['value']['completeness_ratio'] < 1.0
        
    def test_value_ranges(self, sample_data):
        validation_rules = [{
            'type': 'range_check',
            'column': 'value',
            'outlier_threshold': 0
        }]
        
        accuracy_results = validate_accuracy(sample_data, validation_rules)
        assert accuracy_results['value']['status'] == 'PASS'
```

### Integration Testing

```python
class TestEndToEndQuality:
    @pytest.fixture(scope="session")
    def pipeline_config(self):
        return {
            'source_table': 'customers',
            'target_table': 'customer_metrics',
            'quality_rules': [
                {
                    'type': 'completeness',
                    'columns': ['customer_id', 'email'],
                    'threshold': 0.99
                },
                {
                    'type': 'format_check',
                    'column': 'email',
                    'pattern': '^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$'
                }
            ]
        }
    
    def test_pipeline_quality(self, spark, pipeline_config):
        # Execute pipeline
        pipeline_result = execute_pipeline(spark, pipeline_config)
        
        # Validate results
        quality_results = validate_pipeline_output(
            spark,
            pipeline_result,
            pipeline_config['quality_rules']
        )
        
        assert all(result['status'] == 'PASS' for result in quality_results.values())
```

## Modern Data Quality Frameworks

### Great Expectations Implementation

```python
from great_expectations.dataset import SparkDFDataset
from great_expectations.core import ExpectationConfiguration

class DataQualityValidator:
    def __init__(self, spark_df):
        self.ge_df = SparkDFDataset(spark_df)
        
    def validate_dataset(self):
        """Run comprehensive data validation."""
        results = []
        
        # Column value validations
        results.extend([
            self.ge_df.expect_column_values_to_not_be_null("customer_id"),
            self.ge_df.expect_column_values_to_be_unique("email"),
            self.ge_df.expect_column_values_to_match_regex(
                "email",
                "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$"
            )
        ])
        
        # Statistical validations
        results.extend([
            self.ge_df.expect_column_mean_to_be_between(
                "age",
                min_value=0,
                max_value=120
            ),
            self.ge_df.expect_column_quantile_values_to_be_between(
                "transaction_amount",
                quantile_ranges={
                    "quantiles": [0.05, 0.25, 0.5, 0.75, 0.95],
                    "value_ranges": [
                        [0, 1000],
                        [10, 5000],
                        [50, 10000],
                        [100, 50000],
                        [500, 100000]
                    ]
                }
            )
        ])
        
        return self.compile_validation_results(results)
    
    def compile_validation_results(self, results):
        """Compile validation results into a summary."""
        summary = {
            'success': all(result.success for result in results),
            'results': [
                {
                    'expectation_type': result.expectation_config.expectation_type,
                    'success': result.success,
                    'result': result.result
                }
                for result in results
            ]
        }
        
        return summary
```

### Soda Core Integration

```python
from soda.scan import Scan

def run_soda_checks(connection, checks_path):
    """Execute Soda Core checks."""
    scan = Scan()
    scan.add_configuration_yaml_file(connection)
    scan.add_sodacl_yaml_file(checks_path)
    
    scan_result = scan.execute()
    return {
        'passed': scan_result.passed(),
        'failed': scan_result.failed(),
        'errors': scan_result.errors(),
        'warnings': scan_result.warnings()
    }

# Example Soda checks configuration
"""
checks for customers:
  - row_count > 0
  - duplicate_count(email) = 0
  - invalid_count(email) = 0
  - invalid_percent(age) < 1
  - avg(age) between 18 and 90
  
metrics:
  - row_count
  - missing_count
  - invalid_count
  - duplicate_count
"""
```

## Quality Monitoring and Alerting

### Real-time Quality Monitoring

```python
class QualityMonitor:
    def __init__(self, spark, config):
        self.spark = spark
        self.config = config
        self.metrics_store = MetricsStore()
    
    def monitor_batch_quality(self, batch_df, batch_id):
        """Monitor quality metrics for a batch."""
        metrics = {}
        
        # Calculate quality metrics
        metrics['completeness'] = self.check_completeness(batch_df)
        metrics['accuracy'] = self.check_accuracy(batch_df)
        metrics['freshness'] = self.check_freshness(batch_df)
        
        # Store metrics
        self.metrics_store.store_batch_metrics(batch_id, metrics)
        
        # Check for alerts
        self.check_alerts(metrics)
        
        return metrics
    
    def check_alerts(self, metrics):
        """Check metrics against alert thresholds."""
        alerts = []
        
        for metric_name, metric_value in metrics.items():
            threshold = self.config['thresholds'].get(metric_name)
            if threshold and metric_value < threshold:
                alerts.append({
                    'metric': metric_name,
                    'value': metric_value,
                    'threshold': threshold,
                    'timestamp': datetime.now()
                })
        
        if alerts:
            self.send_alerts(alerts)
    
    def send_alerts(self, alerts):
        """Send alerts through configured channels."""
        for alert in alerts:
            # Email alerts
            if 'email' in self.config['alert_channels']:
                send_email_alert(
                    self.config['email_recipients'],
                    f"Quality Alert: {alert['metric']}",
                    f"Metric {alert['metric']} below threshold: {alert['value']} < {alert['threshold']}"
                )
            
            # Slack alerts
            if 'slack' in self.config['alert_channels']:
                send_slack_alert(
                    self.config['slack_webhook'],
                    {
                        'text': f"Quality Alert: {alert['metric']}",
                        'attachments': [{
                            'fields': [
                                {'title': 'Metric', 'value': alert['metric']},
                                {'title': 'Value', 'value': str(alert['value'])},
                                {'title': 'Threshold', 'value': str(alert['threshold'])}
                            ]
                        }]
                    }
                )
```

### Historical Quality Tracking

```python
class QualityMetricsTracker:
    def __init__(self, spark):
        self.spark = spark
        
    def track_historical_metrics(self, table_name, metric_type):
        """Track quality metrics over time."""
        return self.spark.sql(f"""
            SELECT
                date_trunc('day', timestamp) as metric_date,
                metric_type,
                AVG(metric_value) as avg_value,
                MIN(metric_value) as min_value,
                MAX(metric_value) as max_value,
                STDDEV(metric_value) as std_dev
            FROM quality_metrics
            WHERE table_name = '{table_name}'
                AND metric_type = '{metric_type}'
            GROUP BY date_trunc('day', timestamp), metric_type
            ORDER BY metric_date DESC
        """)
    
    def detect_anomalies(self, metrics_df, window_size=30):
        """Detect anomalies in quality metrics."""
        from pyspark.sql.functions import avg, stddev
        from pyspark.sql.window import Window
        
        # Calculate rolling statistics
        window_spec = Window.orderBy('metric_date').rowsBetween(-window_size, 0)
        
        return metrics_df.withColumn(
            'rolling_avg',
            avg('avg_value').over(window_spec)
        ).withColumn(
            'rolling_stddev',
            stddev('avg_value').over(window_spec)
        ).withColumn(
            'is_anomaly',
            abs(col('avg_value') - col('rolling_avg')) > 
            (3 * col('rolling_stddev'))
        )
```

## Quality Gates and Deployment Integration

### CI/CD Integration

```python
def quality_gate_check(spark, config):
    """Quality gate check for CI/CD pipeline."""
    try:
        # Run data quality checks
        quality_results = run_quality_checks(spark, config)
        
        # Check if all critical checks pass
        critical_checks = [
            result for result in quality_results
            if result['severity'] == 'critical'
        ]
        
        if any(check['status'] == 'FAIL' for check in critical_checks):
            raise QualityGateException(
                "Critical quality checks failed. Deployment blocked."
            )
        
        # Check overall quality score
        quality_score = calculate_quality_score(quality_results)
        if quality_score < config['minimum_quality_score']:
            raise QualityGateException(
                f"Quality score {quality_score} below minimum threshold "
                f"{config['minimum_quality_score']}"
            )
        
        return True
    
    except Exception as e:
        logging.error(f"Quality gate check failed: {str(e)}")
        raise

class QualityGateException(Exception):
    """Exception raised when quality gate checks fail."""
    pass
```

### Automated Remediation

```python
class QualityRemediation:
    def __init__(self, spark, config):
        self.spark = spark
        self.config = config
    
    def auto_remediate(self, quality_issues):
        """Attempt to automatically remediate quality issues."""
        remediation_results = []
        
        for issue in quality_issues:
            if issue['type'] in self.config['auto_remediation_rules']:
                rule = self.config['auto_remediation_rules'][issue['type']]
                
                try:
                    if rule['action'] == 'fill_nulls':
                        self.fill_null_values(
                            issue['table'],
                            issue['column'],
                            rule['fill_value']
                        )
                    elif rule['action'] == 'remove_duplicates':
                        self.remove_duplicate_records(
                            issue['table'],
                            rule['key_columns']
                        )
                    elif rule['action'] == 'standardize_format':
                        self.standardize_column_format(
                            issue['table'],
                            issue['column'],
                            rule['format_pattern']
                        )
                    
                    remediation_results.append({
                        'issue': issue,
                        'status': 'SUCCESS',
                        'action_taken': rule['action']
                    })
                    
                except Exception as e:
                    remediation_results.append({
                        'issue': issue,
                        'status': 'FAILED',
                        'error': str(e)
                    })
            else:
                remediation_results.append({
                    'issue': issue,
                    'status': 'SKIPPED',
                    'reason': 'No remediation rule configured'
                })
        
        return remediation_results
```

## Best Practices for Data Quality

### 1. Implement Comprehensive Testing
- Write tests for all critical data transformations
- Include edge cases and boundary conditions
- Test both positive and negative scenarios
- Automate test execution
- Maintain test data sets

### 2. Establish Quality Gates
- Define clear quality criteria
- Set appropriate thresholds
- Implement blocking vs. non-blocking checks
- Monitor false positives/negatives
- Regular threshold reviews

### 3. Monitor Quality Metrics
- Track trends over time
- Set up real-time monitoring
- Implement proactive alerting
- Maintain quality dashboards
- Regular quality reviews

### 4. Document Quality Standards
- Define quality requirements
- Document validation rules
- Maintain quality metrics
- Track quality improvements
- Share quality reports

### 5. Automate Quality Processes
- Implement automated checks
- Set up continuous monitoring
- Automate remediation where possible
- Regular process reviews
- Continuous improvement

## Common Pitfalls and Anti-Patterns

### 1. Insufficient Quality Checks
- Missing critical validations
- Inadequate test coverage
- Poor error handling
- Insufficient monitoring

Solution:
- Implement comprehensive validation framework
- Regular coverage reviews
- Automated test generation
- Continuous monitoring

### 2. Poor Quality Gates
- Inappropriate thresholds
- Missing critical checks
- Insufficient blocking criteria
- Poor documentation

Solution:
- Regular threshold reviews
- Comprehensive gate criteria
- Clear documentation
- Regular effectiveness reviews

### 3. Inadequate Monitoring
- Missing key metrics
- Poor alerting
- Insufficient tracking
- Late detection

Solution:
- Comprehensive monitoring
- Proactive alerting
- Regular metric reviews
- Trend analysis

### 4. Manual Quality Processes
- Manual validations
- Ad-hoc checking
- Inconsistent processes
- Poor documentation

Solution:
- Process automation
- Standard procedures
- Clear documentation
- Regular reviews

## Conclusion

Data quality and testing are critical components of successful data engineering projects. By implementing comprehensive testing strategies, quality monitoring, and automated processes, organizations can ensure reliable and trustworthy data. The combination of modern tools, frameworks, and best practices enables data engineers to build robust quality assurance systems that scale with growing data volumes and complexity.

## Additional Resources

1. **Documentation and Guides**
   - [Great Expectations Documentation](https://docs.greatexpectations.io/)
   - [dbt Testing Guide](https://docs.getdbt.com/docs/building-a-dbt-project/tests)
   - [Soda Core Documentation](https://docs.soda.io/soda-core/overview.html)
   - [Apache Spark Testing Best Practices](https://spark.apache.org/docs/latest/testing.html)

2. **Tools and Frameworks**
   - Great Expectations
   - dbt
   - Soda Core
   - Monte Carlo
   - Apache Spark Testing Utilities

3. **Further Reading**
   - "Testing Data Pipelines" by Holger Krekel
   - "Data Quality: The Field Guide" by Thomas C. Redman
   - "The Data Quality Handbook" by Michelle Knight
   - "Building Reliable Data Pipelines" by James Densmore 