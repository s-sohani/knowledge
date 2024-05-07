Airflow has 3 component:
- Web Server
- Scheduler
- Meta database

Each component deploy in multi Instance to get HA and Fault Tolerance.

### Install Airflow
```bash 
sudo apt update

sudo apt install python3-pip

sudo apt install sqlite3

sudo apt install python3.10-venv

sudo apt-get install libpq-dev

source venv/bin/activate

pip install "apache-airflow[postgres]==2.5.0" --constraint "https://raw.githubusercontent.com/apache/airflow/constraints-2.5.0/constraints-3.7.txt"

airflow db init

sudo apt-get install postgresql postgresql-contrib

sudo -i -u postgres

psql

CREATE DATABASE airflow;
CREATE USER airflow WITH PASSWORD 'airflow';
GRANT ALL PRIVILEGES ON DATABASE airflow TO airflow;

sed -i 's#sqlite:////home/ubuntu/airflow/airflow.db#postgresql+psycopg2://airflow:airflow@localhost/airflow#g' airflow.cfg

sed -i 's#SequentialExecutor#LocalExecutor#g' airflow.cfg

airflow db init

airflow users create -u airflow -f airflow -l airflow -r Admin -e airflow@gmail.com

airflow webserver &

airflow scheduler
```

## Init DAG Scheduling
- start_date: Timestamp that scheduler tempt to backfill
- schedule_interval: How often a DAG runs.
- end_date: Timestamp from which DAG ends. 

![[Pasted image 20240507065906.png]]

## Concepts
### DAG
DAG is Direct Acyclic Graph, that is data pipeline with no cycles, and edges are dependency between nodes, and nodes are tasks.

### Operators
Operators are only tasks in your dag. Airflow has ton of Operators. 
For example:
- Bash Operator
- Python Operator
- Postgres Operator

```python 
training_model_A = PythonOperator( task_id="training_model_A", python_callable=_training_model )
```
### Catchup
If you set `start_date` on one year ago and interval on `daily`, Airflow will create all tasks from one year ago to now, if you set `catchup = False`, Airflow doesn't create tasks earlier.

```python 
with DAG("my_dag", start_date=datetime(2021, 1, 1),
    schedule_interval="@daily", catchup=False) as dag:
```

### xcom
Share data between tasks. Each task can push data to database and another task can pull that data. 
```python 
def _choose_best_model(ti): accuracies = ti.xcom_pull(task_ids=[ 'training_model_A', 'training_model_B', 'training_model_C' ]) best_accuracy = max(accuracies) if (best_accuracy > 8): return 'accurate' return 'inaccurate'
```

### Branch Operator
Choose Next Task base on returning value. 
```python
    if (best_accuracy > 8):
        return 'accurate'
    return 'inaccurate'
```
### Define dependencies in Airflow 
We can Define dependencies with Big Shift Operator. 
Use [ ]  to define tasks in same level.
```
[training_model_A, training_model_B, training_model_C] >> choose_best_model >> [accurate, inaccurate]
```

### Example
```python
from airflow import DAG
from airflow.operators.python import PythonOperator, BranchPythonOperator
from airflow.operators.bash import BashOperator

from random import randint
from datetime import datetime

def _choose_best_model(ti):
    accuracies = ti.xcom_pull(task_ids=[
        'training_model_A',
        'training_model_B',
        'training_model_C'
    ])
    best_accuracy = max(accuracies)
    if (best_accuracy > 8):
        return 'accurate'
    return 'inaccurate'


def _training_model():
    return randint(1, 10)

with DAG("my_dag", start_date=datetime(2021, 1, 1),
    schedule="@daily", tags=["Data engineering team", "Mark"], catchup=False) as dag:

        training_model_A = PythonOperator(
            task_id="training_model_A",
            python_callable=_training_model
        )

        training_model_B = PythonOperator(
            task_id="training_model_B",
            python_callable=_training_model
        )

        training_model_C = PythonOperator(
            task_id="training_model_C",
            python_callable=_training_model
        )

        choose_best_model = BranchPythonOperator(
            task_id="choose_best_model",
            python_callable=_choose_best_model
        )

        accurate = BashOperator(
            task_id="accurate",
            bash_command="echo 'accurate'"
        )

        inaccurate = BashOperator(
            task_id="inaccurate",
            bash_command="echo 'inaccurate'"
        )

        [training_model_A, training_model_B, training_model_C] >> choose_best_model >> [accurate, inaccurate]
```


## Task API
### Share data between tasks
- XCOM_PULL OR XCOM_PUSH
- Task Decorator

### expand
training_model.expand


```python
from airflow import DAG
from airflow.decorators import task
from airflow.operators.bash import BashOperator

from datetime import datetime

with DAG("new_dag", start_date=datetime(2021, 1, 1), 
    schedule="@daily", catchup=False):

        @task
        def training_model(accuracy):
            return accuracy

        @task.branch
        def choose_best_model(accuracies):
            best_accuracy = max(accuracies)
            if best_accuracy > 8:
                return 'accurate'
            return 'inaccurate'

        accurate = BashOperator(
            task_id="accurate",
            bash_command="echo 'accurate'"
        )

        inaccurate = BashOperator(
            task_id="inaccurate",
            bash_command="echo 'inaccurate'"
        )

        choose_best_model(training_model.expand(accuracy=[3, 9, 2])) >> [accurate, inaccurate]
```