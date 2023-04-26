# Postgres_plpgsql

 ## Table of contents
 * [**Introduction**](#introduction)
   * [**PLpgSQL**](#plpgsql)
   * [**Scripts**](#scripts)
   * [**Objectives**](#objectives)
 * [**Functions**](#functions)
   * [**Function without parameters**](#function-without-parameters)
   * [**Function with parameters**](#function-with-parameters)
 * [**Procedures**](#procedures)
 * [**Triggers**](#triggers)
   * [**Tables**](#tables)
   * [**Insert trigger**](#insert-trigger)
   * [**Update trigger**](#update-trigger)
   * [**Alter trigger**](#alter-trigger)
 * [**Bibliography**](#bibliography)
 
# Introduction

### PLpgSQL

It's a procedural language that allows you to develop complex functions and stored procedures in PostgreSQL that may not be possible using plain SQL, e.g., control structures, loops, and complex computations.

### Scripts

### Objectives
In this project we have to use:
- Functions
- Procedures
- Cursors
- Triggers

# Functions

### Function without parameters

The next function calculates the salary statistics for a table of employees. The function has three output variables: __"min_salary"__, __"max_salary"__ and __"avg_salary"__, which contain the minimum, maximum and average values of the salaries in the __"employees"__ table.

The function body begins with the ```BEGIN``` clause and ends with the ```END``` clause. The SQL query in the function body calculates the salary statistics using the PostgreSQL ```MIN", MAX and AVG``` function to calculate the minimum, maximum and average salary in the __"employees"__ table. The ```AVG``` function also uses ```::NUMERIC(6,1)``` to convert the result into a number with a precision of 6 digits and 1 decimal place.

The ```INTO``` clause assigns the results of the SQL query to the output variables __"min_salary"__, __"max_salary"__ and __"avg_salary"__. If no employees are found in the table, an exception is thrown with a custom error message indicating that no employees were found in the table.

Finally, the function returns the values of the output variables __"min_salary"__, __"max_salary"__ and __"avg_salary"__ using the ```RETURN``` clause.

````
CREATE OR REPLACE FUNCTION get_salary_stat(
OUT min_salary NUMERIC,
OUT max_salary NUMERIC,
OUT avg_salary NUMERIC)
LANGUAGE plpgsql
AS $$
BEGIN
SELECT MIN(salary),
MAX(salary),
AVG(salary)::NUMERIC(6,1)
INTO min_salary, max_salary, avg_salary
FROM employees;
IF min_salary IS NULL OR max_salary IS NULL OR avg_salary IS NULL THEN
RAISE EXCEPTION 'No employees found'
USING HINT = 'Insert some data into employees table';
END IF;
RETURN;
END;
$$;
````

Run this command ```SELECT get_salary_stat();```

_Expected output with data_:

![image](https://user-images.githubusercontent.com/114516225/234410743-382466f5-44f9-445a-a520-cf8afc255733.png)

_Expected output use of HINT_:

![image](https://user-images.githubusercontent.com/114516225/234381699-0e866aa8-640b-45e2-98a6-c5c85c73f551.png)

### Function with parameters

This function returns a result set with the columns __"country_name"__ and __"region_name"__ by means of an ```INNER JOIN```. The function takes a __"letter"__ parameter as input and returns the rows matching the country search pattern in the table __"countries"__ and __"regions"__.

The ```RETURNS TABLE``` clause defines the structure of the result table that the function will return. In this case, the results table has two columns: __"country_name"__ and __"region_name"__, both of type VARCHAR.

Within the body of the function, the ```RETURN QUERY``` clause is used to return a result set consisting of the columns of the __"countries"__ table joined to the __"regions"__ table based on the __"region_id"__ field. The ```WHERE``` clause is used to filter the query results and search for partial matches of the search string in the __country_name__ column.

In short, this query returns the country in which our company has a branch office starting with the given entry and the name of the continent to which it belongs.

````
CREATE OR REPLACE FUNCTION location_data(
letter VARCHAR)
RETURNS TABLE (country_name VARCHAR, region_name VARCHAR)
LANGUAGE plpgsql
AS $$
BEGIN
RETURN QUERY
SELECT c.country_name, r.region_name
FROM countries c
INNER JOIN regions r ON c.region_id = r.region_id
WHERE c.country_name LIKE UPPER('%' || letter || '%');
END;
$$;
````

_Expected output_:

![image](https://user-images.githubusercontent.com/114516225/234629067-248ff852-a07d-4b3d-b0d2-86333e0c0037.png)

# Procedures

```Procedure``` is a named block that does a specific task. PL/SQL procedure allows you to encapsulate complex business logic and reuse it in both database layer and application layer.

A ```record``` variable is a variable that contains only one row of a result set.

 ````
CREATE OR REPLACE PROCEDURE get_employee_contact(
letter VARCHAR)
LANGUAGE plpgsql AS $$
DECLARE
employee_data record;
BEGIN
FOR employee_data in(
SELECT "first_name", "email", "phone_number" from employees 
WHERE "first_name" LIKE '%' || letter || '%' 
ORDER BY "first_name")
LOOP
RAISE NOTICE 'NAME: %, E-MAIL: %, PHONE: %', employee_data."first_name", employee_data."email", employee_data."phone_number";
END LOOP;
END;
$$;
````
Run this command ```CALL get_employee_contact();```

_Expected output_:

![image](https://user-images.githubusercontent.com/114516225/234638361-308e0a89-8fb2-406b-8706-0b6a22b59c5d.png)

In this case, this query returns all the contacts that matches the value entered.


# Cursors

A ```CURSOR``` allows you to encapsulate a query and process each individual row at a time.Typically, you use cursors when you want to divide a large result set into parts and process each part individually. 

_Query explanation_:

This function that takes a parameter name of type __VARCHAR__ and returns a __TEXT__ value.

Inside the block, variables are defined to store the names of the employees and their dependents. These variables are named __names, employee_name, employee_last_name, dependent_name__, and __dependent_last_name__.

A ```CURSOR``` is defined that will select the __first_name__ and __last_name__ of __employees__, the __first_name__ and __last_name__ of their __dependents__.
```FETCH``` statement is for create the string concatenation.
```CHR(10)``` is the ASCII value for start a new line.

````
CREATE OR REPLACE FUNCTION get_parent_son(
name VARCHAR)
RETURNS TEXT
AS
$$
DECLARE
names TEXT DEFAULT '';
employee_name VARCHAR;
employee_last_name VARCHAR;
dependent_name VARCHAR;
dependent_last_name VARCHAR;
employee_dependents_cursor CURSOR FOR
SELECT e.first_name, e.last_name, d.first_name, d.last_name
FROM employees e
INNER JOIN dependents d ON e.employee_id = d.employee_id
WHERE CONCAT(e.first_name, ' ', e.last_name) LIKE '%' || name || '%';
BEGIN
OPEN employee_dependents_cursor;
LOOP
FETCH employee_dependents_cursor INTO employee_name, employee_last_name, dependent_name, dependent_last_name;
EXIT WHEN NOT FOUND;
names := names || 'EMPLOYEE: ' || employee_name || ' ' || employee_last_name || 
' --> DEPENDENT: ' ||dependent_name || ' ' || dependent_last_name || ' ' || CHR(10);
END LOOP;
CLOSE employee_dependents_cursor;
RETURN names;
END;
$$ LANGUAGE plpgsql;
````
_Expected output_:

![image](https://user-images.githubusercontent.com/114516225/234644399-215d9435-335e-418d-9494-2b1098569c20.png)

# Triggers

A PostgreSQL trigger is a function invoked automatically whenever an event such as insert, update, or delete occurs.

### Tables

First we have to create the following tables, in which we are going to store the data that will be inserted or updated depending on the command we execute.

* The table __employee_summer__: is for store the employees hired during the summer season.
+ the table __employee_update__: is for store changes in the tables.

````
CREATE TABLE employee_summer(
employee_id INT NOT NULL,
first_name VARCHAR(50) NOT NULL,
last_name VARCHAR(50) NOT NULL,
hire_date TIMESTAMP(6) NOT NULL);

CREATE TABLE employee_update(
employee_id INT NOT NULL,
first_name VARCHAR(50) NOT NULL,
last_name VARCHAR(50) NOT NULL,
hire_date TIMESTAMP(6) NOT NULL);

````

### Insert trigger

As its name indicates, to create this trigger we have to create a function where we set the parameters we are going to insert and specify in the ```RETURN``` clause the word TRIGGER. ```OLD``` and ```NEW``` represent the states of the row in the table before or after the triggering event.

```
CREATE OR REPLACE FUNCTION summer_employees_insert()
RETURNS TRIGGER
LANGUAGE PLPGSQL
AS $$
BEGIN
INSERT INTO employee_summer (employee_id, first_name, last_name, hire_date)
VALUES (NEW.employee_id, NEW.first_name, NEW.last_name, NOW());
RETURN NEW;
END;
$$;
```

In the function below we specify the timing that cause the trigger to fire. It can be ````BEFORE```` or ````AFTER```` an event occurs,the event can be ````INSERT```` , ````DELETE````, ````UPDATE```` or ````TRUNCATE```` in this case we use ````INSERT```` and ````UPDATE````.

We use the ON keyword for specify the name of the table associated with the trigger.

A row-level trigger that is specified by the ````FOR EACH ROW```` clause and we use the above function to complete the trigger.



```
CREATE TRIGGER employee_insert_trigger
AFTER INSERT ON employees
FOR EACH ROW
EXECUTE FUNCTION summer_employees_insert();
```
_Values to insert_:

````
INSERT INTO employees(employee_id,first_name,last_name,email,phone_number,hire_date,job_id,salary,manager_id,department_id) 
VALUES (207,'Will','Git','wgitz@sqltutorial.org','518.023.9771','1990-05-08',1,1300.00,205,9);
````
_Expected output_:

Check values in the table __employee_summer__:

![image](https://user-images.githubusercontent.com/114516225/234704847-ba969f9d-f467-478a-ac88-164cc9e85178.png)


### Update trigger

This function works as the [previous function](https://github.com/SPiedra955/postgres_plpgsql/edit/main/README.md#insert-trigger), only the name is changed.

```
CREATE OR REPLACE FUNCTION employees_update()
RETURNS TRIGGER
LANGUAGE PLPGSQL
AS $$
BEGIN
INSERT INTO employee_update (employee_id, first_name, last_name, hire_date)
VALUES (NEW.employee_id, NEW.first_name, NEW.last_name, NOW());
RETURN NEW;
END;
$$;
```
This [trigger](https://github.com/SPiedra955/postgres_plpgsql/edit/main/README.md#insert-trigger) it's the same as the previous one, only the timing is change with ```AFTER``` and the event is ```UPDATE``` also the table which stores the data.

```
CREATE TRIGGER employee_update_trigger
BEFORE UPDATE ON employees
FOR EACH ROW
EXECUTE FUNCTION employees_update();
CREATE TRIGGER
```
_Values to update_:

````
UPDATE employees SET last_name = 'Gilbert' WHERE employee_id = 207;
````

_Check the values in the table __employee_update__:_

![image](https://user-images.githubusercontent.com/114516225/234704955-91194b7f-7e9c-404e-9422-143e9e8f5818.png)

### Alter trigger

This keyword ```ALTER``` is for ````RENAME````, ```ENABLE``` ,```DISABLE``` a trigger.

* RENAME:

````
ALTER TRIGGER employee_update_trigger
ON employees
RENAME TO new_employee_update_trigger;
````

Execute ```\dS employees```

_Expected output_:

![image](https://user-images.githubusercontent.com/114516225/234722190-0efed42b-4949-4031-b66b-1d088fbe0877.png)

* ENABLE(default):

````
ALTER TABLE employees
ENABLE TRIGGER new_employee_update_trigger;
````

* DISABLE

````
ALTER TABLE employees
DISABLE TRIGGER new_employee_update_trigger;
````

# Bibliography

* [10 PostgreSQL examples to stored procedures](https://www.enterprisedb.com/postgres-tutorials/10-examples-postgresql-stored-procedures)
* [PostgreSQL Triggers](https://w3resource.com/PostgreSQL/postgresql-triggers.php)
* [PostgreSQL PLPGSQL tutorial](https://www.postgresqltutorial.com/postgresql-plpgsql/)
* [Sample database](https://www.sqltutorial.org/sql-sample-database/)
