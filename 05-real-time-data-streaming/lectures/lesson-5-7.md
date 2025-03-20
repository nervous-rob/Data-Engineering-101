# Lesson 5.7: Integration and Testing

## Navigation
- [← Back to Lesson Plan](../5.7-integration-and-testing.md)
- [← Back to Module Overview](../README.md)

## Overview
Testing and monitoring streaming applications presents unique challenges due to their real-time nature and complex state management requirements. This lesson covers comprehensive strategies for testing, monitoring, and debugging streaming applications, with a focus on practical implementations and industry best practices.

## Learning Objectives
After completing this lesson, you'll be able to:
- Design and implement comprehensive testing strategies for streaming applications
- Set up effective monitoring and alerting systems
- Debug complex streaming applications
- Implement integration tests for streaming components
- Use modern testing tools and frameworks
- Apply best practices for streaming system observability
- Handle common testing and monitoring challenges

## Testing Streaming Applications

### 1. Unit Testing Framework

```python
import pytest
from typing import Dict, List, Any
from datetime import datetime
import json

class StreamingTestFramework:
    """Framework for testing streaming components"""
    def __init__(self):
        self.test_events: List[Dict[str, Any]] = []
        self.processed_events: List[Dict[str, Any]] = []
        
    def generate_test_events(self, count: int) -> List[Dict[str, Any]]:
        """Generate test events for streaming tests"""
        return [
            {
                'event_id': f'test-event-{i}',
                'timestamp': datetime.now().isoformat(),
                'data': {
                    'value': i,
                    'type': 'test',
                    'metadata': {'test_run': True}
                }
            }
            for i in range(count)
        ]
        
    def mock_stream_processor(self, processor_func):
        """Decorator for mocking stream processing functions"""
        def wrapper(*args, **kwargs):
            # Track function calls
            self.test_events.extend(args[0] if args else [])
            
            # Process events
            result = processor_func(*args, **kwargs)
            
            # Track results
            if result:
                self.processed_events.extend(
                    [result] if isinstance(result, dict) else result
                )
            
            return result
        return wrapper
    
    def assert_processing_results(
        self,
        expected_count: int,
        condition_func=None
    ) -> None:
        """Assert processing results meet expectations"""
        assert len(self.processed_events) == expected_count, \
            f"Expected {expected_count} events, got {len(self.processed_events)}"
            
        if condition_func:
            assert all(condition_func(event) for event in self.processed_events), \
                "Not all events meet the expected condition"
                
    def reset(self) -> None:
        """Reset test state"""
        self.test_events.clear()
        self.processed_events.clear()
```

### 2. Integration Testing

```python
from kafka import KafkaProducer, KafkaConsumer
import asyncio
from typing import List, Dict, Any, Optional
import json

class StreamingIntegrationTest:
    """Integration testing for streaming components"""
    def __init__(
        self,
        bootstrap_servers: List[str],
        test_topic: str
    ):
        self.bootstrap_servers = bootstrap_servers
        self.test_topic = test_topic
        self.producer = self._create_producer()
        self.consumer = self._create_consumer()
        
    def _create_producer(self) -> KafkaProducer:
        """Create test producer"""
        return KafkaProducer(
            bootstrap_servers=self.bootstrap_servers,
            value_serializer=lambda v: json.dumps(v).encode('utf-8'),
            acks='all'  # Ensure durability for testing
        )
        
    def _create_consumer(self) -> KafkaConsumer:
        """Create test consumer"""
        return KafkaConsumer(
            self.test_topic,
            bootstrap_servers=self.bootstrap_servers,
            group_id='test-group',
            auto_offset_reset='earliest',
            enable_auto_commit=False,  # Manual commit for testing
            value_deserializer=lambda v: json.loads(v.decode('utf-8'))
        )
        
    async def test_end_to_end_flow(
        self,
        test_events: List[Dict[str, Any]],
        timeout_seconds: int = 30
    ) -> bool:
        """Test end-to-end flow with timeout"""
        # Send test events
        for event in test_events:
            self.producer.send(self.test_topic, event)
        self.producer.flush()
        
        # Collect results with timeout
        received_events = []
        start_time = datetime.now()
        
        while len(received_events) < len(test_events):
            if (datetime.now() - start_time).seconds > timeout_seconds:
                raise TimeoutError(
                    f"Timeout waiting for events. "
                    f"Received {len(received_events)} of {len(test_events)}"
                )
                
            messages = self.consumer.poll(timeout_ms=1000)
            for message_set in messages.values():
                for message in message_set:
                    received_events.append(message.value)
                    
        return self._verify_events(test_events, received_events)
        
    def _verify_events(
        self,
        sent_events: List[Dict[str, Any]],
        received_events: List[Dict[str, Any]]
    ) -> bool:
        """Verify received events match sent events"""
        if len(sent_events) != len(received_events):
            return False
            
        # Sort events by ID for comparison
        sent_sorted = sorted(sent_events, key=lambda x: x['event_id'])
        received_sorted = sorted(received_events, key=lambda x: x['event_id'])
        
        return all(
            self._compare_events(sent, received)
            for sent, received in zip(sent_sorted, received_sorted)
        )
        
    def _compare_events(
        self,
        event1: Dict[str, Any],
        event2: Dict[str, Any]
    ) -> bool:
        """Compare two events for equality"""
        return (
            event1['event_id'] == event2['event_id'] and
            event1['data'] == event2['data']
        )
        
    def cleanup(self) -> None:
        """Cleanup test resources"""
        self.producer.close()
        self.consumer.close()
```

