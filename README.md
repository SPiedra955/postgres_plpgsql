# postgres_plpgsql

Create a project that includes at least
One function
One procedure
One cursor
One trigger

CREATE OR REPLACE FUNCTION get_employee(e_name VARCHAR, e_lname VARCHAR)
RETURNS VARCHAR
LANGUAGE plpgsql
AS $$
DECLARE
  employee_data VARCHAR;
BEGIN
  SELECT email || ' ' || phone_number
  INTO employee_data
  FROM employees
  WHERE first_name = e_name AND last_name = e_lname;
  
  IF NOT FOUND THEN
    RAISE EXCEPTION 'Employee % % not found', e_name, e_lname;
  END IF;
  
  RETURN employee_data;
END;
$$;
