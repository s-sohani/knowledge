Airflow has some call backs:

![[Pasted image 20240511070709.png]]

1- Create Channel in Microsoft Team
2- Create include file:
```python
# include/notifications.py
from airflow.models import Variable

def notify_teams(context):
    print("Sending Teams notification")
    import requests
    payload = {
        "@type": "MessageCard",
        "@context": "http://schema.org/extensions",
        "title": "Airflow Task Failed",
        "summary": f"Task {context['task_instance_key_str']} failed",
        "themeColor": "0078D7",
        "sections": [
            {
                "activityTitle": f"Task {context['task_instance_key_str']} failed",
                "activitySubtitle": f"DAG: {context['dag'].dag_id}",
                "facts": [
                    {
                        "name": "Logical Date",
                        "value": context['ds']
                    },
                    {
                        "name": "Log URL",
                        "value": context['task_instance'].log_url
                    }
                ]
            }
        ],
        "potentialAction": [{
            "@type": "OpenUri",
            "name": "View Logs",
            "targets": [{
                "os": "default",
                "uri": context['task_instance'].log_url
            }]
        }]
    }
    
    headers = {"content-type": "application/json"}
    requests.post(Variable.get('teams_webhook_secret'), json=payload, headers=headers)
    print("Teams notification sent")
```

3- Write pipeline
```python
from airflow.decorators import dag, task
from airflow.models.baseoperator import chain
from airflow.exceptions import AirflowFailException
from include.notifications import notify_teams

@dag(schedule=None, catchup=False, on_failure_callback=notify_teams)
def my_dag():
    
    @task
    def a():
        print('good')
        
    @task
    def b():
        print('bad')
        raise AirflowFailException()
    
    chain(a(), b())
    
my_dag()
```