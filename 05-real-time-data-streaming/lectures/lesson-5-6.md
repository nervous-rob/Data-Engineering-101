# Lesson 5.6: Stream Processing Patterns

## Navigation
- [← Back to Lesson Plan](../5.6-stream-processing-patterns.md)
- [← Back to Module Overview](../README.md)

## Overview
Stream processing patterns are essential architectural approaches for building robust, scalable, and maintainable streaming applications. This lesson explores advanced patterns, implementation strategies, and best practices for handling real-time data streams effectively. We'll cover both theoretical concepts and practical implementations of various streaming patterns.

## Learning Objectives
After completing this lesson, you'll be able to:
- Implement core stream processing patterns
- Design fault-tolerant streaming architectures
- Apply advanced windowing and state management patterns
- Handle complex event processing scenarios
- Implement exactly-once processing semantics
- Scale streaming applications effectively
- Choose appropriate patterns for different use cases

## Stream Processing Core Patterns

### 1. Event Sourcing Pattern

```python
from dataclasses import dataclass
from typing import Dict, List, Optional, Any
from datetime import datetime
import json

@dataclass
class Event:
    """Base event class for event sourcing"""
    event_type: str
    data: Dict[str, Any]
    timestamp: datetime
    version: int

class EventStore:
    """Implements event sourcing pattern"""
    def __init__(self, store_name: str):
        self.store_name = store_name
        self.events: List[Event] = []
        self.current_state: Dict[str, Any] = {}
        
    def append_event(self, event: Event) -> None:
        """Append new event to the store"""
        self.events.append(event)
        self._apply_event(event)
        
    def _apply_event(self, event: Event) -> None:
        """Apply event to current state"""
        if event.event_type == "CREATE":
            self.current_state.update(event.data)
        elif event.event_type == "UPDATE":
            self.current_state.update(event.data)
        elif event.event_type == "DELETE":
            for key in event.data.keys():
                self.current_state.pop(key, None)
                
    def replay_events(self, until_version: Optional[int] = None) -> Dict[str, Any]:
        """Replay events to reconstruct state"""
        state = {}
        for event in self.events:
            if until_version and event.version > until_version:
                break
            if event.event_type == "CREATE":
                state.update(event.data)
            elif event.event_type == "UPDATE":
                state.update(event.data)
            elif event.event_type == "DELETE":
                for key in event.data.keys():
                    state.pop(key, None)
        return state
```

### 2. CQRS (Command Query Responsibility Segregation)

```python
from abc import ABC, abstractmethod
from typing import List, Dict, Any
from dataclasses import dataclass
import asyncio

@dataclass
class Command:
    """Base command class"""
    command_type: str
    data: Dict[str, Any]

@dataclass
class Query:
    """Base query class"""
    query_type: str
    parameters: Dict[str, Any]

class CommandHandler(ABC):
    """Abstract base class for command handlers"""
    @abstractmethod
    async def handle(self, command: Command) -> None:
        pass

class QueryHandler(ABC):
    """Abstract base class for query handlers"""
    @abstractmethod
    async def handle(self, query: Query) -> Any:
        pass

class StreamingCQRS:
    """Implements CQRS pattern for streaming data"""
    def __init__(self):
        self.command_handlers: Dict[str, CommandHandler] = {}
        self.query_handlers: Dict[str, QueryHandler] = {}
        self.event_store = EventStore("streaming_store")
        
    async def execute_command(self, command: Command) -> None:
        """Execute command and update write model"""
        handler = self.command_handlers.get(command.command_type)
        if not handler:
            raise ValueError(f"No handler for command type: {command.command_type}")
            
        await handler.handle(command)
        
    async def execute_query(self, query: Query) -> Any:
        """Execute query on read model"""
        handler = self.query_handlers.get(query.query_type)
        if not handler:
            raise ValueError(f"No handler for query type: {query.query_type}")
            
        return await handler.handle(query)
```

### 3. Stream-Table Join Pattern

