.

Data_Engineering_TIL(20210603)


```python
import boto3
import chardet

s3_bucket_name = 'my_bucket'
obj_key='data_folder/my_data.csv'

def encoding_file(s3_bucket_name,obj_key):
    s3=boto3.client('s3')
    s3_resource=boto3.resource('s3')
    obj_origin=obj_key.split('/')[-1]
    asis_obj='/tmp/{}'.format(obj_origin)
    tobe_obj='/tmp/UTF-8_{}'.format(obj_origin)
    
    s3.download_file(s3_bucket_name,obj_key,asis_obj)
    
    with open(asis_obj,'rb') as f:
        encoding_type=chardet.detect(f.read())['encoding']
    print("ASIS encoding type : ", encoding_type)
    
    os.system("iconv -f {encoding} -t UTF-8 '{asis_obj}' > '{tobe_obj}'".format(encoding=encoding_type,asis_obj=asis_obj,tobe_obj=tobe_obj))
    print("iconv -f {encoding} -t UTF-8 '{asis_obj}' > '{tobe_obj}'".format(encoding=encoding_type,asis_obj=asis_obj,tobe_obj=tobe_obj))
    os.system("ls | grep UTF-8")
    print("convert completed")
    s3_resource.meta.client.upload_file(tobe_obj,s3_bucket_name,'encodeing_file_folder'+tobe_obj)
    print(s3_bucket_name+'encodeing_file_folder'+tobe_obj+"upload completed")
    
    return None
```

obj_origin 변수에서 `/tmp/` 경로가 붙어있는거는 aws lambda 함수에서 사용했기 때문이다. 람다는 임시파일 경로로 `/tmp`를 사용하기 때문이다.
