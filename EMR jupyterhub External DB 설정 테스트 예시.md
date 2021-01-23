### [실습목표]

- jupyterhub의 Native Authenticator 플러그인과 postgreSQL(오로라 서버리스 디비)를 이용하여 계정을 관리해본다.


- ASIS EMR에서 계정을 생성하고, TOBE EMR에서도 ASIS EMR 계정정보 그대로 사용이 가능한지 테스트해본다.

### [실습내용 요약]

Step 1) ASIS EMR, TOBE EMR 생성

Step 2) RDS 보안그룹 생성

Step 3) External DB로 사용할 PostgreSQL serverless Aurora 생성

step 4) ASIS EMR 접속 후 jupyterhub 테스트 계정 생성

step 5) external db 연동

step 6) ASIS EMR terminate

step 7) TOBE EMR 접속 후 external db 연동

step 8) step 4)에서 생성한 테스트 계정 그대로 사용가능한지 여부 확인


### [실습 상세내용]

#### Step 1) ASIS EMR, TOBE EMR 생성

EMR 클러스터 두대를 띄운다.

#### Step 2) RDS 보안그룹 생성

RDS를 위한 보안그룹을 하나 생성하고, 5432 포트에 대해서 ASIS&TOBE EMR 마스터노드의 프라이빗 아이피를 허용하는 것을 설정해준다.

#### Step 3) External DB로 사용할 PostgreSQL serverless Aurora 생성

1) Engine type : Amazon Aurora

2) Edition : Amazon Aurora with PostgreSQL compatibility

3) Capacity type : Serverless

4) DB cluster identifier : pms-rds-for-emr-test

5) Master username : postgres

6) Master password : mypasswd123#

7) Capacity settings : 2 Minimum Aurora capacity unit, 2 Maximum Aurora capacity unit

8) Virtual private cloud (VPC) : pms-vpc

** 보안그룹은 STEP 2)에서 만든 보안그룹으로 반드시 설정해준다.

9) the others options : default option 

#### step 4) ASIS EMR 접속 후 jupyterhub 테스트 계정 생성

EMR 마스터노드에 SSH 접속하여 아래와 같이 명령어를 실행하여 계정을 추가한다.


```bash
[hadoop@ip-10-0-1-244 ~]$ sudo docker exec jupyterhub bash -c "pip install jupyterhub-nativeauthenticator"
[hadoop@ip-10-0-1-244 ~]$ echo "c.NativeAuthenticator.open_signup = True" | sudo tee -a /etc/jupyter/conf/jupyterhub_config.py
[hadoop@ip-10-0-1-244 ~]$ echo "c.Authenticator.open_signup = True" | sudo tee -a /etc/jupyter/conf/jupyterhub_config.py
[hadoop@ip-10-0-1-244 ~]$ echo "c.LocalAuthenticator.create_system_users = True" | sudo tee -a /etc/jupyter/conf/jupyterhub_config.py
[hadoop@ip-10-0-1-244 ~]$ echo "from jupyterhub.auth import LocalAuthenticator" | sudo tee -a /etc/jupyter/conf/jupyterhub_config.py
[hadoop@ip-10-0-1-244 ~]$ echo "from nativeauthenticator import NativeAuthenticator" | sudo tee -a /etc/jupyter/conf/jupyterhub_config.py
[hadoop@ip-10-0-1-244 ~]$ echo "class LocalNativeAuthenticator(NativeAuthenticator, LocalAuthenticator):" | sudo tee -a /etc/jupyter/conf/jupyterhub_config.py
[hadoop@ip-10-0-1-244 ~]$ echo "  pass" | sudo tee -a /etc/jupyter/conf/jupyterhub_config.py
[hadoop@ip-10-0-1-244 ~]$ echo "c.JupyterHub.authenticator_class = LocalNativeAuthenticator" | sudo tee -a /etc/jupyter/conf/jupyterhub_config.py
[hadoop@ip-10-0-1-244 ~]$ echo "c.Spawner.default_url = '/lab'" | sudo tee -a /etc/jupyter/conf/jupyterhub_config.py
[hadoop@ip-10-0-1-244 ~]$ sudo docker exec jupyterhub bash -c "apt-get update -y"
[hadoop@ip-10-0-1-244 ~]$ sudo docker exec jupyterhub bash -c "apt-get upgrade -y"
[hadoop@ip-10-0-1-244 ~]$ sudo docker exec jupyterhub bash -c "apt-get install libpq-dev -y"
[hadoop@ip-10-0-1-244 ~]$ sudo docker exec jupyterhub bash -c "pip install psycopg2==2.8.6"
[hadoop@ip-10-0-1-244 ~]$ sudo docker exec jupyterhub bash -c "apt-get install postgresql-client -y"
[hadoop@ip-10-0-1-244 ~]$ sudo docker stop jupyterhub
[hadoop@ip-10-0-1-244 ~]$ sudo docker start jupyterhub

[hadoop@ip-10-0-1-244 ~]$ cat /etc/jupyter/conf/jupyterhub_config.py
# Configuration file for jupyterhub.

import os

notebook_dir = os.environ.get('DOCKER_NOTEBOOK_DIR')
network_name='jupyterhub-network'
s3_endpoint_url = os.environ.get('S3_ENDPOINT_URL')

c.Spawner.debug = True
c.Spawner.environment = {'SPARKMAGIC_CONF_DIR':'/etc/jupyter/conf', 'JUPYTER_ENABLE_LAB': 'yes', 'S3_ENDPOINT_URL': s3_endpoint_url}

c.JupyterHub.hub_ip = '0.0.0.0'
c.JupyterHub.admin_access = True
c.JupyterHub.ssl_key = '/etc/jupyter/conf/server.key'
c.JupyterHub.ssl_cert = '/etc/jupyter/conf/server.crt'
c.JupyterHub.port = 9443

c.Authenticator.admin_users = {'jovyan'}
c.NativeAuthenticator.open_signup = True
c.Authenticator.open_signup = True
c.LocalAuthenticator.create_system_users = True
from jupyterhub.auth import LocalAuthenticator
from nativeauthenticator import NativeAuthenticator
class LocalNativeAuthenticator(NativeAuthenticator, LocalAuthenticator):
  pass
c.JupyterHub.authenticator_class = LocalNativeAuthenticator
c.Spawner.default_url = '/lab'
```

