# PLSQL
## Anonymous Block
```sql
    --declaration section
BEGIN
    --execution section
    EXCEPTION
    --exception section
END;
```

```sql
SET SERVEROUTPUT ON;

BEGIN
    dbms_output.put_line('Hello World');
END;
```
```sql
DECLARE
    num NUMBER;
BEGIN
    num := 10 / 0;
EXCEPTION
    WHEN zero_divide THEN
        dbms_output.put_line('Zero dividde error..');
END;
```

## Data Types
- NUMBER, BINARY_FLOAT, BINARY_DOUBLE, PLS_INTEGER
- BOOLEAN
- CHAR(n) fixed length 1 to 32767
- VARCHAR2(n) varying length 1 to 32767
- LONG, RAW, ROWID 
- DATE, TIMESTAMP

## Variables
```
variable_name datatype [NOT NULL] [:= initial_value];
```
Examples - 
```sql
l_total_sales NUMBER(15,2);
l_credit_limit NUMBER (10,0);    
l_contact_name VARCHAR2(255);
l_product_name VARCHAR2( 100 ) := 'Laptop';
--or
l_product_name VARCHAR2(100) DEFAULT 'Laptop';
```

If Not null constraint is applied then the varible cannot accept a NULL value. Also it must be initialized with a value.

```sql
l_shipping_status VARCHAR2( 25 ) NOT NULL := 'Shipped';
```
Note that PL/SQL treats a zero-length string as a NULL value.

### Varible assignment
```sql
l_lead_for := l_business_parter; 
l_customer_group := 'Gold';
```

### Anchored Declarations
Declaring variable type as per the table column
```sql
l_customer_name customers.name%TYPE;
l_credit_limit customers.credit_limit%TYPE;
```
You can also declare varibles than anchor to another variable
```sql
l_demo l_credit_limit%TYPE;
```

## Comments
### Single line comments
```sql
-- this is a single line comment

/*
this
is multi line command
*/
``` 

## Constant
```sql
co_payment_term   CONSTANT NUMBER   := 45; -- days 
co_payment_status CONSTANT BOOLEAN  := FALSE; 
```

## IF Statement
### IF THEN
```sql
DECLARE
    num NUMBER := 100;
BEGIN
    IF num > 50 THEN
        dbms_output.put_line('INSIDE IF');
    END IF;
EXCEPTION
    WHEN OTHERS THEN
        dbms_output.put_line('ERROR IN THE BLOCK');
END;
```

### TIPS:
1. Avoid clumsy IF statements
```sql
  IF n_sales > n_costs THEN
    b_profitable := true;
  END IF;

-- replace with
b_profitable := n_sales > n_costs;
```
2. Avoid evaluating Boolean variables
```sql
IF b_profitable = TRUE THEN
   DBMS_OUTPUT.PUT_LINE( 'This sales deal is profitable' );
END IF;

-- replace with
IF b_profitable THEN
   DBMS_OUTPUT.PUT_LINE( 'This sales deal is profitable' );
END IF;
```

### IF THEN ELSE
```sql
BEGIN
IF condition THEN
    statements;
ELSE
    else_statements;
END IF;
END;
```

### IF THEN ELSIF
```sql
BEGIN
IF condition1 THEN
    statements1;
ELSIF CONDITION2 THEN
    statements2;
ELSE
    statements;
END IF;
END;
```

## CASE Statement
```sql
BEGIN
CASE c_grade
  WHEN 'A' THEN
    c_rank := 'Excellent' ;
  WHEN 'B' THEN
    c_rank := 'Very Good' ;
  WHEN 'C' THEN
    c_rank := 'Good' ;
  WHEN 'D' THEN
    c_rank := 'Fair' ;
  WHEN 'F' THEN
    c_rank := 'Poor' ;
  ELSE
    c_rank := 'No such grade' ;
  END CASE;
END;
```

