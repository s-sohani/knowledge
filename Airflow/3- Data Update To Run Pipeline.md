Instead of use daily to run task, we can schedule to run task base on data update.

![[Pasted image 20240508071220.png]]

```python 
from airflow import DAG, Dataset
from airflow.decorators import task

from datetime import datetime

my_file = Dataset("/tmp/my_file.txt")

with DAG(
    dag_id="producer",
    schedule="@daily",
    start_date=datetime(2022, 1, 1),
    catchup=False
):

    @task(outlets=[my_file])
    def update_my_file():
        with open(my_file.uri, "a+") as f:
            f.write("producer update")
    
    update_my_file()
```

```python 
from airflow import DAG, Dataset
from airflow.decorators import task

from datetime import date, datetime

my_file = Dataset("/tmp/my_file.txt")

with DAG(
    dag_id="consumer",
    schedule=[my_file],
    start_date=datetime(2022, 1, 1),
    catchup=False
):

    @task
    def read_my_file():
        with open(my_file.uri, "r") as f:
            print(f.read())
    
    read_my_file()
```