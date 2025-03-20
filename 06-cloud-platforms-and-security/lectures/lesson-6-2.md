# Lesson 6.2: AWS Data Services

## Navigation
- [← Back to Lesson Plan](../6.2-aws-data-services.md)
- [← Back to Module Overview](../README.md)

## Overview
Welcome to our deep dive into AWS Data Services! In today's data-driven world, Amazon Web Services (AWS) provides one of the most comprehensive suites of data services available. Whether you're building data lakes, real-time processing pipelines, or sophisticated analytics solutions, understanding AWS's data ecosystem is crucial for modern data engineers.

This lesson will take you through the practical aspects of implementing AWS data solutions, with a focus on real-world scenarios and best practices. We'll explore not just the "what" but the "why" and "how" of AWS data services.

## Learning Objectives
After completing this lesson, you'll be able to:
- Design and implement AWS data lake architectures
- Build scalable data processing pipelines using AWS services
- Implement secure and cost-effective data solutions
- Master AWS analytics and database services
- Deploy real-time data processing solutions
- Optimize AWS data service configurations

## Core AWS Data Services Overview

Before we dive into implementations, let's understand the key services in AWS's data ecosystem:

### Storage Services
1. **Amazon S3 (Simple Storage Service)**
   - Object storage service with 99.999999999% durability
   - Cornerstone of most AWS data architectures
   - Supports multiple storage classes for cost optimization
   - Key features:
     * Versioning
     * Lifecycle management
     * Server-side encryption
     * Access control policies
   - Storage Classes:
     * Standard: Frequent access, lowest latency
     * Intelligent-Tiering: Unknown access patterns
     * Standard-IA: Infrequent access
     * One Zone-IA: Less critical data
     * Glacier: Long-term archival
     * Glacier Deep Archive: Lowest cost

2. **Amazon EFS (Elastic File System)**
   - Managed NFS file system
   - Automatically scales to petabytes
   - Use cases:
     * Shared file storage for data processing
     * Content management systems
     * Development environments

3. **Amazon FSx**
   - Managed file systems for:
     * Windows File Server
     * Lustre (high-performance computing)
   - Ideal for:
     * Windows-based analytics
     * Machine learning training data
     * High-performance workloads

### Database Services
1. **Amazon RDS (Relational Database Service)**
   - Managed relational databases
   - Supports multiple engines:
     * Amazon Aurora
     * PostgreSQL
     * MySQL
     * Oracle
     * SQL Server
   - Key features:
     * Automated backups
     * Point-in-time recovery
     * Read replicas
     * Multi-AZ deployments

2. **Amazon DynamoDB**
   - Fully managed NoSQL database
   - Single-digit millisecond performance
   - Ideal for:
     * High-throughput applications
     * Gaming leaderboards
     * Session management
     * Time-series data

3. **Amazon Redshift**
   - Enterprise-class data warehouse
   - Petabyte-scale analytics
   - Key features:
     * Columnar storage
     * Parallel query execution
     * Automatic compression
     * AQUA (Advanced Query Accelerator)

### Analytics Services
1. **Amazon EMR (Elastic MapReduce)**
   - Managed big data platform
   - Supports:
     * Apache Spark
     * Hive
     * Presto
     * HBase
   - Use cases:
     * Large-scale data processing
     * Machine learning
     * Real-time analytics

2. **Amazon Athena**
   - Serverless query service
   - SQL queries directly on S3 data
   - Pay only for data scanned
   - Perfect for:
     * Ad-hoc analysis
     * Cost-effective querying
     * Data exploration

3. **Amazon Kinesis**
   - Real-time data streaming
   - Components:
     * Kinesis Data Streams
     * Kinesis Data Firehose
     * Kinesis Data Analytics
     * Kinesis Video Streams

## Understanding Data Lake Architecture

Before we implement our data lake, let's understand its key components:

### Data Lake Layers
1. **Landing Zone (Raw)**
   - Purpose: Initial data ingestion
   - Characteristics:
     * Immutable storage
     * Original format preservation
     * Full history retention
   - Example data types:
     * JSON logs
     * CSV files
     * Images
     * Streaming data

