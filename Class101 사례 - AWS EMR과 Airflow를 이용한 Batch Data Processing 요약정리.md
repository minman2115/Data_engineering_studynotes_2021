.

Data_Engineering_TIL(20210605)

[학습자료]

'AWS EMR과 Airflow를 이용한 Batch Data Processing (Class101 데이터팀에서 Spark를 이용해 빅데이터를 ETL하는 법)' 블로그글을 공부하고 정리한 내용입니다.

URL : https://medium.com/class101-dev/aws-emr%EA%B3%BC-airflow%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-batch-data-processing-a100fc2f4f10

[학습내용]

- spark submit processing

step 1) 이미 실행되고 있는 AWS EMR cluster 마스터 노드에 SSH 접속 

step 2) 직접 shell에서 spark-submit커맨드를 실행 

** 이때 실행될 커맨드에 parameter로 PySpark job 이름과 그 job의 application code가 들어있는 Docker Image를 같이 제공

step 3) PySpark job이 완료되면 Airflow Task역시 완료되며 종료

- 참고사항

블로그글 일부 발췌내용

" PySpark job은 Python dependency들을 내재하고 있기 때문에 Spark 클러스터에 Python dependency들을 미리 설치해 Python환경을 조성해주어야 한다. 만약 PySpark Job들이 서로 다른 Python library들을 사용한다면 같은 머신 (클러스터)에서 Job들을 실행할 수 없을 것이다. 이를 해결해주기 위해 Amazon EMR이 6.0.0 release에서 Docker Image Support를 발표했다 (참조). 이제 각 PySpark Job들이 같은 클러스터이지만 서로 다른 Docker 컨테이너 위에 실행될 수 있다!"

참고자료 : https://aws.amazon.com/ko/blogs/big-data/run-spark-applications-with-docker-using-amazon-emr-6-0-0-beta/

- Class101에서 사용하는 PySpark Codebase

Airflow Repo에는 PySpark Job들을 Scheduling 해줄 코드만이 존재할 뿐 PySpark코드를 일체 포함하지 않음. PySpark 애플리케이션 코드들은 다른 Repo에 구성되어있다. 

아래는 그 Repo의 디렉토리 구성임


```console
pyspark-etl/
│   README.md
│   task_runner.py   
│   Dockerfile 
│
└───etl_codes/
│   │   
│   │   users_transform.py
│   │   marketing_transform.py
│   
│   
└───cli_etl_codes/
    │   cli_users_transform.py
    │   cli_marketing_transform.py
```

위의 리포에 정의한 pyspark 코드를 EMR에서 실행시키려면 정의한 method를 CLI커맨드 형태로 실행시키도록 하였는데 이를 위해 Python의 Click이라는 라이브러리를 활용하였다. Pyspark를 코드를 CLI로 정의하는 방법을 users_transform.py를 참고한다.

[소스코드 구현 과정]

step 1) pyspark-etl/etl_codes/cli_users_transform.py (메소드 정의) 작성


```python
def users_transform(
    spark: SparkSession,
    dest_db_prefix: str,
    dest_table_name: str,
    execution_datetime_tag: Union[date, datetime],
) -> None:
    """
    1. Read "raw_users" and "raw_user_infos" tables
    2. preprocess "raw_users"
    3. preprocess "raw_user_infos"
    4. join on "user_id"
    5. write "users"
    """
    # some pyspark codes
    
    s3_path = os.path.join(
        S3_ROOT_PATH,
        f"{dest_db_prefix}_{db_suffix}",
        f"{dest_table_name}",
        f"{execution_datetime_tag.strftime('%Y_%m_%d_%H_%M_%S')}",
    )
    df.repartition(partition_num).write.mode("overwrite").option(
        "path", s3_path
    ).saveAsTable(f"{dest_db_prefix}_{db_suffix}.{dest_table_name}")
```

step 2) pyspark-etl/cli_etl_codes/cli_users_transform.py(Cli command 정의) 작성


```python
@click.command(name="users_transform")
@click.option("--dest_db_prefix", type=click.STRING, required=True)
@click.option("--dest_table_name", type=click.STRING, required=True)
@click.option("--airflow_execution_datetime_tag",type=click_utils.DateTimeParamType(),required=True,help="Airflow Execution Datetime",)
def cli_users_transform(dest_db_prefix: str,dest_table_name: str,airflow_execution_datetime_tag: Union[date, datetime],) -> None:
    """ 
    Join raw lectures and raw steps table, transform, write lectures
    """
    with spark_session(app_name="users_transform") as spark:
        users_transform(spark=spark,dest_db_prefix=dest_db_prefix,dest_table_name=dest_table_name,execution_datetime_tag=airflow_execution_datetime_tag,)
```

step 3) pyspark-etl/task_runner.py의 run_task라는 main method에 위에 형성한 cli를 더해줌


```python
from cli_etl_codes import cli_users_transform

@click.group()
def run_task():
    """Simple group task that will group all sub-tasks together."""
    pass

# pyspark jobs
run_task.add_command(cli_users_transform)
```

