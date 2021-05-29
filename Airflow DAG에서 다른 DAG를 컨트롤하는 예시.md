.

Data_Engineering_TIL(20210529)

[참고한 자료]

베스핀글로벌 데이터 엔지니어 최정민님 'Airflow operator 활용자료'를 공부하고 정리한 자료입니다.

[학습내용]


- DAG 내부 Task에서 다른 DAG를 trigger하거나 다른 DAG의 특정 task가 정상적으로 실행되었는지 확인하는 예시


```python
from airflow.operators import TriggerDagRunOperator
from airflow.sensors.external_task_sensor import ExternalTaskSensor

trigger_test = TriggerDagRunOperator(
    task_id = 'trigger_test',
    trigger_dag_id = 'my_test_external_dag',
    execution_date = "{{ execution_date }}",
    dag=dag
)

waiting_external_dag = ExternalTaskSensor(
    task_id = 'waiting_external_dag',
    external_dag_id = 'my_test_external_dag',
    external_task_id = 'my_test_task',
    execution_date_fn=lambda dt:dt,
    dag = dag,
    execution_timeout=timedelta(minutes=60),
    timeout=14000
)
# timeout = 14000초
```
