.

Data_Engineering_TIL(20210622)

#### [학습자료]

AWS 블로그글

** URL : https://aws.amazon.com/ko/blogs/korea/best-practices-for-successfully-managing-memory-for-apache-spark-applications-on-amazon-emr/

#### [학습내용]

1) spark.executor.instances : 해당 spark application에 할당되는 익스큐터의 수로 spark.dynamicAllocation.enabled가 false 설정된 경우에 이 파라미터를 설정해야함

2) spark.default.parallelism : 파티션 수를 명시적으로 설정하지 않았을 때 join, reduceByKey 및 parallelize와 같은 변환에 의해 반환된 RDD의 파티션 수

3) maximizeResourceAllocation : 해당 spark application에 클러스터에서 사용 가능한 모든 리소스를 사용하려면 True로 설정하면됨

4) spark.network.timeout : 모든 네트워크 트랜잭션에 대한 시간 초과 값

5) spark.executor.heartbeatInterval : 드라이버에 대한 각 익스큐터의 하트비트 간격으로 이 값은 spark.network.timeout보다 작아야함

6) spark.memory.fraction – Spark 실행 및 스토리지에 사용되는 JVM 히프 공간의 비율로 이 수가 낮을수록 누출 및 캐싱된 데이터 제거가 더 자주 발생함

7) spark.memory.storageFraction : spark.memory.fraction에 의해 예비되는 비율로 표현됨. 이 수가 높을수록 실행에 사용 가능한 유휴 메모리가 적어지고 이는 곧 작업이 디스크로 더 자주 누출됨을 의미함

8) spark.yarn.scheduler.reporterThread.maxFailures : YARN에서 애플리케이션을 장애로 처리하기 전 허용되는 최대 익스큐터 실패 횟수

9) spark.rdd.compress : 이 속성을 true로 설정하면 RDD 압축을 통해 일부 추가 CPU 시간을 소비하는 대신 상당한 공간을 절약할 수 있음

10) spark.shuffle.compress : 이 속성을 true로 설정하면 map 연산 output이 압축되어 공간을 절약함

11) spark.shuffle.spill.compress : 이 속성을 true로 설정하면 중간 shuffle spill 파일을 압축함

12) spark.sql.shuffle.partitions : 결합 및 집계를 위한 파티션 수를 설정합니다.

13) spark.serializer : 데이터를 직렬화 또는 역직렬화할 시리얼라이저를 설정함. 일반적으로는 Java 기본 시리얼라이저보다 빠르고 간결한 Kyro(org.apache.spark.serializer.KryoSerializer)를 시리얼라이저를 권장함
