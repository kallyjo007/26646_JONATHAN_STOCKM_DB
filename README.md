# 26646_JONATHAN_STOCKM_DB
PHASE I
PHASE II
PHASE III
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

<img width="700" src=".pictures/Capture.JPG">

```sql
phase V
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