### 3. Performance Testing

```python
from dataclasses import dataclass
from typing import Dict, List, Any, Optional
from datetime import datetime, timedelta
import time
import statistics

@dataclass
class PerformanceMetrics:
    """Container for performance test metrics"""
    throughput: float  # events/second
    latency_avg: float  # milliseconds
    latency_p95: float  # milliseconds
    latency_p99: float  # milliseconds
    error_rate: float  # percentage

class StreamingPerformanceTest:
    """Performance testing for streaming applications"""
    def __init__(self):
        self.latencies: List[float] = []
        self.errors: int = 0
        self.total_events: int = 0
        self.start_time: Optional[datetime] = None
        
    def start_test(self) -> None:
        """Start performance test"""
        self.start_time = datetime.now()
        
    def record_event(self, latency_ms: float, success: bool) -> None:
        """Record test event metrics"""
        self.total_events += 1
        if success:
            self.latencies.append(latency_ms)
        else:
            self.errors += 1
            
    def calculate_metrics(self) -> PerformanceMetrics:
        """Calculate performance metrics"""
        if not self.start_time:
            raise ValueError("Test not started")
            
        test_duration = (datetime.now() - self.start_time).total_seconds()
        
        # Calculate metrics
        throughput = self.total_events / test_duration
        latency_avg = statistics.mean(self.latencies) if self.latencies else 0
        latency_p95 = statistics.quantiles(self.latencies, n=20)[-1] if self.latencies else 0
        latency_p99 = statistics.quantiles(self.latencies, n=100)[-1] if self.latencies else 0
        error_rate = (self.errors / self.total_events) * 100 if self.total_events > 0 else 0
        
        return PerformanceMetrics(
            throughput=throughput,
            latency_avg=latency_avg,
            latency_p95=latency_p95,
            latency_p99=latency_p99,
            error_rate=error_rate
        )
```

## Monitoring and Observability

### 1. Metrics Collection

```python
from prometheus_client import Counter, Gauge, Histogram, start_http_server
import time
from typing import Dict, Any, Optional
from datetime import datetime

class StreamingMetricsCollector:
    """Collects and exposes streaming metrics"""
    def __init__(self, app_name: str, port: int = 8000):
        # Initialize metrics
        self.messages_processed = Counter(
            'messages_processed_total',
            'Total number of messages processed',
            ['app_name', 'status']
        )
        
        self.processing_latency = Histogram(
            'message_processing_latency_seconds',
            'Message processing latency',
            ['app_name'],
            buckets=(0.1, 0.5, 1.0, 2.0, 5.0)
        )
        
        self.consumer_lag = Gauge(
            'consumer_lag_messages',
            'Consumer lag in number of messages',
            ['app_name', 'topic', 'partition']
        )
        
        self.app_name = app_name
        
        # Start metrics server
        start_http_server(port)
        
    def record_message_processed(
        self,
        success: bool,
        processing_time: float
    ) -> None:
        """Record processed message metrics"""
        status = 'success' if success else 'error'
        self.messages_processed.labels(
            app_name=self.app_name,
            status=status
        ).inc()
        
        if success:
            self.processing_latency.labels(
                app_name=self.app_name
            ).observe(processing_time)
            
    def update_consumer_lag(
        self,
        topic: str,
        partition: int,
        lag: int
    ) -> None:
        """Update consumer lag metric"""
        self.consumer_lag.labels(
            app_name=self.app_name,
            topic=topic,
            partition=partition
        ).set(lag)
```