Search case statement
```sql
BEGIN
  n_sales := 150000;
  CASE
  WHEN n_sales    > 200000 THEN
    n_commission := 0.2;
  WHEN n_sales   >= 100000 AND n_sales < 200000 THEN
    n_commission := 0.15;
  WHEN n_sales   >= 50000 AND n_sales < 100000 THEN
    n_commission := 0.1;
  WHEN n_sales    > 30000 THEN
    n_commission := 0.05;
  ELSE
    n_commission := 0;
  END CASE;

  DBMS_OUTPUT.PUT_LINE( 'Commission is ' || n_commission * 100 || '%'
  );
END;
```

## GOTO statement
Allows us to transfer control tolabelled blok or statement
```sql
BEGIN
  GOTO second_message;

  <<first_message>>
  DBMS_OUTPUT.PUT_LINE( 'Hello' );
  GOTO the_end;

  <<second_message>>
  DBMS_OUTPUT.PUT_LINE( 'PL/SQL GOTO Demo' );
  GOTO first_message;

  <<the_end>>
  DBMS_OUTPUT.PUT_LINE( 'and good bye...' );

END;
```

## NULL statements
```sql
BEGIN
IF job_title = 'Sales Representative' THEN
    send_email;
ELSE
    NULL;
END IF;
END;
```

## LOOP statement
### EXIT
```sql
BEGIN
LOOP
    IF condition THEN
        EXIT;
    END IF;
END LOOP;
END;
```
Eg
```sql
DECLARE
  l_counter NUMBER := 0;
BEGIN
  LOOP
    l_counter := l_counter + 1;
    IF l_counter > 3 THEN
      EXIT;
    END IF;
    dbms_output.put_line( 'Inside loop: ' || l_counter )  ;
  END LOOP;
  -- control resumes here after EXIT
  dbms_output.put_line( 'After loop: ' || l_counter );
END;
```

### EXIT WHEN
```sql
DECLARE
  l_counter NUMBER := 0;
BEGIN
  LOOP
    l_counter := l_counter + 1;
    EXIT WHEN l_counter > 3;
    dbms_output.put_line( 'Inside loop: ' || l_counter ) ;
  END LOOP;

  -- control resumes here after EXIT
  dbms_output.put_line( 'After loop: ' || l_counter );
END;
```

### NESTED LOOPS
```sql
DECLARE
    l_i  NUMBER := 0;
    l_j  NUMBER := 0;
BEGIN
    << outer_loop >> LOOP
        l_i := l_i + 1;
        EXIT outer_loop WHEN l_i > 2;
        dbms_output.put_line('Outer counter ' || l_i);
    -- reset inner counter
            l_j := 0;
        << inner_loop >> LOOP
            l_j := l_j + 1;
            EXIT inner_loop WHEN l_j > 3;
            dbms_output.put_line(' Inner counter ' || l_j);
        END LOOP inner_loop;

    END LOOP outer_loop;
END;
```

## FOR Loops
```sql
BEGIN
  FOR l_counter IN 1..5
  LOOP
    DBMS_OUTPUT.PUT_LINE( l_counter );
  END LOOP;
END;
```
```sql
BEGIN
  FOR l_counter IN REVERSE 1..3
  LOOP
    DBMS_OUTPUT.PUT_LINE( l_counter );
  END LOOP;
END;
```

## WHILE Loop
```sql
DECLARE
  n_counter NUMBER := 1;
BEGIN
  WHILE n_counter <= 5
  LOOP
    DBMS_OUTPUT.PUT_LINE( 'Counter : ' || n_counter );
    n_counter := n_counter + 1
  END LOOP;
END;
```

## SELECT INTO
### Single column
```sql
DECLARE
  l_customer_name customers.name%TYPE;
BEGIN
  -- get name of the customer 100 and assign it to l_customer_name
  SELECT name INTO l_customer_name
  FROM customers
  WHERE customer_id = 100;
  -- show the customer name
  dbms_output.put_line( v_customer_name );
END;
```

