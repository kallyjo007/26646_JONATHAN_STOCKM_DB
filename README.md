# 26646_JONATHAN_STOCKM_DB
PHASE I
<h2>Problem Definition</h2>

Construction companies often face challenges in managing their materials inventory. Manual tracking methods result in frequent stockouts, overstocking, and a lack of real-time visibility. These inefficiencies delay construction timelines and inflate operational costs.

<h2>Context</h2>

The system is designed for mid-sized construction firms in Rwanda. It will be deployed in offices and warehouse locations to monitor material usage, stock levels, and automate reordering.

<h2>Target Users</h2>

<li>Warehouse Clerks</li>

<li>Procurement Officers</li>

<li>Site Managers</li>

<li>System Administrators</li>

<h2>Goals</h2>

<li>Track and log daily material usage</li>

<li>Automatically trigger reorders when stock levels fall below thresholds</li>

<li>Prevent unauthorized changes to stock records</li>

<li>Provide reporting for MIS and management decision-making</li>


```sql
PHASE II

 Business Process Modeling (MIS)
```
```sql
Goal
Model the Inventory Usage and Reordering Process to show how information flows between departments and how MIS supports decision-making.
```
```sql
 Step 1: Define the Scope
Business Process:

"Material Usage Logging and Automatic Reordering in a Construction Company."

Objective:
To ensure accurate material tracking, prevent shortages, and automate restocking based on predefined thresholds.

Expected Outcomes:

Reduced material shortages

Timely restocking

Improved procurement decisions
```
```sql
 Step 2: Identify Key Entities / Actors
Actor	Role
Warehouse Clerk	Logs daily material usage
System	Updates stock levels, checks reorder thresholds
Procurement Officer	Approves reorder requests
Supplier	Delivers requested materials
Database	Stores all data and facilitates report generation
```

```sql
PHASE III
```
Step 1: Identify Entities & Attributes
Here are the key entities for the project:

1. materials
```sql
Copy code
material_id       NUMBER PRIMARY KEY  
material_name     VARCHAR2(100)  
unit              VARCHAR2(20)  
current_stock     NUMBER NOT NULL  
threshold_stock   NUMBER DEFAULT 50
```
2. stock_usage
```sql
Copy code
usage_id          NUMBER PRIMARY KEY  
material_id       NUMBER REFERENCES materials(material_id)  
used_by           VARCHAR2(100)  
quantity_used     NUMBER NOT NULL  
usage_date        DATE DEFAULT SYSDATE
```
4.stock_reorders
```sql
Copy code
reorder_id        NUMBER PRIMARY KEY  
material_id       NUMBER REFERENCES materials(material_id)  
quantity_ordered  NUMBER NOT NULL  
order_date        DATE DEFAULT SYSDATE  
status            VARCHAR2(20) CHECK (status IN ('Pending', 'Delivered', 'Cancelled'))
```
users
```sql
Copy code
user_id           NUMBER PRIMARY KEY  
full_name         VARCHAR2(100)  
role              VARCHAR2(50) CHECK (role IN ('Warehouse Clerk', 'Procurement Officer', 'Admin'))
```

5. audit_logs
```sql
Copy code
log_id            NUMBER PRIMARY KEY  
user_id           NUMBER REFERENCES users(user_id)  
operation         VARCHAR2(20)  
timestamp         TIMESTAMP DEFAULT SYSTIMESTAMP  
status            VARCHAR2(10) CHECK (status IN ('Allowed', 'Denied'))
```
6. holidays
```sql
Copy code
holiday_date      DATE PRIMARY KEY  
description       VARCHAR2(100)
```
Step 2: Relationships
Relationship	Type	Explanation
materials – stock_usage   	1:M	Each material can have multiple usage logs
materials – stock_reorders	1:M	Each material can have many reorder records
users – audit_logs        	1:M	Each user can have multiple audit entries

Step 3: Constraints
NOT NULL on all important fields (e.g., quantity_used, current_stock)

CHECK on fields like status, role

FOREIGN KEY constraints to enforce referential integrity

<img width="700" src=".Capture.JPG">

