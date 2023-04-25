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


# function with out
create or replace function get_film_stat(
    out min_len int,
    out max_len int,
    out avg_len numeric) 
language plpgsql
as $$
begin
  
  select min(length),
         max(length),
		 avg(length)::numeric(5,1)
  into min_len, max_len, avg_len
  from film;

end;$$
