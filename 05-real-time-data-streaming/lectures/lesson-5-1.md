# Lesson 5.1: Introduction to Data Streaming

## Navigation
- [← Back to Lesson Plan](../5.1-introduction-to-data-streaming.md)
- [← Back to Module Overview](../README.md)

## Overview
Welcome to the world of real-time data streaming! In this lesson, we'll explore how modern organizations process and analyze data in real-time, enabling immediate insights and rapid decision-making. We'll dive deep into streaming architectures, patterns, and best practices that form the foundation of real-time data processing systems.

## Learning Objectives
After completing this lesson, you'll be able to:
- Master core streaming concepts and architectural patterns
- Understand different stream processing models and their trade-offs
- Implement event-driven architectures effectively
- Handle common streaming challenges with proven solutions
- Apply best practices for stream processing
- Build robust and scalable streaming pipelines

## Real-Time Data Processing Fundamentals

### 1. Core Concepts and Principles

Real-time data processing represents a fundamental shift from traditional batch processing. Let's explore the key characteristics that make streaming systems unique:

#### Key Characteristics
```python
from dataclasses import dataclass
from datetime import datetime
from typing import Any, Dict, List

@dataclass
class StreamingEvent:
    """Represents a streaming event with key metadata"""
    event_id: str
    timestamp: datetime
    source: str
    data: Dict[str, Any]
    schema_version: str
    
    def is_late_arriving(self, window_end: datetime) -> bool:
        """Check if event is late arriving"""
        return self.timestamp < window_end
    
    def validate_schema(self, schema: Dict) -> bool:
        """Validate event against schema"""
        # Schema validation logic
        pass

class StreamProcessor:
    """Base class for stream processing"""
    def __init__(self, name: str):
        self.name = name
        self.processed_count = 0
        self.error_count = 0
        
    def process_event(self, event: StreamingEvent) -> None:
        """Process a single event"""
        try:
            self._validate_event(event)
            self._transform_event(event)
            self._emit_event(event)
            self.processed_count += 1
        except Exception as e:
            self.error_count += 1
            self._handle_error(e, event)
            
    def _validate_event(self, event: StreamingEvent) -> None:
        """Validate incoming event"""
        if not event.validate_schema(self._get_schema()):
            raise ValueError("Invalid event schema")
            
    def _transform_event(self, event: StreamingEvent) -> None:
        """Transform event - override in subclasses"""
        pass
        
    def _emit_event(self, event: StreamingEvent) -> None:
        """Emit processed event"""
        pass
        
    def _handle_error(self, error: Exception, event: StreamingEvent) -> None:
        """Handle processing errors"""
        pass
```

### 2. Stream Processing Models

Different processing models suit different use cases. Let's explore each with its implementation:

#### Record-at-a-Time Processing
```python
class RecordProcessor(StreamProcessor):
    """Process individual records with minimal latency"""
    def __init__(self, name: str, buffer_size: int = 100):
        super().__init__(name)
        self.buffer = []
        self.buffer_size = buffer_size
        
    def process_event(self, event: StreamingEvent) -> None:
        """Process single event immediately"""
        super().process_event(event)
        self.buffer.append(event)
        
        if len(self.buffer) >= self.buffer_size:
            self._flush_buffer()
            
    def _flush_buffer(self) -> None:
        """Flush buffer when full"""
        # Implement buffer flushing logic
        self.buffer.clear()
```

#### Micro-Batch Processing
```python
from typing import List
import time

class MicroBatchProcessor(StreamProcessor):
    """Process events in small batches"""
    def __init__(
        self,
        name: str,
        batch_size: int = 1000,
        batch_timeout: float = 1.0
    ):
        super().__init__(name)
        self.batch_size = batch_size
        self.batch_timeout = batch_timeout
        self.current_batch: List[StreamingEvent] = []
        self.last_flush = time.time()
        
    def process_batch(self, events: List[StreamingEvent]) -> None:
        """Process batch of events"""
        self.current_batch.extend(events)
        
        if self._should_flush():
            self._process_current_batch()
            
    def _should_flush(self) -> bool:
        """Check if batch should be flushed"""
        return (
            len(self.current_batch) >= self.batch_size or
            time.time() - self.last_flush >= self.batch_timeout
        )
        
    def _process_current_batch(self) -> None:
        """Process and flush current batch"""
        try:
            for event in self.current_batch:
                super().process_event(event)
        finally:
            self.current_batch.clear()
            self.last_flush = time.time()
```

