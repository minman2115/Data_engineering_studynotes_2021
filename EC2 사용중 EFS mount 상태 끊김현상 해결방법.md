.

Data_Engineering_TIL(20210207)

### [참고자료]

AWS 도큐먼트 "Troubleshooting Mount Issues"

URL : https://docs.aws.amazon.com/efs/latest/ug/troubleshooting-efs-mounting.html

### [문제상황]

- EC2에 EFS를 mount하여 사용하는 중에 EC2와 EFS 간에 mount가 중단됨


- EC2 서버의 해당 EFS 폴더 경로로 이동해서 ls 등 어떠한 명령어를 실행하였을때 명령어가 실행은 되지만 결과값이 return이 안되고 명령어가 pending 후 먹통이 되는현상이 발생함


- 따라서 해당 EFS 폴더를 EC2로부터 mount를 강제로 해제하려고 하였으나 아래와 같이 unmount도 되지 않는 상황임

```bash
[ec2-user@ip-99-999-99-999 /]$ sudo umount -f /my_efs_folder
umount.nfs4: /my_efs_folder device is busy
```

### [해결방법]

- EFS에 파일이 많다거나 계속해서 쓰기/복사 작업중이었다면 위와 같은 EFS mount 상태가 위와 같이 비정상적이거나 반응이 오래걸릴수 있음

- EFS이 사용중일때 lazy umount를 사용하여 아래 명령어와 같이 강제로 mount를 해제하고, 다시 해당 EFS를 다시 mount하면 됨

```bash
[ec2-user@ip-99-999-99-999 /]$ sudo umount -l /my_efs_folder
```