2. **Processing Zone**
   - Purpose: Data transformation and enrichment
   - Characteristics:
     * Cleaned and validated data
     * Optimized formats
     * Intermediate processing results
   - Common operations:
     * Data cleansing
     * Format conversion
     * Schema evolution
     * Data validation

3. **Curated Zone**
   - Purpose: Business-ready datasets
   - Characteristics:
     * Optimized for analytics
     * Well-defined schemas
     * Quality-assured data
   - Use cases:
     * Business intelligence
     * Machine learning
     * Regulatory reporting

### Data Lake Directory Structure
```plaintext
s3://your-lake/
├── raw/
│   ├── source=web/
│   │   └── date=2024-03-19/
│   │       ├── logs/
│   │       └── events/
│   └── source=mobile/
│       └── date=2024-03-19/
│           ├── user-activity/
│           └── metrics/
├── processed/
│   ├── customer_profiles/
│   │   ├── region=us/
│   │   └── region=eu/
│   └── transactions/
│       ├── year=2024/
│       └── year=2023/
└── curated/
    ├── customer_360/
    │   ├── daily/
    │   └── monthly/
    └── analytics/
        ├── dashboards/
        └── reports/
```

## AWS Data Services Implementation

### 1. Data Lake Implementation

> 📝 **Note:** A data lake is not just a storage solution - it's an architectural pattern that enables you to store, process, and analyze vast amounts of structured and unstructured data.

Let's implement a comprehensive data lake solution:

