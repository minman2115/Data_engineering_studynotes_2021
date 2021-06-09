.

Data_Engineering_TIL(20210609)

[함수내용]


```python
def compare_spark_df_schema(asis_df,compare_df):
    asis_schema=pd.DataFrame(asis_df.dtypes,columns=['column','data_type'])
    compare_schema=pd.DataFrame(compare_df.dtypes,columns=['column','data_type'])
    # return True or False
    return asis_schema.equals(compare_schema)
```
