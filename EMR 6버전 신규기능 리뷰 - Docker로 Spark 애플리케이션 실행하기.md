.

Data_Engineering_TIL(20210606)

[학습자료]

1) AWS 공식가이드 'Run Spark Applications with Docker Using Amazon EMR 6.x'

URL : https://docs.aws.amazon.com/emr/latest/ReleaseGuide/emr-spark-docker.html

2) AWS 블로그글 'Amazon EMR 6.0.0을 사용하여 Docker로 Spark 애플리케이션 실행하기'

https://aws.amazon.com/ko/blogs/korea/run-spark-applications-with-docker-using-amazon-emr-6-0-0-beta/

[학습내용]

- 기본개념 및 장점

#### spark-submit을 하면 YARN은 지정된 Docker 이미지를 가져온 다음에 Docker 컨테이너 내부에서 Spark 애플리케이션을 실행하는 개념

EMR 6버전을 사용하면, Spark 사용자가 Docker Hub 또는 ECR의 Docker 이미지를 사용해서 개발환경과 라이브러리 종속성을 정의할 수 있음. 사용자는 Docker를 사용하여 종속성을 손쉽게 정의하고 개별 작업에 사용할 수 있으므로 개별 클러스터 마다 종속성을 설치할 필요가 없는게 장점임.

따라서 종속성을 쉽게 정의하고 분리할 수 있음. 그러면 작업 실행에 필요한 라이브러리를 사용하여 EMR 클러스터에서 인스턴스를 준비하거나 부트스트래핑하는 데 소요되는 시간을 줄일 수 있음.

이런 개념이 가능할 수 있었던 것은 EMR 6.0에는 Hadoop 3.1.0이 포함되어 있어 YARN NodeManager가 클러스터의 호스트 시스템 또는 Docker 컨테이너 내부에서 직접 컨테이너를 시작할 수 있음. 이는 Docker 컨테이너는 애플리케이션의 코드가 YARN NodeManager 및 기타 애플리케이션의 실행 환경과 분리되어 실행될 수 있다는 의미임.

- Docker로 spark application 실행하는 기본 프로세스

#### step 1) docker 이미지 생성 및 ECR에 등록

로컬PC 또는 EC2 작은거를 하나 띄워서 거기서 작업하면 되는 내용이다.

이미지에 포함할 패키지 및 구성을 정의하는 Dockerfile을 사용하여 생성해야 한다.

아래에 Dockerfile에서 생성된 Docker 이미지에는 Python 3 및 numpy Python 패키지를 설치해서 올리게된다. 이 Dockerfile은 Amazon Linux 2와 Amazon Corretto JDK 8을 사용한다.

이 Dockerfile은 ECR을 사용하게 된다. ECR은 개발자가 Docker 컨테이너 이미지를 저장, 관리 및 배포할 수 있도록 완전히 관리되는 Docker 컨테이너 레지스트리이다. 

태그를 지정하여 ECR에 업로드한다. 업로드한 후에 PySpark 작업을 실행하면 ECR에서 Docker 이미지를 참조한다. 이 참조한 이미지를 갖고 EMR 클러스터에서 SSH를 사용하여 코어 노드에 연결하고 다음 명령을 실행하여 PySpark Dockerfile 예제에서 로컬 Docker 이미지를 빌드한다.

먼저 예제에 사용할 디렉토리와 Dockerfile을 만든다.


```bash
$ mkdir my_pyspark

$ vim my_pyspark/Dockerfile
FROM amazoncorretto:8

RUN yum -y update
yum -y install yum-utils
RUN yum -y groupinstall development

RUN yum list python3*
RUN yum -y install python3 python3-dev python3-pip python3-virtualenv

RUN python -V
RUN python3 -V

ENV PYSPARK_DRIVER_PYTHON python3
ENV PYSPARK_PYTHON python3

RUN pip3 install --upgrade pip
RUN pip3 install numpy panda

RUN python3 -c "import numpy as np"
```

EMR에서 spark-submit을 docker로 하게되면 Docker 컨테이너를 사용하여 YARN 애플리케이션을 실행할 때 YARN은 작업을 제출할 때 지정한 Docker 이미지를 다운로드 한다. YARN이 이 Docker 이미지를 사용하려면 ECR(Docker 레지스트리)을 설정해줘야 한다.


```bash
# PySpark Dockerfile의 내용을 붙여 넣고 다음 명령을 실행하여 Docker 이미지를 생성
$ sudo docker build -t local/pyspark-example my_pyspark/

# emr-docker-examples이라는 ECR repository를 예시로 생성
$ aws ecr create-repository --repository-name emr-docker-examples

# 123456789123.dkr.ecr.us-east-1.amazonaws.com라는 예시 ECR endpoint를 기입하여 로컬로 빌드된 이미지를 태깅 및 ECR에 업로드
$ sudo docker tag local/pyspark-example 123456789123.dkr.ecr.us-east-1.amazonaws.com/emr-docker-examples:pyspark-example
    
$ sudo docker push 123456789123.dkr.ecr.us-east-1.amazonaws.com/emr-docker-examples:pyspark-example
```

