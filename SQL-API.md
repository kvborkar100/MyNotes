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
```sql
SELECT
    col1,
    to_date(col2, "MM/dd/yy") purchase_date,
    date_format(col3, "E") day_of_week
FROM mytable;
```