#### step 5) external db 연동

아래와 같이 명령어를 실행하여 external db를 연동한다.


```bash
[hadoop@ip-10-0-1-244 ~]$ sudo docker exec -it jupyterhub /bin/bash

# psql -h [RDS_endpoint_url] -U postgres -W

root@jupyterhub:~# psql -h pms-rds-test.cluster-xxxxxxxxxxxxx.ap-northeast-2.rds.amazonaws.com -U postgres -W
Password for user postgres:
psql (10.15 (Ubuntu 10.15-0ubuntu0.18.04.1), server 10.12)
SSL connection (protocol: TLSv1.2, cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256, compression: off)
Type "help" for help.

postgres=> CREATE DATABASE jupyterhub;
CREATE DATABASE

postgres=> CREATE USER jupyterhub WITH ENCRYPTED PASSWORD 'mypasswd123#';
CREATE ROLE

postgres-> \q

root@jupyterhub:~# jupyterhub upgrade-db
+ exec
+ exec

# echo 'c.JupyterHub.db_url = "postgresql://jupyterhub:mypasswd123#@[RDS_endpoint_url]:5432/jupyterhub"' | sudo tee -a /etc/jupyter/conf/jupyterhub_config.py

root@jupyterhub:~# echo 'c.JupyterHub.db_url = "postgresql://jupyterhub:mypasswd123#@pms-rds-test.cluster-xxxxxxxxxx.ap-northeast-2.rds.amazonaws.com:5432/jupyterhub"' | sudo tee -a /etc/jupyter/conf/jupyterhub_config.py
c.JupyterHub.db_url = "postgresql://jupyterhub:mypasswd123#@pms-rds-test.cluster-Xxxxxxxxxxxxxx.ap-northeast-2.rds.amazonaws.com:5432/jupyterhub"

root@jupyterhub:~# exit
exit

[hadoop@ip-10-0-1-244 ~]$ sudo docker stop jupyterhub
jupyterhub

[hadoop@ip-10-0-1-244 ~]$ sudo docker start jupyterhub
jupyterhub

##########################################################################################################################
# 위에 명령어를 실행한 후에 웹브라우저를 열고 아래와 같이
# https://[EMR_Masternode_ip]:9443/ 로 접속해서 ronaldo, rooney, jisung, minman 라는 계정들을 생성하고 한번씩 로그인해준다.
# 참고로 jovyan 어드민 유저로 https://[EMR_Masternode_ip]:9443/hub/authorize 에 생성한 계정을 승인해주고,
# https://[EMR_Masternode_ip]:9443/hub/admin 에서 생성한 계정을 추가해준다.
# 그런 다음에 다시 EMR 마스터 노드 터미널로 돌아와서 아래와 같이 명령어를 실행한다.
##########################################################################################################################

[hadoop@ip-10-0-1-244 ~]$ sudo docker exec -it jupyterhub /bin/bash

# psql -h [RDS_endpoint_url] -U jupyterhub -W

root@jupyterhub:~# psql -h pms-rds-test.cluster-xxxxxxxxxxx.ap-northeast-2.rds.amazonaws.com -U jupyterhub -W
Password for user jupyterhub:
psql (10.15 (Ubuntu 10.15-0ubuntu0.18.04.1), server 10.12)
SSL connection (protocol: TLSv1.2, cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256, compression: off)
Type "help" for help.

jupyterhub=> \dt
                 List of relations
 Schema |        Name         | Type  |   Owner
--------+---------------------+-------+------------
 public | alembic_version     | table | jupyterhub
 public | api_tokens          | table | jupyterhub
 public | groups              | table | jupyterhub
 public | oauth_access_tokens | table | jupyterhub
 public | oauth_clients       | table | jupyterhub
 public | oauth_codes         | table | jupyterhub
 public | servers             | table | jupyterhub
 public | services            | table | jupyterhub
 public | spawners            | table | jupyterhub
 public | user_group_map      | table | jupyterhub
 public | users               | table | jupyterhub
(11 rows)

jupyterhub=> select * from users;
 id |  name   | admin |          created           |       last_activity        |            cookie_id             | state | encrypted_auth_state
----+---------+-------+----------------------------+----------------------------+----------------------------------+-------+----------------------
  2 | ronaldo | f     | 2021-01-23 07:11:41.398435 | 2021-01-23 07:12:57.958063 | f534de1cea24422f9bf518a433bffe7a | {}    |
  3 | rooney  | f     | 2021-01-23 07:11:41.404025 | 2021-01-23 07:13:09.274591 | 68bcda254cd64b798748a55d591d3ffb | {}    |
  4 | jisung  | f     | 2021-01-23 07:11:41.409084 | 2021-01-23 07:13:43.590279 | c5fe3716b63249af91b175bb7ab5d892 | {}    |
  1 | jovyan  | t     | 2021-01-23 07:11:41.386407 | 2021-01-23 07:14:04.667675 | a24cc21085e64eaa8570ebc9953c3415 | {}    |
  5 | minman  | f     | 2021-01-23 07:14:28.462603 | 2021-01-23 07:14:40.853971 | 4d3c79a3eed844c99a5d237a13235d69 | {}    |
(5 rows)

jupyterhub=> select * from users_info;
 id | username |                                                          password                                                          | is_authorized | email | has_2fa |    otp_secret
----+----------+----------------------------------------------------------------------------------------------------------------------------+---------------+-------+---------+------------------
  1 | jovyan   | \x2432622431322459614c736d4d32717a326e725557506e33346a316c2e59475374734c314a356249744c4b4f596f726a64587242654c4b756e69554b | t             |       | f       | VROFP2SSBO545Y2Q
  2 | ronaldo  | \x243262243132245a6f4374467a7a76464642464c6d4e4a623663396c7566727461337171783757516e2e36594c44683977766e546630534575344c61 | t             |       | f       | M4OMTBNFR5Z4OBJH
  3 | rooney   | \x2432622431322450734458487464755759317569636d4d467a3574494f4c52572f523444692e6b5a48756768684e48493668746f61496539667a6157 | t             |       | f       | NL5S7B5YWK3HQ265
  4 | jisung   | \x2432622431322431746d6c436a4c3756427052306c764c436155366775642e4649353856392f3039707365367a362e386a6c37726a676f3143797161 | t             |       | f       | DQ32UJNZ6FFWRXG4
  5 | minman   | \x24326224313224683549417835355a486c6f304a4d4630682e6b41364f304151585771475a454f2f534f462e6367525a68377069495551306d41366d | t             |       | f       | JNJE4AJSM6RKG4AM
(5 rows)
```

