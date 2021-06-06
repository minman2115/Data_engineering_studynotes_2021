.

Data_Engineering_TIL(20210606)

[학습자료]

AWSKRUG '데이터 엔지니어가 실무에서 맞닥뜨리는 문제들' 크로키닷컴 강웅석 자료를 공부하고 정리한 내용입니다.

URL : 

[학습내용]

- Schema evolution 상황 발생

개발팀 "판매실적 테이블 스키마가 변경되었습니다", "판매실적 로그테이블 스키마가 변경되었습니다."

아래와 같이 spark application에서 Error가 뻥뻥터짐..

![image](https://user-images.githubusercontent.com/41605276/120914592-199fb800-c6da-11eb-871f-26bee869dd68.PNG)

데이터팀 "..."

** 참고사항

spark application은 parquet file을 소비하고 있음

이런 상황에서 parquet/ORC의 스키마정보는 file 내에 있기 때문에 file을 실질적으로 읽어봐야지 스키마를 알 수 있음

file을 열어보기 전까지는 스키마가 정확히 어떻게 되어 있는지 알기가 어려움

- 대응방안 요약

External metastore(대표적으로 hive, glue)에서 스키마를 명시적으로 주입한다.

이렇게 하면 file을 굳이 읽지 않아도 되고, 관리도 용이하다.

- Glue를 이용한 schema evolution (수동처리 방법)

step 1) data source의 schema 변경을 감지 (보통은 Error 발생으로 확인)

step 2) glue의 metastore의 해당 table 콘솔화면으로 접속해서 엔지니어가 직접 schema evolution을 진행

`edit schema --> add column` 또는 `edit table --> Table properties에서 직접 json 수정`

** 만약에 EMRFS consistent view를 사용한다면 바뀐 table에 대해서 emrfs sync가 필요하다.

Evolution 시점에 이미 띄워져 있는 클러스터는 데이터를 제대로 write 할 수 없음

시점 이후에 띄운 EMR은 괜찮음

- Glue를 이용한 schema evolution (자동처리 방법)

아래 예시와 같이 spark + glue api를 이용하여 자동으로 코드를 구현하면 됨

![image](https://user-images.githubusercontent.com/41605276/120914934-01309d00-c6dc-11eb-8026-8ee21874132d.png)

** 주의사항

테이블이 큰 경우, schema part가 여러개로 쪼개지는데 마지막 part를 수동으로 찾아줘야 함