```python
from typing import Dict, List, Optional, Any
import boto3
import logging
from datetime import datetime

class AWSDataLake:
    """Implements AWS data lake architecture"""
    def __init__(self, config: Dict[str, Any]):
        self.config = config
        self.logger = logging.getLogger(__name__)
        self._initialize_clients()
        
    def _initialize_clients(self) -> None:
        """Initialize AWS service clients"""
        try:
            self.s3 = boto3.client('s3')
            self.glue = boto3.client('glue')
            self.athena = boto3.client('athena')
            self.lakeformation = boto3.client('lakeformation')
            
        except Exception as e:
            self.logger.error(f"Failed to initialize AWS clients: {str(e)}")
            raise
            
    async def setup_data_lake(self, lake_config: Dict[str, Any]) -> None:
        """Set up complete data lake infrastructure"""
        try:
            # 1. Create S3 buckets for different layers
            await self._create_storage_layers(lake_config['storage'])
            
            # 2. Configure Lake Formation permissions
            await self._setup_lake_formation(lake_config['permissions'])
            
            # 3. Create Glue Catalog databases
            await self._create_catalog_databases(lake_config['databases'])
            
            # 4. Set up crawlers for data discovery
            await self._setup_crawlers(lake_config['crawlers'])
            
        except Exception as e:
            self.logger.error(f"Data lake setup failed: {str(e)}")
            raise
            
    async def _create_storage_layers(self, storage_config: Dict[str, Any]) -> None:
        """Create storage layers with appropriate configurations"""
        layers = {
            'raw': {'versioning': True, 'lifecycle_rules': True},
            'processed': {'versioning': True, 'lifecycle_rules': False},
            'curated': {'versioning': False, 'lifecycle_rules': False}
        }
        
        for layer, config in layers.items():
            bucket_name = f"{storage_config['prefix']}-{layer}"
            
            # Create bucket with encryption
            self.s3.create_bucket(
                Bucket=bucket_name,
                CreateBucketConfiguration={
                    'LocationConstraint': storage_config['region']
                }
            )
            
            # Enable encryption
            self.s3.put_bucket_encryption(
                Bucket=bucket_name,
                ServerSideEncryptionConfiguration={
                    'Rules': [{
                        'ApplyServerSideEncryptionByDefault': {
                            'SSEAlgorithm': 'AES256'
                        }
                    }]
                }
            )
            
            # Enable versioning if required
            if config['versioning']:
                self.s3.put_bucket_versioning(
                    Bucket=bucket_name,
                    VersioningConfiguration={'Status': 'Enabled'}
                )
                
            # Set lifecycle rules if required
            if config['lifecycle_rules']:
                self._configure_lifecycle_rules(bucket_name)
                
    def _configure_lifecycle_rules(self, bucket_name: str) -> None:
        """Configure S3 lifecycle rules"""
        self.s3.put_bucket_lifecycle_configuration(
            Bucket=bucket_name,
            LifecycleConfiguration={
                'Rules': [
                    {
                        'ID': 'TransitionToGlacier',
                        'Status': 'Enabled',
                        'Transitions': [
                            {
                                'Days': 90,
                                'StorageClass': 'GLACIER'
                            }
                        ]
                    }
                ]
            }
        )

> 💡 **Pro Tip:** When implementing lifecycle rules, consider your data access patterns carefully. Moving data to Glacier too early can lead to high retrieval costs.

### File Format Selection Guide

| Format | Pros | Cons | Best For | Compression Ratio |
|--------|------|------|-----------|------------------|
| Parquet | - Columnar storage<br>- Efficient compression<br>- Predicate pushdown | - Complex to edit<br>- Limited streaming support | - Analytics queries<br>- Data warehousing | 4-8x |
| Avro | - Schema evolution<br>- Rich data types<br>- Streaming friendly | - Row-based<br>- Less efficient for analytics | - Streaming data<br>- Schema changes | 2-3x |
| JSON | - Human-readable<br>- Flexible schema<br>- Universal support | - Space inefficient<br>- No compression | - Raw data<br>- API responses | 1x |
| ORC | - Highly compressed<br>- ACID support<br>- Predicate pushdown | - Limited tool support<br>- Complex format | - Hive workloads<br>- ACID requirements | 6-10x |

### 2. Real-time Data Processing Pipeline

Real-time data processing is crucial for modern data architectures. Here's how to implement a robust streaming pipeline:

```python
class AWSStreamProcessor:
    """Implements real-time data processing using AWS services"""
    def __init__(self, config: Dict[str, Any]):
        self.config = config
        self.logger = logging.getLogger(__name__)
        self._initialize_clients()
        
    def _initialize_clients(self) -> None:
        """Initialize streaming service clients"""
        self.kinesis = boto3.client('kinesis')
        self.firehose = boto3.client('firehose')
        self.analytics = boto3.client('kinesisanalytics')
        
    async def setup_streaming_pipeline(
        self,
        pipeline_config: Dict[str, Any]
    ) -> None:
        """Set up end-to-end streaming pipeline"""
        try:
            # 1. Create Kinesis Data Stream
            stream_arn = await self._create_data_stream(
                pipeline_config['stream']
            )
            
            # 2. Create Analytics Application
            app_arn = await self._create_analytics_app(
                pipeline_config['analytics'],
                stream_arn
            )
            
            # 3. Create Firehose Delivery Stream
            delivery_arn = await self._create_delivery_stream(
                pipeline_config['delivery'],
                app_arn
            )
            
            # 4. Configure Error Handling
            await self._setup_error_handling(
                stream_arn,
                app_arn,
                delivery_arn
            )
            
        except Exception as e:
            self.logger.error(f"Streaming pipeline setup failed: {str(e)}")
            raise
            
    async def _create_data_stream(
        self,
        stream_config: Dict[str, Any]
    ) -> str:
        """Create Kinesis Data Stream"""
        response = self.kinesis.create_stream(
            StreamName=stream_config['name'],
            ShardCount=stream_config['shard_count']
        )
        
        # Wait for stream to become active
        waiter = self.kinesis.get_waiter('stream_exists')
        waiter.wait(
            StreamName=stream_config['name'],
            WaiterConfig={'Delay': 10, 'MaxAttempts': 30}
        )
        
        return self.kinesis.describe_stream(
            StreamName=stream_config['name']
        )['StreamDescription']['StreamARN']
```

> 🔍 **Deep Dive:** Kinesis shard count is crucial for performance. Each shard supports up to 1MB/second input and 2MB/second output. Plan your sharding strategy based on your data volume and velocity.

### Kinesis Stream Patterns

1. **Fan-out Pattern**
```python
class KinesisFanOut:
    """Implements fan-out pattern for multiple consumers"""
    def __init__(self, stream_name: str):
        self.stream_name = stream_name
        self.kinesis = boto3.client('kinesis')
        
    async def register_consumer(
        self,
        consumer_name: str,
        consumer_config: Dict[str, Any]
    ) -> str:
        """Register new enhanced fan-out consumer"""
        response = self.kinesis.register_stream_consumer(
            StreamARN=self.stream_name,
            ConsumerName=consumer_name
        )
        return response['Consumer']['ConsumerARN']
