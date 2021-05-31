.

Data_Engineering_TIL(20210531)

- pyspark 기준 명령어


```python
import pprint
sc = spark.sparkContext
pprint.pprint(sc.getConf().getAll())
```