#### step 6) ASIS EMR terminate

#### step 7) TOBE EMR 접속 후 external db 연동


```bash
[hadoop@ip-10-0-1-154 ~]$ sudo docker exec jupyterhub bash -c "pip install jupyterhub-nativeauthenticator"
[hadoop@ip-10-0-1-154 ~]$ echo "c.NativeAuthenticator.open_signup = True" | sudo tee -a /etc/jupyter/conf/jupyterhub_config.py
[hadoop@ip-10-0-1-154 ~]$ echo "c.Authenticator.open_signup = True" | sudo tee -a /etc/jupyter/conf/jupyterhub_config.py
[hadoop@ip-10-0-1-154 ~]$ echo "c.LocalAuthenticator.create_system_users = True" | sudo tee -a /etc/jupyter/conf/jupyterhub_config.py
[hadoop@ip-10-0-1-154 ~]$ echo "from jupyterhub.auth import LocalAuthenticator" | sudo tee -a /etc/jupyter/conf/jupyterhub_config.py
[hadoop@ip-10-0-1-154 ~]$ echo "from nativeauthenticator import NativeAuthenticator" | sudo tee -a /etc/jupyter/conf/jupyterhub_config.py
[hadoop@ip-10-0-1-154 ~]$ echo "class LocalNativeAuthenticator(NativeAuthenticator, LocalAuthenticator):" | sudo tee -a /etc/jupyter/conf/jupyterhub_config.py
[hadoop@ip-10-0-1-154 ~]$ echo "  pass" | sudo tee -a /etc/jupyter/conf/jupyterhub_config.py
[hadoop@ip-10-0-1-154 ~]$ echo "c.JupyterHub.authenticator_class = LocalNativeAuthenticator" | sudo tee -a /etc/jupyter/conf/jupyterhub_config.py
[hadoop@ip-10-0-1-154 ~]$ echo "c.Spawner.default_url = '/lab'" | sudo tee -a /etc/jupyter/conf/jupyterhub_config.py

# echo 'c.JupyterHub.db_url = "postgresql://jupyterhub:mypasswd123#@[RDS_endpoint_url]:5432/jupyterhub"' | sudo tee -a /etc/jupyter/conf/jupyterhub_config.py
[hadoop@ip-10-0-1-154 ~]$ echo 'c.JupyterHub.db_url = "postgresql://jupyterhub:mypasswd123#@pms-rds-test.cluster-xxxxxxxxxx.ap-northeast-2.rds.amazonaws.com:5432/jupyterhub"' | sudo tee -a /etc/jupyter/conf/jupyterhub_config.py

[hadoop@ip-10-0-1-154 ~]$ sudo docker exec jupyterhub bash -c "apt-get update -y"
[hadoop@ip-10-0-1-154 ~]$ sudo docker exec jupyterhub bash -c "apt-get upgrade -y"
[hadoop@ip-10-0-1-154 ~]$ sudo docker exec jupyterhub bash -c "apt-get install libpq-dev -y"
[hadoop@ip-10-0-1-154 ~]$ sudo docker exec jupyterhub bash -c "pip install psycopg2==2.8.6"
[hadoop@ip-10-0-1-154 ~]$ sudo docker exec jupyterhub bash -c "apt-get install postgresql-client -y"
[hadoop@ip-10-0-1-154 ~]$ sudo docker stop jupyterhub
[hadoop@ip-10-0-1-154 ~]$ sudo docker start jupyterhub

# 위와 같이 실행후 아래의 명령어로 주피터허브 유저 폴더를 조회하면 AS-IS EMR 과 동일한 형태로 복원된 것을 알 수 있다.

[hadoop@ip-10-0-1-154 ~]$ ll /mnt/var/lib/jupyter/home/
total 0
drwxr-xr-x 2 jupyter      hadoop       57 Jan 23 07:26 jisung
drwxr-xr-x 2 ec2-user     root         91 Jan 23 07:26 jovyan
drwxr-xr-x 2 emr-notebook ec2-user     57 Jan 23 07:26 minman
drwxr-xr-x 2         1004 jupyter      57 Jan 23 07:26 ronaldo
drwxr-xr-x 2         1003 emr-notebook 57 Jan 23 07:26 rooney

##########################################################################################################################
# 위에 명령어를 실행한 후에 웹브라우저를 열고 아래와 같이
# https://[EMR_Masternode_ip]:9443/ 로 접속해서 ronaldo, rooney, jisung, minman 계정에 대해 AS-IS EMR에서 설정한
# 비밀번호 그대로 접속해본다. 과거에 사용했던 비밀번호 그대로 사용할 수 있는 것을 확인할 수 있다.
# 그런 다음에 다시 EMR 마스터 노드 터미널로 돌아와서 아래와 같이 명령어를 실행하여 비디를 조회해보자.
##########################################################################################################################

[hadoop@ip-10-0-1-154 ~]$ sudo docker exec -it jupyterhub /bin/bash

# psql -h [RDS_endpoint_url] -U jupyterhub -W

root@jupyterhub:~# psql -h pms-rds-test.cluster-xxxxxxxxxxx.ap-northeast-2.rds.amazonaws.com -U jupyterhub -W
Password for user jupyterhub:
psql (10.15 (Ubuntu 10.15-0ubuntu0.18.04.1), server 10.12)
SSL connection (protocol: TLSv1.2, cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256, compression: off)
Type "help" for help.

jupyterhub=> \dt
                 List of relations
 Schema |        Name         | Type  |   Owner
--------+---------------------+-------+------------
 public | alembic_version     | table | jupyterhub
 public | api_tokens          | table | jupyterhub
 public | groups              | table | jupyterhub
 public | oauth_access_tokens | table | jupyterhub
 public | oauth_clients       | table | jupyterhub
 public | oauth_codes         | table | jupyterhub
 public | servers             | table | jupyterhub
 public | services            | table | jupyterhub
 public | spawners            | table | jupyterhub
 public | user_group_map      | table | jupyterhub
 public | users               | table | jupyterhub
(11 rows)

jupyterhub=> select * from users;
 id |  name   | admin |          created           |       last_activity        |            cookie_id             | state | encrypted_auth_state
----+---------+-------+----------------------------+----------------------------+----------------------------------+-------+----------------------
  2 | ronaldo | f     | 2021-01-23 07:11:41.398435 | 2021-01-23 07:12:57.958063 | f534de1cea24422f9bf518a433bffe7a | {}    |
  3 | rooney  | f     | 2021-01-23 07:11:41.404025 | 2021-01-23 07:13:09.274591 | 68bcda254cd64b798748a55d591d3ffb | {}    |
  4 | jisung  | f     | 2021-01-23 07:11:41.409084 | 2021-01-23 07:13:43.590279 | c5fe3716b63249af91b175bb7ab5d892 | {}    |
  1 | jovyan  | t     | 2021-01-23 07:11:41.386407 | 2021-01-23 07:14:04.667675 | a24cc21085e64eaa8570ebc9953c3415 | {}    |
  5 | minman  | f     | 2021-01-23 07:14:28.462603 | 2021-01-23 07:14:40.853971 | 4d3c79a3eed844c99a5d237a13235d69 | {}    |
(5 rows)

jupyterhub=> select * from users_info;
 id | username |                                                          password                                                          | is_authorized | email | has_2fa |    otp_secret
----+----------+----------------------------------------------------------------------------------------------------------------------------+---------------+-------+---------+------------------
  1 | jovyan   | \x2432622431322459614c736d4d32717a326e725557506e33346a316c2e59475374734c314a356249744c4b4f596f726a64587242654c4b756e69554b | t             |       | f       | VROFP2SSBO545Y2Q
  2 | ronaldo  | \x243262243132245a6f4374467a7a76464642464c6d4e4a623663396c7566727461337171783757516e2e36594c44683977766e546630534575344c61 | t             |       | f       | M4OMTBNFR5Z4OBJH
  3 | rooney   | \x2432622431322450734458487464755759317569636d4d467a3574494f4c52572f523444692e6b5a48756768684e48493668746f61496539667a6157 | t             |       | f       | NL5S7B5YWK3HQ265
  4 | jisung   | \x2432622431322431746d6c436a4c3756427052306c764c436155366775642e4649353856392f3039707365367a362e386a6c37726a676f3143797161 | t             |       | f       | DQ32UJNZ6FFWRXG4
  5 | minman   | \x24326224313224683549417835355a486c6f304a4d4630682e6b41364f304151585771475a454f2f534f462e6367525a68377069495551306d41366d | t             |       | f       | JNJE4AJSM6RKG4AM
(5 rows)

jupyterhub=> \q

##########################################################################################################################
# 위에 명령어를 실행한 후에 웹브라우저를 열고 아래와 같이
# https://[EMR_Masternode_ip]:9443/ 로 접속해서 songha라는 계정을 또 생성해본다.
# jovyan 어드민 유저로 https://[EMR_Masternode_ip]:9443/hub/authorize 에 생성한 계정을 승인해주고,
# https://[EMR_Masternode_ip]:9443/hub/admin 에서 생성한 계정을 추가해준다.
# 그런 다음에 songha 계정으로 로그인을 한번 해준다.
# 그런 다음에 다시 EMR 마스터 노드 터미널로 돌아와서 아래와 같이 명령어를 실행해서 다시 디비를 조회해본다.
##########################################################################################################################

root@jupyterhub:~# psql -h pms-rds-test.cluster-cnaj6ucovzx2.ap-northeast-2.rds.amazonaws.com -U jupyterhub -W
Password for user jupyterhub:
psql (10.15 (Ubuntu 10.15-0ubuntu0.18.04.1), server 10.12)
SSL connection (protocol: TLSv1.2, cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256, compression: off)
Type "help" for help.

jupyterhub=> select * from users;
 id |  name   | admin |          created           |       last_activity        |            cookie_id             | state | encrypted_auth_state
----+---------+-------+----------------------------+----------------------------+----------------------------------+-------+----------------------
  2 | ronaldo | f     | 2021-01-23 07:11:41.398435 | 2021-01-23 07:28:58.69     | f534de1cea24422f9bf518a433bffe7a | {}    |
  3 | rooney  | f     | 2021-01-23 07:11:41.404025 | 2021-01-23 07:29:08.825    | 68bcda254cd64b798748a55d591d3ffb | {}    |
  1 | jovyan  | t     | 2021-01-23 07:11:41.386407 | 2021-01-23 07:32:13.631967 | a24cc21085e64eaa8570ebc9953c3415 | {}    |
  6 | songha  | f     | 2021-01-23 07:32:33.121975 | 2021-01-23 07:32:42.33866  | 11c37397355a42d38e9b5f9dd081bfe5 | {}    |
  5 | minman  | f     | 2021-01-23 07:14:28.462603 | 2021-01-23 07:21:22.760787 | 4d3c79a3eed844c99a5d237a13235d69 | {}    |
  4 | jisung  | f     | 2021-01-23 07:11:41.409084 | 2021-01-23 07:21:22.861727 | c5fe3716b63249af91b175bb7ab5d892 | {}    |
(6 rows)

jupyterhub=> select * from users_info;
 id | username |                                                          password                                                          | is_authorized | em
ail | has_2fa |    otp_secret
----+----------+----------------------------------------------------------------------------------------------------------------------------+---------------+---
----+---------+------------------
  1 | jovyan   | \x2432622431322459614c736d4d32717a326e725557506e33346a316c2e59475374734c314a356249744c4b4f596f726a64587242654c4b756e69554b | t             |
    | f       | VROFP2SSBO545Y2Q
  2 | ronaldo  | \x243262243132245a6f4374467a7a76464642464c6d4e4a623663396c7566727461337171783757516e2e36594c44683977766e546630534575344c61 | t             |
    | f       | M4OMTBNFR5Z4OBJH
  3 | rooney   | \x2432622431322450734458487464755759317569636d4d467a3574494f4c52572f523444692e6b5a48756768684e48493668746f61496539667a6157 | t             |
    | f       | NL5S7B5YWK3HQ265
  4 | jisung   | \x2432622431322431746d6c436a4c3756427052306c764c436155366775642e4649353856392f3039707365367a362e386a6c37726a676f3143797161 | t             |
    | f       | DQ32UJNZ6FFWRXG4
  5 | minman   | \x24326224313224683549417835355a486c6f304a4d4630682e6b41364f304151585771475a454f2f534f462e6367525a68377069495551306d41366d | t             |
    | f       | JNJE4AJSM6RKG4AM
  6 | songha   | \x243262243132244d426b6f4652704141677167343763747670594341755931507162656353506f44753345697065596e673852674c53453458507257 | t             |
    | f       | TH523FDK3FT2QOIA
(6 rows)

jupyterhub=> \q

root@jupyterhub:~# exit
exit

[hadoop@ip-10-0-1-154 ~]$ ll /mnt/var/lib/jupyter/home/
total 0
drwxr-xr-x 2 jupyter      hadoop        57 Jan 23 07:26 jisung
drwxr-xr-x 5 ec2-user     root         156 Jan 23 07:34 jovyan
drwxr-xr-x 2 emr-notebook ec2-user      57 Jan 23 07:26 minman
drwxr-xr-x 5         1004 jupyter      101 Jan 23 07:28 ronaldo
drwxr-xr-x 5         1003 emr-notebook 101 Jan 23 07:29 rooney
drwxr-xr-x 5         1005         1004 101 Jan 23 07:32 songha
```
