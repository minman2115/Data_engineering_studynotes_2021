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
