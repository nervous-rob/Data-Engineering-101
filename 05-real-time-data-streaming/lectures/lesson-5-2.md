# Lesson 5.2: Apache Kafka Fundamentals

## Navigation
- [← Back to Lesson Plan](../5.2-apache-kafka-fundamentals.md)
- [← Back to Module Overview](../README.md)

## Overview
Apache Kafka has become the de facto standard for real-time data streaming in modern architectures. In this lesson, we'll explore Kafka's core concepts, architecture, and implementation patterns that make it the backbone of distributed streaming systems.

## Learning Objectives
After completing this lesson, you'll be able to:
- Master Kafka's core concepts and architectural components
- Implement robust producer and consumer patterns
- Configure Kafka for optimal performance and reliability
- Handle common failure scenarios and recovery patterns
- Apply best practices for topic design and partitioning
- Build scalable and maintainable Kafka applications

## Core Concepts and Architecture

### 1. Kafka Architecture Components

Let's explore the key components that form Kafka's distributed architecture:

```python
from dataclasses import dataclass
from typing import List, Dict, Optional
from datetime import datetime

@dataclass
class KafkaMessage:
    """Represents a message in Kafka"""
    key: Optional[bytes]
    value: bytes
    topic: str
    partition: int
    offset: int
    timestamp: datetime
    headers: Dict[str, bytes]

@dataclass
class TopicPartition:
    """Represents a Kafka topic partition"""
    topic: str
    partition: int
    leader: int
    replicas: List[int]
    isr: List[int]  # In-Sync Replicas
    
class KafkaCluster:
    """Represents a Kafka cluster configuration"""
    def __init__(
        self,
        bootstrap_servers: List[str],
        num_brokers: int,
        replication_factor: int
    ):
        self.bootstrap_servers = bootstrap_servers
        self.num_brokers = num_brokers
        self.replication_factor = replication_factor
        self.topics: Dict[str, List[TopicPartition]] = {}
        
    def create_topic(
        self,
        topic: str,
        num_partitions: int,
        replication_factor: Optional[int] = None
    ) -> None:
        """Create a new topic with specified partitions"""
        if replication_factor is None:
            replication_factor = self.replication_factor
            
        partitions = []
        for i in range(num_partitions):
            # Simulate partition assignment
            leader = i % self.num_brokers
            replicas = self._calculate_replicas(leader, replication_factor)
            partition = TopicPartition(
                topic=topic,
                partition=i,
                leader=leader,
                replicas=replicas,
                isr=replicas.copy()
            )
            partitions.append(partition)
            
        self.topics[topic] = partitions
        
    def _calculate_replicas(
        self,
        leader: int,
        replication_factor: int
    ) -> List[int]:
        """Calculate replica assignments for a partition"""
        replicas = [leader]
        for i in range(1, replication_factor):
            replica = (leader + i) % self.num_brokers
            replicas.append(replica)
        return replicas
```

### 2. Producer Implementation Patterns

Let's implement robust producer patterns with proper error handling and configuration:

```python
from kafka import KafkaProducer
from kafka.errors import KafkaError
import json
import logging
from typing import Any, Callable, Optional

class RobustKafkaProducer:
    """Implements robust Kafka producer patterns"""
    def __init__(
        self,
        bootstrap_servers: List[str],
        topic: str,
        acks: str = 'all',
        retries: int = 3,
        batch_size: int = 16384,
        linger_ms: int = 1,
        compression_type: str = 'gzip',
        partitioner: Optional[Callable] = None
    ):
        self.topic = topic
        self.logger = logging.getLogger(__name__)
        
        # Configure producer with best practices
        self.producer = KafkaProducer(
            bootstrap_servers=bootstrap_servers,
            acks=acks,  # 'all' for strongest durability
            retries=retries,
            batch_size=batch_size,
            linger_ms=linger_ms,
            compression_type=compression_type,
            value_serializer=lambda x: json.dumps(x).encode('utf-8'),
            partitioner=partitioner
        )
        
    def send_message(
        self,
        key: Optional[str],
        value: Any,
        headers: Optional[Dict[str, str]] = None
    ) -> None:
        """Send message with error handling and retries"""
        try:
            # Convert headers to bytes
            kafka_headers = None
            if headers:
                kafka_headers = [
                    (k, v.encode('utf-8'))
                    for k, v in headers.items()
                ]
            
            # Send message
            future = self.producer.send(
                self.topic,
                key=key.encode('utf-8') if key else None,
                value=value,
                headers=kafka_headers
            )
            
            # Wait for the send to complete
            record_metadata = future.get(timeout=10)
            
            self.logger.info(
                f"Message sent successfully to {record_metadata.topic} "
                f"[{record_metadata.partition}] @ {record_metadata.offset}"
            )
            
        except KafkaError as e:
            self.logger.error(f"Failed to send message: {str(e)}")
            raise
            
    def close(self) -> None:
        """Clean up producer resources"""
        self.producer.flush()
        self.producer.close()
```

### 3. Consumer Implementation Patterns

Implement consumer patterns with proper offset management and error handling:

```python
from kafka import KafkaConsumer, TopicPartition
from kafka.errors import KafkaError
import json
from typing import Callable, Dict, Optional

class RobustKafkaConsumer:
    """Implements robust Kafka consumer patterns"""
    def __init__(
        self,
        bootstrap_servers: List[str],
        topic: str,
        group_id: str,
        auto_offset_reset: str = 'earliest',
        enable_auto_commit: bool = False,
        max_poll_records: int = 500,
        max_poll_interval_ms: int = 300000,
        processor: Optional[Callable] = None
    ):
        self.topic = topic
        self.processor = processor
        self.logger = logging.getLogger(__name__)
        
        # Configure consumer with best practices
        self.consumer = KafkaConsumer(
            topic,
            bootstrap_servers=bootstrap_servers,
            group_id=group_id,
            auto_offset_reset=auto_offset_reset,
            enable_auto_commit=enable_auto_commit,
            max_poll_records=max_poll_records,
            max_poll_interval_ms=max_poll_interval_ms,
            value_deserializer=lambda m: json.loads(m.decode('utf-8'))
        )
        
    def start_consuming(self) -> None:
        """Start consuming messages with error handling"""
        try:
            while True:
                messages = self.consumer.poll(timeout_ms=1000)
                
                for tp, records in messages.items():
                    for record in records:
                        try:
                            if self.processor:
                                self.processor(record)
                            else:
                                self._default_processor(record)
                                
                            # Manually commit offset after successful processing
                            self._commit_offset(tp, record.offset + 1)
                            
                        except Exception as e:
                            self.logger.error(
                                f"Error processing message: {str(e)}"
                            )
                            self._handle_processing_error(record)
                            
        except KafkaError as e:
            self.logger.error(f"Kafka consumer error: {str(e)}")
            raise
        finally:
            self.close()
            
    def _default_processor(self, record) -> None:
        """Default message processor"""
        self.logger.info(
            f"Received message: {record.value} from "
            f"{record.topic}[{record.partition}] @ {record.offset}"
        )
        
    def _commit_offset(
        self,
        tp: TopicPartition,
        offset: int
    ) -> None:
        """Commit offset for a topic partition"""
        self.consumer.commit({tp: offset})
        
    def _handle_processing_error(self, record) -> None:
        """Handle message processing errors"""
        # Implement error handling strategy
        # e.g., dead letter queue, retry logic, etc.
        pass
        
    def close(self) -> None:
        """Clean up consumer resources"""
        self.consumer.close()
```

### 4. Topic and Partition Management

Implement tools for managing topics and partitions:

```python
from kafka.admin import KafkaAdminClient, NewTopic
from kafka.errors import TopicAlreadyExistsError
import math

class KafkaTopicManager:
    """Manages Kafka topics and partitions"""
    def __init__(self, bootstrap_servers: List[str]):
        self.admin_client = KafkaAdminClient(
            bootstrap_servers=bootstrap_servers
        )
        
    def create_topic(
        self,
        topic: str,
        num_partitions: int,
        replication_factor: int,
        config: Optional[Dict[str, str]] = None
    ) -> None:
        """Create a new topic with specified configuration"""
        try:
            new_topic = NewTopic(
                name=topic,
                num_partitions=num_partitions,
                replication_factor=replication_factor,
                topic_configs=config or {}
            )
            
            self.admin_client.create_topics([new_topic])
            
        except TopicAlreadyExistsError:
            self.logger.warning(f"Topic {topic} already exists")
            
    def calculate_num_partitions(
        self,
        target_throughput_mb: float,
        message_size_kb: float,
        consumer_throughput_mb: float
    ) -> int:
        """Calculate optimal number of partitions based on throughput"""
        messages_per_sec = (target_throughput_mb * 1024) / message_size_kb
        consumer_messages_per_sec = (
            consumer_throughput_mb * 1024
        ) / message_size_kb
        
        return math.ceil(
            messages_per_sec / consumer_messages_per_sec
        )
```

## Best Practices and Patterns

### 1. Configuration Best Practices

```python
def get_recommended_configs() -> Dict[str, Dict[str, Any]]:
    """Get recommended configurations for different components"""
    return {
        'broker': {
            'num.network.threads': 3,
            'num.io.threads': 8,
            'socket.send.buffer.bytes': 102400,
            'socket.receive.buffer.bytes': 102400,
            'socket.request.max.bytes': 104857600,
            'log.retention.hours': 168,
            'num.partitions': 1,
            'num.recovery.threads.per.data.dir': 1,
        },
        'producer': {
            'acks': 'all',
            'retries': 3,
            'batch.size': 16384,
            'linger.ms': 1,
            'compression.type': 'gzip',
            'max.in.flight.requests.per.connection': 5,
        },
        'consumer': {
            'fetch.min.bytes': 1,
            'fetch.max.wait.ms': 500,
            'max.partition.fetch.bytes': 1048576,
            'enable.auto.commit': False,
            'auto.offset.reset': 'earliest',
            'max.poll.records': 500,
        }
    }
```

### 2. Monitoring and Metrics

```python
from dataclasses import dataclass
from typing import Dict
import time

@dataclass
class KafkaMetrics:
    """Metrics for Kafka monitoring"""
    messages_sent: int = 0
    messages_received: int = 0
    bytes_sent: int = 0
    bytes_received: int = 0
    errors: int = 0
    avg_latency_ms: float = 0.0
    
class KafkaMonitor:
    """Monitor Kafka performance and health"""
    def __init__(self):
        self.metrics: Dict[str, KafkaMetrics] = {}
        self.start_time = time.time()
        
    def record_producer_metrics(
        self,
        topic: str,
        message_size: int,
        latency_ms: float
    ) -> None:
        """Record producer metrics"""
        if topic not in self.metrics:
            self.metrics[topic] = KafkaMetrics()
            
        metrics = self.metrics[topic]
        metrics.messages_sent += 1
        metrics.bytes_sent += message_size
        metrics.avg_latency_ms = (
            (metrics.avg_latency_ms * (metrics.messages_sent - 1) + latency_ms)
            / metrics.messages_sent
        )
        
    def record_consumer_metrics(
        self,
        topic: str,
        message_size: int
    ) -> None:
        """Record consumer metrics"""
        if topic not in self.metrics:
            self.metrics[topic] = KafkaMetrics()
            
        metrics = self.metrics[topic]
        metrics.messages_received += 1
        metrics.bytes_received += message_size
        
    def record_error(self, topic: str) -> None:
        """Record processing error"""
        if topic not in self.metrics:
            self.metrics[topic] = KafkaMetrics()
            
        self.metrics[topic].errors += 1
        
    def get_metrics(self, topic: str) -> KafkaMetrics:
        """Get metrics for a specific topic"""
        return self.metrics.get(topic, KafkaMetrics())
```