### 2. Logging and Tracing

```python
import logging
import json
from datetime import datetime
from typing import Dict, Any, Optional
import traceback

class StreamingLogger:
    """Enhanced logging for streaming applications"""
    def __init__(
        self,
        app_name: str,
        log_level: int = logging.INFO
    ):
        self.logger = logging.getLogger(app_name)
        self.logger.setLevel(log_level)
        
        # Add JSON formatter
        handler = logging.StreamHandler()
        handler.setFormatter(self._create_json_formatter())
        self.logger.addHandler(handler)
        
    def _create_json_formatter(self):
        """Create JSON formatter for structured logging"""
        class JsonFormatter(logging.Formatter):
            def format(self, record):
                log_entry = {
                    'timestamp': datetime.now().isoformat(),
                    'level': record.levelname,
                    'message': record.getMessage(),
                    'logger': record.name
                }
                
                if hasattr(record, 'event_id'):
                    log_entry['event_id'] = record.event_id
                    
                if record.exc_info:
                    log_entry['exception'] = {
                        'type': record.exc_info[0].__name__,
                        'message': str(record.exc_info[1]),
                        'stacktrace': traceback.format_exception(*record.exc_info)
                    }
                    
                return json.dumps(log_entry)
                
        return JsonFormatter()
        
    def log_event_processing(
        self,
        event: Dict[str, Any],
        status: str,
        processing_time: float,
        error: Optional[Exception] = None
    ) -> None:
        """Log event processing details"""
        log_data = {
            'event_id': event.get('event_id', 'unknown'),
            'status': status,
            'processing_time_ms': processing_time * 1000
        }
        
        if error:
            self.logger.error(
                f"Event processing failed",
                extra={'event_data': log_data},
                exc_info=error
            )
        else:
            self.logger.info(
                f"Event processed successfully",
                extra={'event_data': log_data}
            )
```

## Best Practices

1. **Testing Strategy**
   - Write comprehensive unit tests for stream processing logic
   - Implement integration tests for end-to-end flows
   - Conduct performance testing under realistic conditions
   - Test failure scenarios and recovery mechanisms

2. **Monitoring Setup**
   - Implement detailed metrics collection
   - Set up alerting for critical conditions
   - Use structured logging for better observability
   - Monitor system resources and performance

3. **Debugging Approach**
   - Implement detailed logging at key points
   - Use distributed tracing for complex flows
   - Maintain test environments for debugging
   - Document common failure patterns

4. **Performance Optimization**
   - Regular performance testing
   - Monitor and optimize resource usage
   - Implement proper error handling
   - Use appropriate batching strategies

## Common Pitfalls
1. Insufficient test coverage
2. Poor error handling in tests
3. Inadequate monitoring setup
4. Missing performance baselines
5. Incomplete integration tests

## Additional Resources
- [Kafka Testing Best Practices](https://www.confluent.io/blog/testing-kafka-streams/)
- [Streaming Systems Testing Guide](https://www.oreilly.com/library/view/streaming-systems/9781491983867/)
- [Monitoring Distributed Systems](https://sre.google/sre-book/monitoring-distributed-systems/)
- [Performance Testing Strategies](https://www.databricks.com/blog/2022/12/12/streaming-production-collected-best-practices)

## Next Steps
- Explore advanced testing patterns
- Study monitoring best practices
- Practice debugging techniques
- Learn about performance optimization 