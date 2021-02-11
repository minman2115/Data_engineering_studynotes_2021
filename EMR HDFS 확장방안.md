Data_Engineering_TIL(20210212)

### [학습자료]

AWS 사용자모임 QnA 글을 읽고 공부한 내용입니다.

https://awskrug.slack.com/archives/C08A9550X/p1611028996088600

### [학습내용]

- 궁금한점

EMR core node 용량이 부족해서 확장을 하려고하는데, 해당 볼륨이 root 볼륨이 아니라 HDFS가 들어있는 볼륨이다. 결론적으로 HDFS를 확장하고 싶다. 

참고로 현재 EMR HDFS를 데이터를 저장하고, 사용중에 있다.

- 해답

1) core node EBS 볼륨을 100g까지 확장가능하다. 하지만 금액적으로나 기능적으로 s3에 저장하는 것이 좋다.

2) EMR을 계속 띄워놓고 사용중이라면, HDFS에 persist data나 로그 등이 계속 적재되기 때문에 실제로 데이터를 HDFS에 쓰지 않아도 사용 중인 용량이 계속 늘어날 수 있다. 

#### Core node를 추가하는 것으로 전체 HDFS 용량을 늘릴 수 있고 (rebalance가 일어남), EBS를 확장하는 방법으로도 용량을 늘릴 수 있다.

3) HDFS가 들어있는 EBS를 확장하는 작업을 진행해도 HDFS가 깨지거나 하는 문제는 없다.