```sql
phase V
```
```sql
-- Table: materials
CREATE TABLE materials (
    material_id      NUMBER PRIMARY KEY,
    material_name    VARCHAR2(100) NOT NULL,
    unit             VARCHAR2(20) NOT NULL,
    current_stock    NUMBER NOT NULL,
    threshold_stock  NUMBER DEFAULT 50
);

-- Table: stock_usage
CREATE TABLE stock_usage (
    usage_id         NUMBER PRIMARY KEY,
    material_id      NUMBER REFERENCES materials(material_id),
    used_by          VARCHAR2(100),
    quantity_used    NUMBER NOT NULL,
    usage_date       DATE DEFAULT SYSDATE
);

-- Table: stock_reorders
CREATE TABLE stock_reorders (
    reorder_id        NUMBER PRIMARY KEY,
    material_id       NUMBER REFERENCES materials(material_id),
    quantity_ordered  NUMBER NOT NULL,
    order_date        DATE DEFAULT SYSDATE,
    status            VARCHAR2(20) CHECK (status IN ('Pending', 'Delivered', 'Cancelled'))
);

-- Table: users
CREATE TABLE users (
    user_id    NUMBER PRIMARY KEY,
    full_name  VARCHAR2(100) NOT NULL,
    role       VARCHAR2(50) CHECK (role IN ('Warehouse Clerk', 'Procurement Officer', 'Admin'))
);

-- Table: audit_logs
CREATE TABLE audit_logs (
    log_id     NUMBER PRIMARY KEY,
    user_id    NUMBER REFERENCES users(user_id),
    operation  VARCHAR2(20),
    timestamp  TIMESTAMP DEFAULT SYSTIMESTAMP,
    status     VARCHAR2(10) CHECK (status IN ('Allowed', 'Denied'))
);
-- Table: holidays
CREATE TABLE holidays (
    holiday_date DATE PRIMARY KEY,
    description  VARCHAR2(100)
);
```

insert_data.sql
```sql
Copy
Edit
-- Insert into materials
INSERT INTO materials VALUES (1, 'Cement', 'bags', 100, 50);
INSERT INTO materials VALUES (2, 'Bricks', 'pieces', 300, 100);
INSERT INTO materials VALUES (3, 'Steel Rods', 'meters', 80, 30);

-- Insert into stock_usage
INSERT INTO stock_usage VALUES (1, 1, 'John', 20, DATE '2025-04-01');
INSERT INTO stock_usage VALUES (2, 2, 'Alice', 50, DATE '2025-04-02');
INSERT INTO stock_usage VALUES (3, 1, 'John', 15, DATE '2025-04-03');
INSERT INTO stock_usage VALUES (4, 3, 'Mike', 10, DATE '2025-04-03');

-- Insert into stock_reorders
INSERT INTO stock_reorders VALUES (1, 1, 100, DATE '2025-04-05', 'Pending');
INSERT INTO stock_reorders VALUES (2, 2, 150, DATE '2025-04-06', 'Delivered');

-- Insert into users
INSERT INTO users VALUES (1, 'John Doe', 'Warehouse Clerk');
INSERT INTO users VALUES (2, 'Alice Smith', 'Procurement Officer');
INSERT INTO users VALUES (3, 'Admin User', 'Admin');

-- Insert into audit_logs
INSERT INTO audit_logs VALUES (1, 1, 'INSERT', SYSTIMESTAMP, 'Allowed');
INSERT INTO audit_logs VALUES (2, 2, 'UPDATE', SYSTIMESTAMP, 'Denied');

-- Insert into holidays
INSERT INTO holidays VALUES (DATE '2025-06-01', 'National Construction Day');
INSERT INTO holidays VALUES (DATE '2025-06-15', 'Maintenance Break');
```

✅ PHASE VI: Database Interaction & Transactions
This phase involves procedures, functions, exception handling, transactions, and optionally packages.

📌 Step 1: DML & DDL Operations
Already covered:

INSERT into materials, stock_usage, etc.

Table definitions (DDL)

Now we’ll build procedures and functions to interact with the data.

