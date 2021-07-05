.

Data_Engineering_TIL(20210705)

#### [문제상황]

프로덕션 시스템에서 특정시간에 정기적으로 실행되는 spark batch job이 있다. 문제가 발생한 시점까지 항상 정상적으로 실행되었던 job이다. 하지만 어느날 갑자기 Error가 발생했다.

#### [원인]

해당 EMR Step의 stderror 로그를 확인해보니 다음과 같았다.

...

21/07/05 18:02:10 INFO Client : Submitting application_xxxxxxxxxxxxxxxxxxx to Resource Manager

...

21/07/05 19:19:20 ERROR TransportClient : Failed to send RPC RPC xxxxxxxxxxxxxxxx to /112.38.7.281:52638:java.nio.channels.ClosedChannelException

java.nio.channels.ClosedChannelException at io.netty.channel.AbstractChannel$AbstractUnsafe.newClosedChannelException(AbstractChannel.java:957)

...

해당 step이 Resource manager에 application_xxxxxxxxxxxxxxxxxxx로 제출되었고, "java.nio.channels.ClosedChannelException"이 발생하였다. ClosedChannelException가 발생한 원인을 찾기 위해 application log를 체크해보면 다음과 같다.

...

21/07/05 19:19:14 INFO ApplicationMaster: Final app status: FAILED, exitCode: 11,(reason: Due to executor failures all avaliable nodes are blacklisted)

21/07/05 19:19:17 INFO ShutdownHookManager: Shutdown hook called

...

가용한 모든 노드가 blacklist에 포함되어 job이 실패하였다. 그리고 아래와 같이 같은 시간대에 instance controller 로그에서 클러스터 scale down 이벤트가 발생한 것을 확인하였다. 


...

2021-07-05 19:19:15,243 INFO Poller: identified 4 instances in Task_04 to shrink: i-xxxxxxxxxxxxxxx, i-yyyyyyyyyyyyy, i-zzzzzzzzzzzzzz, i-kkkkkkkkkkkkkk

2021-07-05 19:19:15,243 INFO Poller: Task_05 active instance count 0 matches configuration

2021-07-05 19:19:15,243 INFO Poller: Task_08 active instance count 0 matches configuration

2021-07-05 19:19:15,243 INFO Poller: core active instance count 3 matches configuration

2021-07-05 19:19:15,243 INFO Poller: shrinkYarnNodes for 4 nodes to shrink and 0 nodes to unshrink

...

2021-07-05 19:19:56,436 INFO Poller: Update i-qcnweklqnlcw instanceStates RUNNING => DECOMMISIONED

2021-07-05 19:19:56,436 INFO Poller: Instance i-qcnweklqnlcw DECOMMISIONED at 2021-07-05T19:19:56.436Z and took 1162.547 seconds

2021-07-05 19:19:56,436 INFO Poller: Update i-zxcaclkanclkc instanceStates RUNNING => DECOMMISIONED

2021-07-05 19:19:56,436 INFO Poller: Instance i-zxcaclkanclkc DECOMMISIONED at 2021-07-05T19:19:56.436Z and took 737.527 seconds

2021-07-05 19:19:56,436 INFO Poller: Update i-wnecqlwneql instanceStates RUNNING => DECOMMISIONED

2021-07-05 19:19:56,436 INFO Poller: Instance i-wnecqlwneql DECOMMISIONED at 2021-07-05T19:19:56.436Z and took 1162.547 seconds

2021-07-05 19:19:56,436 INFO Poller: Update i-casdasxdadxa instanceStates RUNNING => DECOMMISIONED

2021-07-05 19:19:56,436 INFO Poller: Instance i-casdasxdadxa DECOMMISIONED at 2021-07-05T19:19:56.436Z and took 737.527 seconds

...

이것은 spark known issue로 scale in 이벤트가 발생하여 노드가 해제되었고, 이 해제된 노드는 blacklist에 추가되어 결국에 spark application이 fail이 난 것이다. https://github.com/apache/spark/blob/v2.4.5/resource-managers/yarn/src/main/scala/org/apache/spark/deploy/yarn/ApplicationMaster.scala 를 확인해보면 isAllNodeBlacklisted 값이 True가 되면, 즉 https://github.com/apache/spark/blob/v2.4.5/resource-managers/yarn/src/main/scala/org/apache/spark/deploy/yarn/YarnAllocatorBlacklistTracker.scala#L104 를 참고하면currentBlacklistedYarnNodes.size >= numClusterNodes 인 경우(def isAllNodeBlacklisted: Boolean = currentBlacklistedYarnNodes.size >= numClusterNodes) job이 실패하게 된다. 

이 이슈의 내용은 아래의 링크를 참고한다.

https://issues.apache.org/jira/browse/SPARK-29683?focusedCommentId=17223929&page=com.atlassian.jira.plugin.system.issuetabpanels%3Acomment-tabpanels#comment-17223929

위에 이슈의 내용을 요약하자면 scale in 이벤트로 노드해제가 발생하였고, 이렇게 해제된 노드들은 blacklist에 등록이 되었다. blacklist에 등록된 노드 수가 클러스터의 노드수와 같아져서 spark는 가용한 노드가 없는 것으로 판단해서 job에 필요한 container를 할당할 수 없었고 spark job이 fail이 난 것이다.

#### [해결방안]

방안 1. EMR 6.2.0 (spark 3.0.1) --> EMR 6.3.0 (spark 3.1.1-amazon) 로 버전 업그레이드

방안 2. 임시패치 적용

** 임시패치 내용 URL : http://awssupportdatasvcs.com/bootstrap-actions/Spark/SPARK-29683/

아래와 같이 EMR 재실행시 아래와 같은 내용으로 부트스트랩 액션에 반영하면 된다. 

Name: "spark_29683_patch"  

JAR Location: "s3://awssupportdatasvcs.com/bootstrap-actions/Spark/SPARK-29683/spark_29683_patcher.rb"  

Arguments: ""

** spark_29683_patcher.rb은 http://awssupportdatasvcs.com/bootstrap-actions/Spark/SPARK-29683/ 에서도 다운로드 가능

임시파일 적용시 참고사항

By default, this script will look for the patched RPM's in the "s3://awssupportdatasvcs.com/bootstrap-actions/Spark/SPARK-29683" bucket location. If you require the RPM's to be hosted in another bucket, please note:

The following s3 directory structure is required:

your-s3-bucket/path/  
├── spark-29683-patched-spark-2.4.5-amzn-0.tar.gz  
├── spark-29683-patched-spark-2.4.6-amzn-0.tar.gz  
├── spark-29683-patched-spark-2.4.7-amzn-0.tar.gz  
├── spark-29683-patched-spark-3.0.0-amzn-0.tar.gz  
└── spark-29683-patched-spark-3.0.1-amzn-0.tar.gz  
Hence, Bootstrap Action Install Script will also need to be updated at line 29:

@@s3_bucket = "s3://your-bucket/path"

** 패키지는 https://awssupportdatasvcs.com/bootstrap-actions/Spark/SPARK-29683/spark-29683-patched-spark-3.0.1-amzn-0.tar.gz 에서 다운로드 가능하기 때문에 다운로드 받아서 @@s3_bucket = "s3://your-bucket/path" 에 업로드 하면 된다.
