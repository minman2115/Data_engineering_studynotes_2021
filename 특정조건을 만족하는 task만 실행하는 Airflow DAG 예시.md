.

Data_Engineering_TIL(20210529)

[참고한 자료]

베스핀글로벌 데이터 엔지니어 최정민님 'Airflow operator 활용자료'를 공부하고 정리한 자료입니다.

[학습내용]

ShortCircuitOperator, AirflowSkipException을 활용하여 특정 조건에 따라 하위 task들의 실행을 skip할 수 있음


```python
from airflow import DAG
from airflow.operators.python_operator import PythonOperator
from airflow.operators.python_operator import ShortCircuitOperator
from airflow.operators.dummy_operator import DummyOperator
from airflow.exceptions import AirflowSkipException
import os
from airflow.models import Variable
my_module_path=Variable.get('my_module_path')
import sys
sys.path.append(my_module_path)
from my_custom_lib import *
from datetime import datetime, timedelta
import time

default_args={
    "owner":"minman",
    "depends_on_past":False,
    "start_date":datetime(2021,3,10),
    "provide_context":True,
    "max_active_runs":5
}

dag = DAG(
    "my_test_dag",
    default_args=default_args,
    schedule_interval=None,
    catchup=False,
    concurrency=10
)

# 조건을 부여하는 함수
# 현재시간과 task case time이 일치하면 True이고 그 외의 경우에는 False를 반환
def get_time(time,check_time):
    time = str(time).split("+")[0]
    dt_time = datetime.strptime(time, '%Y-%m-%dT%H:%M:%S.%f') + timedelta(hours=9) + timedelta(minutes=1)
    print(dt_time)
    print(dt_time.strftime('%H:%M:%s'))
    return dt_time.strftime('%H:%M') == check_time


# task_first_case_time, task_second_case_time, task_third_case_time에서 시간을 체크하고 True일 경우에만 하위 task가 실행되도록 구현
task_first_case_time = "23:11"
task_second_case_time = "23:11"
task_third_case_time = "23:11"

# 각각의 케이스의 시간을 체크하여 일치하면 True, 그렇지 않으면 Exception 처리함
def task_case_time_check(task_case, **kwargs):
    if get_time(kwargs['execution_date'],task_case):
        return True
    else:
        raise AirflowSkipException("Skip this task and indivisual downstream tasks while respecting trigger rules")
   
# ShortCircuitOperator 는 PythonOperator처럼 사용가능
#  Return 값이 True or False
# True일 경우 하위 task 계속 진행
# False일 경우 하위 task들은 모두 skip

first_case = ShortCircuitOperator(
    task_id = 'first_case',
    python_callable = task_case_time_check,
    op_kwargs={'task_case': task_first_case_time},
    provide_context = True,
    dag = dag
)

second_case = ShortCircuitOperator(
    task_id = 'second_case',
    python_callable = task_case_time_check,
    op_kwargs={'task_case': task_second_case_time},
    provide_context = True,
    dag = dag,
    trigger_rule = 'none_failed'
)

third_case = ShortCircuitOperator(
    task_id = 'third_case',
    python_callable = task_case_time_check,
    op_kwargs={'task_case': task_third_case_time},
    provide_context = True,
    dag = dag
)

task_A = DummyOperator(task_id='task_A',dag=dag,depends_on_past=False)
task_B = DummyOperator(task_id='task_B',dag=dag,depends_on_past=False)
task_C = DummyOperator(task_id='task_C',dag=dag)
task_D = DummyOperator(task_id='task_D',dag=dag,depends_on_past=False)

first_case >> task_A >> second_case >> task_B >> task_C
third_case >> task_D
```