```python
from typing import Dict, Optional
from datetime import datetime, timedelta

class StreamTableJoin:
    """Implements stream-table join pattern"""
    def __init__(self, window_size: timedelta = timedelta(minutes=5)):
        self.reference_table: Dict[str, Any] = {}
        self.stream_buffer: List[Dict[str, Any]] = []
        self.window_size = window_size
        
    def update_reference_table(self, key: str, value: Any) -> None:
        """Update reference table with new data"""
        self.reference_table[key] = {
            'value': value,
            'timestamp': datetime.now()
        }
        
    def process_stream_event(self, event: Dict[str, Any]) -> Optional[Dict[str, Any]]:
        """Process streaming event with reference data"""
        event_key = event.get('key')
        if not event_key:
            return None
            
        ref_data = self.reference_table.get(event_key)
        if not ref_data:
            # Buffer event for later processing
            self.stream_buffer.append({
                'event': event,
                'timestamp': datetime.now()
            })
            return None
            
        return self._enrich_event(event, ref_data['value'])
        
    def _enrich_event(self, event: Dict[str, Any], ref_data: Any) -> Dict[str, Any]:
        """Enrich event with reference data"""
        return {
            **event,
            'enriched_data': ref_data,
            'processing_timestamp': datetime.now()
        }
        
    def cleanup_old_events(self) -> None:
        """Remove events older than window size"""
        cutoff_time = datetime.now() - self.window_size
        self.stream_buffer = [
            buffer_item for buffer_item in self.stream_buffer
            if buffer_item['timestamp'] > cutoff_time
        ]
```

### 4. Windowing Pattern

```python
from collections import defaultdict
from datetime import datetime, timedelta
from typing import Dict, List, Any, Callable

class WindowProcessor:
    """Implements various windowing patterns"""
    def __init__(
        self,
        window_size: timedelta,
        slide_interval: Optional[timedelta] = None
    ):
        self.window_size = window_size
        self.slide_interval = slide_interval or window_size
        self.windows: Dict[datetime, List[Any]] = defaultdict(list)
        
    def process_event(
        self,
        event: Any,
        timestamp: datetime,
        aggregator: Callable[[List[Any]], Any]
    ) -> Dict[datetime, Any]:
        """Process event and return window results"""
        # Determine relevant windows for event
        window_start = self._get_window_start(timestamp)
        
        # Add event to relevant windows
        self.windows[window_start].append(event)
        
        # Process completed windows
        results = {}
        current_time = datetime.now()
        for start_time, events in list(self.windows.items()):
            if start_time + self.window_size <= current_time:
                results[start_time] = aggregator(events)
                if self.slide_interval == self.window_size:
                    del self.windows[start_time]
                    
        return results
        
    def _get_window_start(self, timestamp: datetime) -> datetime:
        """Calculate window start time for timestamp"""
        elapsed = timestamp - datetime.min
        window_seconds = self.slide_interval.total_seconds()
        windows = int(elapsed.total_seconds() // window_seconds)
        return datetime.min + timedelta(seconds=windows * window_seconds)
```

## Advanced Stream Processing Patterns

### 1. Exactly-Once Processing

```python
from typing import Dict, Set, Any
import hashlib

class ExactlyOnceProcessor:
    """Implements exactly-once processing pattern"""
    def __init__(self):
        self.processed_events: Set[str] = set()
        self.state: Dict[str, Any] = {}
        
    def process_event(self, event: Dict[str, Any]) -> Optional[Dict[str, Any]]:
        """Process event exactly once"""
        # Generate event ID
        event_id = self._generate_event_id(event)
        
        # Check if already processed
        if event_id in self.processed_events:
            return None
            
        try:
            # Process event
            result = self._process_event_logic(event)
            
            # Mark as processed
            self.processed_events.add(event_id)
            
            return result
        except Exception as e:
            # Handle processing error
            self._handle_processing_error(event_id, e)
            raise
            
    def _generate_event_id(self, event: Dict[str, Any]) -> str:
        """Generate unique event ID"""
        event_str = json.dumps(event, sort_keys=True)
        return hashlib.sha256(event_str.encode()).hexdigest()
        
    def _process_event_logic(self, event: Dict[str, Any]) -> Dict[str, Any]:
        """Implement specific event processing logic"""
        # Example processing logic
        key = event.get('key')
        value = event.get('value')
        
        if key is not None:
            self.state[key] = value
            
        return {
            'processed_event': event,
            'current_state': self.state.get(key)
        }
        
    def _handle_processing_error(self, event_id: str, error: Exception) -> None:
        """Handle processing errors"""
        # Remove from processed set if error occurs
        self.processed_events.discard(event_id)
```

### 2. Dead Letter Queue Pattern

