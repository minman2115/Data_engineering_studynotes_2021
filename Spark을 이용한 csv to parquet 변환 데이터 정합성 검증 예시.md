.

Data_Engineering_TIL(20210603)

- Spark을 이용한 csv to parquet 변환 데이터 정합성 검증 예시

spark 데이터 프레임으로 로우카운트, 컬럼을 비교하여 데이터 정합성을 간단하게 검증한다


```python
from pyspark import SparkContext
from pyspark import SparkConf
from pyspark.sql import *

def check_csv_to_parquet(parquet_s3_location,csv_s3_location):
    
    parquet_df=spark.read.parquet(parquet_s3_location)
    print("parquet file 전체 row 수 : ", parquet_df.count())
    print("parquet file 전체 column : ", parquet_df.columns) # 리스트형태임
    
    csv_df=spark.read.csv(csv_s3_location,header=True,inferSchema='false',multiLine=True,\
                          quote='"',escape='\\',sep=',',ignoreLeadingWhiteSpace='true',\
                          ignoreTrailingWhiteSpace='true',mode='PERMISSIVE',encoding="UTF-8")
    print("csv file 전체 row 수 : ", csv_df.count())
    print("csv file 전체 column : ", csv_df.columns)
    
    compare_list=csv_df.columns
    for col_name in compare_list:
        if col_name not in parquet_df.columns:
            compare_list.remove(col_name)
    
    result = parquet_df.exceptAll(csv_df).orderBy(compare_list).count()==0
    
    if result is False:
        print("컬럼 정합성 이상없음")
    else:
        print("서로 컬럼이 다른 내용이 있음. 확인필요.")
        print("해당리스트 : ",list(set(csv_df.columns).difference(set(parquet_df.columns))))
    
    return None

check_csv_to_parquet("s3a://my_bucket/my_parquet","s3a://my_bucket/UTF8_my_csv.csv")
```

** 참고사항

csv 파일은 UTF-8로 인코딩된 파일을 load 해야함

위에서 `s3a://my_bucket/my_parquet`은 폴더로 폴더 하위는 아래와 같은 파케이 파일 구조임


```console
$ tree my_parquet
my_parquet
├── _common_metadata
├── _metadata
├── _SUCCESS
├── part-r-0000-4412a71-63cf-29aa-dwkdja1.gz.parquet
├── part-r-0001-4412a71-63cf-29aa-dwkdja1.gz.parquet
├── part-r-0002-4412a71-63cf-29aa-dwkdja1.gz.parquet
├── part-r-0003-4412a71-63cf-29aa-dwkdja1.gz.parquet
...
├── part-r-0099-4412a71-63cf-29aa-dwkdja1.gz.parquet
```

_common_metadata `contains the merged schemas for the parquet files in that directory`

_metadata `will contain only the schema of the most recently written parquet file in that directory`