```

2. **Aggregation Pattern**
```python
class KinesisAggregation:
    """Implements record aggregation for better throughput"""
    def __init__(self, max_size: int = 1000000):  # 1MB
        self.max_size = max_size
        self.buffer = []
        self.current_size = 0
        
    def add_record(self, record: Dict[str, Any]) -> bool:
        """Add record to buffer, return True if buffer is full"""
        record_size = len(json.dumps(record).encode())
        if self.current_size + record_size > self.max_size:
            return True
        self.buffer.append(record)
        self.current_size += record_size
        return False
```

[Previous content remains the same until Analytics Services section...]

## Architectural Considerations for Reliable Scalability

> 🏗️ **Architecture Principle:** "Build today with tomorrow in mind" - Your data services architecture should cater to both current scale requirements and anticipated growth.

### 1. Modularity in Data Architecture

Modern data architectures should break complex components into smaller, more manageable parts. For example:

```python
class ModularDataArchitecture:
    """Implements modular data architecture pattern"""
    def __init__(self):
        self.ingestion_layer = DataIngestionLayer()
        self.processing_layer = DataProcessingLayer()
        self.serving_layer = DataServingLayer()
        
    async def process_data_pipeline(self, data: Dict[str, Any]) -> None:
        """Process data through modular pipeline"""
        try:
            # 1. Ingest data through dedicated layer
            ingested_data = await self.ingestion_layer.ingest(data)
            
            # 2. Process in isolated processing layer
            processed_data = await self.processing_layer.process(ingested_data)
            
            # 3. Serve through optimized serving layer
            await self.serving_layer.serve(processed_data)
            
        except Exception as e:
            # Handle errors at appropriate layer
            await self._handle_layer_error(e)
```

### 2. Design for Failure

Following AWS's reliability principles:

```python
class ResilientDataPipeline:
    """Implements resilient data pipeline pattern"""
    def __init__(self, config: Dict[str, Any]):
        self.primary_region = config['primary_region']
        self.backup_region = config['backup_region']
        self.setup_multi_region_resources()
        
    def setup_multi_region_resources(self) -> None:
        """Set up resources across regions for resilience"""
        # Primary region setup
        self.primary_kinesis = boto3.client('kinesis', 
                                          region_name=self.primary_region)
        self.primary_s3 = boto3.client('s3', 
                                     region_name=self.primary_region)
        
        # Backup region setup
        self.backup_kinesis = boto3.client('kinesis', 
                                         region_name=self.backup_region)
        self.backup_s3 = boto3.client('s3', 
                                    region_name=self.backup_region)
```

### 3. Horizontal Scaling Strategy

Implement stateless scaling for data processing:

```python
class StatelessDataProcessor:
    """Implements stateless processing for horizontal scaling"""
    def __init__(self):
        self.state_store = boto3.client('dynamodb')  # External state storage
        
    async def process_data(self, data: Dict[str, Any]) -> None:
        """Process data in stateless manner"""
        # Retrieve any needed state from external store
        state = await self._get_state(data['id'])
        
        # Process data
        result = await self._process(data, state)
        
        # Update state externally
        await self._save_state(data['id'], result)
```

### 4. Content Delivery Integration

Leverage CloudFront for global data distribution:

```python
class GlobalDataDistribution:
    """Implements CDN-based data distribution"""
    def __init__(self):
        self.cloudfront = boto3.client('cloudfront')
        self.s3 = boto3.client('s3')
        
    async def setup_distribution(self, config: Dict[str, Any]) -> None:
        """Set up CloudFront distribution for data access"""
        try:
            # Create CloudFront distribution
            response = self.cloudfront.create_distribution(
                DistributionConfig={
                    'Origins': {
                        'Items': [{
                            'Id': 'DataLakeOrigin',
                            'DomainName': f"{config['bucket']}.s3.amazonaws.com",
                            'S3OriginConfig': {
                                'OriginAccessIdentity': config['oai']
                            }
                        }]
                    },
                    'DefaultCacheBehavior': {
                        'TargetOriginId': 'DataLakeOrigin',
                        'ViewerProtocolPolicy': 'https-only',
                        'AllowedMethods': {
                            'Items': ['GET', 'HEAD'],
                            'Quantity': 2
                        }
                    },
                    'Enabled': True
                }
            )
            
            return response['Distribution']['Id']
            
        except Exception as e:
            self.logger.error(f"Failed to set up distribution: {str(e)}")
            raise