#### Windowed Processing
```python
from datetime import timedelta
from typing import Dict, Set

class WindowedProcessor(StreamProcessor):
    """Process events using time windows"""
    def __init__(
        self,
        name: str,
        window_size: timedelta,
        slide_interval: timedelta
    ):
        super().__init__(name)
        self.window_size = window_size
        self.slide_interval = slide_interval
        self.windows: Dict[datetime, Set[StreamingEvent]] = {}
        
    def process_event(self, event: StreamingEvent) -> None:
        """Process event and assign to windows"""
        window_start = self._get_window_start(event.timestamp)
        
        if window_start not in self.windows:
            self.windows[window_start] = set()
        
        self.windows[window_start].add(event)
        self._process_expired_windows(event.timestamp)
        
    def _get_window_start(self, timestamp: datetime) -> datetime:
        """Calculate window start time"""
        return timestamp - (
            timestamp - datetime.min
        ) % self.slide_interval
        
    def _process_expired_windows(self, current_time: datetime) -> None:
        """Process and remove expired windows"""
        expired = [
            start for start, events in self.windows.items()
            if start + self.window_size < current_time
        ]
        
        for window_start in expired:
            self._process_window(window_start, self.windows[window_start])
            del self.windows[window_start]
            
    def _process_window(
        self,
        window_start: datetime,
        events: Set[StreamingEvent]
    ) -> None:
        """Process events in a window"""
        # Implement window processing logic
        pass
```

## Event-Driven Architecture

### 1. Core Components

Event-driven architecture forms the backbone of streaming systems. Let's implement key components:

#### Event Producer
```python
from abc import ABC, abstractmethod
import json
from kafka import KafkaProducer

class EventProducer(ABC):
    """Abstract base class for event producers"""
    @abstractmethod
    def produce_event(self, event: StreamingEvent) -> None:
        """Produce a single event"""
        pass
        
    @abstractmethod
    def flush(self) -> None:
        """Flush any buffered events"""
        pass

class KafkaEventProducer(EventProducer):
    """Kafka-based event producer"""
    def __init__(
        self,
        bootstrap_servers: List[str],
        topic: str,
        batch_size: int = 100
    ):
        self.topic = topic
        self.producer = KafkaProducer(
            bootstrap_servers=bootstrap_servers,
            value_serializer=lambda v: json.dumps(v).encode('utf-8'),
            batch_size=batch_size,
            acks='all'
        )
        
    def produce_event(self, event: StreamingEvent) -> None:
        """Produce event to Kafka topic"""
        try:
            future = self.producer.send(
                self.topic,
                value=event.__dict__,
                timestamp_ms=int(event.timestamp.timestamp() * 1000)
            )
            # Wait for the send to complete
            future.get(timeout=10)
        except Exception as e:
            self._handle_error(e, event)
            
    def flush(self) -> None:
        """Flush producer buffer"""
        self.producer.flush()
        
    def _handle_error(self, error: Exception, event: StreamingEvent) -> None:
        """Handle producer errors"""
        # Implement error handling logic
        pass
```

#### Event Consumer
```python
from kafka import KafkaConsumer
from typing import Callable, Optional

class EventConsumer:
    """Generic event consumer"""
    def __init__(
        self,
        bootstrap_servers: List[str],
        topic: str,
        group_id: str,
        processor: StreamProcessor,
        error_handler: Optional[Callable] = None
    ):
        self.consumer = KafkaConsumer(
            topic,
            bootstrap_servers=bootstrap_servers,
            group_id=group_id,
            value_deserializer=lambda m: json.loads(m.decode('utf-8')),
            enable_auto_commit=False,
            auto_offset_reset='earliest'
        )
        self.processor = processor
        self.error_handler = error_handler
        
    def start_consuming(self) -> None:
        """Start consuming events"""
        try:
            for message in self.consumer:
                try:
                    event = self._parse_event(message.value)
                    self.processor.process_event(event)
                    self._commit_offset(message)
                except Exception as e:
                    if self.error_handler:
                        self.error_handler(e, message)
                    else:
                        raise
        finally:
            self._cleanup()
            
    def _parse_event(self, value: Dict) -> StreamingEvent:
        """Parse message value into StreamingEvent"""
        return StreamingEvent(**value)
        
    def _commit_offset(self, message) -> None:
        """Commit offset after successful processing"""
        self.consumer.commit()
        
    def _cleanup(self) -> None:
        """Cleanup resources"""
        self.consumer.close()
```

### 2. Advanced Patterns

Let's implement advanced streaming patterns:

