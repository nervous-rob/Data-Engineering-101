# Lesson 5.8: Streaming System Design

## Navigation
- [← Back to Module Overview](../README.md)
- [← Previous Lesson](./lesson-5-7.md)
- [Next Module →](../../06-cloud-platforms-and-security/README.md)

## Overview
Designing robust streaming systems requires careful consideration of architecture patterns, scalability requirements, and reliability mechanisms. This lesson explores modern streaming system design approaches, with a focus on practical implementations and industry best practices for building production-grade streaming applications.

## Learning Objectives
After completing this lesson, you'll be able to:
- Design scalable streaming architectures
- Implement modern streaming patterns
- Handle system reliability and fault tolerance
- Apply performance optimization techniques
- Choose appropriate design patterns for different use cases
- Implement monitoring and observability
- Design for high availability and disaster recovery

## Modern Streaming Architecture Patterns

### 1. Event-Driven Architecture

```python
from abc import ABC, abstractmethod
from typing import Dict, List, Any
from datetime import datetime
import asyncio
import json

class EventPublisher:
    """Implements event publishing pattern"""
    def __init__(self, broker_config: Dict[str, Any]):
        self.subscribers: Dict[str, List[callable]] = {}
        self.broker_config = broker_config
        
    async def publish(self, event_type: str, event_data: Dict[str, Any]) -> None:
        """Publish event to all subscribers"""
        event = {
            'type': event_type,
            'data': event_data,
            'timestamp': datetime.now().isoformat(),
            'id': self._generate_event_id(event_data)
        }
        
        if event_type in self.subscribers:
            await asyncio.gather(
                *[
                    subscriber(event)
                    for subscriber in self.subscribers[event_type]
                ]
            )
            
    def subscribe(self, event_type: str, handler: callable) -> None:
        """Subscribe to event type"""
        if event_type not in self.subscribers:
            self.subscribers[event_type] = []
        self.subscribers[event_type].append(handler)
        
    def _generate_event_id(self, event_data: Dict[str, Any]) -> str:
        """Generate unique event ID"""
        return f"{hash(json.dumps(event_data, sort_keys=True))}-{datetime.now().timestamp()}"

class EventHandler(ABC):
    """Abstract base class for event handlers"""
    @abstractmethod
    async def handle(self, event: Dict[str, Any]) -> None:
        """Handle event"""
        pass

class OrderProcessor(EventHandler):
    """Example event handler for order processing"""
    async def handle(self, event: Dict[str, Any]) -> None:
        """Process order event"""
        order_data = event['data']
        # Process order logic
        print(f"Processing order: {order_data['order_id']}")
        
class InventoryManager(EventHandler):
    """Example event handler for inventory management"""
    async def handle(self, event: Dict[str, Any]) -> None:
        """Update inventory"""
        order_data = event['data']
        # Update inventory logic
        print(f"Updating inventory for order: {order_data['order_id']}")
```

### 2. Scalable Stream Processing

```python
from typing import Dict, List, Any, Optional
from datetime import datetime, timedelta
import asyncio
import json

class StreamProcessor:
    """Implements scalable stream processing pattern"""
    def __init__(
        self,
        partition_count: int,
        batch_size: int = 1000,
        processing_window: timedelta = timedelta(minutes=1)
    ):
        self.partition_count = partition_count
        self.batch_size = batch_size
        self.processing_window = processing_window
        self.partitions: Dict[int, List[Dict[str, Any]]] = {
            i: [] for i in range(partition_count)
        }
        self.processors: Dict[int, asyncio.Task] = {}
        
    async def process_event(self, event: Dict[str, Any]) -> None:
        """Process incoming event"""
        partition = self._get_partition(event)
        self.partitions[partition].append(event)
        
        # Start processor if batch size reached
        if len(self.partitions[partition]) >= self.batch_size:
            await self._process_partition(partition)
            
    async def _process_partition(self, partition: int) -> None:
        """Process events in partition"""
        if partition in self.processors and not self.processors[partition].done():
            return
            
        events = self.partitions[partition]
        self.partitions[partition] = []
        
        # Create processing task
        self.processors[partition] = asyncio.create_task(
            self._process_batch(events)
        )
        
    async def _process_batch(self, events: List[Dict[str, Any]]) -> None:
        """Process batch of events"""
        try:
            # Group events by type
            event_groups = self._group_events(events)
            
            # Process each group
            for event_type, group_events in event_groups.items():
                await self._process_event_group(event_type, group_events)
                
        except Exception as e:
            print(f"Error processing batch: {str(e)}")
            # Handle error (e.g., dead letter queue)
            
    def _get_partition(self, event: Dict[str, Any]) -> int:
        """Get partition for event"""
        # Simple hash-based partitioning
        return hash(event['id']) % self.partition_count
        
    def _group_events(
        self,
        events: List[Dict[str, Any]]
    ) -> Dict[str, List[Dict[str, Any]]]:
        """Group events by type"""
        groups: Dict[str, List[Dict[str, Any]]] = {}
        for event in events:
            event_type = event['type']
            if event_type not in groups:
                groups[event_type] = []
            groups[event_type].append(event)
        return groups
        
    async def _process_event_group(
        self,
        event_type: str,
        events: List[Dict[str, Any]]
    ) -> None:
        """Process group of events of same type"""
        # Implement specific processing logic
        print(f"Processing {len(events)} events of type {event_type}")
```