```

[Previous content continues with Best Practices section...]

> 🔒 **Security Note:** Following AWS's "secure by design" principle, security should be integrated into your data architecture from the start, not added as an afterthought.

[Continue with the rest of the existing content...]

## Best Practices and Optimization

### 1. Cost Optimization

```python
class AWSCostOptimizer:
    """Implements AWS cost optimization strategies"""
    def __init__(self):
        self.s3 = boto3.client('s3')
        self.cloudwatch = boto3.client('cloudwatch')
        self.ce = boto3.client('ce')  # Cost Explorer
        
    async def optimize_storage(self, bucket_name: str) -> None:
        """Implement storage optimization strategies"""
        # 1. Configure Intelligent-Tiering
        self.s3.put_bucket_intelligent_tiering_configuration(
            Bucket=bucket_name,
            Id='default-tiering',
            IntelligentTieringConfiguration={
                'Status': 'Enabled',
                'Tierings': [
                    {
                        'Days': 90,
                        'AccessTier': 'ARCHIVE_ACCESS'
                    },
                    {
                        'Days': 180,
                        'AccessTier': 'DEEP_ARCHIVE_ACCESS'
                    }
                ]
            }
        )
        
        # 2. Set up lifecycle rules
        self.s3.put_bucket_lifecycle_configuration(
            Bucket=bucket_name,
            LifecycleConfiguration={
                'Rules': [
                    {
                        'ID': 'archive-old-data',
                        'Status': 'Enabled',
                        'Prefix': 'data/',
                        'Transitions': [
                            {
                                'Days': 30,
                                'StorageClass': 'STANDARD_IA'
                            },
                            {
                                'Days': 60,
                                'StorageClass': 'GLACIER'
                            }
                        ]
                    }
                ]
            }
        )
        
    async def setup_cost_monitoring(self) -> None:
        """Set up cost monitoring and alerts"""
        # Create monthly budget
        self.ce.create_budget(
            AccountId='ACCOUNT_ID',
            Budget={
                'BudgetName': 'MonthlyDataStorageBudget',
                'BudgetLimit': {
                    'Amount': '1000',
                    'Unit': 'USD'
                },
                'TimeUnit': 'MONTHLY',
                'BudgetType': 'COST',
                'CostFilters': {
                    'Service': ['Amazon Simple Storage Service']
                }
            },
            NotificationsWithSubscribers=[
                {
                    'Notification': {
                        'NotificationType': 'ACTUAL',
                        'ComparisonOperator': 'GREATER_THAN',
                        'Threshold': 80.0
                    },
                    'Subscribers': [
                        {
                            'SubscriptionType': 'EMAIL',
                            'Address': 'admin@example.com'
                        }
                    ]
                }
            ]
        )
        
    async def get_cost_recommendations(self) -> List[Dict[str, Any]]:
        """Get cost optimization recommendations"""
        response = self.ce.get_cost_and_usage(
            TimePeriod={
                'Start': (datetime.now() - timedelta(days=30)).strftime('%Y-%m-%d'),
                'End': datetime.now().strftime('%Y-%m-%d')
            },
            Granularity='MONTHLY',
            Metrics=['UnblendedCost'],
            GroupBy=[
                {'Type': 'DIMENSION', 'Key': 'SERVICE'},
                {'Type': 'DIMENSION', 'Key': 'USAGE_TYPE'}
            ]
        )
        return response['ResultsByTime']
