- Wait for file to run next task.
- Wait for complete a task to run another task. 

Sensor is Airflow Operation that return True Or False, For example if file exist in specific location, Sensor return true other wise Sensor return false.
Airflow has ton of sensors, For example it has sensor for exist file, Or sql Sensor Or external task Sensor.

```python

from airflown.sensors.filesystem import FileSensor
with DAG("my_dag", start_date=datetime(2021, 1, 1),
    schedule="@daily", tags=["Data engineering team", "Mark"], catchup=False) as dag:

	wait_for_file = FileSensor(
		task_id = 'waiting_for_file'
		poke_interval = 30
		timeout = 60 * 5  #important to avoid deadlock
		mode = 'reschedule'
	)

```

### Sensor mode
- `poke` (default): The Sensor takes up a worker slot for its entire runtime
- `reschedule`: The Sensor takes up a worker slot only when it is checking, and sleeps for a set duration between checks

Something that is checking every second should be in `poke` mode, while something that is checking every minute should be in `reschedule` mode.

### Timeout
If don't set timeout, task will take slot all time and doesn't release slot, For example if airflow has 12 slot and your task run every 5 min, after a short all slots become busy with single task. 

