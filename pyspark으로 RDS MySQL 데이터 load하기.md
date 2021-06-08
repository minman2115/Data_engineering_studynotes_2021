.

Data_Engineering_TIL(20210608)

#### [실습목적]

EMR pyspark에서 RDS mysql 디비로부터 데이터를 가져와 데이터 프레임으로 만들어본다.

#### [실습내용]

STEP 1) EMR 마스터노드 spark library 폴더에 Mysql connector 다운로드

step 1-1) https://dev.mysql.com/downloads/ 접속

step 1-2) 화면중간에 'Connector/J' 클릭

--> https://dev.mysql.com/downloads/connector/j/ 로 리다이렉트됨

step 1-3) 화면중간에 'Select operating System'에서 'Microsoft Windows'를 클릭한 후 'Platform Independent' 클릭

step 1-4) 화면중간에 'Platform Independent (Architecture Independent), ZIP Archive(mysql-connector-java-8.0.25.zip)'의 'Download' 클릭

step 1-5) 'MySQL Community Downloads'라는 화면이 뜨면서 'Login Now or Sign Up for a free account.' 메세지가 보이는데 화면 하단에 'No thanks, just start my download.'를 클릭한다.

step 1-6) mysql-connector-java-8.0.25.zip 가 다운로드되고 이거를 압축을 푼다.

step 1-7) 압축파일을 풀고 해당 폴더로 들어가보면 my-connector-java-8.0.25.jar가 보이는데 이게 우리가 사용할 mysql connector이다.

step 1-8) 이 파일을 사용해야할 EMR 마스터노드의 spark library 폴더에 아래와 같이 다운받는다.


```console
[hadoop@ip-10-0-2-17 ~]$ aws s3 cp s3://my-s3bucket-test/mysql-connector-java-8.0.25.jar .
download: s3://pms-s3bucket-test/mysql-connector-java-8.0.25.jar to ./mysql-connector-java-8.0.25.jar

[hadoop@ip-10-0-2-17 ~]$ ll
total 2372
-rw-rw-r-- 1 hadoop hadoop 2428320 Jun  8 12:10 mysql-connector-java-8.0.25.jar

[hadoop@ip-10-0-2-17 ~]$ sudo mv mysql-connector-java-8.0.25.jar /usr/lib/spark/jars

[hadoop@ip-10-0-2-17 ~]$ cd /usr/lib/spark/jars

[hadoop@ip-10-0-2-17 jars]$ ll
total 227924
lrwxrwxrwx 1 root   root         33 Jun  8 11:40 * -> /usr/share/aws/emr/emrfs/auxlib/*
-rw-r--r-- 1 root   root      30035 Mar 31 00:55 accessors-smart-1.2.jar
-rw-r--r-- 1 root   root      69409 Mar 31 00:55 activation-1.1.1.jar
...
-rw-r--r-- 1 root   root       5711 Mar 31 00:55 minlog-1.3.0.jar
lrwxrwxrwx 1 root   root         53 Jun  8 11:40 mockito-core-1.10.19.jar -> /usr/share/aws/emr/emrfs/lib/mockito-core-1.10.19.jar
-rw-rw-r-- 1 hadoop hadoop  2428320 Jun  8 12:10 mysql-connector-java-8.0.25.jar
-rw-r--r-- 1 root   root      54461 Mar 31 00:55 native_ref-java-1.1.jar
...
    
[hadoop@ip-10-0-2-17 jars]$ sudo chown root:root mysql-connector-java-8.0.25.jar
```

STEP 2) 아래와 같은 pyspark 스크립트를 이용해 RDS mysql 디비로부터 데이터를 가져와 데이터 프레임으로 load한다.


```python
from pyspark.sql import SparkSession

hostname='pms-rds-test.xxxxxxxxxx.ap-northeast-2.rds.amazonaws.com'
jdbcPort=3306
dbname='airflow'
username='minman'
password='mypasswd123#'

spark=SparkSession.builder.appName("test_app").enableHiveSupport().getOrCreate()

df = spark.read.format("jdbc").option("url","jdbc:mysql://{0}:{1}/{2}".format(hostname,jdbcPort,dbname))\
     .option("driver","com.mysql.jdbc.Driver").option("dbtable","my_test")\
     .option("user",username).option("password",password).load()

df.show(30)
```


    FloatProgress(value=0.0, bar_style='info', description='Progress:', layout=Layout(height='25px', width='50%'),…


    +---+------+-----------+-------+
    | ID|  Name|ReserveDate|RoomNum|
    +---+------+-----------+-------+
    |  5|이순신| 2016-02-16|   1108|
    |  1|박지성| 2016-03-14|   1102|
    |  2|은지원| 2016-02-15|   1002|
    |  3|강호동| 2016-02-11|   1001|
    |  4|이수근| 2016-01-11|   1003|
    +---+------+-----------+-------+