```

### Case Study 1: E-commerce Data Platform Implementation

```python
class EcommerceDataPlatform:
    """Implements scalable e-commerce data platform"""
    def __init__(self, config: Dict[str, Any]):
        self.config = config
        self._initialize_services()
        
    def _initialize_services(self) -> None:
        """Initialize AWS services"""
        self.kinesis = boto3.client('kinesis')
        self.s3 = boto3.client('s3')
        self.dynamodb = boto3.client('dynamodb')
        self.athena = boto3.client('athena')
        
    async def process_transaction(self, transaction: Dict[str, Any]) -> None:
        """Process real-time transaction data"""
        # 1. Stream transaction to Kinesis
        await self._stream_transaction(transaction)
        
        # 2. Update real-time aggregations in DynamoDB
        await self._update_aggregations(transaction)
        
        # 3. Archive to S3 data lake
        await self._archive_transaction(transaction)
        
    async def _stream_transaction(self, transaction: Dict[str, Any]) -> None:
        """Stream transaction to Kinesis"""
        self.kinesis.put_record(
            StreamName=self.config['stream_name'],
            Data=json.dumps(transaction),
            PartitionKey=str(transaction['customer_id'])
        )
        
    async def _update_aggregations(self, transaction: Dict[str, Any]) -> None:
        """Update real-time aggregations"""
        self.dynamodb.update_item(
            TableName=self.config['aggregations_table'],
            Key={
                'customer_id': {'S': str(transaction['customer_id'])},
                'date': {'S': datetime.now().strftime('%Y-%m-%d')}
            },
            UpdateExpression='ADD total_spend :val',
            ExpressionAttributeValues={
                ':val': {'N': str(transaction['amount'])}
            }
        )
        
    async def _archive_transaction(self, transaction: Dict[str, Any]) -> None:
        """Archive transaction to S3"""
        date_path = datetime.now().strftime('%Y/%m/%d/%H')
        key = f"transactions/{date_path}/{transaction['id']}.json"
        
        self.s3.put_object(
            Bucket=self.config['archive_bucket'],
            Key=key,
            Body=json.dumps(transaction),
            ContentType='application/json'
        )
        
    async def generate_daily_report(self, date: str) -> Dict[str, Any]:
        """Generate daily analytics report"""
        query = f"""
        SELECT 
            customer_id,
            COUNT(*) as transaction_count,
            SUM(amount) as total_amount,
            AVG(amount) as avg_amount
        FROM transactions
        WHERE date = '{date}'
        GROUP BY customer_id
        HAVING total_amount > 1000
        ORDER BY total_amount DESC
        LIMIT 100
        """
        
        # Execute Athena query
        response = self.athena.start_query_execution(
            QueryString=query,
            QueryExecutionContext={
                'Database': self.config['athena_database']
            },
            ResultConfiguration={
                'OutputLocation': f"s3://{self.config['query_results_bucket']}/temp/"
            }
        )
        
        return {
            'execution_id': response['QueryExecutionId'],
            'status': 'RUNNING'
        }
```

# ... rest of the existing content ...

## Real-World Case Studies

### Case Study 1: E-commerce Data Platform
A major e-commerce company needed to process 10TB of daily transaction data while maintaining real-time analytics capabilities.

**Challenge:**
- High data volume
- Real-time requirements
- Cost constraints
- Complex analytics needs

**Solution:**
```python
# Implementation example of the solution
class EcommerceDataPlatform:
    """Implements scalable e-commerce data platform"""
    # ... implementation details ...
