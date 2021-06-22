.

Data_Engineering_TIL(20210622)

#### [학습자료]

AWS 블로그글

** URL : https://aws.amazon.com/ko/blogs/korea/best-practices-for-successfully-managing-memory-for-apache-spark-applications-on-amazon-emr/

#### [학습내용]

** 일괄 r5.4xlarge (16 core, 128 memory) 사양으로 마스터 1, 코어 3일 경우

#### step 1) spark.executor.cores 설정

익스큐터에 많은 수의 코어를 할당하면 익스큐터 수가 적어지고 병렬 프로세스가 감소함. 반면에 적은 수의 가상 코어를 할당하면 익스큐터 수가 많아져 더 많은 양의 I/O 작업을 유발합니다. 일반적으로는 매직넘버가 5로 알려져 있음 (spark.executors.cores = 5 (vCPU))

#### step 2) spark.executor.memory 설정

먼저 인스턴스당 익스큐터 수를 계산

`인스턴스당 익스큐터 수 = (인스턴스당 코어 수 - 1)/ spark.executors.cores`

인스턴스당 익스큐터 수 = (16 - 1)/ 5 = 15 / 5 = 3 (가까운 정수로 내림)

그 다음에 익스큐터 메모리를 계산

`총 익스큐터 메모리 = 인스턴스당 RAM / 인스턴스당 익스큐터 수`

총 익스큐터 메모리 = 128 / 5 = 25 (가까운 정수로 내림)

이 총 익스큐터 메모리는 익스큐터 메모리와 오버헤드(spark.yarn.executor.memoryOverhead)를 포함한다. 이 총 익스큐터 메모리의 10%를 메모리 오버헤드에 할당하고 나머지 90%는 익스큐터 메모리에 할당해야 한다.

`spark.executors.memory = 총 익스큐터 메모리 * 0.90`

spark.executors.memory = 25 * 0.9 = 23 (가까운 정수로 내림)

`spark.yarn.executor.memoryOverhead = 총 익스큐터 메모리 * 0.10`

spark.yarn.executor.memoryOverhead = 25 * 0.1 = 2 (가까운 정수로 올림)

#### step 3) spark.driver.memory는 spark.executors.memory와 같은 값으로 설정

#### step 4) spark.driver.cores도 spark.executors.cores와 같은 값으로 설정

#### step 5) spark.executor.instances 설정

`spark.executor.instances = (인스턴스당 익스큐터 수 * 코어 인스턴스 수) - 드라이버용 1개`

spark.executor.instances = (3 * 3) - 1 = 8

#### step 6) spark.default.parallelism 설정

`spark.default.parallelism = spark.executor.instances * spark.executors.cores * 2`

spark.default.parallelism = 8 * 5 * 2 = 80

** 주의사항 : 각 파티션의 크기를 예측하고 coalesce 또는 repartition을 사용하여 적절히 이 수를 조절할 것을 권장
