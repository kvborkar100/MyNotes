# What is Airflow
Open source tool for Data orchestration, managing, scheduling data pipelines

**Benefits** - Dynamic, Scalability, UI, and Extensibility 

**Core Components**-   
Web Server - Flask with Gunicorn for UI  
scheduler - Daemon for scheduling workflow  
Metastore - DB for metadata  
Executor - class for how tasks wil be excuted   

**DAG (Directed Acyclic Graph)** -  
Directed edges with no loops

**Operator** -  
Wrapper around the class
```python
db = connect(host, credentials)
db.insert(sql_request)
```  
- Action Operator - Excuting commands, functions eg. python, bash script  
- Transfer Operator - allows to transfer data eg. any operator transfering data 
- Sensor Operators - wait for some thing to happen eg. waiting for file to land at specific location


**Task** - is an operator  
**Task Instance** - when operator is triggered it becomes task instance

**Workflow** - DAG with all components
 
![alt text](single-node-architecture.png "Single Node Architecture")


![alt text](Multinode-airflow.png "Multi Node Architecture")

![alt image](working-airflow.png)

1. add dag.py in dag folder
2. web server and scheduler parses the dag
3. scheduler creates dagRun object
4. scheduler schedules task instance object
5. scheduler sends task instance to the executor
6. executor runs the task instance
7. task instance state is updated in metastore
8. scheduler checkes for task status
9. web server updates the status in UI


airflow UI -> localhost:8080

**Airflow CLI**

```sh
> pip install apache-airflow

> airflow db init  #setup DB first time

> airflow users create -u kvborkar100 -p Blackpanther22 -f Krushna -l Borkar -r Admin -e test@email.com     #creating user for airflow UI

> airflow db upgrade   # upgrade airflow version

> airflow db reset   # everything will be reset

> airflow webserver --port 8080 # starts webserver at localhost:8080

> airflow scheduler  # start/stop the scheduler

> airflow worker   # if in distributed mode like celery

> airflow dags list   # show list of dags

> airflow tasks list example_complex  # show tasks for the given dag_id

> airflow dags trigger -e 2021-02-23 dag_id  # triggers the DAG at given execution type

> airflow test dag_id task_id execution_date # test the task in your DAG

> airflow tasks test user_processing creating_table 2020-01-01  #eg
```
Note: Create one task per operator to easily trigger a failed task.

Creating a DAG - 
```python
from airflow import DAG
from datetime import datetime

default_args = {
    'start_date': datetime(2020, 1, 1)

}

with DAG(dag_id='user_processing',
         schedule_interval='@daily',
         default_args=default_args,
         catchup=False) as dag:
    pass

```

If DAG is scheduled to run at 2020-1-1 10AM after every 10 minutes then the first run will be 2020-1-1 10.10 AM.  i.e start_date + schedule_time

If DAG is scheduled to run everyday from 1, 2, stopped on 3-4-5 and resumed on 6 then it will first trigger the 3-4-5 and then 6.
catchup = True
It will only trigger the DAGs from latest execution start date.  
Dates in Airflow are in UTC


Credits: 
Marc Lamberti Courses Udemy