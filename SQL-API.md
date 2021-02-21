## Create Table using different file formats
```sql
DROP TABLE IF EXISTS People10M;
CREATE TABLE People10M
USING csv
OPTIONS (
    path "/path/to/the/file",
    header "true",
    inferschema "true"
)
```
File types - parquet, json, avro, orc


## Create temporary views
```sql
CREATE OR REPLACE TEMPORARY VIEW PeopleSavings AS
SELECT 
    col1,
    col2,
    col3
FROM People10M;
```

## Describe Table
```sql
DESCRIBE EXTENDED People10M;  --extended gives more details about the table
```


## Cast column to different datatype
```sql
SELECT ratings,
       CAST(timeRecorded AS timestamp)
FROM movieRatings;
```

## Select sample for table
```sql
SELECT * FROM Mytable
LIMIT 3;

SELECT * FROM outdoorProductsRaw TABLESAMPLE (5 ROWS);

SELECT * FROM outdoorProductsRaw TABLESAMPLE (2 PERCENT) ORDER BY InvoiceDate;
```


## Explode nested object
```sql
SELECT EXPLODE(column1)
FROM myTable;

```

## Collect exploded objects
```sql
SELECT COLLECT(column1)
FROM myTable;
```

## Accessing nested columns
```sql

SELECT value.description,
       value.ip,
       value.name
FROM MyTable;
```

## Common Table Expression (CTE)
```sql
WITH ExplodeSource
AS
(
    SELECT col1,
           col2,
           EXPLODE(source)
    FROM rawTable
)
SELECT
    col1,
    col2,
    col3,
    value.col1,
    value.col2,
    value.col3
FROM ExplodeSource
```

## Create table as select (CTAS)
CTEs are temporary and cannot queried again. Create a table using CTE.
```sql
DROP TABLE IF EXISTS DeviceData;
CREATE TABLE DeviceData                 
WITH ExplodeSource                       -- The start of the CTE from the last cell
AS
  (
  SELECT 
  dc_id,
  to_date(date) AS date,
  EXPLODE (source)
  FROM DCDataRaw
  )
SELECT 
  dc_id,
  key device_type,                       
  date,
  value.description,
  value.ip,
  value.temps,
  value.co2_level
FROM ExplodeSource;
```


## COALESCE, SPLIT

COALESCE() - will replace the null with a value you include.  
SPLIT() - splits a string value around a specified character and returns an zero index array

```sql
CREATE
OR REPLACE TEMPORARY VIEW outdoorProducts AS
SELECT
  InvoiceNo,
  StockCode,
  COALESCE(Description, "Misc") AS Description,
  Quantity,
  SPLIT(InvoiceDate, "/")[0] month,  -- "12/17/21 09:30" 
  SPLIT(InvoiceDate, "/")[1] day,
  SPLIT(SPLIT(InvoiceDate, " ")[0], "/")[2] year, 
  UnitPrice,
  Country
FROM
  outdoorProductsRaw
```
## LPAD, CONCAT_WS
LPAD() -  inserts characters to the left of a string until the string reachers a certain length  
CONCAT_WS() function to join the string
```sql
DROP TABLE IF EXISTS standardDate;
CREATE TABLE standardDate
WITH padStrings AS
(
SELECT 
  InvoiceNo,
  StockCode,
  Description,
  Quantity, 
  LPAD(month, 2, 0) AS month,
  LPAD(day, 2, 0) AS day,
  year,
  UnitPrice, 
  Country
FROM outdoorProducts
)
SELECT 
 InvoiceNo,
  StockCode,
  Description,
  Quantity, 
  concat_ws("/", month, day, year) sDate,
  UnitPrice,
  Country
FROM padStrings;
```

## TO_DATE, DATE_FORMAT
TO_DATE - converts the value to a date MM/dd/yy
date_format() - converts a timestamp to a string in the format specified
to_timestamp
```sql
SELECT
    col1,
    to_date(col2, "MM/dd/yy") purchase_date,
    date_format(col3, "E") day_of_week,
    to_timestamp(col4, 'yyyy/MM/dd HH:mm:ss') timeStamp
FROM mytable;
```
# Higher Order Functions
## FILTER
filter - creates a new column based on whether or not values in an array meets the conditions
```sql
SELECT
  categories,
  FILTER (categories, category -> category <> "Engineering Blog") woEngineering
FROM DatabricksBlog
```