#### Back-pressure Handling
```python
import asyncio
from asyncio import Queue
from typing import AsyncIterator

class BackPressureHandler:
    """Handle back-pressure in streaming systems"""
    def __init__(
        self,
        max_queue_size: int,
        processing_time: float = 0.1
    ):
        self.queue = Queue(maxsize=max_queue_size)
        self.processing_time = processing_time
        
    async def produce(self, event: StreamingEvent) -> None:
        """Produce event with back-pressure"""
        try:
            await self.queue.put(event)
        except asyncio.QueueFull:
            await self._handle_backpressure()
            await self.queue.put(event)
            
    async def consume(self) -> AsyncIterator[StreamingEvent]:
        """Consume events with controlled rate"""
        while True:
            event = await self.queue.get()
            yield event
            await asyncio.sleep(self.processing_time)
            self.queue.task_done()
            
    async def _handle_backpressure(self) -> None:
        """Handle back-pressure condition"""
        # Implement back-pressure strategies
        await asyncio.sleep(1)  # Simple delay strategy
```

#### Exactly-Once Processing
```python
from typing import Set

class ExactlyOnceProcessor(StreamProcessor):
    """Ensure exactly-once processing semantics"""
    def __init__(self, name: str):
        super().__init__(name)
        self.processed_ids: Set[str] = set()
        
    def process_event(self, event: StreamingEvent) -> None:
        """Process event exactly once"""
        if event.event_id in self.processed_ids:
            return  # Skip duplicate event
            
        try:
            super().process_event(event)
            self.processed_ids.add(event.event_id)
        except Exception as e:
            # Handle error but don't mark as processed
            self._handle_error(e, event)
```

## Best Practices and Patterns

### 1. Error Handling
```python
from typing import Optional
import logging

class ErrorHandler:
    """Sophisticated error handling for streaming systems"""
    def __init__(
        self,
        max_retries: int = 3,
        dead_letter_queue: Optional[EventProducer] = None
    ):
        self.max_retries = max_retries
        self.dead_letter_queue = dead_letter_queue
        self.logger = logging.getLogger(__name__)
        
    def handle_error(
        self,
        error: Exception,
        event: StreamingEvent,
        retry_count: int = 0
    ) -> None:
        """Handle processing errors"""
        self.logger.error(
            f"Error processing event {event.event_id}: {str(error)}"
        )
        
        if retry_count < self.max_retries:
            self._retry_processing(event, retry_count + 1)
        else:
            self._send_to_dead_letter_queue(event, error)
            
    def _retry_processing(
        self,
        event: StreamingEvent,
        retry_count: int
    ) -> None:
        """Retry event processing"""
        # Implement retry logic with exponential backoff
        pass
        
    def _send_to_dead_letter_queue(
        self,
        event: StreamingEvent,
        error: Exception
    ) -> None:
        """Send failed event to dead letter queue"""
        if self.dead_letter_queue:
            self.dead_letter_queue.produce_event(event)
```

### 2. Monitoring and Metrics
```python
from dataclasses import dataclass
from typing import Dict
import time

@dataclass
class StreamingMetrics:
    """Metrics for streaming system monitoring"""
    events_processed: int = 0
    events_failed: int = 0
    processing_time_ms: float = 0.0
    last_event_timestamp: Optional[datetime] = None
    
class MetricsCollector:
    """Collect and report streaming metrics"""
    def __init__(self):
        self.metrics: Dict[str, StreamingMetrics] = {}
        
    def record_event_processed(
        self,
        processor_name: str,
        processing_time_ms: float,
        event: StreamingEvent
    ) -> None:
        """Record successful event processing"""
        if processor_name not in self.metrics:
            self.metrics[processor_name] = StreamingMetrics()
            
        metrics = self.metrics[processor_name]
        metrics.events_processed += 1
        metrics.processing_time_ms += processing_time_ms
        metrics.last_event_timestamp = event.timestamp
        
    def record_event_failed(
        self,
        processor_name: str,
        event: StreamingEvent
    ) -> None:
        """Record failed event processing"""
        if processor_name not in self.metrics:
            self.metrics[processor_name] = StreamingMetrics()
            
        self.metrics[processor_name].events_failed += 1
        
    def get_metrics(self, processor_name: str) -> StreamingMetrics:
        """Get metrics for specific processor"""
        return self.metrics.get(processor_name, StreamingMetrics())
```

## Practical Exercises

