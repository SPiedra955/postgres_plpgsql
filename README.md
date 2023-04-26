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

# Introduction

### PlpgSQL

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

Within the body of the function, the ```RETURN QUERY``` clause is used to return a result set consisting of the __"country_name"__ and __"region_name"__ columns of the __"countries"__ table joined to the __"regions"__ table based on the __"region_id"__ field. The ```WHERE``` clause is used to filter the query results and search for partial matches of the search string in the __country_name__ column.

````
 CREATE OR REPLACE FUNCTION location_data(
    letter VARCHAR
)
RETURNS TABLE (country_name VARCHAR, region_name VARCHAR)
LANGUAGE plpgsql
AS $$
BEGIN
    RETURN QUERY
    SELECT c.country_name, r.region_name
    FROM countries c
    INNER JOIN regions r ON c.region_id = r.region_id
    WHERE c.country_name LIKE '%' || letter || '%';
END;
$$;
````
_Expected output_:

![image](https://user-images.githubusercontent.com/114516225/234418084-6a14b15c-b549-4d63-a6fc-2dcecd795c6a.png)

# Procedures

A procedure is a named block that does a specific task. PL/SQL procedure allows you to encapsulate complex business logic and reuse it in both database layer and application layer.

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

![image](https://user-images.githubusercontent.com/114516225/234425265-e6769685-4a29-441d-866e-c3994e3c1766.png)

# Cursors

````
 CREATE OR REPLACE FUNCTION get_parent_son(
    letter VARCHAR)
RETURNS TEXT
AS
$$
DECLARE
    names TEXT DEFAULT '';
    employee_dependents RECORD;
    name_dependents RECORD;
    employee_dependents_cursor CURSOR(letter_param VARCHAR) FOR
        SELECT e.first_name, e.last_name, d.first_name
        FROM employees e
        INNER JOIN dependents d ON e.employee_id = d.employee_id
        WHERE e.first_name LIKE '%' || letter_param || '%';
BEGIN
    OPEN employee_dependents_cursor(letter);
    LOOP
        FETCH employee_dependents_cursor INTO employee_dependents;
        EXIT WHEN NOT FOUND;
        FETCH employee_dependents_cursor INTO name_dependents;
        EXIT WHEN NOT FOUND;
        IF employee_dependents.first_name LIKE '%' || letter || '%' THEN
            names := names || 'EMPLOYEE: ' || employee_dependents.first_name || ' ' || employee_dependents.last_name
            || '...DEPENDENT: ' ||  name_dependents.first_name || ' ';
        END IF;
    END LOOP;
    CLOSE employee_dependents_cursor;
    RETURN names;
END;
$$ LANGUAGE plpgsql;

````

# Bibliography

https://www.enterprisedb.com/postgres-tutorials/10-examples-postgresql-stored-procedures

