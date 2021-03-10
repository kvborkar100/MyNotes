# CX_PYTHON

## Creating connection
```python
import cx_Oracle
import sys

try:
    cx_Oracle.init_oracle_client(
        config_dir="/home/krushna/Documents/Krushna/practice/cx_oracle/wallet")
    print("Successful...")
    connection = cx_Oracle.connect(
        "ADMIN", "Blackpanther22", "mydb_high", encoding="UTF-8")
    print("Connected")
    print(connection.version)
except Exception as err:
    print("Whoops!")
    print(err)
    sys.exit(1)

```

### Creating session pool
```python
# Create the session pool
pool = cx_Oracle.SessionPool(
    config.username,
    config.password,
    config.database,
    min=100,
    max=100,
    increment=0,
    encoding=config.encoding
)

# Acquire a connection from the pool
connection = pool.acquire()

# Release the connection to the pool
pool.release(connection)

# Close the pool
pool.close()
```
## Closing connection
```python
if connection:
    connection.close()
```

## fetchone(), fetchmany(), fetchall()
```python
import cx_Oracle
import config

sql = 'select customer_id, name ' \
    'from customers ' \
    'order by name'
try:
    with cx_Oracle.connect(
                config.username,
                config.password,
                config.dsn,
                encoding=config.encoding) as connection:
        with connection.cursor() as cursor:
            cursor.execute(sql)
            while True:
                row = cursor.fetchone() # similarly fetchall, fetchmany
                if row is None:
                    break
                print(row)
except cx_Oracle.Error as error:
    print(error)
```
### using bind variables
```python
cursor.execute(sql, [100])  # here 100 is passed as bind variable
cursor.execute(sql, price=600, cost=500) 
#or
cursor.execute(sql, {price:600, cost:500})
#or binding by position
cursor.execute(sql, [600, 500])
```

## Cursor.executemany()
```python
cursor.executemany(sql, billings)
```

## Transaction Management
```python
#commit
cursor.execute('<DML statement>')
connection.commit()

#rollback
cursor.execute('<DML statement>')
connection.rollback()

#autocommit
connection.autocommit = True
``` 

## PLSQL Procedure
```python
order_count = cursor.var(int)   #create variable to hold out parameter
cursor.callproc('get_order_count',
                [salesman_id, year, order_count])   #calling procedure
return order_count.getvalue()    #returning procedure out variable
```

## PLSQL Function
```python
revenue = cursor.callfunc('get_revenue',
                        float,
                        [salesman_id, year])
```
