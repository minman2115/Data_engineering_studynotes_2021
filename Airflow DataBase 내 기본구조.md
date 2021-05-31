.

Data_Engineering_TIL(20210531)


```bash
[ec2-user@ip-10-172-372-192 airflow]$ mysql -h my-airflow-db.cluster-xxxxxxxx-ap-northeast-2.rds.amazonaws.com -u minman -p
Enter password: *********

...

mysql> use airflow
...
Database changed

mysql> show tables;
+-------------------------------------------------------+
| Tables_in_airflow                                           |
+-------------------------------------------------------+
| ab_permission                                               |
| ab_permission_view                                          |
| ab_permission_view_role                                     |
| ab_register_user                                            |
| ab_role                                                     |
| ab_user                                                     |
| ab_user_role                                                |
| ab_view_menu                                                |
| alembic_version                                             |
| celery_taskmeta                                             |
| celery_tasksetmeta                                          |
| chart                                                       |
| connection                                                  |
| dag                                                         |
| dag_code                                                    |
| dag_pickle                                                  |
| dag_run                                                     |
| dag_tag                                                     |
| import_error                                                |
| job                                                         |
| known_event                                                 |
| known_event_type                                            |
| kube_resource_version                                       |
| kube_worker_uuid                                            |
| log                                                         |
| rendered_task_instance_fields                               |
| serialized_dag                                              |
| sla_miss                                                    |
| slot_pool                                                   |
| task_fail                                                   |
| task_instance                                               |
| task_reschedule                                             |
| users                                                       |
| variable                                                    |
| xcom                                                        |
+-------------------------------------------------------+
35 rows in set (0.00 sec)

# airflow webserver에 접근하는 사용자 조회
mysql> select * from ab_user;
id, first_name, username, password(암호화되어있음), login_count,  등의 데이터가 있는 테이블

# airflow 내부에 직접접근하는 system 사용자 조회
# REST API 등으로 접근할때 이 계정정보를 이용해야함
mysql> select * from users;
+------------------------------------------------------------------------------+--------------+
| id | username | email                                   | password                   | superuser      |
+----+---------+------------------------------------+--------------------------+--------------+
|  1 | my_sys_user| minsupark@korea.com                   | qlwnecklqncelqqwenkxq(난수)|             0  |
...
+----+---------+------------------------------------+--------------------------+--------------+
```