```python
from typing import Dict, List, Any, Callable
from datetime import datetime
import logging

class DeadLetterQueue:
    """Implements dead letter queue pattern"""
    def __init__(
        self,
        max_retries: int = 3,
        retry_delay: timedelta = timedelta(minutes=5)
    ):
        self.max_retries = max_retries
        self.retry_delay = retry_delay
        self.failed_events: List[Dict[str, Any]] = []
        self.logger = logging.getLogger(__name__)
        
    def process_with_dlq(
        self,
        event: Dict[str, Any],
        processor: Callable[[Dict[str, Any]], Any]
    ) -> Optional[Any]:
        """Process event with dead letter queue handling"""
        try:
            return processor(event)
        except Exception as e:
            self._handle_failed_event(event, e)
            return None
            
    def _handle_failed_event(
        self,
        event: Dict[str, Any],
        error: Exception
    ) -> None:
        """Handle failed event processing"""
        retry_count = event.get('retry_count', 0)
        
        if retry_count < self.max_retries:
            # Add to retry queue
            self.failed_events.append({
                'event': event,
                'error': str(error),
                'retry_count': retry_count + 1,
                'next_retry': datetime.now() + self.retry_delay
            })
            self.logger.warning(
                f"Event processing failed, attempt {retry_count + 1}/{self.max_retries}"
            )
        else:
            # Log permanent failure
            self.logger.error(
                f"Event processing permanently failed after {self.max_retries} attempts"
            )
            
    def retry_failed_events(
        self,
        processor: Callable[[Dict[str, Any]], Any]
    ) -> None:
        """Retry processing of failed events"""
        current_time = datetime.now()
        remaining_events = []
        
        for failed_event in self.failed_events:
            if failed_event['next_retry'] <= current_time:
                try:
                    processor(failed_event['event'])
                except Exception as e:
                    # Update retry information
                    failed_event['error'] = str(e)
                    failed_event['next_retry'] = current_time + self.retry_delay
                    remaining_events.append(failed_event)
            else:
                remaining_events.append(failed_event)
                
        self.failed_events = remaining_events
```

## Practical Exercise: Building a Robust Stream Processing Pipeline

Implement a complete stream processing pipeline that combines multiple patterns:

```python
class StreamProcessor:
    """Complete stream processing pipeline implementation"""
    def __init__(self):
        self.event_store = EventStore("main_store")
        self.cqrs = StreamingCQRS()
        self.stream_table_join = StreamTableJoin()
        self.window_processor = WindowProcessor(
            window_size=timedelta(minutes=5),
            slide_interval=timedelta(minutes=1)
        )
        self.exactly_once = ExactlyOnceProcessor()
        self.dlq = DeadLetterQueue()
        
    async def process_event(self, event: Dict[str, Any]) -> Optional[Dict[str, Any]]:
        """Process event through the pipeline"""
        try:
            # 1. Store event
            stored_event = Event(
                event_type="PROCESS",
                data=event,
                timestamp=datetime.now(),
                version=len(self.event_store.events) + 1
            )
            self.event_store.append_event(stored_event)
            
            # 2. Apply CQRS pattern
            command = Command("PROCESS_EVENT", event)
            await self.cqrs.execute_command(command)
            
            # 3. Enrich with reference data
            enriched_event = self.stream_table_join.process_stream_event(event)
            
            # 4. Process with exactly-once semantics
            if enriched_event:
                processed_event = self.exactly_once.process_event(enriched_event)
                
                # 5. Apply windowing
                if processed_event:
                    window_results = self.window_processor.process_event(
                        processed_event,
                        datetime.now(),
                        lambda events: sum(e.get('value', 0) for e in events)
                    )
                    
                    return {
                        'processed_event': processed_event,
                        'window_results': window_results
                    }
                    
            return None
            
        except Exception as e:
            # Handle failures with DLQ pattern
            return self.dlq.process_with_dlq(event, lambda e: self.process_event(e))
```

## Best Practices

1. **Pattern Selection**
   - Choose patterns based on use case requirements
   - Consider scalability and maintenance implications
   - Evaluate pattern combinations carefully
   - Document pattern decisions and rationale

2. **Implementation Guidelines**
   - Implement proper error handling
   - Use appropriate serialization formats
   - Maintain clean separation of concerns
   - Follow idempotency principles

3. **Performance Optimization**
   - Monitor and tune window sizes
   - Optimize state management
   - Implement efficient event storage
   - Use appropriate batching strategies

4. **Testing and Monitoring**
   - Test pattern implementations thoroughly
   - Monitor pattern performance
   - Track error rates and processing times
   - Implement proper logging and alerting

## Common Pitfalls
1. Incorrect window boundary handling
2. Memory leaks in state management
3. Poor error handling strategies
4. Inefficient event storage
5. Inadequate monitoring and alerting

## Additional Resources
- [Stream Processing Design Patterns](https://www.confluent.io/blog/stream-processing-design-patterns/)
- [Event Sourcing Pattern Guide](https://martinfowler.com/eaaDev/EventSourcing.html)
- [CQRS Pattern Documentation](https://docs.microsoft.com/en-us/azure/architecture/patterns/cqrs)
- [Kafka Streams Patterns](https://kafka.apache.org/documentation/streams/)

## Next Steps
- Explore advanced streaming patterns
- Study real-world implementations
- Practice with different pattern combinations
- Learn about pattern optimization techniques 