### Select complete row
```sql
DECLARE
  r_customer customers%ROWTYPE;
BEGIN
  -- get the information of the customer 100
  SELECT * INTO r_customer
  FROM customers
  WHERE customer_id = 100;
  -- show the customer info
  dbms_output.put_line( r_customer.name || ', website: ' || r_customer.website );
END;
```
### Selecting multiple columns
```sql
DECLARE
  l_customer_name customers.name%TYPE;
  l_contact_first_name contacts.first_name%TYPE;
  l_contact_last_name contacts.last_name%TYPE;
BEGIN
  -- get customer and contact names
  SELECT
    name, 
    first_name, 
    last_name
  INTO
    l_customer_name, 
    l_contact_first_name, 
    l_contact_last_name
  FROM
    customers
  INNER JOIN contacts USING( customer_id )
  WHERE
    customer_id = 100;
  -- show the information  
  dbms_output.put_line( 
    l_customer_name || ', Contact Person: ' ||
    l_contact_first_name || ' ' || l_contact_last_name );
END;
```

## EXCEPTION Handling
```sql
DECLARE
    l_name customers.NAME%TYPE;
    l_customer_id customers.customer_id%TYPE := &customer_id;
BEGIN
    -- get the customer
    SELECT NAME INTO l_name
    FROM customers
    WHERE customer_id > l_customer_id;
    
    -- show the customer name   
    dbms_output.put_line('Customer name is ' || l_name);
    EXCEPTION 
        WHEN NO_DATA_FOUND THEN
            dbms_output.put_line('Customer ' || l_customer_id ||  ' does not exist');
        WHEN TOO_MANY_ROWS THEN
            dbms_output.put_line('The database returns more than one customer');    
END;
```
### Raising an exception 
```sql
DECLARE
    exception_name EXCEPTION;
    PRAGMA EXCEPTION_INIT (exception_name, error_number);
```
the **error_code** is an integer that ranges from -20,999 to -20,000. And the message is a character string with a maximum length of 2,048 bytes.

procedure **raise_application_error** allows you to issue an user-defined error from a code block or stored program.

Eg. 
```sql
DECLARE
    e1 EXCEPTION;
    PRAGMA exception_init (e1, -20001);
    e2 EXCEPTION;
    PRAGMA exception_init (e2, -20002);
    e3 EXCEPTION;
    PRAGMA exception_init (e2, -20003);
    l_input NUMBER := &input_number;
BEGIN
    -- inner block
    BEGIN
        IF l_input = 1 THEN
            raise_application_error(-20001,'Exception: the input number is 1');
        ELSIF l_input = 2 THEN
            raise_application_error(-20002,'Exception: the input number is 2');
        ELSE
            raise_application_error(-20003,'Exception: the input number is not 1 or 2');
        END IF;
    -- exception handling of the inner block
    EXCEPTION
        WHEN e1 THEN 
            dbms_output.put_line('Handle exception when the input number is 1');
    END;
    -- exception handling of the outer block
    EXCEPTION 
        WHEN e2 THEN
            dbms_output.put_line('Handle exception when the input number is 2');
END;
/
```

## Records

### Table Based
```sql
DECLARE
   record_name table_name%ROWTYPE;

DECLARE
   r_contact contacts%ROWTYPE;
```

### Cursor based
```sql
DECLARE
    record_name cursor_name%ROWTYPE;

DECLARE
    CURSOR c_contacts IS
        SELECT first_name, last_name, phone
        FROM contacts;
    r_contact c_contacts%ROWTYPE;
```

## Cursors
Whenever Oracle executes an SQL statement such as SELECT INTO, INSERT, UPDATE, and DELETE, it automatically creates an implicit cursor.

```sql
CURSOR cursor_name IS query;
```
Before start fetching rows from the cursor, you must open it. To open a cursor, you use the following syntax:
```
OPEN cursor_name;
```
fetch from cursor
```
FETCH cursor_name INTO variable_list;
```
After fetching all rows, you need to close the cursor with the CLOSE statement:
```
CLOSE cursor_name;
```
### Cursor atributes
```
cursor_name%attribute
```
- %ISOPEN
- %FOUND
- %NOTFOUND
- %ROWCOUNT

