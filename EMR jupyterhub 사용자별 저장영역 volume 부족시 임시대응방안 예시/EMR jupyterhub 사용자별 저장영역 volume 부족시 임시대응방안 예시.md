.

Data_Engineering_TIL(20210602)

[EMR 운영시 발생하는 문제]

- EMR jupyterhub에 여러 유저가 접속해서 작업할 경우 마스터 노드 /mnt 하위 폴더(주피터허브 저장영역으로 사용자별 파일들이 저장된 공간임)에 file이 쌓이면서 해당 볼륨에 할당된 용량에 한계가 오는 문제가 있음

[해결방안]

- /mnt 볼륨의 용량을 증가시키는 방안

[해결방안 상세내용]

step 1) 해당 EMR 마스터에 SSH 접속하여 볼륨현황을 아래와 같이 파악한다.

** 아래에 `/dev/nvme1n1p2   27G  7.4G   20G  28% /mnt` 부분의 용량을 증가시키면 된다. 문제가 발생하면 `/dev/nvme1n1p2   27G  27G   0G  100% /mnt` 와 같이 볼륨이 꽉찬게 보이기 때문이다.


```bash
Last login: Wed Jun  2 11:46:26 2021

       __|  __|_  )
       _|  (     /   Amazon Linux 2 AMI
      ___|\___|___|

https://aws.amazon.com/amazon-linux-2/
17 package(s) needed for security, out of 41 available
Run "sudo yum update" to apply all updates.

EEEEEEEEEEEEEEEEEEEE MMMMMMMM           MMMMMMMM RRRRRRRRRRRRRRR
E::::::::::::::::::E M:::::::M         M:::::::M R::::::::::::::R
EE:::::EEEEEEEEE:::E M::::::::M       M::::::::M R:::::RRRRRR:::::R
  E::::E       EEEEE M:::::::::M     M:::::::::M RR::::R      R::::R
  E::::E             M::::::M:::M   M:::M::::::M   R:::R      R::::R
  E:::::EEEEEEEEEE   M:::::M M:::M M:::M M:::::M   R:::RRRRRR:::::R
  E::::::::::::::E   M:::::M  M:::M:::M  M:::::M   R:::::::::::RR
  E:::::EEEEEEEEEE   M:::::M   M:::::M   M:::::M   R:::RRRRRR::::R
  E::::E             M:::::M    M:::M    M:::::M   R:::R      R::::R
  E::::E       EEEEE M:::::M     MMM     M:::::M   R:::R      R::::R
EE:::::EEEEEEEE::::E M:::::M             M:::::M   R:::R      R::::R
E::::::::::::::::::E M:::::M             M:::::M RR::::R      R::::R
EEEEEEEEEEEEEEEEEEEE MMMMMMM             MMMMMMM RRRRRRR      RRRRRR

# mount point 확인
[hadoop@ip-10-0-2-103 ~]$ lsblk
NAME          MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
nvme1n1       259:0    0  32G  0 disk
├─nvme1n1p1   259:5    0   5G  0 part /emr
└─nvme1n1p2   259:6    0  27G  0 part /mnt
nvme2n1       259:1    0  32G  0 disk /mnt1
nvme0n1       259:2    0  10G  0 disk
├─nvme0n1p1   259:3    0  10G  0 part /
└─nvme0n1p128 259:4    0   1M  0 part

# file system size 확인
[hadoop@ip-10-0-2-103 ~]$ df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs         16G     0   16G   0% /dev
tmpfs            16G     0   16G   0% /dev/shm
tmpfs            16G  1.5M   16G   1% /run
tmpfs            16G     0   16G   0% /sys/fs/cgroup
/dev/nvme0n1p1   10G  6.1G  4.0G  61% /
/dev/nvme1n1p1  5.0G   40M  5.0G   1% /emr
/dev/nvme1n1p2   27G  7.4G   20G  28% /mnt
/dev/nvme2n1     32G  184M   32G   1% /mnt1
tmpfs           3.1G     0  3.1G   0% /run/user/1001
tmpfs           3.1G     0  3.1G   0% /run/user/993
tmpfs           3.1G     0  3.1G   0% /run/user/990
tmpfs           3.1G     0  3.1G   0% /run/user/991
tmpfs           3.1G     0  3.1G   0% /run/user/989
tmpfs           3.1G     0  3.1G   0% /run/user/985
tmpfs           3.1G     0  3.1G   0% /run/user/984
tmpfs           3.1G     0  3.1G   0% /run/user/994
tmpfs           3.1G     0  3.1G   0% /run/user/987
overlay          27G  7.4G   20G  28% /mnt/var/lib/docker/overlay2/f9655c245925924e2f9da0d60530d9a76436982fda373394e36d08380bd65d84/merged
tmpfs           3.1G     0  3.1G   0% /run/user/983
tmpfs           3.1G     0  3.1G   0% /run/user/986
tmpfs           3.1G     0  3.1G   0% /run/user/995
```

step 2) EMR console에서 마스터노드의 해당 볼륨 사이즈를 아래와 같이 32g에서 62g로 증가시킨다.

![test](https://user-images.githubusercontent.com/41605276/120482848-7eae9180-c3ec-11eb-82f2-b2bb7034f574.png)

step 3) EMR 서버에서 실제 볼륨이 확장되었는지 아래와 같이 확인해본다.

