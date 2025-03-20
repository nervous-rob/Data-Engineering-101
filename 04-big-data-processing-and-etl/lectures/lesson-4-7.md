# Lesson 4.7: Advanced Airflow Concepts

## Navigation
- [← Back to Lesson Plan](../4.7-advanced-airflow-concepts.md)
- [← Back to Module Overview](../README.md)

## Overview
Ready to take your Airflow skills to the next level? In this lesson, we'll explore advanced concepts that will help you build more sophisticated, efficient, and maintainable data pipelines. We'll dive into custom components, dynamic task generation, advanced monitoring, and complex workflow patterns. These advanced features will give you the tools to tackle even the most challenging workflow orchestration needs.

## Learning Objectives
After completing this lesson, you'll be able to:
- Build custom operators and hooks for specialized tasks
- Generate tasks dynamically based on runtime conditions
- Implement advanced monitoring and alerting systems
- Design complex workflow patterns
- Optimize Airflow performance at scale
- Handle errors and recovery with sophistication

## Building Custom Components

### 1. Creating Powerful Custom Operators
Let's build an operator that handles data processing with enhanced features:

```python
from typing import Any, Dict, Optional
from airflow.models.baseoperator import BaseOperator
from airflow.utils.decorators import apply_defaults
from airflow.hooks.base import BaseHook

class EnhancedDataOperator(BaseOperator):
    """
    A sophisticated operator with advanced error handling and logging
    """
    # Fields that support templating
    template_fields = ['sql_query', 'target_table']
    template_ext = ['.sql']
    ui_color = '#e8f7e4'  # Custom UI color

    @apply_defaults
    def __init__(
        self,
        conn_id: str,
        sql_query: str,
        target_table: str,
        batch_size: int = 1000,  # Process data in batches
        retry_delay: int = 300,
        *args, **kwargs
    ) -> None:
        super().__init__(*args, **kwargs)
        self.conn_id = conn_id
        self.sql_query = sql_query
        self.target_table = target_table
        self.batch_size = batch_size
        self.retry_delay = retry_delay
        
    def execute(self, context: Dict[str, Any]) -> Any:
        self.log.info(f"Starting execution with batch size: {self.batch_size}")
        
        try:
            # Get database connection
            hook = BaseHook.get_hook(self.conn_id)
            connection = hook.get_conn()
            
            # Process data in batches
            with connection.cursor() as cursor:
                cursor.execute(self.sql_query)
                while True:
                    batch = cursor.fetchmany(self.batch_size)
                    if not batch:
                        break
                    
                    self._process_batch(batch, connection)
                    
            self.log.info("Successfully completed processing")
            return True
            
        except Exception as e:
            self.log.error(f"Error during execution: {str(e)}")
            raise
            
    def _process_batch(self, batch: list, connection: Any) -> None:
        """Process a batch of records with transaction handling"""
        try:
            with connection.cursor() as cursor:
                cursor.executemany(
                    f"INSERT INTO {self.target_table} VALUES (%s, %s, %s)",
                    batch
                )
                connection.commit()
                
        except Exception as e:
            connection.rollback()
            raise
```

### 2. Building Smart Database Hooks
Create hooks that handle connection pooling and retry logic:

```python
from typing import Any, Dict, Optional
from airflow.hooks.base import BaseHook

class EnhancedDataHook(BaseHook):
    """
    Smart hook with connection pooling and retry logic
    """
    def __init__(
        self,
        conn_id: str,
        pool_size: int = 5,
        *args, **kwargs
    ) -> None:
        super().__init__(*args, **kwargs)
        self.conn_id = conn_id
        self.pool_size = pool_size
        self._pool = None
        
    def get_conn(self) -> Any:
        """Get connection from pool"""
        if self._pool is None:
            self._setup_connection_pool()
        return self._pool.getconn()
        
    def _setup_connection_pool(self) -> None:
        """Initialize connection pool for better performance"""
        import psycopg2
        from psycopg2 import pool
        
        conn_params = self.get_connection(self.conn_id)
        
        self._pool = psycopg2.pool.SimpleConnectionPool(
            1, self.pool_size,
            host=conn_params.host,
            database=conn_params.schema,
            user=conn_params.login,
            password=conn_params.password,
            port=conn_params.port
        )
```

## Dynamic Task Generation

### 1. Task Mapping: Dynamic Workflows
Create tasks based on runtime conditions:

```python
from airflow.decorators import task, dag
from datetime import datetime
from typing import List

@dag(
    start_date=datetime(2024, 1, 1),
    schedule_interval='@daily',
    catchup=False
)
def dynamic_etl_pipeline():
    
    @task
    def get_table_list() -> List[str]:
        """Dynamically discover tables to process"""
        return ['users', 'orders', 'products']
    
    @task
    def process_table(table_name: str) -> dict:
        """Process each table independently"""
        return {
            'table': table_name,
            'records_processed': 1000,
            'status': 'success'
        }
    
    @task
    def aggregate_results(results: List[dict]) -> dict:
        """Combine results from all table processing"""
        total_records = sum(r['records_processed'] for r in results)
        return {
            'total_tables': len(results),
            'total_records': total_records,
            'status': 'complete'
        }
    
    # Create dynamic workflow
    tables = get_table_list()
    process_results = process_table.expand(table_name=tables)
    final_result = aggregate_results(process_results)

dynamic_etl_dag = dynamic_etl_pipeline()
```

### 2. Smart Branching: Conditional Workflows
Create workflows that adapt to conditions:

```python
from airflow.operators.python import BranchPythonOperator
from airflow.operators.dummy import DummyOperator

def branch_func(**context):
    """Smart branching logic"""
    execution_date = context['execution_date']
    if execution_date.day == 1:
        return 'monthly_processing'  # First day of month
    elif execution_date.weekday() == 0:
        return 'weekly_processing'   # Monday
    else:
        return 'daily_processing'    # Regular day

branch_task = BranchPythonOperator(
    task_id='branch_task',
    python_callable=branch_func,
    provide_context=True
)

monthly_task = DummyOperator(task_id='monthly_processing')
weekly_task = DummyOperator(task_id='weekly_processing')
daily_task = DummyOperator(task_id='daily_processing')

branch_task >> [monthly_task, weekly_task, daily_task]
```

## Advanced Monitoring

### 1. Custom Metrics: Track What Matters
Create a plugin to monitor custom metrics:

```python
from airflow.plugins_manager import AirflowPlugin
from airflow.stats import Stats
from flask_admin.base import MenuLink

class MetricsPlugin(AirflowPlugin):
    """Plugin for tracking custom metrics"""
    name = 'metrics_plugin'
    hooks = []
    executors = []
    macros = []
    admin_views = []
    flask_blueprints = []
    menu_links = [MenuLink(
        category='Monitoring',
        name='Custom Metrics',
        url='/metrics'
    )]
    
    def on_task_success(self, context):
        """Track successful task metrics"""
        task_instance = context['task_instance']
        duration = (task_instance.end_date - task_instance.start_date).total_seconds()
        
        # Record task duration
        Stats.timing(
            f'task_duration.{task_instance.dag_id}.{task_instance.task_id}',
            duration
        )
        
        # Count successful tasks
        Stats.incr(
            f'task_success.{task_instance.dag_id}.{task_instance.task_id}'
        )
```

### 2. Smart Alerting: Stay Informed
Create an operator that handles sophisticated alerting:

```python
from airflow.models import BaseOperator
from airflow.utils.email import send_email
from slack_sdk import WebClient

class EnhancedAlertOperator(BaseOperator):
    """
    Smart alerting operator with multiple channels
    """
    def __init__(
        self,
        alert_channels: List[str],
        slack_token: Optional[str] = None,
        *args, **kwargs
    ) -> None:
        super().__init__(*args, **kwargs)
        self.alert_channels = alert_channels
        self.slack_token = slack_token
        
    def execute(self, context: Dict[str, Any]) -> None:
        alert_message = self._generate_alert_message(context)
        
        # Send alerts through configured channels
        for channel in self.alert_channels:
            if channel == 'email':
                self._send_email_alert(alert_message)
            elif channel == 'slack':
                self._send_slack_alert(alert_message)
                
    def _generate_alert_message(self, context: Dict[str, Any]) -> str:
        """Create detailed alert message"""
        task_instance = context['task_instance']
        return f"""
        DAG: {task_instance.dag_id}
        Task: {task_instance.task_id}
        Execution Time: {task_instance.start_date}
        Status: {task_instance.state}
        Log URL: {task_instance.log_url}
        """
        
    def _send_slack_alert(self, message: str) -> None:
        """Send alert to Slack"""
        if not self.slack_token:
            raise ValueError("Slack token not provided")
            
        client = WebClient(token=self.slack_token)
        client.chat_postMessage(
            channel="#airflow-alerts",
            text=message
        )
```

## Complex Workflow Patterns

### 1. SubDAGs: Modular Workflows
Create reusable workflow components:

```python
from airflow.models import DAG
from airflow.operators.subdag import SubDagOperator

def create_processing_subdag(
    parent_dag_name: str,
    child_dag_name: str,
    start_date: datetime,
    schedule_interval: str
) -> DAG:
    """Create a reusable processing workflow"""
    dag = DAG(
        f"{parent_dag_name}.{child_dag_name}",
        schedule_interval=schedule_interval,
        start_date=start_date,
    )
    
    with dag:
        # Define your reusable workflow
        task1 = PythonOperator(...)
        task2 = PythonOperator(...)
        task3 = PythonOperator(...)
        
        task1 >> [task2, task3]
        
    return dag

# Use the SubDAG in your main workflow
processing_subdag = SubDagOperator(
    task_id='processing_subdag',
    subdag=create_processing_subdag(
        'main_dag',
        'processing_subdag',
        start_date,
        schedule_interval
    ),
    dag=main_dag
)
```

### 2. Task Groups: Organized Workflows
Create well-organized task groups:

```python
from airflow.utils.task_group import TaskGroup

def create_processing_group(dag: DAG) -> TaskGroup:
    """Create an organized group of processing tasks"""
    with TaskGroup(
        group_id='processing_tasks',
        tooltip='Data processing tasks'
    ) as processing_group:
        
        validate = PythonOperator(
            task_id='validate_data',
            python_callable=validate_data
        )
        
        # Transform task group
        with TaskGroup(group_id='transform_tasks') as transform_group:
            transform1 = PythonOperator(...)
            transform2 = PythonOperator(...)
            
            transform1 >> transform2
            
        # Load task group
        with TaskGroup(group_id='load_tasks') as load_group:
            load1 = PythonOperator(...)
            load2 = PythonOperator(...)
            
            [load1, load2]
            
        validate >> transform_group >> load_group
        
    return processing_group
```

## Performance Optimization

### 1. Resource Management: Smart Resource Use
Configure task-level resources:

```python
from airflow.models import DAG
from airflow.operators.python import PythonOperator

# Configure task resources
task = PythonOperator(
    task_id='resource_intensive_task',
    python_callable=process_data,
    execution_timeout=timedelta(hours=1),
    pool='high_cpu_pool',
    pool_slots=2,
    retry_delay=timedelta(minutes=5),
    retry_exponential_backoff=True,
    max_retry_delay=timedelta(hours=1)
)

# Configure DAG-level resources
dag = DAG(
    'optimized_pipeline',
    default_args={
        'pool': 'default_pool',
        'execution_timeout': timedelta(hours=2),
        'retry_delay': timedelta(minutes=5),
        'max_active_runs': 1
    }
)
```

### 2. Database Optimization: Better Performance
Optimize database interactions:

```python
# airflow.cfg optimizations
[core]
sql_alchemy_pool_size = 5
sql_alchemy_pool_recycle = 1800
sql_alchemy_max_overflow = 10

# Smart session handling
def get_optimized_session():
    """Get optimized database session"""
    from airflow.utils.db import create_session
    
    with create_session() as session:
        session.execute('SET statement_timeout = 3600000')  # 1 hour
        return session
```

## Error Handling and Recovery

### 1. Smart Error Handling
Implement sophisticated error handling:

```python
from airflow.operators.python import PythonOperator
from airflow.exceptions import AirflowException

def handle_task_failure(context):
    """Smart failure handler"""
    task_instance = context['task_instance']
    dag_run = context['dag_run']
    
    # Detailed failure logging
    logging.error(f"""
    Task Failure:
    DAG: {task_instance.dag_id}
    Task: {task_instance.task_id}
    Execution Date: {dag_run.execution_date}
    Error: {context['exception']}
    """)
    
    # Smart recovery logic
    if isinstance(context['exception'], ConnectionError):
        # Retry with exponential backoff
        task_instance.max_tries = 5
        task_instance.retry_delay = timedelta(minutes=5)
        task_instance.retry_exponential_backoff = True
    
    # Alert relevant teams
    send_failure_alert(context)

task = PythonOperator(
    task_id='task_with_smart_error_handling',
    python_callable=process_data,
    on_failure_callback=handle_task_failure,
    provide_context=True
)
```

### 2. Recovery Strategies: Graceful Recovery
Implement checkpointing for recovery:

```python
def implement_recovery_checkpoint():
    """Implement smart recovery checkpoints"""
    from airflow.models import XCom
    
    def save_checkpoint(task_instance, checkpoint_data):
        """Save progress checkpoint"""
        task_instance.xcom_push(
            key='checkpoint',
            value=checkpoint_data
        )
    
    def restore_from_checkpoint(task_instance):
        """Restore from last checkpoint"""
        checkpoint_data = task_instance.xcom_pull(
            task_ids=task_instance.task_id,
            key='checkpoint'
        )
        return checkpoint_data

# Use in your tasks
def process_with_checkpoints(**context):
    task_instance = context['task_instance']
    
    # Try to restore from checkpoint
    checkpoint = restore_from_checkpoint(task_instance)
    if checkpoint:
        start_from = checkpoint['last_processed']
    else:
        start_from = 0
    
    try:
        # Process with regular checkpoints
        for i in range(start_from, total_items):
            process_item(i)
            if i % 100 == 0:  # Checkpoint every 100 items
                save_checkpoint(task_instance, {'last_processed': i})
    except Exception as e:
        # Save final checkpoint before failing
        save_checkpoint(task_instance, {'last_processed': i})
        raise
```

## Best Practices for Advanced Workflows

### 1. Code Organization
- Use clear, consistent naming conventions
- Implement modular, reusable code
- Separate configuration from logic
- Group related tasks together
- Document complex workflows thoroughly

### 2. Testing Strategies
```python
from airflow.utils.dag_cycle_tester import check_cycle
from airflow.models import DagBag

def test_dag_integrity():
    """Comprehensive DAG testing"""
    dag_bag = DagBag()
    
    # Check for import errors
    assert len(dag_bag.import_errors) == 0, \
        f"DAG import failures: {dag_bag.import_errors}"
    
    # Test specific DAG
    dag = dag_bag.get_dag('my_dag')
    
    # Check for cycles
    check_cycle(dag)
    
    # Verify task dependencies
    assert len(dag.tasks) > 0
    
    # Check task properties
    task = dag.get_task('my_task')
    assert task.retries == 3
    assert task.retry_delay == timedelta(minutes=5)
```

### 3. Monitoring Best Practices
- Implement detailed logging
- Set meaningful alert thresholds
- Monitor resource utilization
- Track performance trends
- Regular health checks

## Common Pitfalls and Solutions

### 1. Performance Issues
Problem: Too many concurrent tasks
Solution: Implement proper pooling and queuing

Problem: Inefficient database queries
Solution: Optimize queries and use connection pooling

Problem: Resource contention
Solution: Configure resource pools and limits

### 2. Design Problems
Problem: Complex task dependencies
Solution: Use task groups and modular design

Problem: Large monolithic tasks
Solution: Break into smaller, focused tasks

Problem: Poor error handling
Solution: Implement comprehensive error handling

### 3. Operational Challenges
Problem: Insufficient testing
Solution: Implement automated testing

Problem: Poor version control
Solution: Use proper CI/CD practices

Problem: Unclear ownership
Solution: Establish clear responsibilities

## Conclusion

Congratulations! You've mastered advanced Airflow concepts. Remember these key points:
- Custom components give you ultimate flexibility
- Dynamic task generation adapts to your needs
- Advanced monitoring keeps you informed
- Smart error handling ensures reliability
- Best practices prevent problems before they start

## Additional Resources

1. **Advanced Documentation**
   - [Airflow Advanced Concepts Guide](https://airflow.apache.org/docs/apache-airflow/stable/concepts/index.html)
   - [Custom Operators Guide](https://airflow.apache.org/docs/apache-airflow/stable/howto/custom-operator.html)
   - [Dynamic Task Mapping](https://airflow.apache.org/docs/apache-airflow/stable/concepts/dynamic-task-mapping.html)

2. **Best Practices**
   - [Airflow Performance Tuning](https://airflow.apache.org/docs/apache-airflow/stable/best-practices.html)
   - [Production Deployment Guide](https://airflow.apache.org/docs/apache-airflow/stable/production-deployment.html)
   - [Security Best Practices](https://airflow.apache.org/docs/apache-airflow/stable/security/best-practices.html)

3. **Community Resources**
   - [Airflow GitHub Discussions](https://github.com/apache/airflow/discussions)
   - [Airflow Summit Presentations](https://airflowsummit.org/)
   - [Airflow Slack Community](https://apache-airflow.slack.com/)

4. **Books and Tutorials**
   - "Data Pipelines with Apache Airflow" by Bas P. Harenslak and Julian de Ruiter
   - "Fundamentals of Data Engineering" by Joe Reis and Matt Housley
   - "Building Data Pipelines with Apache Airflow" by Marc Lamberti 