### Exercise 1: Building a Real-Time Analytics Pipeline
```python
from typing import Dict, List
import json

class AnalyticsPipeline:
    """Real-time analytics pipeline implementation"""
    def __init__(
        self,
        window_size: timedelta = timedelta(minutes=5)
    ):
        self.window_size = window_size
        self.metrics: Dict[str, int] = {}
        self.window_start = datetime.now()
        
    def process_event(self, event: Dict[str, Any]) -> None:
        """Process event for analytics"""
        current_time = datetime.now()
        
        # Check if window has expired
        if current_time - self.window_start >= self.window_size:
            self._emit_window_results()
            self._reset_window()
            
        # Update metrics
        event_type = event.get('type', 'unknown')
        self.metrics[event_type] = self.metrics.get(event_type, 0) + 1
        
    def _emit_window_results(self) -> None:
        """Emit window results"""
        results = {
            'window_start': self.window_start.isoformat(),
            'window_end': datetime.now().isoformat(),
            'metrics': self.metrics
        }
        print(f"Window Results: {json.dumps(results, indent=2)}")
        
    def _reset_window(self) -> None:
        """Reset window state"""
        self.metrics.clear()
        self.window_start = datetime.now()

# Usage example
pipeline = AnalyticsPipeline()

# Simulate events
events = [
    {'type': 'pageview', 'user_id': '123', 'page': '/home'},
    {'type': 'click', 'user_id': '123', 'element': 'button'},
    {'type': 'pageview', 'user_id': '456', 'page': '/products'}
]

for event in events:
    pipeline.process_event(event)
```

### Exercise 2: Implementing a Streaming ETL Pipeline
```python
from abc import ABC, abstractmethod
from typing import Any, Dict, List, Optional
import json

class StreamingETL:
    """Streaming ETL pipeline implementation"""
    def __init__(
        self,
        extractors: List['DataExtractor'],
        transformers: List['DataTransformer'],
        loaders: List['DataLoader']
    ):
        self.extractors = extractors
        self.transformers = transformers
        self.loaders = loaders
        
    async def process_stream(self, stream: AsyncIterator[bytes]) -> None:
        """Process streaming data"""
        async for raw_data in stream:
            # Extract
            data = await self._extract(raw_data)
            if not data:
                continue
                
            # Transform
            transformed_data = await self._transform(data)
            if not transformed_data:
                continue
                
            # Load
            await self._load(transformed_data)
            
    async def _extract(self, raw_data: bytes) -> Optional[Dict[str, Any]]:
        """Extract data using configured extractors"""
        data = None
        for extractor in self.extractors:
            try:
                data = await extractor.extract(raw_data)
                if data:
                    break
            except Exception as e:
                print(f"Extraction error: {str(e)}")
        return data
        
    async def _transform(
        self,
        data: Dict[str, Any]
    ) -> Optional[Dict[str, Any]]:
        """Transform data using configured transformers"""
        transformed_data = data
        for transformer in self.transformers:
            try:
                transformed_data = await transformer.transform(transformed_data)
                if not transformed_data:
                    break
            except Exception as e:
                print(f"Transformation error: {str(e)}")
        return transformed_data
        
    async def _load(self, data: Dict[str, Any]) -> None:
        """Load data using configured loaders"""
        for loader in self.loaders:
            try:
                await loader.load(data)
            except Exception as e:
                print(f"Loading error: {str(e)}")

class DataExtractor(ABC):
    """Abstract base class for data extractors"""
    @abstractmethod
    async def extract(self, raw_data: bytes) -> Optional[Dict[str, Any]]:
        """Extract data from raw input"""
        pass

class DataTransformer(ABC):
    """Abstract base class for data transformers"""
    @abstractmethod
    async def transform(
        self,
        data: Dict[str, Any]
    ) -> Optional[Dict[str, Any]]:
        """Transform extracted data"""
        pass

class DataLoader(ABC):
    """Abstract base class for data loaders"""
    @abstractmethod
    async def load(self, data: Dict[str, Any]) -> None:
        """Load transformed data"""
        pass
```

## Additional Resources

1. **Stream Processing Frameworks**
   - [Apache Kafka Documentation](https://kafka.apache.org/documentation/)
   - [Apache Flink Documentation](https://flink.apache.org/docs/stable/)
   - [Apache Spark Streaming Guide](https://spark.apache.org/docs/latest/streaming-programming-guide.html)

2. **Best Practices and Patterns**
   - [Streaming Systems Book](https://www.oreilly.com/library/view/streaming-systems/9781491983867/)
   - [Kafka Streams Documentation](https://kafka.apache.org/documentation/streams/)
   - [Real-Time Analytics Best Practices](https://www.confluent.io/blog/real-time-analytics-with-kafka-streams/)

3. **Advanced Topics**
   - [Event-Driven Architecture](https://www.confluent.io/blog/event-driven-architecture-implementation/)
   - [Stream Processing Design Patterns](https://www.oreilly.com/library/view/streaming-systems/9781491983867/)
   - [Exactly-Once Semantics](https://www.confluent.io/blog/exactly-once-semantics-are-possible-heres-how-apache-kafka-does-it/)

## Next Steps
- Dive into Apache Kafka fundamentals
- Explore stream processing frameworks
- Practice building streaming pipelines
- Study monitoring and observability
- Learn about advanced streaming patterns 