## Practical Exercises

### Exercise 1: Building a Fault-Tolerant Producer
```python
# Implementation of a fault-tolerant producer with retry logic
class FaultTolerantProducer:
    """Implements a fault-tolerant Kafka producer"""
    def __init__(
        self,
        bootstrap_servers: List[str],
        topic: str,
        retries: int = 3,
        retry_backoff_ms: int = 100
    ):
        self.producer = RobustKafkaProducer(
            bootstrap_servers=bootstrap_servers,
            topic=topic,
            retries=retries
        )
        self.retry_backoff_ms = retry_backoff_ms
        
    def send_with_retry(
        self,
        key: str,
        value: Dict[str, Any],
        max_retries: int = 3
    ) -> None:
        """Send message with retry logic"""
        retries = 0
        while retries <= max_retries:
            try:
                self.producer.send_message(key, value)
                return
            except KafkaError as e:
                retries += 1
                if retries > max_retries:
                    raise
                    
                # Exponential backoff
                wait_time = (
                    self.retry_backoff_ms * (2 ** (retries - 1))
                )
                time.sleep(wait_time / 1000)  # Convert to seconds

# Usage example
producer = FaultTolerantProducer(
    bootstrap_servers=['localhost:9092'],
    topic='test-topic'
)

message = {
    'id': '123',
    'timestamp': datetime.now().isoformat(),
    'data': 'test message'
}

producer.send_with_retry('key1', message)
```

### Exercise 2: Implementing a Scalable Consumer Group
```python
class ScalableConsumerGroup:
    """Implements a scalable consumer group pattern"""
    def __init__(
        self,
        bootstrap_servers: List[str],
        topic: str,
        group_id: str,
        num_consumers: int
    ):
        self.consumers = []
        self.topic = topic
        self.group_id = group_id
        
        # Create multiple consumers
        for i in range(num_consumers):
            consumer = RobustKafkaConsumer(
                bootstrap_servers=bootstrap_servers,
                topic=topic,
                group_id=group_id,
                processor=self._process_message
            )
            self.consumers.append(consumer)
            
    def start(self) -> None:
        """Start all consumers"""
        for consumer in self.consumers:
            # In practice, you'd want to run these in separate threads
            consumer.start_consuming()
            
    def _process_message(self, message) -> None:
        """Process received message"""
        print(f"Processing message: {message.value}")
        
    def stop(self) -> None:
        """Stop all consumers"""
        for consumer in self.consumers:
            consumer.close()

# Usage example
consumer_group = ScalableConsumerGroup(
    bootstrap_servers=['localhost:9092'],
    topic='test-topic',
    group_id='test-group',
    num_consumers=3
)

try:
    consumer_group.start()
except KeyboardInterrupt:
    consumer_group.stop()
```

## Additional Resources

1. **Kafka Documentation and Guides**
   - [Apache Kafka Documentation](https://kafka.apache.org/documentation/)
   - [Kafka Protocol Guide](https://kafka.apache.org/protocol.html)
   - [Kafka Streams Documentation](https://kafka.apache.org/documentation/streams/)

2. **Best Practices and Patterns**
   - [Kafka Design Patterns](https://www.confluent.io/blog/event-driven-architecture-patterns/)
   - [Kafka Security Best Practices](https://www.confluent.io/blog/apache-kafka-security-best-practices/)
   - [Kafka Performance Tuning](https://www.confluent.io/blog/kafka-fastest-messaging-system/)

3. **Advanced Topics**
   - [Kafka Connect](https://kafka.apache.org/documentation/#connect)
   - [Kafka Monitoring](https://kafka.apache.org/documentation/#monitoring)
   - [Kafka Multi-Cluster Architectures](https://www.confluent.io/blog/multi-cluster-kafka-architectures/)

## Next Steps
- Explore advanced Kafka concepts
- Learn about Kafka Streams
- Study monitoring and operations
- Practice with real-world scenarios
- Implement security patterns 