아래와 같이 spark-submit하는 간단한 pyspark script를 준비한다.

(아래 spark-submit할때 main.py를 말함)


```python
from pyspark.sql import SparkSession
spark = SparkSession.builder.appName("docker-numpy").getOrCreate()
sc = spark.sparkContext

import numpy as np
a = np.arange(15).reshape(3, 5)
print(a)
```

#### step 2) Docker 레지스트리 configuration 을 적용한 EMR 클러스터 구동

당연한 얘기지만 ECR을 사용하는 경우 클러스터는 ECR 인스턴스를 신뢰하도록 구성되어야 하며 클러스터가 ECR의 Docker 이미지를 사용하려면 인증을 구성해야 하는점을 주의해야 한다.

Docker 이미지를 확인하는 데 사용되는 특정 레지스트리를 신뢰하도록 Docker를 구성해야 한다. 기본 신뢰 레지스트리는 local(프라이빗) 및 centos(퍼블릭 Docker 허브)이다. `/etc/hadoop/conf/container-executor.cfg`의 docker.trusted.registrations을 재정의하여 다른 도커허브 또는 ECR을 사용할 수 있다. 이 configuration을 재정의하려면 EMR classification API을 container-executor classification key와 함께 사용한다.

아래 예시는 공용 리포지토리(your-public-repo)와 ECR 레지스트리(123456789123.dkr.ecr.us-east-1.amazonaws.com)를 모두 신뢰하도록 클러스터를 구성하는 configuration 예시임다. ECR을 사용하는 경우 이 endpoint을 특정 ECR endpoint로 바꾸면된다.  Docker Hub를 사용할 때는 이 리포지토리 이름을 실제 리포지토리 이름으로 바꾸면 된다.

container-executor.json(EMR configuration file)


```json
[
  {
    "Classification": "container-executor",
    "Configurations": [
        {
            "Classification": "docker",
            "Properties": {
                "docker.trusted.registries": "local,centos,123456789123.dkr.ecr.us-east-1.amazonaws.com",
                "docker.privileged-containers.registries": "local,centos, 123456789123.dkr.ecr.us-east-1.amazonaws.com"
            }
        }
    ]
  }
]
```

그런 다음에 아래에 AWS CLI 명령어 양식을 사용하여 EMR 6 클러스터를 구동한다. 


```bash
$ export KEYPAIR=<Amazon EC2 키 페어 이름>
$ export SUBNET_ID=<클러스터에 배포할 서브넷의 ID>
$ export INSTANCE_TYPE=<사용할 인스턴스 유형의 이름>
$ export REGION=<클러스터가 배포되는 리전>

$ aws emr create-cluster \
    --name "EMR-6- Cluster" \
    --region $REGION \
    --release-label emr-6 \
    --applications Name=Hadoop Name=Spark \
    --service-role EMR_DefaultRole \
    --ec2-attributes KeyName=$KEYPAIR,InstanceProfile=EMR_EC2_DefaultRole,SubnetId=$SUBNET_ID \
    --instance-groups InstanceGroupType=MASTER,InstanceCount=1,InstanceType=$INSTANCE_TYPE InstanceGroupType=CORE,InstanceCount=2,InstanceType=$INSTANCE_TYPE \
    --configuration file://container-executor.json
```

#### step 3) 생성한 도커 이미지를 이용하여 spark-submit 하기

먼저 생성한 EMR 클러스터의 각 인스턴스에서 ECR에 액세스할 수 있는지 확인한다.

docker 명령을 사용하여 ECR에 액세스하려면 먼저 자격 증명을 생성해야 한다. YARN이 ECR의 이미지에 액세스할 수 있도록 하려면 컨테이너 환경 변수 `YARN_CONTAINER_RUNTIME_DOCKER_CLIENT_CONFIG`를 사용하여 생성된 자격 증명에 대한 참조를 전달해야 한다.

코어 노드 중 하나에서 다음 명령을 실행하여 ECR 계정에 대한 로그인 라인을 가져온다.


```bash
$ aws ecr get-login --region us-east-1 --no-include-email
```

get-login 명령은 Docker CLI 명령을 생성하여 자격 증명을 생성한다. 그런 다음에 get-login에서 출력을 복사하여 아래와 같이 실행한다.


```bash
$ sudo docker login -u AWS -p <password> https://<account-id>.dkr.ecr.us-east-1.amazonaws.com
```

이 명령은 `/root/.docker` 폴더에 `config.json` 파일을 생성한다.  클러스터에 제출된 작업이 이 파일을 사용하여 ECR을 인증할 수 있도록 이 파일을 HDFS로 복사한다.

아래 명령을 실행하여 config.json 파일을 홈 디렉토리에 복사한다.


```bash
$ mkdir -p ~/.docker
$ sudo cp /root/.docker/config.json ~/.docker/config.json
$ sudo chmod 644 ~/.docker/config.json
```

