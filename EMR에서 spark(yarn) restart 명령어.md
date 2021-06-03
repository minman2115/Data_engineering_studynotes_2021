.

Data_Engineering_TIL(20210603)

[참고자료]

aws 도큐먼트 : https://aws.amazon.com/ko/premiumsupport/knowledge-center/restart-service-emr/

[학습내용]

```console
sudo systemctl stop hadoop-yarn-resourcemanager
sudo systemctl status hadoop-yarn-resourcemanager
sudo systemctl start hadoop-yarn-resourcemanager
```