```sql
DECLARE
  l_budget NUMBER := 1000000;
   -- cursor
  CURSOR c_sales IS
  SELECT  *  FROM sales  
  ORDER BY total DESC;
   -- record    
   r_sales c_sales%ROWTYPE;
BEGIN

  -- reset credit limit of all customers
  UPDATE customers SET credit_limit = 0;

  OPEN c_sales;

  LOOP
    FETCH  c_sales  INTO r_sales;
    EXIT WHEN c_sales%NOTFOUND;

    -- update credit for the current customer
    UPDATE 
        customers
    SET  
        credit_limit = 
            CASE WHEN l_budget > r_sales.credit 
                        THEN r_sales.credit 
                            ELSE l_budget
            END
    WHERE 
        customer_id = r_sales.customer_id;

    --  reduce the budget for credit limit
    l_budget := l_budget - r_sales.credit;

    DBMS_OUTPUT.PUT_LINE( 'Customer id: ' ||r_sales.customer_id || 
' Credit: ' || r_sales.credit || ' Remaining Budget: ' || l_budget );

    -- check the budget
    EXIT WHEN l_budget <= 0;
  END LOOP;

  CLOSE c_sales;
END;
```

## CURSOR with FOR Loop
```
FOR record IN cursor_name
LOOP
    process_record_statements;
END LOOP;
```
```sql
DECLARE
  CURSOR c_product
  IS
    SELECT 
        product_name, list_price
    FROM 
        products 
    ORDER BY 
        list_price DESC;
BEGIN
  FOR r_product IN c_product
  LOOP
    dbms_output.put_line( r_product.product_name || ': $' ||  r_product.list_price );
  END LOOP;
END;
```
```
FOR record IN (select_statement)
LOOP
    process_record_statements;
END LOOP;
```
```sql
BEGIN
  FOR r_product IN (
        SELECT 
            product_name, list_price 
        FROM 
            products
        ORDER BY list_price DESC
    )
  LOOP
     dbms_output.put_line( r_product.product_name ||
        ': $' || 
        r_product.list_price );
  END LOOP;
END;
```

## Parameterized Cursors
```
CURSOR cursor_name (parameter_list) 
IS
cursor_query;

OPEN cursor_name (value_list);
```

```sql
DECLARE
    CURSOR c_revenue (in_year NUMBER :=2017 , in_customer_id NUMBER := 1)
    IS
        SELECT SUM(quantity * unit_price) revenue
        FROM order_items
        INNER JOIN orders USING (order_id)
        WHERE status = 'Shipped' AND EXTRACT( YEAR FROM order_date) = in_year
        GROUP BY customer_id
        HAVING customer_id = in_customer_id;
        
    r_revenue c_revenue%rowtype;
BEGIN
    OPEN c_revenue;
    LOOP
        FETCH c_revenue INTO r_revenue;
        EXIT    WHEN c_revenue%notfound;
        -- show the revenue
        dbms_output.put_line(r_revenue.revenue);
    END LOOP;
    CLOSE c_revenue;
END;
```

## PROCEDURE
```
CREATE [OR REPLACE ] PROCEDURE procedure_name (parameter_list)     
IS
```
IN - An IN parameter is read-only.
OUT - An OUT parameter is writable.
INOUT - An INOUT parameter is both readable and writable. The procedure can read and modify it.

```sql
CREATE OR REPLACE PROCEDURE print_contact(
    in_customer_id NUMBER 
)
IS
  r_contact contacts%ROWTYPE;
BEGIN
  -- get contact based on customer id
  SELECT *
  INTO r_contact
  FROM contacts
  WHERE customer_id = p_customer_id;

  -- print out contact's information
  dbms_output.put_line( r_contact.first_name || ' ' ||
  r_contact.last_name || '<' || r_contact.email ||'>' );

EXCEPTION
   WHEN OTHERS THEN
      dbms_output.put_line( SQLERRM );
END;
```

### EXECUTING procedure
```
EXECUTE procedure_name( arguments);
EXEC print_contact(100);
```

## FUNCTION
```
CREATE [OR REPLACE] FUNCTION function_name (parameter_list)
    RETURN return_type
IS
```
```sql
CREATE OR REPLACE FUNCTION get_total_sales(
    in_year PLS_INTEGER
) 
RETURN NUMBER
IS
    l_total_sales NUMBER := 0;
BEGIN
    -- get total sales
    SELECT SUM(unit_price * quantity)
    INTO l_total_sales
    FROM order_items
    INNER JOIN orders USING (order_id)
    WHERE status = 'Shipped'
    GROUP BY EXTRACT(YEAR FROM order_date)
    HAVING EXTRACT(YEAR FROM order_date) = in_year;
    
    -- return the total sales
    RETURN l_total_sales;
END;
```
## Package
In PL/SQL, a package is a schema object that contains definitions for a group of related functionalities. A package includes variables, constants, cursors, exceptions, procedures, functions, and subprograms. It is compiled and stored in the Oracle Database.

