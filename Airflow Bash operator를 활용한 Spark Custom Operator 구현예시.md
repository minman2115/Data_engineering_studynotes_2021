.

Data_Engineering_TIL(20210605)

[학습자료]

'[Airflow] BashOperator 확장을 통한 Spark Custom Operator' 블로그글을 공부하고 정리한 내용입니다.

URL : https://louisdev.tistory.com/21?category=894285

[학습내용]

Airflow를 이용한 spark-submit 명령을 위해 커스텀하게 만든 operator에 대한 구현예시임

- Airflow DAG script


```python
default_args = {
    'start_date': datetime(2015, 12, 1),
    'retries': 0,
    'catchup': False,
    'retry_delay': timedelta(minutes=5),
}

spark_args = dict(driver_cores=1,
                  driver_memory="2g",
                  executor_cores="5",
                  num_executors="1",
                  executor_memory="5g",
                  max_attempts=1,
                  yarn_queue="default",
                  spark_opts=["--conf \"spark.executor.extraJavaOptions=-XX:+PrintGCDetails -XX:+PrintGCTimeStamps\"",
                              "--conf spark.eventLog.enabled=false"
                              ],
                  spark_class="className",
                  jar="jar path",
                  keytab=var_json['keytab'],
                  principal=var_json['principal'],
                  job_args=['--table=tableName',
                            '--prefix=prefix',
                            ],
                  )

dag = DAG('spark', default_args=default_args, schedule_interval="@once")

cdl_to_cdp = SparkBashOperator(
    task_id='spark_bash_operator',
    **spark_args,
    dag=dag)
```

- SparkBashOperator


```python
from airflow.operators.bash_operator import BashOperator
from airflow.utils.decorators import apply_defaults
import re

class SparkBashOperator(BashOperator):
    
    @apply_defaults
    def __init__(
            self,
            spark_opts=[],
            driver_cores=1,
            driver_memory="2g",
            executor_cores="5",
            num_executors="1",
            executor_memory="5g",
            max_attempts=1,
            yarn_queue="root.default",
            spark_class="",
            jar="",
            keytab="",
            principal="",
            job_args=[],
            *args, **kwargs) -> None:
        super(SparkBashOperator, self).__init__(bash_command="", *args, **kwargs)

        self.driver_cores = driver_cores
        self.driver_memory = driver_memory
        self.executor_cores = executor_cores
        self.nun_executors = num_executors
        self.executor_memory = executor_memory
        self.max_attempts = max_attempts
        self.yarn_queue = yarn_queue
        self.spark_class = spark_class
        self.jar = jar
        self.keytab = keytab
        self.principal = principal
        self.job_args = job_args
        self.spark_opts = spark_opts

    def execute(self, context):
        command = """
            spark-submit --master yarn \
            --deploy-mode cluster \
            --driver-cores {driver_cores} \
            --driver-memory {driver_memory} \
            --executor-cores {executor_cores} \
            --num-executors {nun_executors} \
            --executor-memory {executor_memory} \
            --keytab {keytab} \
            --principal {principal} \
            --conf spark.yarn.maxAppAttempts={max_attempts} \
            {spark_opts} \
            --queue {yarn_queue} \
            --class {spark_class} \
            {jar} {job_args}
            """.format(driver_cores=self.driver_cores,
                       driver_memory=self.driver_memory,
                       executor_cores=self.executor_cores,
                       nun_executors=self.nun_executors,
                       executor_memory=self.executor_memory,
                       max_attempts=self.max_attempts,
                       queue=self.yarn_queue,
                       spark_class=self.spark_class,
                       jar=self.jar,
                       keytab=self.keytab,
                       principal=self.principal,
                       job_args=' '.join(self.job_args),
                       spark_opts=' '.join(self.spark_opts))

        self.bash_command = re.sub("\s\s+", " ", command)

        super(SparkBashOperator, self).execute(context)
```