실제로 `nvme1n1       259:0    0  64G  0 disk` 64기가로 늘어난 것을 확인할 수 있다.


```bash
[hadoop@ip-10-0-2-103 ~]$ lsblk
NAME          MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
nvme1n1       259:0    0  64G  0 disk
├─nvme1n1p1   259:5    0   5G  0 part /emr
└─nvme1n1p2   259:6    0  27G  0 part /mnt
nvme2n1       259:1    0  32G  0 disk /mnt1
nvme0n1       259:2    0  10G  0 disk
├─nvme0n1p1   259:3    0  10G  0 part /
└─nvme0n1p128 259:4    0   1M  0 part
    
[hadoop@ip-10-0-2-103 ~]$ df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs         16G     0   16G   0% /dev
tmpfs            16G     0   16G   0% /dev/shm
tmpfs            16G  1.5M   16G   1% /run
tmpfs            16G     0   16G   0% /sys/fs/cgroup
/dev/nvme0n1p1   10G  6.1G  4.0G  61% /
/dev/nvme1n1p1  5.0G   40M  5.0G   1% /emr
/dev/nvme1n1p2   27G  7.4G   20G  28% /mnt
/dev/nvme2n1     32G  184M   32G   1% /mnt1
tmpfs           3.1G     0  3.1G   0% /run/user/1001
tmpfs           3.1G     0  3.1G   0% /run/user/993
tmpfs           3.1G     0  3.1G   0% /run/user/990
tmpfs           3.1G     0  3.1G   0% /run/user/991
tmpfs           3.1G     0  3.1G   0% /run/user/989
tmpfs           3.1G     0  3.1G   0% /run/user/985
tmpfs           3.1G     0  3.1G   0% /run/user/984
tmpfs           3.1G     0  3.1G   0% /run/user/994
tmpfs           3.1G     0  3.1G   0% /run/user/987
overlay          27G  7.4G   20G  28% /mnt/var/lib/docker/overlay2/f9655c245925924e2f9da0d60530d9a76436982fda373394e36d08380bd65d84/merged
tmpfs           3.1G     0  3.1G   0% /run/user/983
tmpfs           3.1G     0  3.1G   0% /run/user/986
tmpfs           3.1G     0  3.1G   0% /run/user/995
```

step 4) 아래와 같이 파티션을 확장시킨다.


```bash
[hadoop@ip-10-0-2-103 ~]$ sudo growpart /dev/nvme1n1 2
CHANGED: partition=2 start=10487808 old: size=56621023 end=67108831 new: size=123729887 end=134217695

# nvme1n1p2가 27g에서 59g로 늘어난 것을 확인할 수 있다.
[hadoop@ip-10-0-2-103 ~]$ lsblk
NAME          MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
nvme1n1       259:0    0  64G  0 disk
├─nvme1n1p1   259:5    0   5G  0 part /emr
└─nvme1n1p2   259:6    0  59G  0 part /mnt
nvme2n1       259:1    0  64G  0 disk /mnt1
nvme0n1       259:2    0  10G  0 disk
├─nvme0n1p1   259:3    0  10G  0 part /
└─nvme0n1p128 259:4    0   1M  0 part
```

step 5) 파티션 확장된 부분에 대해 실제 mnt 영역에 할당하도록 xfs_growfs 명령을 실행한다.


```bash
[hadoop@ip-10-0-2-103 ~]$ sudo xfs_growfs -d /mnt
meta-data=/dev/nvme1n1p2         isize=512    agcount=4, agsize=1769407 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1 spinodes=0
data     =                       bsize=4096   blocks=7077627, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal               bsize=4096   blocks=3455, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 7077627 to 15466235

# /mnt 부분이 기존에 20g에서 52g로 늘어난 것을 확인할 수 있다.
[hadoop@ip-10-0-2-103 ~]$ df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs         16G     0   16G   0% /dev
tmpfs            16G     0   16G   0% /dev/shm
tmpfs            16G  1.5M   16G   1% /run
tmpfs            16G     0   16G   0% /sys/fs/cgroup
/dev/nvme0n1p1   10G  6.1G  4.0G  61% /
/dev/nvme1n1p1  5.0G   42M  5.0G   1% /emr
/dev/nvme1n1p2   59G  7.5G   52G  13% /mnt
/dev/nvme2n1     32G  184M   32G   1% /mnt1
tmpfs           3.1G     0  3.1G   0% /run/user/1001
tmpfs           3.1G     0  3.1G   0% /run/user/993
tmpfs           3.1G     0  3.1G   0% /run/user/990
tmpfs           3.1G     0  3.1G   0% /run/user/991
tmpfs           3.1G     0  3.1G   0% /run/user/989
tmpfs           3.1G     0  3.1G   0% /run/user/985
tmpfs           3.1G     0  3.1G   0% /run/user/984
tmpfs           3.1G     0  3.1G   0% /run/user/994
tmpfs           3.1G     0  3.1G   0% /run/user/987
overlay          59G  7.5G   52G  13% /mnt/var/lib/docker/overlay2/f9655c245925924e2f9da0d60530d9a76436982fda373394e36d08380bd65d84/merged
tmpfs           3.1G     0  3.1G   0% /run/user/983
tmpfs           3.1G     0  3.1G   0% /run/user/986
tmpfs           3.1G     0  3.1G   0% /run/user/995
```
