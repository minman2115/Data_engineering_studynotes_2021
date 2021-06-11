.

Data_Engineering_TIL(20210611)

[요약설명]

매일 특정시간에 s3간에 데이터를 복사하는 Airflow DAG 스크립트

관련자료 : https://minman2115.github.io/DE_TIL209/

[아키텍처]

Airflow -----------------> EMR  ----------------------> s3 to s3 copy

** 참고사항

1) 매일 23시에 Airflow DAG 실행

2) EMR은 s3 copy 명령을 실행하는 주체

3) s3 copy 명령 실행중 Error 발생할 경우 재시도 하는 로직이 있음

[구현한 스크립트]

- s3_bucket_data_copy.py


```python
from airflow import DAG
from airflow.operators.python_operator import PythonOperator
from datetime import datetime, timedelta
from airflow.contrib.operators.emr_add_steps_operator import EmrAddStepsOperator
from airflow.contrib.sensors.emr_step_sensor import EmrStepSensor
from airflow.models import DagBag
from airflow.models import TaskInstance
import requests
import json
import time
import os
from airflow.models import Variable
module_path=Variable.get("my_module_path")
import sys
sys.path.append(module_path)
from my_custom_lib import *
from pprint import pprint
import logging
log = logging.getLogger(__name__)

###############################################################
# custom parameter settings
###############################################################

ENV = Variable.get('ENV') # prod or dev
current_time=check_current_time()

user_info = Variable.get("airflow_sys_user_info")
airflow_sys_user_id = json.load(user_info)['id']
airflow_sys_user_pwd = json.load(user_info)['pwd']

airflow_master_node_ip = Variable.get("airflow_master_node_ip")
airflow_master_node_ip = json.load(airflow_master_node_ip)['{}'.format(ENV)]

###############################################################
# DAG configuration
###############################################################

default_args = {
    'owner' : 'minman',
    'depends_on_past' : False,
    'start_date' : datetime(2021,2,11),
    'provide_context' : True,
    'max_active_runs' : 5
}

dag = DAG(
    's3_bucket_data_copy',
    default_args=default_args,
    schedule_interval = '* 23 * * *',
    catchup=False,
    concurrency = 3,
    tags=['data_management']
)

get_emr_cluster_id = PythonOperatorperator(
    task_id = 'get_emr_cluster_id',
    provide_context=True,
    python_callable=get_my_emr_cluster_id,
    op_kwargs={"cluster_name","minman_emr_test"},
    dag=dag
)

s3_copy = EmrAddStepsOperator(
    task_id = 's3_copy',
    job_flow_id="{{ task_instance.xcom_pull(task_ids='get_emr_cluster_id',key='jobflow_id')}}",
    aws_conn_id='aws_seoul',
    steps=[
        {
            'Name':'s3_copy',
            'ActionOnFailure':'CONTINUE',
            'HadoopJarStep':{
                'Jar':'command-runner.jar',
                'Args':["aws","s3","cp","s3://my_data_lake/raw_data/","s3://my_warehouse/data/","--recursive"]                
            }
        
        }
        
    ],
    default_args=default_args,
    trigger_rule='all_done',
    retries=5,
    dag=dag
)

check_s3_copy = EmrStepSensor(
    task_id = 'check_s3_copy',
    job_flow_id= "{{ task_instance.xcom_pull(task_ids='get_emr_cluster_id',key='jobflow_id')}}",
    step_id = "{{ task_instance.xcom_pull(task_ids='s3_copy',key='return_value')[0]}}",
    aws_conn_id='aws_seoul',
    trigger_rule='all_done',
    retries=5,
    dag=dag
)

def trigger_external_dag(**kwargs):
    execution_date = kwargs['execution_date']
    dag_instance = kwargs['dag']
    operator_instance = dag_instance.get_task("check_s3_copy")
    task_status = TaskInstance(operator_instance,execution_date).current_state()
    # my_airflow_task_checker TASK 상태가 fail이 나면 REST API를 이용해서 해당 DAG를 다시 RUNNING 시킴
    if task_status == 'failed':
#         result = requests.post(
#             # 참고로 아래와 같이 단일 Airflow에서 실행할 경우 localhost 주소로 api를 날릴 수 있지만,
#             # 클러스터 형태의 airflow는 반드시 airflow master node의 ip로 호출해야 한다.
#             "http://localhost:8080/api/experimental/dags/"+DAG_NAME+"/dag_runs",
#             #"http://{Airflow_master_server_IP}:8080/api/experimental/dags/"+DAG_NAME+"/dag_runs",
#             data = json.dumps("{}"),
#             auth = (username,password)
#         )
#         pprint(result.content.decode('utf-8'))
        c = Client(None,None)
        c.trigger_dag(dag_id='s3_bucket_data_copy',run_id='trigger_retry_logic_{}'.format(current_time),conf={})
    else:
        pprint(task_status)
        
retry_logic = PythonOperator(
    task_id = 'retry_logic',
    provide_context=True,
    python_callable=trigger_external_dag,
    trigger_rule = 'all_done',
    dag=dag
)

##################################################################################
# Design dag structure
##################################################################################

get_emr_cluster_id >> s3_copy >> check_s3_copy >> retry_logic
```

- my_custom_lib.py


```python
from airflow import DAG
from airflow.operators.python_operator import PythonOperator
import boto3
import json
import datetime
from airflow.models import Variable
airflow_env=Variable('ENV') # prod or dev

def check_current_time():
    utcnow = datetime.datetime.utcnow()
    time_gap = datetime.timedelta(hours=9)
    kor_time = utcnow + time_gap
    return kor_time.strftime('%Y%m%d%H%M%S')

def get_my_emr_cluster_id(cluster_name,**kwargs):
    from datetime import datetime, timedelta
    client=boto3.client('emr',region_name='us-west-2')
    response=client.list_cluster(
        CreatedAfter=datetime(2020,1,1),
        CreatedBefore=datetime(2999,1,1),
        ClusterStates=["WAITING","RUNNING"]
    )
    index=0
    for key in response['Clusters']:
        res = client.describe_cluster(ClusterId=key['Id'])
        name = res['Cluster']['Name']
        name = name.lower()
        if cluster_name.lower() in name:
            print(name)
            print(key['Id'])
            kwargs['ti'].xcom_push(key='jobflow_id',value=key['Id'])
```

- airflow variable 설정

1) airflow_master_node_ip :{"prod":"10.342.38.10","dev":"10.281.23.12"}

2) airflow_sys_user_info : {"id":"minman","pwd" :"mypwd123@"}

3) ENV : dev
