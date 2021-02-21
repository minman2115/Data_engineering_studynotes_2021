Data_Engineering_TIL(20210221)

### [학습자료]

AWS 사용자모임 QnA 글을 읽고 공부한 내용입니다.

URL : https://awskrug.slack.com/archives/C3D5K9LHY/p1613440043023200

### [학습내용]

- 궁금한점

고객의 개인정보 같은 데이터들을 Data Source에서 가지고 와서 S3(Data Lake)에 저장하려 하는데, 이런 민감한 데이터들은 어떻게 처리를 해서 Data Lake나 DW에 저장을 해야하는지 궁금함. 민감한 데이터 자체를 난독화를 해서 저장을 해야하지는 않을까 생각함.

- 방안

방안 1) source에서 가져올때부터 masking 하는 방법도 있음. 핵심은 데이터 레이크에 적재되기 이전부터 masking 되는 방안임. source에서 가져오는 job은 주로 Spark job으로 하는 방안이 일반적임. 거기에서 masking하는 function을 만들어 사용하면 됨.

DMS(Database Migration Service)에서도 Masking Function을 만들어서 위와 같이 처리하는 방식에 대해서 제안하고 있음

** 참고자료 : https://aws.amazon.com/ko/blogs/database/automating-database-migration-and-refreshing-activities-with-aws-dms

방안 2) Athena의 UDF(user-defined functions)에서 개인식별정보, 신용카드 정보와 같이 민감한 정보를 XXXXX 와 같이 마스킹 처리하는 방안이 있음

** 참고자료 : https://aws.amazon.com/ko/blogs/big-data/redacting-sensitive-information-with-user-defined-functions-in-amazon-athena

방안 3) AWS S3/Redshift에 저장할때부터 KMS - S3를 이용해서 암호화 해서 저장을 하는 방안도 있음.

(이것은 난독화/마스킹 라기 보다는 파일이 AWS 외부로 유출되더라도 확인이 불가능하게 처리하는 방안임 - 객체 암호화)

RDS/Redshift에 지정하는 암호화 방식도 비슷한 형태로 블럭 암호화를 할 수 있음

** 참고자료 : https://docs.aws.amazon.com/ko_kr/redshift/latest/mgmt/working-with-db-encryption.html

방안 4) 개인정보가 포함된 데이터를 RSA + AES 를 이용해서 암호화해서 저장

데이터 종류에 따라서 RSA 또는 AES 로 저장하면 됨. AES 128 -> 256 으로 법령이 변경되고 각 데이터 종류별로 별도의 키를 이용해 별개로 암호화를 함. (서로 볼 수 없게)

위와 같은 경우에는 암호화 기준은 법령에서 정하는 기준이 있기 때문에 반드시 법령이 정하는 기준을 충족해야함.

나라에서 정하는 익명성이란 개념이 있어 마스킹의 수준이나 해시를 한다 하더라도 특정 조건이 만족하면 조인해서 사용할 수 있는 모든 정보가 개인정보가 되기 때문에 상당히 조심해서 다루어야함.

방안 5) RDS에서 S3로 옮기는 spark code에서부터 꼭 필요하지 않은 민감정보는 컬럼 자체를 제외하고 옮기게 했고, 식별이 가능한 키 값의 경우는 SHA512로 해시하여 실서비스에서의 키와 매칭이 어렵게 하는 방안도 있음