### 3. Fault-Tolerant Design

```python
from typing import Dict, List, Any, Optional
from datetime import datetime
import asyncio
import json

class FaultTolerantProcessor:
    """Implements fault-tolerant processing pattern"""
    def __init__(
        self,
        max_retries: int = 3,
        retry_delay: float = 1.0,
        circuit_breaker_threshold: int = 5
    ):
        self.max_retries = max_retries
        self.retry_delay = retry_delay
        self.circuit_breaker_threshold = circuit_breaker_threshold
        self.failure_counts: Dict[str, int] = {}
        self.circuit_state: Dict[str, bool] = {}  # True = Open
        
    async def process_with_retries(
        self,
        processor_func: callable,
        event: Dict[str, Any]
    ) -> Optional[Any]:
        """Process event with retry logic"""
        service = self._get_service_name(event)
        
        if self._is_circuit_open(service):
            raise Exception(f"Circuit breaker open for service: {service}")
            
        retries = 0
        while retries < self.max_retries:
            try:
                result = await processor_func(event)
                self._reset_failure_count(service)
                return result
            except Exception as e:
                retries += 1
                if retries == self.max_retries:
                    self._increment_failure_count(service)
                    raise
                await asyncio.sleep(self.retry_delay * (2 ** retries))
                
    def _get_service_name(self, event: Dict[str, Any]) -> str:
        """Extract service name from event"""
        return event.get('service', 'default')
        
    def _increment_failure_count(self, service: str) -> None:
        """Increment failure count and check circuit breaker"""
        self.failure_counts[service] = self.failure_counts.get(service, 0) + 1
        
        if self.failure_counts[service] >= self.circuit_breaker_threshold:
            self.circuit_state[service] = True  # Open circuit
            
    def _reset_failure_count(self, service: str) -> None:
        """Reset failure count for service"""
        self.failure_counts[service] = 0
        self.circuit_state[service] = False  # Close circuit
        
    def _is_circuit_open(self, service: str) -> bool:
        """Check if circuit breaker is open"""
        return self.circuit_state.get(service, False)
```

### 4. High-Availability Design

```python
from typing import Dict, List, Any, Optional
from datetime import datetime
import asyncio
import json

class HighAvailabilityManager:
    """Implements high-availability pattern"""
    def __init__(self, node_count: int = 3):
        self.node_count = node_count
        self.nodes: Dict[int, Dict[str, Any]] = {}
        self.leader: Optional[int] = None
        self.state: Dict[str, Any] = {}
        
    async def initialize_cluster(self) -> None:
        """Initialize cluster nodes"""
        for i in range(self.node_count):
            self.nodes[i] = {
                'id': i,
                'status': 'active',
                'last_heartbeat': datetime.now(),
                'state': {}
            }
        await self._elect_leader()
        
    async def _elect_leader(self) -> None:
        """Elect cluster leader"""
        active_nodes = [
            node_id for node_id, node in self.nodes.items()
            if node['status'] == 'active'
        ]
        
        if active_nodes:
            self.leader = min(active_nodes)
            print(f"Node {self.leader} elected as leader")
            
    async def handle_node_failure(self, node_id: int) -> None:
        """Handle node failure"""
        if node_id in self.nodes:
            self.nodes[node_id]['status'] = 'failed'
            
            # Re-elect leader if needed
            if node_id == self.leader:
                await self._elect_leader()
                
            # Redistribute workload
            await self._redistribute_workload(node_id)
            
    async def _redistribute_workload(self, failed_node: int) -> None:
        """Redistribute workload from failed node"""
        workload = self.nodes[failed_node]['state']
        active_nodes = [
            node_id for node_id, node in self.nodes.items()
            if node['status'] == 'active' and node_id != failed_node
        ]
        
        if not active_nodes:
            raise Exception("No active nodes available")
            
        # Distribute workload
        for i, (key, value) in enumerate(workload.items()):
            target_node = active_nodes[i % len(active_nodes)]
            self.nodes[target_node]['state'][key] = value
```

## Best Practices

1. **Architecture Design**
   - Use event-driven patterns for loose coupling
   - Implement proper partitioning strategies
   - Design for horizontal scalability
   - Consider state management carefully

2. **Scalability**
   - Use appropriate partitioning schemes
   - Implement backpressure mechanisms
   - Design for horizontal scaling
   - Consider resource allocation

3. **Reliability**
   - Implement proper error handling
   - Use circuit breakers for external services
   - Design for fault tolerance
   - Implement proper monitoring

4. **Performance**
   - Optimize batch processing
   - Use appropriate window sizes
   - Implement caching strategies
   - Monitor and tune performance

## Common Pitfalls
1. Inadequate error handling
2. Poor scalability design
3. Missing monitoring
4. Inefficient resource usage
5. Incomplete failure recovery

## Additional Resources
- [Modern Streaming Architectures](https://www.softkraft.co/streaming-data-architecture/)
- [Event-Driven Design Patterns](https://www.confluent.io/blog/event-driven-architecture-patterns/)
- [Scalability Best Practices](https://aws.amazon.com/blogs/big-data/best-practices-for-scaling-apache-kafka/)
- [Fault Tolerance in Streaming](https://www.oreilly.com/library/view/streaming-systems/9781491983867/)

## Next Steps
- Study advanced streaming patterns
- Explore cloud-native streaming
- Practice with real-world scenarios
- Learn about stream processing frameworks 