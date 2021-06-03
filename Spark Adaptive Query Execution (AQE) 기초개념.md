.

Data_Engineering_TIL(20210603)

[학습 참고자료]

spark 도큐먼트 : https://spark.apache.org/docs/latest/sql-performance-tuning.html

AWS EMR 도큐먼트 : https://docs.aws.amazon.com/emr/latest/ReleaseGuide/emr-spark-performance.html

[학습내용]

- Adaptive Query Execution (AQE) 

1) spark sql에서 runtime statistics를 고려해서 가장 효율적인 query execution plan을 채택하는 기법

2) spark 3.0 버전 이상 사용가능

3) 'spark.sql.adaptive.enabled' True 또는 False로 옵션적용 가능

** EMR 6 버전이상에서는 기본적으로 True 임

4) Adaptive Query Execution는 크게 3가지 개념이 있음

coalescing post-shuffle partitions

converting sort-merge join to broadcast join

skew join optimization

- coalescing post-shuffle partitions

1) Shffle Partitions 수를 적절한 수로 조절해주는 옵션

2) spark.sql.adaptive.coalescePartitions.enabled True 또는 False 설정으로 적용가능함

3) EMR에서는 spark.sql.shuffle.partitions가 설정되어 있지 않는 이상 spark.sql.adaptive.coalescePartitions.enabled는 기본적으로 True임

- converting sort-merge join to broadcast join

말 그대로 sort-merge를 broadcast join으로 컨버팅하여 shuffle 연산을 피하는 기법

- skew join optimization

1) sort-merge 시  다이나믹하게 데이터 스큐를 조절해서 task들의 데이터 사이즈를 고르게 하는 기법

2) spark.sql.adaptive.skewJoin.enabled True 또는 False 설정으로 적용가능함