📁 procedures_functions.sql
🔧 1. Function: Get_Total_Usage(material_id)
```sql
CREATE OR REPLACE FUNCTION Get_Total_Usage(p_material_id IN NUMBER)
RETURN NUMBER IS
    v_total NUMBER;
BEGIN
    SELECT NVL(SUM(quantity_used), 0)
    INTO v_total
    FROM stock_usage
    WHERE material_id = p_material_id;

    RETURN v_total;
EXCEPTION
    WHEN OTHERS THEN
        RETURN -1;
END;
/
```
2. Procedure: Restock_If_Low(material_id)
```sql
   CREATE OR REPLACE PROCEDURE Restock_If_Low(p_material_id IN NUMBER) IS
    v_stock NUMBER;
BEGIN
    SELECT current_stock INTO v_stock FROM materials WHERE material_id = p_material_id;

    IF v_stock < 50 THEN
        INSERT INTO stock_reorders (reorder_id, material_id, quantity_ordered, order_date, status)
        VALUES (stock_reorders_seq.NEXTVAL, p_material_id, 100, SYSDATE, 'Pending');
    END IF;
EXCEPTION
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Error occurred during reorder.');
END;
/
```

 3. Cursor-based Reporting Procedure
```sql
    CREATE OR REPLACE PROCEDURE Print_Stock_Usage IS
    CURSOR usage_cur IS
        SELECT m.material_name, s.quantity_used, s.usage_date
        FROM stock_usage s
        JOIN materials m ON s.material_id = m.material_id;
BEGIN
    FOR rec IN usage_cur LOOP
        DBMS_OUTPUT.PUT_LINE('Material: ' || rec.material_name || 
                             ', Used: ' || rec.quantity_used || 
                             ', Date: ' || rec.usage_date);
    END LOOP;
END;
/
```
 PHASE VII: Advanced Logic & Auditing
📌 1. Holidays Table

Already created: holidays(holiday_date, description)

📌 2. Trigger: Block DML on Weekdays & Holidays
```sql
CREATE OR REPLACE TRIGGER trg_block_weekday_holiday_dml
BEFORE INSERT OR UPDATE OR DELETE ON stock_usage
FOR EACH ROW
DECLARE
    v_day VARCHAR2(10);
    v_today DATE := TRUNC(SYSDATE);
    v_is_holiday NUMBER;
BEGIN
    SELECT TO_CHAR(v_today, 'DY') INTO v_day FROM DUAL;
    
    SELECT COUNT(*) INTO v_is_holiday
    FROM holidays
    WHERE holiday_date = v_today;
    
    IF v_day IN ('MON', 'TUE', 'WED', 'THU', 'FRI') OR v_is_holiday > 0 THEN
        RAISE_APPLICATION_ERROR(-20001, 'DML not allowed on weekdays or holidays.');
    END IF;
END;
/
```

3. Auditing Trigger
```sql
   CREATE OR REPLACE TRIGGER trg_audit_dml
AFTER INSERT OR UPDATE OR DELETE ON stock_usage
FOR EACH ROW
DECLARE
    v_user_id NUMBER := 1; -- simulate logged-in user
BEGIN
    INSERT INTO audit_logs (log_id, user_id, operation, timestamp, status)
    VALUES (audit_logs_seq.NEXTVAL, v_user_id, 'DML', SYSTIMESTAMP, 'Allowed');
END;
/
```
```sql
CREATE OR REPLACE PACKAGE stock_pkg IS
  FUNCTION Get_Total_Usage(p_material_id NUMBER) RETURN NUMBER;
  PROCEDURE Restock_If_Low(p_material_id NUMBER);
END;
/

CREATE OR REPLACE PACKAGE BODY stock_pkg IS

  FUNCTION Get_Total_Usage(p_material_id NUMBER) RETURN NUMBER IS
    v_total NUMBER;
  BEGIN
    SELECT NVL(SUM(quantity_used), 0)
    INTO v_total
    FROM stock_usage
    WHERE material_id = p_material_id;
    RETURN v_total;
  END;

  PROCEDURE Restock_If_Low(p_material_id NUMBER) IS
    v_stock NUMBER;
  BEGIN
    SELECT current_stock INTO v_stock FROM materials WHERE material_id = p_material_id;

    IF v_stock < 50 THEN
      INSERT INTO stock_reorders (reorder_id, material_id, quantity_ordered, order_date, status)
      VALUES (stock_reorders_seq.NEXTVAL, p_material_id, 100, SYSDATE, 'Pending');
    END IF;
  END;

END stock_pkg;
/

```
4. Add a Package
### Normalization Summary

- **1NF**: All tables have atomic values (e.g., `quantity_used`, `material_name`)  
- **2NF**: No partial dependencies — all attributes depend fully on the primary key  
- **3NF**: No transitive dependencies — every non-key field depends only on the primary key

