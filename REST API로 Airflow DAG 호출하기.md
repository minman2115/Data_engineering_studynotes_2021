Data_Engineering_TIL(20210529)

[참고한 자료]

베스핀글로벌 데이터 엔지니어 최정민님 'Airflow operator 활용자료'를 공부하고 정리한 자료입니다.


[학습내용]

### REST API로 Airflow DAG를 호출하기

step 1) REST API용 Airflow User 생성

** 유의사항 : Airflow CLI 또는 Airflow UI에서 생성한 User는 Airflow Web server를 이용하기 위한 User임. REST API를 이용해서 DAG를 컨트롤 하기 위해서는 별도의 REST API 용 User(Airflow를 사용하는 계정이 아니라 별도로 Airflow 디비에 접근할 수 있는 시스템 유저라고 생각하면됨)를 생성해야함

- Create for Airflow REST API user


```python
from airflow import models, settings
from airflow.contrib.auth.backends.password_auth import PasswordUser
User = PasswordUser(models.User())

# User Info
user.username = 'minsupark'
user.email = 'minsu@korea.com'
user.password = 'mypassword1234'

session = settings.Session()
user_exits = session.query(models.User.id).filter_by(username=user.username).scalar() is not None
if not user_exits:
    session.add(user)
    session.commit()
session.close()
```

step 2) REST API로 DAG 호출하기

- 위에서 생성한 User info를 이용해서 REST API로 DAG를 호출하는 예시


```python
import requests
import json
from pprint import pprint

username = 'minsupark'
password = 'mypassword1234'

result = requests.post(
    "http://{Airflow_master_server_IP}:8080/api/experimental/dags/{DAG_NAME}/dag_runs",
    data = json.dumps("{}"),
    auth = (username,password)
)

# REST API는 8080 포트를 이용하기 때문에 당연한 얘기지만 
# Airflow master server의 8080포트가 API를 날리는 쪽에는 개방이 되어 있어야함

pprint(result.content.decode('utf-8'))
```

- Airflow DAG 활용예시

```python

username=Variable.get("minsupark_username")
password=Variable.get("minsupark_password")


# my_airflow_task_checker 라는 DAG 내에 task가 실행하다가 fail이 나면 restapi로 이 DAG를 재실행하는 함수
# DAG_NAME 라는 변수는 지금 이 스크립트를 돌리는 DAG이름임, 외부의 DAG를 트리거 하는것도 가능
def call_external_dag(**kwargs):
    # 실행시간 정보 조회
    execution_date = kwargs['execution_date']
    # DAG 정보 조회
    dag_instance = kwargs['dag']
    # 특정 TASK 정보 조회
    operator_instance = dag_instance.get_task('my_airflow_task_checker')
    # TASK 상태조회
    task_status = TaskInstance(operator_instance,execution_date).current_state()
    # my_airflow_task_checker TASK 상태가 fail이 나면 REST API를 이용해서 해당 DAG를 다시 RUNNING 시킴
    if task_status == 'failed':
        result = requests.post(
            # 참고로 아래와 같이 단일 Airflow에서 실행할 경우 localhost 주소로 api를 날릴 수 있지만,
            # 클러스터 형태의 airflow는 반드시 airflow master node의 ip로 호출해야 한다.
            "http://localhost:8080/api/experimental/dags/"+DAG_NAME+"/dag_runs",
            #"http://{Airflow_master_server_IP}:8080/api/experimental/dags/"+DAG_NAME+"/dag_runs",
            data = json.dumps("{}"),
            auth = (username,password)
        )
        pprint(result.content.decode('utf-8'))
    else:
        pprint(task_status)
        
retry_dag = PythonOperator(
    task_id = 'retry_dag',
    provide_context=True,
    python_callable=call_external_dag,
    trigger_rule = 'all_done',
    dag=dag
)

my_airflow_task >> my_airflow_task_checker >> retry_dag
```