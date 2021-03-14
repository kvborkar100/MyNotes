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

## adding literal/constant
```python
convertedUsersDF = (salesDF.select("email").dropDuplicates().withColumn("converted", lit(True))
)
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
# Datetime Functions
## cast()
casts column to different datatype
```python
timestampDF = df.withColumn("timestamp", (col("timestamp") / 1e6).cast("timestamp"))

#or
from pyspark.sql.types import TimestampType

timestampDF = df.withColumn("timestamp", (col("timestamp") / 1e6).cast(TimestampType()))
```

## date_format()
Converts a date/timestamp/string to a string formatted with the given date time pattern.
```python
from pyspark.sql.functions import date_format

formattedDF = (timestampDF.withColumn("date string", date_format("timestamp", "MMMM dd, yyyy"))
  .withColumn("time string", date_format("timestamp", "HH:mm:ss.SSSSSS"))
) 
```

## extacting datetime attributes
```python
from pyspark.sql.functions import year, month, dayofweek, minute, second

datetimeDF = (timestampDF.withColumn("year", year(col("timestamp")))
  .withColumn("month", month(col("timestamp")))
  .withColumn("dayofweek", dayofweek(col("timestamp")))
  .withColumn("minute", minute(col("timestamp")))
  .withColumn("second", second(col("timestamp")))              
)
```

## to_date()
Converts the column into DateType by casting rules to DateType.
```python
from pyspark.sql.functions import to_date

dateDF = timestampDF.withColumn("date", to_date(col("timestamp")))
```

## manipulating datetime- date_add()
```python
from pyspark.sql.functions import date_add

plus2DF = timestampDF.withColumn("plus_two_days", date_add(col("timestamp"), 2))
```

# Complex Types
## explode(), split()

```python
from pyspark.sql.functions import *

detailsDF = (df.withColumn("items", explode("items"))    #explode the column
  .select("email", "items.item_name")    #select from exploded column
  .withColumn("details", split(col("item_name"), " ")) #split using space and form an array     
)
#kmunoz@powell-duran.com | Premium King Mattress | ["Premium", "King", "Mattress"]
```
## array_contains(), element_at()
Extracting details from arrays
```python
mattressDF = (detailsDF.filter(array_contains(col("details"), "Mattress"))
  .withColumn("size", element_at(col("details"), 2))
  .withColumn("quality", element_at(col("details"), 1))
) 
#kmunoz@powell-duran.com | Premium King Mattress | ["Premium", "King", "Mattress"] | King | Premium
```
## unionByName()
```python
unionDF = (mattressDF.unionByName(pillowDF)
  .drop("details"))
```
## collect_set()
```python
optionsDF = (unionDF.groupBy("email")
  .agg(collect_set("size").alias("size options"),
       collect_set("quality").alias("quality options"))
)
#aallen43@hotmail.com | ["Queen", "Twin"] | ["Premium", "Standard"]
```
# Joins
```python
from pyspark.sql.functions import col
conversionsDF = (usersDF.join(convertedUsersDF, "email", "outer")
                 .filter(col("email").isNotNull()).fillna(False, ["converted"])
)

from pyspark.sql.functions import explode,collect_set
cartsDF = (eventsDF.withColumn("items", explode("items"))
           .groupBy("user_id").agg(collect_set("items.item_id").alias("cart"))
)

emailCartsDF = conversionsDF.join(cartsDF, "user_id", "left")

abandonedCartsDF = (emailCartsDF.filter((emailCartsDF.converted == False ) & emailCartsDF.cart.isNotNull())
)
```
# UDFs
```python
#define a function
def firstLetterFunction(email):
  return email[0]

firstLetterFunction("annagray@kaufman.com")

#define a UDF to wrap the function
firstLetterUDF = udf(firstLetterFunction)

#apply the UDF
from pyspark.sql.functions import col
display(salesDF.select(firstLetterUDF(col("email"))))
```

## registering udf to use in SQL
```python
spark.udf.register("sql_udf", firstLetterFunction)

%sql
SELECT sql_udf(email) AS firstLetter FROM sales
```

## Decorator syntax
```python
@udf("string")
def decoratorUDF(email: str) -> str:
  return email[0]

display(salesDF.select(decoratorUDF(col("email"))))
```

```python
import pandas as pd
from pyspark.sql.functions import pandas_udf

# We have a string input/output
@pandas_udf("string")
def vectorizedUDF(email: pd.Series) -> pd.Series:
  return email.str[0]

# Alternatively
vectorizedUDF = pandas_udf(lambda s: s.str[0], "string")  

#or
display(salesDF.select(vectorizedUDF(col("email"))))
```
# Caching

## cache(), persist() alias
A call to cache() does not immediately materialize the data in cache.
An action using the DataFrame must be executed for Spark to actually cache the data.
```python
df.cache()
```
As a best practice, you should always evict your DataFrames from cache when you no longer need them

```python
df.unpersist()
```

## cache table
```python
df.createOrReplaceTempView("Pageviews_DF_Python")
spark.catalog.cacheTable("Pageviews_DF_Python")
```
# Query Optimizations

## explain()
Prints the plans (logical and physical), optionally formatted by a given explain mode.
```python
limitEventsDF.explain(True)
```
# Partitioning
```python
df = spark.read.parquet(eventsPath)
df.rdd.getNumPartitions()

print(spark.sparkContext.defaultParallelism)
# print(sc.defaultParallelism)
```

## repartition()
```python
repartitionedDF = df.repartition(8)

repartitionedDF.rdd.getNumPartitions()
```
## coalesce()
Returns a new DataFrame that has exactly n partitions, when the fewer partitions are requested

If a larger number of partitions is requested, it will stay at the current number of partitions
```python
coalesceDF = df.coalesce(8)
coalesceDF.rdd.getNumPartitions()
```

## configure default partitions
Use SparkConf to access the spark configuration parameter for default shuffle partitions
```python
spark.conf.get("spark.sql.shuffle.partitions")
spark.conf.set("spark.sql.shuffle.partitions", "8")
```

## AQE (Adaptive Query Execution)
In Spark 3, AQE is now able to dynamically coalesce shuffle partitions at runtime

Spark SQL can use spark.sql.adaptive.enabled to control whether AQE is turned on/off (disabled by default)

```python
spark.conf.get("spark.sql.adaptive.enabled")
```
