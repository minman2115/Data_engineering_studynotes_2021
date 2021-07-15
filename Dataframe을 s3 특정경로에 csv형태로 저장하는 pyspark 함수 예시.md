.

Data_Engineering_TIL(20210715)



```python
##########################################################################################
# spark df convert to csv and upload s3
##########################################################################################

def spark_df_to_csv(spark_df, s3_location, sep):
    import pandas
    import boto3
    import StringIO
    pandas_df=spark_df.toPandas()
    bucket_name=s3_location.split("/")[2]
    obj_key='/'.join(s3_location.split("/")[3:])
    csv_buffer = StringIO()
    pandas_df.to_csv(csv_buffer,sep=sep,encoding='UTF-8',index=False)
    s3_resource=boto3.resource('s3')
    s3_resource.Object(bucket_name,obj_key).put(Body=csv_buffer.getvalue())
    
spark_df_to_csv(my_spark_df,'s3://my-bucket-test/testfolder/test_my_file.csv',',')

##########################################################################################
# load the csv file
##########################################################################################

schema = StructType() \
      .add("RecordNumber",IntegerType(),True) \
      .add("Zipcode",IntegerType(),True) \
      .add("ZipCodeType",StringType(),True) \
      .add("City",StringType(),True) \
      .add("State",StringType(),True) \
      .add("LocationType",StringType(),True) \
      .add("Lat",DoubleType(),True) \
      .add("Long",DoubleType(),True) \
      .add("Xaxis",IntegerType(),True) \
      .add("Yaxis",DoubleType(),True) \
      .add("Zaxis",DoubleType(),True) \
      .add("WorldRegion",StringType(),True) \
      .add("Country",StringType(),True) \
      .add("LocationText",StringType(),True) \
      .add("Location",StringType(),True) \
      .add("Decommisioned",BooleanType(),True) \
      .add("TaxReturnsFiled",StringType(),True) \
      .add("EstimatedPopulation",IntegerType(),True) \
      .add("TotalWages",IntegerType(),True) \
      .add("Notes",StringType(),True)

df=spark.read.schema(schema).csv('s3://my-bucket-test/testfolder/test_my_file.csv',\
                                header=True,inferSchema='false',multiLine=True,quote='"',\
                                escape='\\', sep=',', ignoreLeadingWhiteSpace='true',\
                                ignoreTrailingWhiteSpace='true', mode='PERMISSIVE',encoding='UTF-8')

df.show()
```