```

**Results:**
- 60% reduction in processing costs
- 99.99% data availability
- Real-time analytics capabilities
- Improved customer insights

## Common Pitfalls and Solutions

### 1. Cost Management Issues
❌ **Problem:** Unoptimized S3 storage leading to high costs
✅ **Solution:** Implement lifecycle policies and use appropriate storage classes

```python
class StorageCostOptimizer:
    """Implements cost optimization strategies for AWS storage"""
    def __init__(self):
        self.s3 = boto3.client('s3')
        self.cloudwatch = boto3.client('cloudwatch')
        
    async def optimize_storage_costs(self, bucket: str) -> None:
        """Implement cost-optimized storage strategy"""
        try:
            # 1. Analyze current storage patterns
            metrics = await self._analyze_storage_patterns(bucket)
            
            # 2. Configure lifecycle rules based on access patterns
            await self._configure_lifecycle_rules(bucket, metrics)
            
            # 3. Set up monitoring and alerts
            await self._setup_cost_alerts(bucket)
            
        except Exception as e:
            logging.error(f"Failed to optimize storage costs: {str(e)}")
            raise
            
    async def _analyze_storage_patterns(self, bucket: str) -> Dict[str, Any]:
        """Analyze storage access patterns"""
        response = self.cloudwatch.get_metric_statistics(
            Namespace='AWS/S3',
            MetricName='GetRequests',
            Dimensions=[{'Name': 'BucketName', 'Value': bucket}],
            StartTime=datetime.now() - timedelta(days=30),
            EndTime=datetime.now(),
            Period=86400,  # Daily statistics
            Statistics=['Sum']
        )
        
        return {
            'access_frequency': response['Datapoints'],
            'last_accessed': max(p['Timestamp'] for p in response['Datapoints'])
        }
        
    async def _configure_lifecycle_rules(
        self, 
        bucket: str, 
        metrics: Dict[str, Any]
    ) -> None:
        """Configure optimal lifecycle rules"""
        rules = [
            {
                'ID': 'InfrequentAccessTransition',
                'Status': 'Enabled',
                'Filter': {'Prefix': 'data/'},
                'Transitions': [
                    {
                        'Days': 30,
                        'StorageClass': 'STANDARD_IA'
                    }
                ]
            },
            {
                'ID': 'ArchiveOldData',
                'Status': 'Enabled',
                'Filter': {'Prefix': 'archive/'},
                'Transitions': [
                    {
                        'Days': 90,
                        'StorageClass': 'GLACIER'
                    }
                ]
            }
        ]
        
        self.s3.put_bucket_lifecycle_configuration(
            Bucket=bucket,
            LifecycleConfiguration={'Rules': rules}
        )
        
    async def _setup_cost_alerts(self, bucket: str) -> None:
        """Set up cost monitoring alerts"""
        self.cloudwatch.put_metric_alarm(
            AlarmName=f"{bucket}-storage-cost-alarm",
            MetricName='BucketSizeBytes',
            Namespace='AWS/S3',
            Statistic='Average',
            Period=86400,
            EvaluationPeriods=1,
            Threshold=1000000000000,  # 1TB
            ComparisonOperator='GreaterThanThreshold',
            AlarmActions=['arn:aws:sns:region:account-id:topic-name']
        )
