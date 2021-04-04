# Columns
```sql
create table students(
    name VARCHAR2(100),
    roll_num numbeNUMBER,
    student_id NUMBER GENERATED ALWAYS AS IDENTITY
);

--or
--id NUMBER GENERATED BY DEFAULT AS IDENTITY
--GENERATED ALWAYS AS IDENTITY START WITH 1 INCREMENT BY 1
```
## adding column
```sql
alter table student add(
    last_name VARCHAR2(100)
);
```

## removing column
```sql
alter table student drop(
    last_name VARCHAR2(100)
)
```

# Constraint
## drop constraint
```sql
ALTER TABLE table_name
DROP CONSTRAINT constraint_name;
```

# Joins
## Cross Join
Returns every row of first table matched with every row of second table.

```sql
select *
from tabl1, table2;

select *
from table1
cross join table2;
```

## Inner Join
returns only matching rows
```sql
select *
from table1 t1,
     table2 t2
where t1.id = t2.id;

select *
from table1 t2
inner join tablet2
on t1.id = t2.id
```

## Outer join

### Left outer join
Returns all rows from table1 to matching rows from table2.
```sql
SELECT *
FROM table1 t1
LEFT JOIN table2 t2 ON t1.id = t2.id;

select *
from table1 t1,
     table2 t2
where t1.id = t2.id(+);
```

### Right outer Join
Returns all rows from table2 to matching rows from table1.
```sql
SELECT *
FROM table1 t1
RIGHT JOIN table2 t2 ON t1.id = t2.id;


select *
from table1 t1,
     table2 t2
where t1.id(+) = t2.id;
```

### Full Outer Join
returns the result of left and right join
```sql
SELECT *
FROM table1 t1
FULL JOIN table2 t2 ON t1.id = t2.id;

select *
from table1 t1,
     table2 t2
where t1.id = t2.id(+)
UNION ALL
select *
from table1 t1,
     table2 t2
where t1.id(+) = t2.id;
```
# Insert
```sql
insert into table_name(col1, col2,...)
values(val1,val2,...);
```
## Multi table insert
```sql
insert all
into table1..
into table2.. 
```

# update
```sql
update table_name
set col1 = val1,
    col2 = val2
where id = 3;
```

## select for update
no one can update, delete or select the record
```sql
update table_name
set col1 = val1,
    col2 = val2
where id = 3
for update;
```
# Delete & Truncate
Delete - Removes all rows from table. If rollback issued it will revert all rows.

TRUNCATE is DDL. Cannot be roll back. Cannot specify where condition.
```sql
delete from table_name
where id = 3;

DROP TABLE Shippers;

TRUNCATE TABLE Categories;
```
DDL - CREATE, ALTER, TRUNCATE, DROP, RENAME
DML - SELECT, INSERT, UPDATE, DELETE

# NULL
is NULL  
is NOT NULL
col > 1000 -> null is excluded

## NVL
```sql
select NVL(col1, 1) 
from table_name;
```

## COALESCE
Can take any no of arguments, return first non null value.
```sql
select COALESCE(col1, col2, col3, 1)
from table_name;
```

# Subqueries
## Inline views
```sql
select *
from (select * from table1) table2;
```

## Nested subqueries
```sql
select *
from table1
where col1 in (
    select col2 from table2
);

select *
from table1 t1
where 1 = 1
and exists (
    select 1
    from table2 t2
    where t1.id = t2.id
)
```

## CTE - Common table Expressions
```sql
with mySUB as(
    select col1,
           col2
    from table1
)
select *
from mySUB;
```

# Top N
```sql
select *
from (
    select *
    from table1
    order by col1
)
where rownum <= 3;

select *
from ( 1.*
    row_number() over (order by marks) rn
    from table1
)
where rn >= 3

select *
from table1
order by col1 desc
fetch first 3 rows only;

select *
from table1
order by sal desc
fetch first 3 rows with ties; --can be 3 or more rows
```

# Analytical Queries
## Ranking
- RANK()
- DENSE_RANK()
- ROW_NUMBER()
- PERCENT_RANK()
- CUM_DIST()
- NTILE(n)

## Partitioning
- sum()
- row_number()
- avg()
- min()
- max()
- count()

## Windows
```sql
--given same result i.e running total
select *
from (
    select a.*,
            sum(salary) over (partition by department_id
                                order by salary desc) rn1,
            sum(salary) over (partition by department_id
                                order by salary desc
                                rows between unbounded preceding and current row) rn2
                                
    from employees a
);
```
- rows between unbounded preceding and current row  
- rows between 1 preceding and 1 following  
- range between interval 5 year preceding and interval 5 year following


nth_value(trip_count, 2) over()

## LAG LEAD

```sql
select *
from (
    select a.*,
            sum(salary) over (partition by department_id
                                order by salary desc) rn1,
            sum(salary) over (partition by department_id
                                order by salary desc
                                rows between unbounded preceding and current row) rn2,
            sum(salary) over (partition by department_id
                                order by salary desc
                                rows between 1 preceding and 1 following) rn3,
            nth_value(salary,2) over (partition by department_id
                                order by salary desc
                                rows between unbounded preceding and current row) rn4,
            nth_value(salary,2) over (partition by department_id
                                order by salary desc
                                rows between unbounded preceding and unbounded following) rn5,
            lag(employee_id,1) over (partition by department_id
                                order by salary desc) rn6,
            lead(employee_id,1) over (partition by department_id
                                order by salary desc) rn7
                                
    from employees a
);
```


# Order of execution of SQL
from and join
where
group by
having
select
distinct
order by
limit