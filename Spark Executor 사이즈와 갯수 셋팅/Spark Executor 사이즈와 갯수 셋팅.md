.

Data_Engineering_TIL(20201231)

[참고자료]

'Jaemun Jung'님 블로그글 "[Apache Spark] Executor 사이즈와 개수 정하기" 을 학습하고 정리한 노트입니다.

URL : https://jaemunbro.medium.com/spark-executor-%EA%B0%9C%EC%88%98-%EC%A0%95%ED%95%98%EA%B8%B0-b9f0e0cc1fd8

[학습내용]

![1](https://user-images.githubusercontent.com/41605276/103435619-95419880-4c54-11eb-94e7-46fbc7ca1342.jpg)

![2](https://user-images.githubusercontent.com/41605276/103435623-a38fb480-4c54-11eb-937c-71000dcc76b5.jpg)

![3](https://user-images.githubusercontent.com/41605276/103435647-ef425e00-4c54-11eb-852d-7da4df32e57b.png)

- EMR specification 현황

1) Core 서버 : m5.24xlarge 10대

2) 서버당 vCore : 96개

3) 서버당 Memory : 384GiB

- 노드당 executor 수 : 96 vcore / 4 = 24개 (executor당 core 수 4개로 지정했다고 가정했을 시)

executor 당 core 수를 먼저 정의하고, 이를 통해 vCore에서 활용할 수 있는 전체 executor 수가 정의될 수 있다. 

- executor당 memory : 360GB / 24 = 15G

(실제로 가용 memory는 Resource Manager - Cluster metrics에서 확인가능. 여기서는 360GB로 가정함)

** 참고사항 : m5.xlarge (4core/16GiB memory)를 core로 2대 띄운 EMR 클러스터의 가용 메모리는 24GB

![4](https://user-images.githubusercontent.com/41605276/103435687-89a2a180-4c55-11eb-9a2a-fe1ed9e8d47e.png)

