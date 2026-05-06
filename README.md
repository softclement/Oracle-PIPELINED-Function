# 🍕 Oracle PIPELINED Function – Complete Demo (Single Frame)

This demo explains how **PIPELINED functions in Oracle** work using a simple, step-by-step approach.

---

## 📌 What This Demonstrates

- Row-by-row streaming using `PIPE ROW`
- Real-time processing proof using logging
- Difference between batch vs pipelined execution

---

## 🧱 Complete Script

```sql
/*------------------------------------------------------------
 🍕 Oracle PIPELINED FUNCTION – COMPLETE DEMO (Single Frame)
------------------------------------------------------------*/

/*------------------------------------------------------------
 STEP 1: Create OBJECT type
         - Represents ONE row returned by the pipelined function
------------------------------------------------------------*/
CREATE OR REPLACE TYPE t_emp_row AS OBJECT (
  emp_name   VARCHAR2(50),
  emp_salary NUMBER
);
/

/*------------------------------------------------------------
 STEP 2: Create TABLE type
         - Collection of object rows
         - Required return type for PIPELINED function
------------------------------------------------------------*/
CREATE OR REPLACE TYPE t_emp_tab AS TABLE OF t_emp_row;
/

/*------------------------------------------------------------
 STEP 3: Create LOG table
         - Used to PROVE row-by-row execution
         - Each PIPE ROW will write one log entry
------------------------------------------------------------*/
CREATE TABLE pipe_log (
  log_time TIMESTAMP,
  emp_name VARCHAR2(50)
);

/*------------------------------------------------------------
 STEP 4: Create LOGGING procedure
         - Uses AUTONOMOUS TRANSACTION
         - Allows DML inside function called from SQL
------------------------------------------------------------*/
CREATE OR REPLACE PROCEDURE log_pipe (p_name VARCHAR2)
IS
  PRAGMA AUTONOMOUS_TRANSACTION;
BEGIN
  INSERT INTO pipe_log (log_time, emp_name)
  VALUES (SYSTIMESTAMP(6), p_name);

  COMMIT;
END;
/

/*------------------------------------------------------------
 STEP 5: Create PIPELINED function
         - Reads EMP table row-by-row
         - Logs each row independently
         - Pipes rows incrementally to SQL engine
------------------------------------------------------------*/
CREATE OR REPLACE FUNCTION get_emp_data
  RETURN t_emp_tab PIPELINED
AS
BEGIN
  FOR rec IN (SELECT ename, sal FROM emp) LOOP

    -- Log proves this row is processed NOW (not later)
    log_pipe(rec.ename);

    -- Stream one row at a time to SQL engine
    PIPE ROW (t_emp_row(rec.ename, rec.sal));

  END LOOP;

  RETURN; -- Required for pipelined functions
END;
/

/*------------------------------------------------------------
 STEP 6: Query PIPELINED function like a table
         - Rows are returned incrementally
------------------------------------------------------------*/
SELECT *
FROM TABLE(get_emp_data);

/*------------------------------------------------------------
 STEP 7: Verify ROW-BY-ROW processing proof
         - Each record has its own timestamp
------------------------------------------------------------*/
SELECT *
FROM pipe_log
ORDER BY log_time;
