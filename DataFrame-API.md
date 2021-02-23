# Creating Spark session
```python
from pyspark.sql import SparkSession
spark = (SparkSession
        .builder
        .master("url")
        .appName("application name")
        .config("spark-config-options")
        getOrCreate())
```
# Reading and Writing DataFrame

## Reading DataFrame
```python
df = (spark.read
    .format("csv")
    .option("inferSchema", "true")
    .option("header", True)
    .schema(schema)
    .load(file_path)  # If not given in option "path"
```
format can be = parquet, csv, json, avro, orc, image, binary file

```python
df = spark.read.csv("/file/path")
df = spark.read.json("/file/path")
df = spark.read.text("/file/path")
```
## Creating a schema
```python
from pyspark.sql.types import LongType, StringType, StructType, StructField

userDefinedSchema = StructType([
  StructField("user_id", StringType(), True),  
  StructField("user_first_touch_timestamp", LongType(), True),
  StructField("email", StringType(), True)
])
```

or using DDL
```python
DDLSchema = "user_id string, user_first_touch_timestamp long, email string"
```

## Printing schema
```python
df.printSchema()
```

## Writing DataFrame
```python
(df.write.format("format")
    .option("options")
    .bucketBy(args)
    .mode(mode)   # append, overwrite, ignore, error/errorifexists
    .partitionBy(args)
    .save(path)
)

(df.write.format(args)
    .option(args)
    .sortBy(args)
    .saveAsTable(table)
)

# writing to delta tables
(eventsDF.write
  .format("delta")
  .mode("overwrite")
  .save(eventsOutputPath)
)
```

# Creating DataFrame
## From RDD
```python
createDataFrame(rdd)

toDF()

toDF(*cols)

createDataFrame(dataList)

createDataFrame(rowData,columns)

createDataFrame(dataList, schema)
```

# DataFrame and Column


# Aggregation

# Datetimes

# Complex Types

# Joins

# UDFs

# Caching

# Query Optimizations


# Partitioning






