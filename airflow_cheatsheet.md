# Airflow Cheat Sheet for Data Engineering

A practical reference for Apache Airflow — defining DAGs, operators, scheduling, and managing pipelines.

---

## Core Concepts

| Concept | What it means |
|---------|---------------|
| `DAG` | Directed Acyclic Graph — the definition of a pipeline and its task order |
| `Task` | A single unit of work inside a DAG |
| `Operator` | The type of task — Python, Bash, SQL, etc. |
| `DAG Run` | One execution instance of a DAG |
| `Task Instance` | One execution of a single task within a DAG run |
| `Schedule` | How often the DAG runs — defined as a cron expression or preset |
| `XCom` | Cross-communication — how tasks pass data to each other |
| `Connection` | A stored credential for an external system (DB, S3, API) |
| `Variable` | A key-value store for config values accessible across DAGs |

---

## Defining a DAG

### Basic DAG structure
```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime

with DAG(
    dag_id="my_pipeline",
    start_date=datetime(2024, 1, 1),
    schedule="@daily",
    catchup=False
) as dag:

    def my_task():
        print("running")

    task = PythonOperator(
        task_id="my_task",
        python_callable=my_task
    )
```

### Common schedule presets

| Preset | What it means |
|--------|---------------|
| `@once` | Run once and never again |
| `@hourly` | Run every hour |
| `@daily` | Run once a day at midnight |
| `@weekly` | Run once a week |
| `@monthly` | Run once a month |
| `"0 6 * * *"` | Run every day at 6 AM (cron syntax) |
| `None` | Never run automatically — only manual trigger |

### DAG parameters reference

| Parameter | What it does |
|-----------|--------------|
| `dag_id` | Unique name of the DAG |
| `start_date` | When the DAG becomes active |
| `schedule` | How often it runs |
| `catchup=False` | Don't run missed intervals from the past |
| `max_active_runs=1` | Only one DAG run active at a time |
| `default_args` | Default settings applied to all tasks |
| `tags` | Labels for filtering in the UI |

### Using default_args
```python
default_args = {
    "owner": "sajjad",
    "retries": 2,
    "retry_delay": timedelta(minutes=5),
    "email_on_failure": False
}

with DAG("my_pipeline", default_args=default_args, ...):
    ...
```

---

## Operators

### PythonOperator — run a Python function
```python
from airflow.operators.python import PythonOperator

def extract():
    print("extracting data")

task = PythonOperator(
    task_id="extract",
    python_callable=extract
)
```

### BashOperator — run a shell command
```python
from airflow.operators.bash import BashOperator

task = BashOperator(
    task_id="run_script",
    bash_command="python /opt/pipelines/transform.py"
)
```

### EmptyOperator — placeholder task (useful for structuring flow)
```python
from airflow.operators.empty import EmptyOperator

start = EmptyOperator(task_id="start")
end = EmptyOperator(task_id="end")
```

### BranchPythonOperator — conditionally run different tasks
```python
from airflow.operators.python import BranchPythonOperator

def decide():
    if condition:
        return "task_a"
    return "task_b"

branch = BranchPythonOperator(
    task_id="branch",
    python_callable=decide
)
```

---

## Task Dependencies

### Set task order with `>>` and `<<`
```python
extract >> transform >> load
```

### Fan out — one task triggers multiple
```python
extract >> [transform_a, transform_b]
```

### Fan in — multiple tasks must finish before the next
```python
[transform_a, transform_b] >> load
```

### Chain multiple tasks
```python
from airflow.models.baseoperator import chain

chain(extract, transform, validate, load)
```

---

## XCom — Passing Data Between Tasks

### Push a value from a task
```python
def extract(**context):
    context["ti"].xcom_push(key="row_count", value=1500)
```

### Pull a value in another task
```python
def load(**context):
    count = context["ti"].xcom_pull(task_ids="extract", key="row_count")
    print(count)
```

### Return value shortcut — return value is automatically pushed as `return_value`
```python
def extract():
    return {"rows": 1500}

def load(**context):
    data = context["ti"].xcom_pull(task_ids="extract")
    print(data["rows"])
```

---

## Variables & Connections

### Read a Variable in a DAG
```python
from airflow.models import Variable

bucket = Variable.get("s3_bucket_name")
config = Variable.get("pipeline_config", deserialize_json=True)
```

### Use a Connection in a task
```python
from airflow.hooks.base import BaseHook

conn = BaseHook.get_connection("my_postgres")
print(conn.host, conn.login, conn.password)
```

---

## Airflow CLI

| Command | What it does |
|---------|--------------|
| `airflow dags list` | List all DAGs |
| `airflow dags trigger my_dag` | Manually trigger a DAG run |
| `airflow dags pause my_dag` | Pause a DAG |
| `airflow dags unpause my_dag` | Unpause a DAG |
| `airflow tasks list my_dag` | List all tasks in a DAG |
| `airflow tasks test my_dag my_task 2024-01-01` | Test a single task without recording it |
| `airflow db init` | Initialize the Airflow metadata database |
| `airflow webserver` | Start the web UI |
| `airflow scheduler` | Start the scheduler |

---

## ETL Patterns

### Classic ETL DAG — extract, transform, load in sequence
```python
extract >> transform >> load
```

### ETL with a start and end marker
```python
start >> extract >> transform >> load >> end
```

### Run a pipeline daily, skip past missed runs
```python
with DAG(
    dag_id="daily_pipeline",
    start_date=datetime(2024, 1, 1),
    schedule="@daily",
    catchup=False
) as dag:
    ...
```

### Retry a task up to 3 times with a 5 minute wait
```python
task = PythonOperator(
    task_id="load",
    python_callable=load_data,
    retries=3,
    retry_delay=timedelta(minutes=5)
)
```

### Pass a date parameter into a task using Jinja templating
```python
task = BashOperator(
    task_id="run_with_date",
    bash_command="python pipeline.py --date {{ ds }}"
)
```

### Trigger a DAG only after another DAG finishes
```python
from airflow.sensors.external_task import ExternalTaskSensor

wait = ExternalTaskSensor(
    task_id="wait_for_upstream",
    external_dag_id="upstream_pipeline",
    external_task_id="load"
)

wait >> my_task
```

### Read config from a Variable and pass to a task
```python
def run(**context):
    config = Variable.get("pipeline_config", deserialize_json=True)
    process(config["batch_size"])
```

---

*Last updated: 2026*
