.

Data_Engineering_TIL(20210622)

#### [학습자료]

AWS EMR 도큐먼트

** URL 

1) https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-master-core-task-nodes.html

2) https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-plan-instances-guidelines.html#emr-plan-spot-YARN

#### [학습내용]

#### 1) 마스터 노드 

#### 클러스터를 관리하는 노드임 따라서 일반적으로 분산 애플리케이션의 마스터 구성 요소를 실행함

예를 들면 마스터 노드는 애플리케이션용 리소스를 관리하는 YARN ResourceManager 서비스는 물론, HDFS NameNode 서비스도 실행함 또한 클러스터로 전송된 작업의 상태를 추적하고 인스턴스 그룹의 상태를 모니터링함

#### 2) 코어 노드

#### HDFS DataNode 데몬을 실행하여 HDFS의 일부 구성노드임 

#### Task Tracker 데몬을 실행하고 spark job 등 Hadoop 기반 에코 애플리케이션을 실행하는 노드임

예를 들면, 코어 노드는 YARN NodeManager 데몬, 하둡 MapReduce 작업 및 Spark job을 실행함.

주의해야할 점은 HDFS 데몬을 제거하거나 코어 노드를 종료하면 데이터가 손실될 위험이 있음 따라서 스팟 인스턴스를 사용하도록 코어 노드를 구성할 때는 이를 염두해야함.

#### 3) 테스크 노드

#### Hadoop MapReduce 및 Spark job 등 어플리케이션을 실행하는 노드임. 

#### 테스크 노드는 HDFS DataNode 데몬을 실행하지 않으며 HDFS에 데이터를 저장하지도 않음. 


** 참고사항

- YARN 노드 레이블 기능 

Because Spot Instances are often used to run task nodes, EMR has default functionality for scheduling YARN jobs so that running jobs do not fail when task nodes running on Spot Instances are terminated. Amazon EMR does this by allowing application master processes to run only on core nodes. The application master process controls running jobs and needs to stay alive for the life of the job. 

EMR release version 5.19.0 and later uses the built-in YARN node labels feature to achieve this. (Earlier versions used a code patch). Properties in the `yarn-site` and `capacity-scheduler` configuration classifications are configured by default so that the YARN capacity-scheduler and fair-scheduler take advantage of node labels. Amazon EMR automatically labels core nodes with the CORE label, and sets properties so that application masters are scheduled only on nodes with the CORE label. Manually modifying related properties in the yarn-site and capacity-scheduler configuration classifications, or directly in associated XML files, could break this feature or modify this functionality.
