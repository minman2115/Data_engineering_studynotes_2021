.

Data_Engineering_TIL(20210529)

[참고한 자료]

베스핀글로벌 데이터 엔지니어 최정민님 'Airflow operator 활용자료'를 공부하고 정리한 자료입니다.

[학습내용]


- task를 pending 시키다가 특정시간이 되면 하위 task를 실행하는 custom operator 예시


- 활용예시

DAG process

create transient EMR cluster --> wait_execution_time --> if 15:00 --> running spark job workflow

만약에 정확히 15:00에 spark job workflow를 실행해야하는 상황이라고 하자 그러나 DAG process에는 해당 spark job workflow를 실행하기 앞서서 Transient한 EMR 클러스터를 띄우는 task가 있는데 구동하는 시간이 20분 ~ 40분정도 소요된다. 그렇다고 하면 14:00에 미리 EMR 클러스터를 띄워놓고, 대기하다가 15:00에 정확하게 workflow를 실행하도록 하면 된다. 이를 가능하게 해주는게 아래와 같은 operator라고 할 수 있다.


```python
# execution time 설정
execution_time = "15:00"

def wait_execution_time(**kwargs):
    while True:
        cur_time = (datetime.now()).strftime("%H:%M")
        # cur_time 과 execution_time 이 일치하는지 확인
        if cur_time == execution_time:
            break
        else :
            # 1분씩 poking 하면서 대기함
            print("poking for {} on {}".format(kwargs['task_instance_key_str'],cur_time))
            time.sleep(60)

wait_execution_time = PythonOperator(
    task_id = 'wait_execution_time',
    python_callable = wait_execution_time,
    provide_context = True,
    dag=dag
)
```
