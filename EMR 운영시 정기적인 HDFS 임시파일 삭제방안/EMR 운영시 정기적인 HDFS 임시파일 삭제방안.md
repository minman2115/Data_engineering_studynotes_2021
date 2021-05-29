
Data_Engineering_TIL(20210417)

[학습내용]

- EMR 운영시 문제가 발생하는 상황

다수의 인원이 EMR jupyterhub에 접속해서 spark 작업을 하면 HDFS 등에 임시파일이 쌓이게 되는데 방치하면 spark job을 실행할 수 없는 문제가 발생함

- 해결방안

Airflow 스케쥴링을 이용하여 EMR에 쌓여있는 HDFS 임시파일, jupyterhub log 등을 정기적으로 삭제

** Airflow 스케쥴링을 적용하는 내용은 https://minman2115.github.io/DE_TIL208/ 을 참고할 것

- HDFS temp file을 Cleaning 하는 bash script 예시

https://github.com/minman2115/Data_engineering_studynotes_2021/blob/master/EMR%20%EC%9A%B4%EC%98%81%EC%8B%9C%20%EC%A0%95%EA%B8%B0%EC%A0%81%EC%9D%B8%20HDFS%20%EC%9E%84%EC%8B%9C%ED%8C%8C%EC%9D%BC%20%EC%82%AD%EC%A0%9C%EB%B0%A9%EC%95%88/cleaner_example.zip
