.

Data_Engineering_TIL(20210611)

- 아키텍처

Airflow DAG에서 10분마다 어떤 로직을 실행하는데 이 로직은 뭐냐면 운영정책 미준수하는 EC2를 STOP 시킴

** EC2 운영정책 체크내용 : TAG naming rule 준수여부, 특정시간 동안의 자원사용량

- 구현내용

1) Airflow DAG : ec2_monitor_DAG.py


```python
from airflow import DAG
from airflow.operators.python_operator import PythonOperator
from datetime import datetime, timedelta
from airflow.operators.bash_operator import BashOperator
from airflow.operators.dummy_operator import DummyOperator
from airflow.contrib.operators.ssh_operator import SSHOperator
from airlfow.contrib.hooks import SSHHook
import boto3
import json
import time
import sys
import logging
log = logging.getLogger(__name__)

default_args = {
    'owner' : 'minman',
    'depends_on_past' : False,
    'start_date' : datetime(2021,2,11),
    'provide_context' : True,
    'max_active_runs' : 5
}

dag = DAG(
    'ec2_monitor',
    default_args=default_args,
    schedule_interval = '*/10 * * * *',
    catchup=False,
    concurrency = 3,
    tags=['infra_management']
)

start = DummyOperator(
    task_id = 'start',
    dag=dag
)

end = DummyOperator(
    task_id = 'end',
    dag=dag
)

def ssh_conn(connid,bash_command,dag):
    sshHook = SSHHook(connid)
    ssh_operator = SSHOperator(
        ssh_hook=sshHook,
        task_id=connid,
        command=bash_command,
        dag=dag
    )
    return ssh_operator
    
ec2_monitor = ssh_conn('minman_ssh_conn','python3 /home/minman/ec2_monitor.py',dag)

start >> ec2_monitor >> end
```

** minman_ssh_conn

============================================================================================

[EditConnection]

Conn id : minman_ssh_conn

Conn Type : SSH

Host : 182.982.17.292

Username : ec2-user

Password :

Port : 22

Extra : {“key_file”:”/home/minman/airflow/dags/my_emr_pemkey”,”no_host_key_check”:”true”}

============================================================================================

2) 운영정책 미준수하는 EC2를 STOP logic : ec2_monitor.py

아래의 URL의 자료를 참고할 것

https://github.com/minman2115/Data_engineering_studynotes_2021/blob/master/%EC%9A%B4%EC%98%81%EC%A0%95%EC%B1%85%20%EB%AF%B8%EC%A4%80%EC%88%98%20EC2%EB%A5%BC%20%EB%AA%A8%EB%8B%88%ED%84%B0%EB%A7%81%ED%95%B4%EC%84%9C%20Stop%20%EC%8B%9C%ED%82%A4%EB%8F%84%EB%A1%9D%20%EC%9E%90%EB%8F%99%ED%99%94%ED%95%98%EB%8A%94%20%EC%98%88%EC%8B%9C/%EC%9A%B4%EC%98%81%EC%A0%95%EC%B1%85%20%EB%AF%B8%EC%A4%80%EC%88%98%ED%95%98%EB%8A%94%20EC2%EB%A5%BC%20STOP%20logic.zip
