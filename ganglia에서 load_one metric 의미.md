.

Data_Engineering_TIL(20210629)

[학습내용]

- load_one : the load average over one minute


- load_average란 프로세스의 상태가 R(러닝중인 상태로 프로세스가 실제로 CPU 자원을 소모하고 있는 실행중인 상태) 또는 D(디스크 또는 네트워크 등 I/O 작업을 대기중인 프로세스의 상태)인 프로세스들의 개수를 1분, 5분, 15분 단위로 평균값을 낸 수치


- load_average가 높다는 것은 많은 프로세스가 현재 실행중이거나, 네트워크 또는 디스크 작업을 처리하기 위한 대기상태에 있다는 것을 의미함
