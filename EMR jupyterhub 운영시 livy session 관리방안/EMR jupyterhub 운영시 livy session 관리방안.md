Data_Engineering_TIL(20210417)

[학습내용]

- EMR 운영시 문제가 발생하는 상황

다수의 사용자가 jupyterhub에 접속해서 notebook을 열고 작업할 경우 불필요한 livy session이 누적되어 livy session을 더이상 추가해서 사용할 수 없는 경우가 발생함

- 해결방안

정기적으로 오래된 휴무중인 livy session을 kill 해줘야함
