.

Data_Engineering_TIL(20210529)

- Airflow SSHOperator 사용예시 아키텍처

![architecture](https://user-images.githubusercontent.com/41605276/120059635-b4bae100-c08d-11eb-9ccc-27427569f095.png)

- Airflow SSHOperator 사용시 필요한 준비물

1) SSHOperation job을 수행하는 DAG

2) SSHOperation을 위한 pem key

- Airflow SSHOperator 사용을 위한 connection 설정방법

step 1) Airflow web server UI에 접속 후 상단 'Admin'메뉴 클릭 --> 'Connections' 메뉴 클릭

step 2) 좌측 상단에 '+' 버튼을 클릭해서 Connection을 추가하는 화면으로 이동

step 3) 아래와 같이 입력 후 save (예시)

============================================================================================

[EditConnection]

Conn id : emr_master_ssh_conn

Conn Type : SSH

Host : 182.982.17.292

Username : hadoop

Password : 



Port : 22

Extra : {"key_file":"/home/minman/airflow/dags/my_emr_pemkey","no_host_key_check":"true"}


============================================================================================


** 참고사항 

1) 'Host : 182.982.17.292' 는 접속하고자하는 EMR master node의 private ip임

2) 'Password : ' 는 특별히 기입해야할 내용이 없다는 의미임

3) 'my_emr_pemkey'라는 emr master node에 접속하기 위한 pem key를 /home/minman/airflow/dag/에 사전에 위치시켜야함

- 아래 URL 자료와 같이 SSHOperator를 이용하여 DAG 개발
