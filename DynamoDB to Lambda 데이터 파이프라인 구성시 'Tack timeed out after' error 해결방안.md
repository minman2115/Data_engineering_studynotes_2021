Data_Engineering_TIL(20210213)

### [학습자료]

AWS 사용자모임 QnA 글을 읽고 공부한 내용입니다.

URL : https://awskrug.slack.com/archives/C08A9550X/p1612501401334800

### [학습내용]

- 현재상황

1) DynamoDB=> DynamoDB stream => Lambda => RDS (mariaDB)로 데이터 파이프라인을 구성함

2) 위와 같이 데이터 파이프라인을 구성하고 나서 데이터를 흘려보고 확인해보니 데이터가 개수가 달랐음

3) Lambda로그를 확인해보니 Tack timeed out after 현상을 확인함

4) 그래서 메모리도 늘려보고 시간도 늘려봤지만 최대시간 만큼 계속 Tack timeed out after 현상이 확인됨

5) 30~50MB 밖에 안되는 데이터인데 왜 그런지 잘 모르겠음

6) Lambda 코드는 Records 내용들을 확인하고 mysql pool을 이용해서 update 해주고 있음

7) RDS 로그를 확인해보니 통신패킷 읽기오류가 발생함

- 해결방안

방안 1) 

일반적인 서버환경에서는 고정적인 서버대수에 connection pool을 관리해서 접속이 넘치지 않도록 하지만 람다 환경에서는 사용량에 따라 서버 자체가 증식하기 때문에 pool걸면 오히려 max connection error가 발생함. 다시말해서 DynamoDB에서 stream으로 람다를 실행하면 실행 건마다 서버가 켜진다는 것임

그래서 javascript로 보면 npm모듈중에 람다용 mysql 모듈이 있음. 이 모듈의 핵심은 pool을 안쓰고 접속이 될때까지 재시도하는 방식인데 이 모듈을 활용하는 방법이 있음

** 참고자료 : https://www.npmjs.com/package/serverless-mysql

--> 질문자가 방안 1)을 참고해서 NPM 모듈 Serverless MySQL을 사용하니 실제로 7~9초 대로 시간이 줄어들고 안정적으로 데이터가 들어오고 있는 것을 확인함. 하지만 아직 오류 발생 시 알려주는 부분을 구현하지 않아서 모니터링으로만 확인중이지만 오류 없이 잘 되는 것으로 확인됨.

** 방안 1) 답변자의 추가 피드백 

벡엔드 배치작업을 할때는 주로 바로 람다로 실행 안하고 SQS넣어서 돌리는 것을 추천함. SNS와 SQS가 큐에 넣으면 람다를 실행시켜준다는 점에서 비슷한데 확실히 실행되도록 보장하려면 SQS를 사용해야함

참고자료 : https://jaemunbro.medium.com/aws-%EB%A9%94%EC%8B%9C%EC%A7%95%EC%84%9C%EB%B9%84%EC%8A%A4-%EB%B9%84%EA%B5%90-kinesis-sqs-sns-ab397a07cb1d

방안 2) 

AWS에서도 위와 같은 상황때문에 RDS Proxy 기능을 출시하였기에 이 RDS Proxy를 사용하는 방안도 있음. 단, 비용이 발생한다는 점과 비밀키를 직접 지정할수 없기 때문에 디비를 한두대 개발용으로 돌리기에는 불편할 수 있음.

** 참고자료 : https://aws.amazon.com/ko/blogs/korea/amazon-rds-proxy-now-generally-available/

방안 3) 

반드시 실시간성이 보장되야 하는게 아니면 SQS 같은데 보내고 Pool에 맞게 속도를 조절해서 넣는 방법도 있음

- 참고사항

Kinesis DataSteam을 통해 들어오는 데이터를 Lambda 로 처리했을때 위와 유사한 장애가 나는 경우도 있음. Lambda에 메모리나 실행 시간에 제약을 심하게 받았었는데 결국은 SQS를 통해서 처리하고 Daed-letter-queue를 활성화 시켜서 처리가 가능하였음. 다이렉트로 쏘기에는 Lambda 문제도 있고 RDS의 퍼포먼스 문제도 있기에 비용적으로는 SQS를 최대한 활용하는게 좋은 방법임.