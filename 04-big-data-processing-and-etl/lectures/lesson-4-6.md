# Lesson 4.6: Apache Airflow Basics

## Navigation
- [← Back to Lesson Plan](../4.6-apache-airflow-basics.md)
- [← Back to Module Overview](../README.md)

## Overview
Welcome to Apache Airflow! Think of Airflow as your personal workflow conductor, orchestrating complex data pipelines with elegance and reliability. In this lesson, we'll explore how Airflow helps you build, schedule, and monitor your data workflows. Whether you're handling ETL processes, training machine learning models, or managing complex data transformations, Airflow provides the tools you need to succeed.

## Learning Objectives
After completing this lesson, you'll be able to:
- Understand Airflow's architecture and how its components work together
- Create and manage DAGs (Directed Acyclic Graphs) effectively
- Work with operators and sensors to build workflow tasks
- Implement task dependencies and scheduling strategies
- Monitor and maintain your workflows
- Apply best practices and avoid common pitfalls

## Airflow Architecture: The Building Blocks

Let's explore the key components that make Airflow tick. Think of these components as members of an orchestra, each playing a crucial role in the symphony of workflow automation.

### 1. Web Server: Your Control Center
The Web Server is your window into Airflow's world. It provides:
- A user-friendly interface to monitor and manage workflows
- Real-time task status and progress tracking
- Access to logs and historical data
- Visual representations of your DAGs

Here's how you might configure it:
```python
# Example configuration in airflow.cfg
[webserver]
base_url = http://localhost:8080
web_server_host = 0.0.0.0
web_server_port = 8080
web_server_worker_timeout = 120
```

### 2. Scheduler: The Conductor
The Scheduler is the brain of Airflow, ensuring tasks run at the right time and in the right order. It handles:
- DAG parsing and validation
- Task scheduling and dependency resolution
- Worker task assignment
- State management

Configuration example:
```python
# Example configuration
[scheduler]
job_heartbeat_sec = 5
scheduler_heartbeat_sec = 5
num_runs = -1  # Continuous operation
max_threads = 2
```

### 3. Metadata Database: The Memory Bank
Think of the Metadata Database as Airflow's long-term memory, storing:
- DAG and task states
- Variables and connections
- Historical execution data
- Cross-task communication (XCom) data

Here's a typical configuration:
```python
# Example database configuration
[database]
sql_alchemy_conn = postgresql+psycopg2://airflow:airflow@postgres/airflow
sql_engine_encoding = utf-8
sql_alchemy_pool_enabled = True
```

### 4. Executors: The Task Performers
Executors are the workhorses of Airflow, responsible for running your tasks. Choose from:
- Sequential Executor (for development)
- Local Executor (for small deployments)
- Celery Executor (for distributed setups)
- Kubernetes Executor (for cloud-native applications)

Configuration examples:
```python
# Different executor configurations
[core]
# Sequential Executor (default)
executor = SequentialExecutor

# Local Executor
executor = LocalExecutor

# Celery Executor
executor = CeleryExecutor
```

## Building Your First DAG

Let's create a simple yet powerful data processing pipeline. Think of a DAG as a recipe, with each task being a step in the preparation.

### 1. Basic DAG Structure
```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime, timedelta

# Define default arguments - think of these as your recipe's standard settings
default_args = {
    'owner': 'data_engineer',
    'depends_on_past': False,
    'start_date': datetime(2024, 1, 1),
    'email': ['alerts@example.com'],
    'email_on_failure': True,
    'email_on_retry': False,
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
}

# Create the DAG - your recipe's main structure
with DAG(
    'data_processing_pipeline',
    default_args=default_args,
    description='ETL pipeline for data processing',
    schedule_interval='@daily',
    catchup=False
) as dag:
    
    def extract():
        # Your data extraction logic here
        pass
    
    def transform():
        # Your data transformation logic here
        pass
    
    def load():
        # Your data loading logic here
        pass
    
    # Define tasks - the steps in your recipe
    extract_task = PythonOperator(
        task_id='extract_data',
        python_callable=extract
    )
    
    transform_task = PythonOperator(
        task_id='transform_data',
        python_callable=transform
    )
    
    load_task = PythonOperator(
        task_id='load_data',
        python_callable=load
    )
    
    # Set task dependencies - the order of your recipe steps
    extract_task >> transform_task >> load_task
```

### 2. Task Dependencies: Building the Workflow
There are several ways to define how your tasks should flow:

```python
# Method 1: Using the intuitive bitshift operator
task1 >> task2 >> task3

# Method 2: Using explicit set_upstream/set_downstream
task2.set_upstream(task1)
task2.set_downstream(task3)

# Method 3: For complex dependencies, use cross_downstream
from airflow.models.baseoperator import cross_downstream
cross_downstream([task1, task2], [task3, task4])
```

### 3. Scheduling: When Should It Run?
Airflow provides flexible scheduling options:

```python
# Common scheduling patterns
schedule_interval='@daily'           # Once a day
schedule_interval='0 0 * * *'       # Midnight every day (cron style)
schedule_interval=timedelta(hours=1) # Every hour
schedule_interval=None              # Manual triggers only
```

## Working with Operators and Sensors

Operators and sensors are your building blocks for tasks. Let's explore some common ones:

### 1. Common Operators: Your Task Toolkit
```python
# Python Operator - for running Python functions
from airflow.operators.python import PythonOperator

python_task = PythonOperator(
    task_id='process_data',
    python_callable=process_function,
    op_kwargs={'param1': 'value1'}
)

# Bash Operator - for running shell commands
from airflow.operators.bash import BashOperator

bash_task = BashOperator(
    task_id='run_script',
    bash_command='python /path/to/script.py'
)

# SQL Operator - for database operations
from airflow.providers.postgres.operators.postgres import PostgresOperator

sql_task = PostgresOperator(
    task_id='query_data',
    postgres_conn_id='postgres_default',
    sql='SELECT * FROM table'
)
```

### 2. Sensors: Waiting for Conditions
```python
# File Sensor - wait for a file to appear
from airflow.sensors.filesystem import FileSensor

file_sensor = FileSensor(
    task_id='wait_for_file',
    filepath='/path/to/file.csv',
    poke_interval=60,  # Check every minute
    timeout=3600      # Timeout after an hour
)

# External Task Sensor - wait for another DAG
from airflow.sensors.external_task import ExternalTaskSensor

wait_for_other_dag = ExternalTaskSensor(
    task_id='wait_for_task',
    external_dag_id='other_dag',
    external_task_id='final_task',
    timeout=3600
)
```

### 3. Building Custom Operators
Sometimes you need a specialized tool. Here's how to create one:

```python
from airflow.models.baseoperator import BaseOperator
from typing import Any, Dict

class CustomOperator(BaseOperator):
    def __init__(
        self,
        custom_param: str,
        **kwargs
    ) -> None:
        super().__init__(**kwargs)
        self.custom_param = custom_param
    
    def execute(self, context: Dict[str, Any]) -> Any:
        """Your custom execution logic goes here"""
        self.log.info(f"Executing with param: {self.custom_param}")
        # Implementation logic
        return result
```

## Task Communication: Sharing Data

Tasks often need to share information. Airflow provides two main ways to do this:

### 1. XCom: Cross-Communication
```python
def push_data(**context):
    """Share data with other tasks"""
    context['task_instance'].xcom_push(
        key='sample_data',
        value={'status': 'success', 'count': 100}
    )

def pull_data(**context):
    """Get data from another task"""
    data = context['task_instance'].xcom_pull(
        task_ids='push_task',
        key='sample_data'
    )
    print(f"Retrieved data: {data}")

# Create tasks using XCom
push_task = PythonOperator(
    task_id='push_task',
    python_callable=push_data,
    provide_context=True
)

pull_task = PythonOperator(
    task_id='pull_task',
    python_callable=pull_data,
    provide_context=True
)
```

### 2. Variables: Global Settings
```python
from airflow.models import Variable

# Use variables in your DAGs
api_key = Variable.get("api_key")
config = Variable.get("config", deserialize_json=True)
```

## Best Practices: Building Reliable Workflows

### 1. DAG Design
- Give tasks meaningful, descriptive IDs
- Keep tasks atomic and idempotent
- Implement proper error handling
- Set appropriate retries and timeouts
- Use task groups for organization

### 2. Performance Tips
```python
# Use Task Groups for better organization
from airflow.utils.task_group import TaskGroup

with TaskGroup(group_id='transform_tasks') as transform_group:
    task1 = PythonOperator(...)
    task2 = PythonOperator(...)
    task3 = PythonOperator(...)
    
    task1 >> [task2, task3]
```

### 3. Monitoring and Maintenance
- Set up alerts for task failures
- Monitor task duration trends
- Track success rates
- Implement comprehensive logging
- Review DAGs regularly

## Common Pitfalls and How to Avoid Them

### 1. Task Design Issues
- Keep tasks simple and focused
- Always handle errors gracefully
- Manage dependencies carefully
- Watch for resource bottlenecks
- Set appropriate timeouts

### 2. Performance Problems
- Don't create too many tasks
- Schedule efficiently
- Watch for memory leaks
- Manage database connections
- Monitor resource usage

### 3. Maintenance Challenges
- Keep dependencies updated
- Monitor system health
- Document everything
- Establish clear ownership
- Address security concerns

## Conclusion

Congratulations! You now understand the basics of Apache Airflow. Remember these key points:
- DAGs are your recipes for workflow success
- Operators and sensors are your building blocks
- Task dependencies define your workflow flow
- Proper monitoring ensures reliability
- Best practices prevent common problems

## Additional Resources

1. **Official Documentation**
   - [Apache Airflow Documentation](https://airflow.apache.org/docs/apache-airflow/stable/index.html)
   - [Airflow API Reference](https://airflow.apache.org/docs/apache-airflow/stable/stable-rest-api-ref.html)
   - [Operators and Hooks Reference](https://airflow.apache.org/docs/apache-airflow/stable/operators-and-hooks-ref.html)

2. **Best Practices**
   - [Airflow Best Practices Guide](https://airflow.apache.org/docs/apache-airflow/stable/best-practices.html)
   - [Common Pitfalls](https://airflow.apache.org/docs/apache-airflow/stable/best-practices.html#common-pitfalls)
   - [Security Best Practices](https://airflow.apache.org/docs/apache-airflow/stable/security/best-practices.html)

3. **Community Resources**
   - [Airflow GitHub Repository](https://github.com/apache/airflow)
   - [Airflow Blog](https://airflow.apache.org/blog/)
   - [Airflow Slack Channel](https://apache-airflow.slack.com/)

4. **Books and Tutorials**
   - "Data Pipelines with Apache Airflow" by Bas P. Harenslak and Julian de Ruiter
   - "Fundamentals of Data Engineering" by Joe Reis and Matt Housley
   - "Building Data Pipelines with Apache Airflow" by Marc Lamberti 