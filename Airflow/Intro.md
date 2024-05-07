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


