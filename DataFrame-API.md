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
```python
eventsDF = spark.read.parquet(eventsPath)
display(eventsDF)        #displays dataframe
```

## Creating new columns
```python
from pyspark.sql.functions import col

col("device")
eventsDF.device
eventsDF["device"]

eventsDF.select("device").show()   # selecting a columns
```

## Performing calculations on column
```python
col("ecommerce.purchase_revenue_in_usd") + col("ecommerce.total_item_quantity")

col("event_timestamp").desc()

(col("ecommerce.purchase_revenue_in_usd") * 100).cast("int")
```

## Seleting subset of columns - select()
```python
devicesDF = eventsDF.select("user_id", "device")
display(devicesDF)
```

Giving alias to column and selecting nested columns
```python
from pyspark.sql.functions import col

locationsDF = eventsDF.select("user_id", 
  col("geo.city").alias("city"),
  col("geo.state").alias("state"))
```
## Selecting using SQL expression
```python
appleDF = eventsDF.selectExpr("user_id", "device in ('macOS', 'iOS') as apple_user")
display(appleDF)
```

## Dropping a column
```python
anonymousDF = eventsDF.drop("user_id", "geo", "device")
noSalesDF = eventsDF.drop(col("ecommerce"))
```

## Adding or replacing a column - withColumn(), withColumnRenamed()
withColumn- Returns a new DataFrame by adding a column or replacing the existing column that has the same name
withColumnRenamed - Returns a new DataFrame with a column renamed
```python
mobileDF = eventsDF.withColumn("mobile", col("device").isin("iOS", "Android"))

purchaseQuantityDF = eventsDF.withColumn("purchase_quantity", col("ecommerce.total_item_quantity").cast("int"))
```

## Filtering rows
Filters rows using the given SQL expression or column based condition

```python
purchasesDF = eventsDF.filter("ecommerce.total_item_quantity > 0")
revenueDF = eventsDF.filter(col("ecommerce.purchase_revenue_in_usd").isNotNull())
androidDF = eventsDF.filter((col("traffic_source") != "direct") & (col("device") == "Android"))
```

## dropping duplicate columns - distinct(), dropDuplicates()
Returns a new DataFrame with duplicate rows removed, optionally considering only a subset of columns.
```python
eventsDF.distinct()
distinctUsersDF = eventsDF.dropDuplicates(["user_id"])
```

## Limiting df rows - limit()
```python
limitDF = eventsDF.limit(100)
```

## Sorting rows- sort(), orderBy()
```python
increaseTimestampsDF = eventsDF.sort("event_timestamp")
decreaseTimestampsDF = eventsDF.sort(col("event_timestamp").desc())
increaseSessionsDF = eventsDF.orderBy(["user_first_touch_timestamp", "event_timestamp"])
``` 

# Aggregation
```python
df.groupBy("event_name")
#count
eventCountsDF = df.groupBy("event_name").count() 
#avg 
avgStatePurchasesDF = df.groupBy("geo.state").avg("ecommerce.purchase_revenue_in_usd") 
#sum
cityPurchaseQuantitiesDF = df.groupBy("geo.state", "geo.city").sum("ecommerce.total_item_quantity")
```

Builtin aggregate functions
Use the grouped data method agg to apply built-in aggregate functions. This allows you to apply other transformations on the resulting columns, such as alias

```python
from pyspark.sql.functions import sum

statePurchasesDF = df.groupBy("geo.state").agg(sum("ecommerce.total_item_quantity").alias("total_purchases"))

#applying multiple aggregate functions
from pyspark.sql.functions import avg, approx_count_distinct

stateAggregatesDF = df.groupBy("geo.state").agg(
  avg("ecommerce.total_item_quantity").alias("avg_quantity"),
  approx_count_distinct("user_id").alias("distinct_users"))
```
# Datetimes

# Complex Types

# Joins

# UDFs

# Caching

# Query Optimizations


# Partitioning