```

## Practice Exercises

### Exercise 1: Basic Data Lake Setup

```python
class BasicDataLake:
    """Starter implementation for practice data lake"""
    def __init__(self, config: Dict[str, Any]):
        self.config = config
        self.logger = logging.getLogger(__name__)
        self._initialize_clients()
        
    def _initialize_clients(self) -> None:
        """Initialize AWS service clients"""
        self.s3 = boto3.client('s3')
        self.glue = boto3.client('glue')
        self.athena = boto3.client('athena')
        
    async def setup_basic_lake(self) -> None:
        """Set up basic data lake infrastructure"""
        try:
            # 1. Create S3 buckets
            await self._create_buckets()
            
            # 2. Set up Glue Catalog
            await self._setup_glue_catalog()
            
            # 3. Configure Athena
            await self._setup_athena()
            
        except Exception as e:
            self.logger.error(f"Failed to set up data lake: {str(e)}")
            raise
            
    async def _create_buckets(self) -> None:
        """Create required S3 buckets"""
        buckets = {
            'raw': f"{self.config['prefix']}-raw",
            'processed': f"{self.config['prefix']}-processed",
            'athena': f"{self.config['prefix']}-athena-results"
        }
        
        for purpose, bucket in buckets.items():
            self.s3.create_bucket(
                Bucket=bucket,
                CreateBucketConfiguration={
                    'LocationConstraint': self.config['region']
                }
            )
            
            # Enable versioning
            self.s3.put_bucket_versioning(
                Bucket=bucket,
                VersioningConfiguration={'Status': 'Enabled'}
            )
            
    async def _setup_glue_catalog(self) -> None:
        """Set up AWS Glue Data Catalog"""
        # Create database
        self.glue.create_database(
            DatabaseInput={
                'Name': self.config['database_name'],
                'Description': 'Practice data lake catalog'
            }
        )
        
        # Create crawler for data discovery
        self.glue.create_crawler(
            Name=f"{self.config['prefix']}-crawler",
            Role=self.config['crawler_role'],
            DatabaseName=self.config['database_name'],
            Targets={
                'S3Targets': [
                    {'Path': f"s3://{self.config['prefix']}-processed/"}
                ]
            },
            Schedule='cron(0 0 * * ? *)'  # Run daily
        )
        
    async def _setup_athena(self) -> None:
        """Configure Athena for querying"""
        self.athena.create_work_group(
            Name=f"{self.config['prefix']}-workgroup",
            Configuration={
                'ResultConfiguration': {
                    'OutputLocation': f"s3://{self.config['prefix']}-athena-results/"
                },
                'EnforceWorkGroupConfiguration': True,
                'PublishCloudWatchMetricsEnabled': True,
                'BytesScannedCutoffPerQuery': 1073741824  # 1GB
            }
        )
        
    async def ingest_data(self, data: Dict[str, Any], dataset: str) -> None:
        """Ingest raw data into the lake"""
        key = f"raw/{dataset}/{datetime.now().strftime('%Y/%m/%d/%H/%M')}.json"
        
        self.s3.put_object(
            Bucket=f"{self.config['prefix']}-raw",
            Key=key,
            Body=json.dumps(data),
            ContentType='application/json'
        )
        
    async def transform_data(
        self, 
        source_key: str, 
        transformation: Callable
    ) -> None:
        """Apply basic transformation to data"""
        # Get source data
        response = self.s3.get_object(
            Bucket=f"{self.config['prefix']}-raw",
            Key=source_key
        )
        data = json.loads(response['Body'].read())
        
        # Apply transformation
        transformed_data = transformation(data)
        
        # Save transformed data
        target_key = source_key.replace('raw', 'processed')
        self.s3.put_object(
            Bucket=f"{self.config['prefix']}-processed",
            Key=target_key,
            Body=json.dumps(transformed_data),
            ContentType='application/json'
        )
        
    async def query_data(self, query: str) -> Dict[str, Any]:
        """Execute query on processed data"""
        response = self.athena.start_query_execution(
            QueryString=query,
            QueryExecutionContext={
                'Database': self.config['database_name']
            },
            WorkGroup=f"{self.config['prefix']}-workgroup"
        )
        
        return {
            'execution_id': response['QueryExecutionId'],
            'status': 'RUNNING'
        }
        
    async def monitor_metrics(self) -> Dict[str, Any]:
        """Get basic monitoring metrics"""
        metrics = {}
        
        # Get storage metrics
        for bucket_type in ['raw', 'processed']:
            bucket = f"{self.config['prefix']}-{bucket_type}"
            response = self.cloudwatch.get_metric_statistics(
                Namespace='AWS/S3',
                MetricName='BucketSizeBytes',
                Dimensions=[{'Name': 'BucketName', 'Value': bucket}],
                StartTime=datetime.now() - timedelta(days=1),
                EndTime=datetime.now(),
                Period=3600,
                Statistics=['Average']
            )
            metrics[f"{bucket_type}_size"] = response['Datapoints'][-1]['Average']
            
        return metrics
```

## Review Questions and Discussion Topics

1. How would you design a cost-effective data lake architecture for a startup?
   - Consider starting with basic storage tiers
   - Implement auto-archiving for infrequently accessed data
   - Use Athena for cost-effective querying
   - Monitor and optimize based on usage patterns

2. What are the trade-offs between different storage formats in AWS?
   - Parquet: Best for analytical queries, but complex to modify
   - Avro: Good for streaming, but less efficient for analytics
   - JSON: Flexible but storage inefficient
   - ORC: Highly compressed but limited tool support

3. How do you handle schema evolution in a data lake?
   - Use schema-flexible formats like Avro
   - Implement versioning in your data catalog
   - Maintain backward compatibility
   - Document schema changes

4. Discuss the implications of different partitioning strategies:
   - Time-based partitioning for time-series data
   - Geographic partitioning for regional data
   - Category-based partitioning for product data
   - Hybrid approaches for complex datasets

## Next Steps
- Explore AWS Glue for ETL workflows
- Learn about AWS Lake Formation security features
- Practice with real datasets using the provided implementations
- Build a portfolio project using the concepts covered

Remember: The best way to learn is by doing. Use these implementations as a starting point and experiment with different configurations and optimizations.

> 📝 **Note:** Always follow AWS best practices for security and cost optimization when implementing these solutions in production environments.
