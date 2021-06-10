.

Data_Engineering_TIL(20210609)

[참고자료]

https://docs.aws.amazon.com/ko_kr/emr/latest/ReleaseGuide/emr-jupyterhub-install-kernels-libs.html

[구현내용]

EMR의 모든 노드에 python 라이브러리를 설치하기 위해서 boto3를 이용한 python 스크립트를 구현함. 특정 EMR 클러스터의 마스터노드를 제외한 모든 running중인 노드에 python 라이브러리를 설치하는 쉘스크립트를 실행해서 python 라이브러리를 일괄 모든 노드에 설치하는 원리임.

STEP 1) 아래와 같이 준비물 스크립트를 작성한다.

준비물 1. python_lib_install.py


```python
import argparse
import time
import boto3


def install_libraries_all_nodes(cluster_id, script_path, emr_client, ssm_client):
    
    ids=[]
    for resp in emr_client.get_paginator('list_instances').paginate(ClusterId=cluster_id,InstanceGroupTypes=['TASK','CORE']):
        for instance in resp['Instances']:
            if instance['Status']['State']=='RUNNING':
                ids.append(instance['Ec2InstanceId'])    
    print(f"Found instances: {ids}.")

    script_file_name = script_path.split('/')[-1]
    commands = [# Copy the shell script from Amazon S3 to each node instance.
                f"aws s3 cp {script_path} /home/hadoop",
                # Run the shell script to install libraries on each node instance.
                "bash /home/hadoop/{}".format(script_file_name)]
    
    for command in commands:
        print(f"Sending '{command}' to all nodes...")
        command_id = ssm_client.send_command(
            InstanceIds=ids,
            DocumentName='AWS-RunShellScript',
            Parameters={"commands": [command]},
            TimeoutSeconds=3600)['Command']['CommandId']
        while True:
            # Verify the previous step succeeded before running the next step.
            cmd_result = ssm_client.list_commands(
                CommandId=command_id)['Commands'][0]
            if cmd_result['StatusDetails'] == 'Success':
                print(f"Command succeeded.")
                break
            elif cmd_result['StatusDetails'] in ['Pending', 'InProgress']:
                print(f"Command status is {cmd_result['StatusDetails']}, waiting...")
                time.sleep(10)
            else:
                print(f"Command status is {cmd_result['StatusDetails']}, quitting.")
                raise RuntimeError(
                    f"Command {command} failed to run. "
                    f"Details: {cmd_result['StatusDetails']}")

def main():
    parser = argparse.ArgumentParser()
    parser.add_argument('cluster_id', help="The ID of the cluster.")
    parser.add_argument('script_path', help="The path to the script in Amazon S3.")
    args = parser.parse_args()

    emr_client = boto3.client('emr')
    ssm_client = boto3.client('ssm')

    install_libraries_all_nodes(args.cluster_id, args.script_path, emr_client, ssm_client)


if __name__ == '__main__':
    main()
```

준비물 2. my_lib_install.sh

flask 라이브러리를 설치 해보는거를 가정해서 아래와 같이 스크립트를 작성했다.


```bash
#!bin/bash
sudo python3 -m pip install Flask==2.0.1
sudo docker exec jupyterhub bash -c "conda install -c conda-forge flask"
```

STEP 2) 준비물을 이용해서 작성한 스크립트를 실행한다.

위와 같이 준비물이 준비되면 boto3 권한이 있는 서버나 로컬에서 아래 리눅스 명령어와 같이 Cluster_id와 python 라이브러리를 설치하는 쉘스크립트의 s3 경로를 파라미터로 넣어서 실행하면 된다.


```bash
[hadoop@ip-172-99-27-281 my_folder] python3 python_lib_install.py j-ZXXXAXXXXXXX s3://my_bucket/my_lib_install.sh
Found instances : ['i-qwnceklqwcnelq','i-0129ndkqnwckqnd', ... , 'i-01asdcakncwckqnd'].
Sending 'aws s3 cp s3://my_bucket/my_lib_install.sh' to all nodes ....
Command status is Pending, waiting...
Command status is InProgress, waiting...
Command succeeded.
Sending 'bash /home/hadoop/my_lib_install.sh' to all instances...
Command status is Pending, waiting...
Command status is InProgress, waiting...
Command succeeded.
```