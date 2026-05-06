# 🍕 Oracle PIPELINED Function – Complete Demo

This demo explains how **PIPELINED functions in Oracle** work using a simple, step-by-step approach.

---

## 📌 What This Demonstrates

- Row-by-row streaming using `PIPE ROW`
- Real-time processing proof using logging
- Difference between batch vs pipelined execution

---

## 🧱 STEP 1: Create OBJECT Type

Represents **one row** returned by the pipelined function.

```sql
CREATE OR REPLACE TYPE t_emp_row AS OBJECT (
  emp_name   VARCHAR2(50),
  emp_salary NUMBER
);
/
