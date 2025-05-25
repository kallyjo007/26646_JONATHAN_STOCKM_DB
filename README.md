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

<img width="700" src="./pictures/capture.jpg">