아래 명령을 실행하여 config.json을 HDFS에 저장하여 클러스터에서 실행 중인 작업에 사용할 수 있도록 한다.


```bash
$ hadoop fs -put ~/.docker/config.json /user/hadoop/
```

그러면 YARN은 작업 실행 중에 Docker 이미지 레지스트리로 ECR에 액세스하고 container를 가져올 수 있게 된다.

그러면 spark-submit을 하면 된다.

spark job을 submit하려면 Docker 이름을 참조해야 한다. EMR configuration 추가 구성 매개 변수를 정의하여 작업 실행이 Docker를 런타임으로 사용하도록 한다. 

즉 아래와 같이 spark-submit을 할때 아래와 같은 변수를 사용하여 Docker 런타임과 이미지를 지정하는 것이다.

`YARN_CONTAINER_RUNTIME_TYPE=docker`

`YARN_CONTAINER_RUNTIME_DOCKER_IMAGE={DOCKER_IMAGE_NAME}`

아까 위에서 설정한 YARN_CONTAINER_RUNTIME_DOCKER_CLIENT_CONFIG는 ECR 인증에 사용되는 자격 증명이 포함된 config.json도 명시를 해야한다.


```bash
$ DOCKER_IMAGE_NAME=123456789123.dkr.ecr.us-east-1.amazonaws.com/emr-docker-examples:pyspark-example
$ DOCKER_CLIENT_CONFIG=hdfs:///user/hadoop/config.json
$ spark-submit --master yarn \
--deploy-mode cluster \
--conf spark.executorEnv.YARN_CONTAINER_RUNTIME_TYPE=docker \
--conf spark.executorEnv.YARN_CONTAINER_RUNTIME_DOCKER_IMAGE=$DOCKER_IMAGE_NAME \
--conf spark.executorEnv.YARN_CONTAINER_RUNTIME_DOCKER_CLIENT_CONFIG=$DOCKER_CLIENT_CONFIG \
--conf spark.executorEnv.YARN_CONTAINER_RUNTIME_DOCKER_MOUNTS=/etc/passwd:/etc/passwd:ro \
--conf spark.yarn.appMasterEnv.YARN_CONTAINER_RUNTIME_TYPE=docker \
--conf spark.yarn.appMasterEnv.YARN_CONTAINER_RUNTIME_DOCKER_IMAGE=$DOCKER_IMAGE_NAME \
--conf spark.yarn.appMasterEnv.YARN_CONTAINER_RUNTIME_DOCKER_CLIENT_CONFIG=$DOCKER_CLIENT_CONFIG \
--conf spark.yarn.appMasterEnv.YARN_CONTAINER_RUNTIME_DOCKER_MOUNTS=/etc/passwd:/etc/passwd:ro \
--num-executors 2 \
main.py -v
                
# spark-submit이 완료되면 YARN 애플리케이션 ID를 확인하고 다음 명령을 사용하여 PySpark 작업의 출력을 확인할 수 있음
$ yarn logs --applicationId application_id | grep -C2 '\[\['
LogLength:55
LogContents:
[[ 0  1  2  3  4]
 [ 5  6  7  8  9]
 [10 11 12 13 14]]
```

** Docker와 함께 Spark를 사용할때 주의사항

1) docker 패키지와 CLI는 코어 및 작업 노드에만 설치된다.

2) spark-submit 명령은 항상 EMR 클러스터의 마스터 인스턴스에서 실행되어야 함

3) 클러스터 시작 시 추가 매개 변수를 정의하려면 아래와 같이 Classification API( Container-executor 분류 키)를 사용하여 Docker 이미지를 확인하는 데 사용되는 Docker 레지스트리를 정의해야함.

`docker.trusted.registries`

`docker.privileged-containers.registries`

4) Docker 컨테이너에서 Spark 애플리케이션을 실행하려면 다음 구성 옵션이 필요함

`YARN_CONTAINER_RUNTIME_TYPE=docker`

`YARN_CONTAINER_RUNTIME_DOCKER_IMAGE={DOCKER_IMAGE_NAME}`

5) ECR을 사용하여 Docker 이미지를 검색할 때는 자체 인증을 위해 아래와 같이 클러스터를 구성해야함. 

`YARN_CONTAINER_RUNTIME_DOCKER_CLIENT_CONFIG={DOCKER_CLIENT_CONFIG_PATH_ON_HDFS}`

6) 작업을 실행하는 사용자를 Docker 컨테이너에서 식별할 수 있도록 /etc/passwd 파일을 컨테이너에 마운트한다.

`YARN_CONTAINER_RUNTIME_DOCKER_MOUNTS=/etc/passwd:/etc/passwd:ro`

7) Spark와 함께 사용되는 모든 Docker 이미지는 Docker 이미지에 Java가 설치되어 있어야 합니다.

8) EMR 클러스터의 각 인스턴스에서 ECR에 액세스할 수 있는지 확인해야 한다.
