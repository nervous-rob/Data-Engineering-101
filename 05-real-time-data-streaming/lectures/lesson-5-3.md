# Lesson 5.3: Advanced Kafka Concepts

## Navigation
- [← Back to Lesson Plan](../5.3-advanced-kafka-concepts.md)
- [← Back to Module Overview](../README.md)

## Overview
This lesson explores advanced Apache Kafka concepts, focusing on exactly-once semantics, Kafka Streams, and sophisticated streaming patterns. We'll dive deep into the internals of Kafka's transaction system and learn how to build robust streaming applications.

## Learning Objectives
After completing this lesson, you'll be able to:
- Implement exactly-once semantics in Kafka applications
- Master Kafka Streams for complex stream processing
- Handle advanced consumer group scenarios
- Design fault-tolerant streaming architectures
- Implement sophisticated message delivery patterns
- Build scalable stream processing applications

## Advanced Kafka Concepts

### 1. Exactly-Once Semantics (EOS)

Let's implement exactly-once processing with Kafka Streams:

```python
from typing import Dict, Optional
from kafka.errors import (
    ProducerFencedException,
    InvalidProducerEpochException
)
import json
import logging

class ExactlyOnceProcessor:
    """Implements exactly-once processing semantics"""
    def __init__(
        self,
        application_id: str,
        bootstrap_servers: List[str],
        input_topic: str,
        output_topic: str
    ):
        self.application_id = application_id
        self.input_topic = input_topic
        self.output_topic = output_topic
        self.logger = logging.getLogger(__name__)
        
        # Configure with exactly-once semantics
        self.configs = {
            'bootstrap.servers': bootstrap_servers,
            'application.id': application_id,
            'processing.guarantee': 'exactly_once_v2',
            'transaction.timeout.ms': 900000,  # 15 minutes
            'commit.interval.ms': 100,
            'replication.factor': 3
        }
        
    def process_stream(self) -> None:
        """Process stream with exactly-once guarantees"""
        try:
            # Initialize stream processor
            topology = self._build_topology()
            streams = KafkaStreams(topology, self.configs)
            
            # Add state listener
            streams.setStateListener(self._state_listener)
            
            # Start processing
            streams.start()
            
        except ProducerFencedException as e:
            self.logger.error("Producer was fenced out")
            self._handle_fencing_error(e)
        except InvalidProducerEpochException as e:
            self.logger.error("Invalid producer epoch")
            self._handle_epoch_error(e)
            
    def _build_topology(self) -> Topology:
        """Build processing topology"""
        topology = Topology()
        
        # Add source node
        topology.addSource(
            "Source",
            self.input_topic
        )
        
        # Add processor node
        topology.addProcessor(
            "Process",
            self._process_record,
            ["Source"]
        )
        
        # Add sink node
        topology.addSink(
            "Sink",
            self.output_topic,
            ["Process"]
        )
        
        return topology
        
    def _process_record(
        self,
        record: ConsumerRecord
    ) -> Optional[ProducerRecord]:
        """Process a single record with exactly-once semantics"""
        try:
            # Process record
            result = self._transform_record(record)
            
            # Create output record
            if result:
                return ProducerRecord(
                    self.output_topic,
                    key=record.key,
                    value=result,
                    timestamp=record.timestamp
                )
            
        except Exception as e:
            self.logger.error(f"Error processing record: {str(e)}")
            self._handle_processing_error(record, e)
            
        return None
        
    def _transform_record(
        self,
        record: ConsumerRecord
    ) -> Optional[Dict]:
        """Transform record - override in subclasses"""
        pass
        
    def _state_listener(
        self,
        new_state: KafkaStreams.State,
        old_state: KafkaStreams.State
    ) -> None:
        """Monitor state transitions"""
        self.logger.info(
            f"State transition: {old_state} -> {new_state}"
        )
        
    def _handle_fencing_error(self, error: Exception) -> None:
        """Handle producer fencing errors"""
        # Implement error handling strategy
        pass
        
    def _handle_epoch_error(self, error: Exception) -> None:
        """Handle producer epoch errors"""
        # Implement error handling strategy
        pass
```

### 2. Advanced Consumer Group Patterns

Implement sophisticated consumer group management:

```python
from kafka import KafkaConsumer, TopicPartition
from kafka.coordinator.assignors import RangeAssignor
from threading import Thread, Event
import time

class AdvancedConsumerGroup:
    """Implements advanced consumer group patterns"""
    def __init__(
        self,
        bootstrap_servers: List[str],
        topic: str,
        group_id: str,
        num_consumers: int,
        session_timeout_ms: int = 10000,
        max_poll_interval_ms: int = 300000
    ):
        self.topic = topic
        self.group_id = group_id
        self.num_consumers = num_consumers
        self.consumers: List[KafkaConsumer] = []
        self.consumer_threads: List[Thread] = []
        self.stop_event = Event()
        
        # Create consumers
        for i in range(num_consumers):
            consumer = KafkaConsumer(
                topic,
                bootstrap_servers=bootstrap_servers,
                group_id=group_id,
                enable_auto_commit=False,
                partition_assignment_strategy=[RangeAssignor],
                session_timeout_ms=session_timeout_ms,
                max_poll_interval_ms=max_poll_interval_ms
            )
            self.consumers.append(consumer)
            
    def start(self) -> None:
        """Start all consumers"""
        for i, consumer in enumerate(self.consumers):
            thread = Thread(
                target=self._consume_loop,
                args=(consumer, i)
            )
            self.consumer_threads.append(thread)
            thread.start()
            
    def stop(self) -> None:
        """Stop all consumers"""
        self.stop_event.set()
        
        for thread in self.consumer_threads:
            thread.join()
            
        for consumer in self.consumers:
            consumer.close()
            
    def _consume_loop(
        self,
        consumer: KafkaConsumer,
        consumer_id: int
    ) -> None:
        """Main consumer loop"""
        try:
            while not self.stop_event.is_set():
                messages = consumer.poll(timeout_ms=1000)
                
                if not messages:
                    continue
                    
                for tp, records in messages.items():
                    for record in records:
                        self._process_record(record, consumer_id)
                        
                    # Commit offsets for partition
                    self._commit_offsets(consumer, tp, records)
                    
        except Exception as e:
            self.logger.error(
                f"Consumer {consumer_id} error: {str(e)}"
            )
            self._handle_consumer_error(consumer_id, e)
            
    def _process_record(
        self,
        record: ConsumerRecord,
        consumer_id: int
    ) -> None:
        """Process a single record"""
        try:
            # Process record
            result = self._transform_record(record)
            
            if result:
                self._handle_result(result, consumer_id)
                
        except Exception as e:
            self.logger.error(
                f"Error processing record in consumer {consumer_id}: "
                f"{str(e)}"
            )
            self._handle_processing_error(record, consumer_id, e)
            
    def _commit_offsets(
        self,
        consumer: KafkaConsumer,
        tp: TopicPartition,
        records: List[ConsumerRecord]
    ) -> None:
        """Commit offsets for processed records"""
        last_offset = records[-1].offset
        consumer.commit({
            tp: last_offset + 1
        })
```

### 3. Kafka Streams Patterns

Implement advanced Kafka Streams patterns:

```python
from typing import Dict, Optional
from datetime import datetime, timedelta

class StreamProcessor:
    """Advanced stream processing patterns"""
    def __init__(
        self,
        application_id: str,
        bootstrap_servers: List[str],
        input_topics: List[str],
        output_topic: str,
        window_size_ms: int = 60000,
        retention_ms: int = 604800000  # 7 days
    ):
        self.application_id = application_id
        self.input_topics = input_topics
        self.output_topic = output_topic
        
        # Configure streams
        self.configs = {
            'application.id': application_id,
            'bootstrap.servers': bootstrap_servers,
            'processing.guarantee': 'exactly_once_v2',
            'commit.interval.ms': 100,
            'window.size.ms': window_size_ms,
            'retention.ms': retention_ms
        }
        
    def build_topology(self) -> Topology:
        """Build processing topology"""
        topology = Topology()
        
        # Add source nodes
        for topic in self.input_topics:
            topology.addSource(
                f"Source-{topic}",
                topic
            )
            
        # Add processor nodes
        topology.addProcessor(
            "Windowed-Aggregator",
            self._window_aggregate_processor,
            self.input_topics
        )
        
        # Add state store
        topology.addStateStore(
            self._create_window_store(),
            ["Windowed-Aggregator"]
        )
        
        # Add sink node
        topology.addSink(
            "Sink",
            self.output_topic,
            ["Windowed-Aggregator"]
        )
        
        return topology
        
    def _window_aggregate_processor(
        self,
        record: ConsumerRecord,
        state: WindowStore
    ) -> Optional[ProducerRecord]:
        """Process records with windowed aggregation"""
        try:
            # Get window for record
            window = self._get_window(record.timestamp)
            
            # Update window state
            current = state.get(window) or {}
            updated = self._aggregate_record(record, current)
            state.put(window, updated)
            
            # Check if window is complete
            if self._is_window_complete(window):
                result = self._finalize_window(window, updated)
                return ProducerRecord(
                    self.output_topic,
                    key=str(window.start),
                    value=result
                )
                
        except Exception as e:
            self.logger.error(f"Processing error: {str(e)}")
            self._handle_processing_error(record, e)
            
        return None
        
    def _create_window_store(self) -> WindowStore:
        """Create windowed state store"""
        return Stores.windowStoreBuilder(
            Stores.persistentWindowStore(
                "window-store",
                timedelta(
                    milliseconds=self.configs['window.size.ms']
                ),
                timedelta(
                    milliseconds=self.configs['retention.ms']
                ),
                3,  # Number of segments
                True,  # Retain duplicates
                timedelta(milliseconds=60000)  # Segment interval
            ),
            Serdes.String(),
            Serdes.String()
        ).build()
```