### Package Specification
```sql
CREATE OR REPLACE PACKAGE order_mgmt
AS
  gc_shipped_status  CONSTANT VARCHAR(10) := 'Shipped';
  gc_pending_status CONSTANT VARCHAR(10) := 'Pending';
  gc_canceled_status CONSTANT VARCHAR(10) := 'Canceled';

  -- cursor that returns the order detail
  CURSOR g_cur_order(p_order_id NUMBER)
  IS
    SELECT
      customer_id,
      status,
      salesman_id,
      order_date,
      item_id,
      product_name,
      quantity,
      unit_price
    FROM
      order_items
    INNER JOIN orders USING (order_id)
    INNER JOIN products USING (product_id)
    WHERE
      order_id = p_order_id;

  -- get net value of a order
  FUNCTION get_net_value(
      p_order_id NUMBER)
    RETURN NUMBER;

  -- Get net value by customer
  FUNCTION get_net_value_by_customer(
      p_customer_id NUMBER,
      p_year        NUMBER)
    RETURN NUMBER;

END order_mgmt;
```

### Package Body
```sql
CREATE OR REPLACE PACKAGE BODY order_mgmt
AS
  -- get net value of a order
  FUNCTION get_net_value(
      p_order_id NUMBER)
    RETURN NUMBER
  IS
    ln_net_value NUMBER 
  BEGIN
    SELECT
      SUM(unit_price * quantity)
    INTO
      ln_net_value
    FROM
      order_items
    WHERE
      order_id = p_order_id;

    RETURN p_order_id;

  EXCEPTION
  WHEN no_data_found THEN
    DBMS_OUTPUT.PUT_LINE( SQLERRM );
  END get_net_value;

-- Get net value by customer
  FUNCTION get_net_value_by_customer(
      p_customer_id NUMBER,
      p_year        NUMBER)
    RETURN NUMBER
  IS
    ln_net_value NUMBER 
  BEGIN
    SELECT
      SUM(quantity * unit_price)
    INTO
      ln_net_value
    FROM
      order_items
    INNER JOIN orders USING (order_id)
    WHERE
      extract(YEAR FROM order_date) = p_year
    AND customer_id                 = p_customer_id
    AND status                      = gc_shipped_status;
    RETURN ln_net_value;
  EXCEPTION
  WHEN no_data_found THEN
    DBMS_OUTPUT.PUT_LINE( SQLERRM );
  END get_net_value_by_customer;

END order_mgmt;
```

## TRIGGER
```
CREATE [OR REPLACE] TRIGGER trigger_name
    {BEFORE | AFTER } triggering_event ON table_name
    [FOR EACH ROW]
    [FOLLOWS | PRECEDES another_trigger]
    [ENABLE / DISABLE ]
    [WHEN condition]
DECLARE
    declaration statements
BEGIN
    executable statements
EXCEPTION
    exception_handling statements
END;
```

```sql
CREATE OR REPLACE TRIGGER customers_audit_trg
    AFTER 
    UPDATE OR DELETE 
    ON customers
    FOR EACH ROW    
DECLARE
   l_transaction VARCHAR2(10);
BEGIN
   -- determine the transaction type
   l_transaction := CASE  
         WHEN UPDATING THEN 'UPDATE'
         WHEN DELETING THEN 'DELETE'
   END;

   -- insert a row into the audit table   
   INSERT INTO audits (table_name, transaction_name, by_user, transaction_date)
   VALUES('CUSTOMERS', l_transaction, USER, SYSDATE);
END;
/
```

Enable/Disble Trigger - 
```
ALTER TRIGGER trigger_name ENABLE;
ALTER TRIGGER trigger_name DISABLE;
```