## EXISTS
exists - checks whether a statement is true for one or more elements in an array.
```sql
SELECT
  categories,
  EXISTS (categories, c -> c = "Company Blog") companyFlag
FROM DatabricksBlog;
```

## TRANSFORM
transform - transform all elements of an array
```sql
SELECT
  TRANSFORM(categories, cat -> LOWER(cat)) lwrCategories
FROM DatabricksBlog;
```

## Checking size of an array
```sql
SELECT size(col1)
FROM myTable;
```

## REDUCE
reduce - More advanced than tranform. It takes two lambda functions.You can use it to reduce the elements of an array to a single value by merging the elements into a buffer, and applying a finishing function on the final buffer.
```sql
CREATE OR REPLACE TEMPORARY VIEW Co2LevelsTemporary
AS
  SELECT
    dc_id, 
    device_type,
    co2Level,
    REDUCE(co2Level, 0, (c, acc) -> c + acc, acc ->(acc div size(co2Level))) as averageCo2Level
  FROM DeviceData  
  SORT BY averageCo2Level DESC;

SELECT * FROM Co2LevelsTemporary
```


## PIVOTE
pivote - A pivot table allows you to transform rows into columns and group by any data field.
```sql
-- Example 1

SELECT * FROM (
  SELECT device_type, averageCo2Level 
  FROM Co2LevelsTemporary
)
PIVOT (
  ROUND(AVG(averageCo2Level), 2) avg_co2 
  FOR device_type IN ('sensor-ipad', 'sensor-inest', 
    'sensor-istick', 'sensor-igauge')
  );

-- Example 2
SELECT
  *
FROM
  (
    SELECT
      month(date) month,
      REDUCE(co2_Level, 0, (c, acc) -> c + acc, acc ->(acc div size(co2_Level))) averageCo2Level
    FROM
      DeviceData
  ) PIVOT (
    avg(averageCo2Level) avg FOR month IN (7 JUL, 8 AUG, 9 SEPT, 10 OCT, 11 NOV)
  )
```

## ROLLUP
rollup - allow you to summarize data based on the columns passed to the ROLLUP operator
```sql
SELECT 
  COALESCE(dc_id, "All data centers") AS dc_id,
  COALESCE(device_type, "All devices") AS device_type,
  ROUND(AVG(averageCo2Level))  AS avgCo2Level
FROM Co2LevelsTemporary
GROUP BY ROLLUP (dc_id, device_type)
ORDER BY dc_id, device_type;
```
## CUBE
cube - CUBE to generate summary values for sub-elements grouped by column value. 
```sql
SELECT 
  COALESCE(dc_id, "All data centers") AS dc_id,
  COALESCE(device_type, "All devices") AS device_type,
  ROUND(AVG(averageCo2Level))  AS avgCo2Level
FROM Co2LevelsTemporary
GROUP BY CUBE (dc_id, device_type)
ORDER BY dc_id, device_type;
```

# Partitioning Table

```sql
CREATE TABLE IF NOT EXISTS AvgTemps
PARTITIONED BY (device_type)
AS
  SELECT
    dc_id,
    date,
    temps,
    REDUCE(temps, 0, (t, acc) -> t + acc, acc ->(acc div size(temps))) as avg_daily_temp_c,
    device_type
  FROM DeviceData;
  
SELECT * FROM AvgTemps;
```

## See partitions
```sql
SHOW PARTITIONS AvgTemps;
```

## Creating widgets in databricks 
```sql
CREATE WIDGET DROPDOWN selectedDeviceType DEFAULT "sensor-inest" CHOICES
SELECT
  DISTINCT device_type
FROM
  DeviceData
```

use widget in your query - 
```sql
SELECT 
  device_type,
  ROUND(AVG(avg_daily_temp_c),4) AS avgTemp,
  ROUND(STD(avg_daily_temp_c), 2) AS stdTemp
FROM AvgTemps
WHERE device_type = getArgument("selectedDeviceType")
GROUP BY device_type
```

remove widget 
```sql
REMOVE WIDGET selectedDeviceType;
```

## Window Functions
```sql
SELECT 
  dc_id,
  month(date),
  avg_daily_temp_c,
  AVG(avg_daily_temp_c)
  OVER (PARTITION BY month(date), dc_id) AS avg_monthly_temp_c
FROM AvgTemps
WHERE month(date)="8" AND dc_id = "dc-102";
```