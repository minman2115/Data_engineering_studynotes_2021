.

Data_Engineering_TIL(20210602)

- create_my_chardet_lib_layer.sh

** 스크립트 작성시 주의사항 : 라이브러리가 설치되는 폴더이름은 반드시 "python"이어야 함


```bash
#!/bin/bash

mkdir python
pip3 install chardet -t python/
zip -r python.zip .
aws s3 cp python.zip s3://my_bucket/test_temp/
aws lambda publish-layer-version --layer-name my_chardet_lib --description "my_chardet_lib" --content S3Bucket=my_bucket,S3Key=test_temp/python.zip --compatible-runtimes python3.7
```

위에 쉘스크립트 실행 후 lambda function 콘솔 접속 --> 스크롤를 내려서 하단 Layers 메뉴 우측에 'Add a layer' 클릭 --> 중간에 'custom layers' 단추 클릭 --> custom layers에 my_chardet_lib 클릭, 버전 1 클릭 --> 하단에 Add 버튼 클릭

- aws lambda publish-layer-version cli 명령어 실행결과 화면 예시


```bash
$ aws lambda publish-layer-version --layer-name my_chardet_lib --description "my_chardet_lib" --content S3Bucket=my_bucket,S3Key=test_temp/python.zip --compatible-runtimes python3.7
{
    "LayerVersionArn":"arn:aws:lambda:ap-northeast-2:239192389:layer:my_chardet_lib:1",
    "Description":"my_chardet_lib",
    "CreatedData":"2021-06-02T....",
    "LayerArn":"arn:aws:lambda:ap-northeast-2:239192389:layer:my_chardet_lib",
    "Content":{
        "CodeSize":27272,
        "CodeSha256":"qneckqnweclqwkeqcweklqxn",
        "Location":"https://awslambda-ap-ne-2-layers.s3.ap-northeast-2.........",
    },
    "Version":1,
    "CompatibleRuntimes":[
        "python3.7"
    ]
}
```
