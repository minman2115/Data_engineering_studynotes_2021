. 

Data_Engineering_TIL(20210205)

### [참고자료]

stack overflow "Unable to view ganglia dashboard"

URL : https://stackoverflow.com/questions/23515934/unable-to-view-ganglia-dashboard

### [문제상황]

AWS EMR ganglia web UI 접속시 아래와 같은 Error 발생

1) 웹브라우저 화면

#### Forbidden 

#### You don't have permission to access this server.

2) EMR master server terminal

```bash
$ sudo cat /etc/httpd/logs/error_log
[Wed May 07 10:50:42 2014] [error] [client 118.102.239.85] client denied by server configuration: /usr/share/ganglia
```

### [해결방법]

```bash
$ sudo vim /etc/httpd/conf.d/ganglia.conf

<Location /ganglia>
 Order deny,allow
 Allow from all
 Required all granted
</Location>

를 아래와 같이 변경

<Location /ganglia>
 Allow from all
 Required all granted
</Location>

$ sudo service httpd restart
Redirecting to /bin/systemctl restart httpd.service

```

위와 같이 하였음에도 해결이 안될경우 아래와 같이 명령어 실행후 다시 접근해볼 것

```bash
$ chown -R ganglia:ganglia /usr/share/ganglia
$ sudo service httpd restart
Redirecting to /bin/systemctl restart httpd.service
```