step 4) Docker Image를 build하여 push한다.

그러면 EMR master 노드 shell에서 아래와 같은 커맨드를 실행이 가능하다.


```console
spark-submit                 
--master yarn                 
--deploy-mode cluster                 
--conf spark.executorEnv.YARN_CONTAINER_RUNTIME_DOCKER_IMAGE=my_imgage/pyspark-etl:latest                 
--conf spark.yarn.appMasterEnv.YARN_CONTAINER_RUNTIME_DOCKER_IMAGE=my_imgage/pyspark-etl:latest                 
--executor-memory 3G                 
--executor-cores 1                 
--driver-memory 2G                 
--driver-cores 1                 
run_task.py users_transform --dest_db_prefix='etl_production'
--dest_table_name='users' --execution_datetime_tag='2020-07-19T00:00:00+00:00'
```

step 5) Airflow에서 위에 PySpark job을 Airflow Task로 정의한다.

Class101 데이터팀이 Airflow에서 EMR로 PySpark job을 submit하는 방법 : EMRSSHOperator 라는 커스텀한 Operator를 만들어서 사용함


```python
dag = DAG(
    "transform-dag",
    schedule_interval="30 15 * * *",  # every 00:30 AM KST
    catchup=False,
    max_active_runs=1,
    default_args={
        **PYSPARK_ETL_DEFAULT_ARGS,
        "start_date": datetime(2020, 8, 6),
    },
)

users_transform = EMRSSHOperator(
    dag=dag,
    task_id="users_transform",
    image="my_imgage/pyspark-etl:latest",
    spark_program_args=shell_command_args(
        "users_transform",
        dest_db_prefix="etl",
        dest_table_name="users",
        execution_datetime_tag="{{execution_date}}",
    ),
)
```

- EMRSSHOperator


```python
class EMRSSHOperator(ssh_operator.SSHOperator):
    """
    Operator that inherits SSHOperator to run spark-submit command on remote EMR cluster
    
    """
    
    @decorators.apply_defaults
    def __init__(
        self,
        spark_program_args: str,
        driver_cores: str = "1",
        driver_memory: str = "2G",
        executor_cores: str = "1",
        executor_memory: str = "3G",
        spark_confs: list = [],
        extra_spark_conf_overrides: typing.Optional[dict] = None,
        image: str = None,
        *args: typing.Any,
        **kwargs: typing.Any,
    ) -> None:
        self.driver_cores = driver_cores
        self.driver_memory = driver_memory
        self.executor_cores = executor_cores
        self.executor_memory = executor_memory
        self.image = image
        self.spark_program_args = spark_program_args
        super().__init__(
            ssh_conn_id=f"my_emr_conn_ssh", command="", *args, **kwargs,
        )
        
    def execute(self, context: typing.Dict[str, typing.Any]) -> None:
        self.command = f"spark-submit \\
                --master yarn \\
                --deploy-mode cluster \\
                --conf spark.executorEnv.YARN_CONTAINER_RUNTIME_DOCKER_IMAGE={self.image} \\
                --conf spark.yarn.appMasterEnv.YARN_CONTAINER_RUNTIME_DOCKER_IMAGE={self.image} \\
                --executor-memory {self.executor_memory} \\
                --executor-cores {self.executor_cores} \\
                --driver-memory {self.driver_memory} \\
                --driver-cores {self.driver_cores} \\
                {self.spark_confs} \\
                run_task.py \\
                {self.spark_program_args}"
        super().execute(context)
        self.cleanup()
```

- 위에 pyspark-etl/etl_codes/cli_users_transform.py에서 users라는 ETL 결과 테이블을 S3에 write하였는데 이 테이블을 어떻게 SQL을 통해 쿼리할 수 있을까?


```python
s3_path = os.path.join(
    "s3://my-data-lake",
    f"{dest_db_prefix}_{db_suffix}",
    f"{dest_table_name}",
    f"{execution_datetime_tag.strftime('%Y_%m_%d_%H_%M_%S')}",
)

df.repartition(partition_num).write.mode("overwrite").option(
    "path", s3_path
).saveAsTable(f"{dest_db_prefix}_{db_suffix}.{dest_table_name}")
```

users라는 테이블을 s3://my-data-lake/etl_production/users/2020-08_06_00_00_00/ 에 parquet형태로 write한다. 그런다음에 Redshift/Athena에서 쿼리가 가능하도록 이 테이블에 대한 metadata를 Glue 카탈로그에 싱크하고 테이블을 etl_production이라는 external schema에 등록한다. 이는 etl_production이라는 external schema가 Redshift에 존재하고 EMR클러스터가 hive metastore로 Glue Catalog를 사용하다면 Pyspark의 saveAsTableoperation에서 자동으로 이루어진다. 그러면 이 테이블은 Redshift에서 etl_production.users의 형태로 존재하게 되어 SQL통하여 쿼리가 가능해진다.
