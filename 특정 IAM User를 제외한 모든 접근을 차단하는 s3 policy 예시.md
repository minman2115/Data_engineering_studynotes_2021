- 구현 시나리오

pms-bucket-test 라는 버킷에 minsu.park이라는 IAM User를 제외한 모든 접근을 막고 싶다.

- 방법

minman-bucket s3 버킷에 아래와 같은 정책을 반영


```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Deny",
            "Principal": "*",
            "Action": "s3:*",
            "Resource": [
                "arn:aws:s3:::pms-bucket-test/*",
                "arn:aws:s3:::pms-bucket-test"
            ],
            "Condition": {
                "StringNotEquals": {
                    "aws:username": "minsu.park"
                }
            }
        }
    ]
}
```

방법 2. pms-bucket-test-2에 대한 접근권한 policy를 아래와 같이 생성하고, IAM User 들에 할당한다.

- policy 예시


```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Deny",
            "Action": "s3:*",
            "Resource": [
                "arn:aws:s3:::pms-bucket-test/*",
                "arn:aws:s3:::pms-bucket-test"
            ],
            "Condition": {
                "StringNotEquals": {
                    "aws:username": "minsu.park"
                }
            }
        }
    ]
}
```

- 위와 같은 policy를 설정했을때 minsu.park외의 유저가 pms-bucket-test 버킷에 접근했을때는 아래와 같이 액세스 권한이 없다는 애러메세지가 뜨면서 접근이 불가하다.

Insufficient permissions to list objects

After you or your AWS administrator have updated your permissions to allow the s3:ListBucket action, refresh the page. Learn more about identity and access management in Amazon S3


- pms-bucket-test bucket에 적용한 policy에서 "Effect"를 "Allow"로 설정하는 정책을 적용하면 다른 계정에서는 접근이 불가능한가?

Allow로 policy를 설정하면 의미가 없다. User IAM에 s3에 접근하는 권한이 있다면 해당 유저들은 모두 접근이 가능하다.

반면에 위와 같이 Deny로 설정하면 User IAM에 s3 full access 권한이 있더라고 해당 유저가 deny 대상이면 접근이 불가하다.