### 4. Advanced Error Handling

Implement sophisticated error handling patterns:

```python
from typing import Optional, Callable
from kafka.errors import KafkaError
import json
import logging

class ErrorHandler:
    """Advanced error handling patterns"""
    def __init__(
        self,
        dead_letter_topic: str,
        max_retries: int = 3,
        retry_backoff_ms: int = 1000,
        error_callback: Optional[Callable] = None
    ):
        self.dead_letter_topic = dead_letter_topic
        self.max_retries = max_retries
        self.retry_backoff_ms = retry_backoff_ms
        self.error_callback = error_callback
        self.logger = logging.getLogger(__name__)
        
    def handle_processing_error(
        self,
        error: Exception,
        record: ConsumerRecord,
        retry_count: int = 0
    ) -> None:
        """Handle processing errors with retry logic"""
        try:
            if retry_count < self.max_retries:
                # Implement retry with backoff
                wait_time = self.retry_backoff_ms * (2 ** retry_count)
                time.sleep(wait_time / 1000)
                
                # Retry processing
                self._retry_processing(record, retry_count + 1)
            else:
                # Send to dead letter topic
                self._send_to_dead_letter_topic(record, error)
                
        except Exception as e:
            self.logger.error(
                f"Error handling failed: {str(e)}"
            )
            if self.error_callback:
                self.error_callback(e, record)
                
    def _retry_processing(
        self,
        record: ConsumerRecord,
        retry_count: int
    ) -> None:
        """Retry record processing"""
        try:
            # Implement retry logic
            pass
        except Exception as e:
            self.handle_processing_error(e, record, retry_count)
            
    def _send_to_dead_letter_topic(
        self,
        record: ConsumerRecord,
        error: Exception
    ) -> None:
        """Send failed record to dead letter topic"""
        try:
            dead_letter_record = {
                'original_topic': record.topic,
                'original_partition': record.partition,
                'original_offset': record.offset,
                'original_timestamp': record.timestamp,
                'error_message': str(error),
                'error_timestamp': datetime.now().isoformat(),
                'original_value': record.value
            }
            
            # Send to dead letter topic
            producer = KafkaProducer(
                bootstrap_servers=self.bootstrap_servers,
                value_serializer=lambda v: json.dumps(v).encode('utf-8')
            )
            
            producer.send(
                self.dead_letter_topic,
                value=dead_letter_record
            )
            producer.flush()
            producer.close()
            
        except Exception as e:
            self.logger.error(
                f"Failed to send to dead letter topic: {str(e)}"
            )
            if self.error_callback:
                self.error_callback(e, record)
```

## Best Practices and Patterns

### 1. Transaction Management
- Use exactly-once semantics for critical applications
- Configure appropriate transaction timeouts
- Implement proper error handling for transaction failures
- Monitor transaction health and performance

### 2. Consumer Group Management
- Implement proper rebalancing protocols
- Handle consumer failures gracefully
- Monitor consumer lag and health
- Use appropriate partition assignment strategies

### 3. State Management
- Use persistent state stores for reliability
- Implement proper backup and recovery
- Monitor state store performance
- Handle state store failures

### 4. Performance Optimization
- Configure appropriate batch sizes
- Use compression when needed
- Monitor and tune resource usage
- Implement proper caching strategies

## Practical Exercises

### Exercise 1: Implementing Exactly-Once Processing
```python
# Example implementation in previous sections
```

### Exercise 2: Building a Stateful Stream Processor
```python
# Example implementation in previous sections
```

## Additional Resources

1. **Advanced Kafka Documentation**
   - [Kafka Streams Architecture](https://kafka.apache.org/documentation/streams/architecture)
   - [Kafka Transaction Protocol](https://kafka.apache.org/protocol.html#transaction)
   - [Kafka Streams DSL](https://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html)

2. **Best Practices and Patterns**
   - [Kafka Streams Design Patterns](https://www.confluent.io/blog/kafka-streams-design-patterns/)
   - [Exactly-Once Semantics](https://www.confluent.io/blog/exactly-once-semantics-are-possible-heres-how-apache-kafka-does-it/)
   - [State Store Management](https://www.confluent.io/blog/kafka-streams-state-store-internals/)

3. **Advanced Topics**
   - [Kafka Streams Testing](https://kafka.apache.org/documentation/streams/developer-guide/testing.html)
   - [Monitoring and Metrics](https://kafka.apache.org/documentation/#monitoring)
   - [Security Patterns](https://kafka.apache.org/documentation/#security)

## Next Steps
- Explore Spark Streaming integration
- Study advanced monitoring techniques
- Practice with real-world scenarios
- Implement security patterns 