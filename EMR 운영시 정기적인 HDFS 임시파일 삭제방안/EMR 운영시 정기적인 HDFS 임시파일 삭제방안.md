
Data_Engineering_TIL(20210417)

[학습내용]

- EMR 운영시 문제가 발생하는 상황

다수의 인원이 EMR jupyterhub에 접속해서 spark 작업을 하면 HDFS 등에 임시파일이 쌓이게 되는데 방치하면 spark job을 실행할 수 없는 문제가 발생항

- 해결방안

Airflow 스케쥴링을 이용하여 EMR에 쌓여있는 HDFS 임시파일, jupyterhub log 등을 정기적으로 삭제

** 아래 URL을 참고할것
