
---

# 📌 DATA WAREHOUSE KYA HOTA HAI?

Data Warehouse = centralized storage system jo **historical data analysis** ke liye use hota hai.

Example:

```text
Sales data of last 10 years
Customer behavior trends
Business reports
Dashboards
```

Used for:

```text
Analytics
Reporting
Decision making
Forecasting
```

NOT used for:

```text
Daily transactions
Real-time updates
```

---

# 1️⃣ OLTP vs OLAP (MOST IMPORTANT INTERVIEW QUESTION)

| Feature | OLTP               | OLAP            |
| ------- | ------------------ | --------------- |
| Purpose | transactions       | analytics       |
| Users   | clerks, apps       | analysts        |
| Queries | simple             | complex         |
| Data    | current            | historical      |
| Speed   | fast insert/update | fast read       |
| Example | ATM system         | sales dashboard |

Example OLTP query:

```sql
SELECT balance
FROM accounts
WHERE account_id = 101;
```

Example OLAP query:

```sql
SELECT region, SUM(sales)
FROM sales
GROUP BY region;
```

🎯 Interview tip:

```text
OLTP = operational systems
OLAP = analytical systems
```

---

# 2️⃣ DATA WAREHOUSE CHARACTERISTICS (INMON DEFINITION)

Data Warehouse is:

```text
Subject-oriented
Integrated
Time-variant
Non-volatile
```

Meaning:

| Term             | Meaning                     |
| ---------------- | --------------------------- |
| Subject-oriented | organized by business topic |
| Integrated       | multiple sources merged     |
| Time-variant     | stores historical data      |
| Non-volatile     | data rarely updated         |

---

# 3️⃣ DATA WAREHOUSE ARCHITECTURE

Layers:

```text
Data Sources
↓
ETL Layer
↓
Staging Area
↓
Data Warehouse
↓
Data Marts
↓
BI Tools
```

Example flow:

```text
MySQL + CSV + Logs
→ ETL
→ Warehouse
→ Dashboard
```

---

# 4️⃣ FACT TABLE KYA HOTI HAI?

Fact table stores:

```text
numeric measurable values
```

Example:

```text
sales_amount
quantity
revenue
profit
```

Example structure:

```sql
sales_fact(
product_id,
customer_id,
time_id,
sales_amount
)
```

Contains:

```text
foreign keys + metrics
```

---

# 5️⃣ DIMENSION TABLE KYA HOTI HAI?

Dimension table stores:

```text
descriptive attributes
```

Example:

```sql
product_dim(
product_id,
product_name,
category
)
```

Example:

```sql
time_dim(
time_id,
month,
year
)
```

---

# 6️⃣ STAR SCHEMA (MOST COMMON DESIGN)

Structure:

```text
1 fact table
multiple dimension tables
```

Example:

```
        product_dim
             |
customer_dim — sales_fact — time_dim
             |
        store_dim
```

Advantages:

```text
Simple structure
Fast queries
Easy joins
Best for BI tools
```

Example table design:

```sql
sales_fact(
product_id,
customer_id,
store_id,
time_id,
sales_amount
)
```

---

# 7️⃣ SNOWFLAKE SCHEMA

Snowflake schema = normalized version of star schema

Example:

```
product_dim
   |
category_dim
   |
sales_fact
```

Structure:

```text
dimension tables further split into sub-dimensions
```

Advantages:

```text
Less redundancy
Better storage efficiency
```

Disadvantages:

```text
More joins required
Slower queries
```

---

# 8️⃣ STAR vs SNOWFLAKE SCHEMA DIFFERENCE

| Feature       | Star Schema | Snowflake Schema |
| ------------- | ----------- | ---------------- |
| Structure     | simple      | complex          |
| Normalization | low         | high             |
| Speed         | faster      | slower           |
| Storage       | more        | less             |
| Joins         | fewer       | more             |

Interview answer shortcut:

```text
Star schema = performance optimized
Snowflake schema = storage optimized
```

---

# 9️⃣ DATA MART KYA HOTA HAI?

Data mart = subset of data warehouse

Example:

```text
Sales data mart
HR data mart
Finance data mart
```

Structure:

```
Enterprise Warehouse
     |
 ┌───┴────┐
Sales   HR
Mart    Mart
```

Use:

```text
department-level analytics
```

---

# 🔟 ETL PROCESS (VERY IMPORTANT)

ETL =

```text
Extract
Transform
Load
```

Example:

```
Extract → MySQL, CSV
Transform → clean, filter
Load → Data Warehouse
```

Example Spark ETL:

```python
df = spark.read.csv("sales.csv")
df.write.parquet("warehouse/sales")
```

---

# 1️⃣1️⃣ OLAP OPERATIONS (EXAM FAVORITE)

Four main operations:

| Operation  | Meaning                    |
| ---------- | -------------------------- |
| Roll-up    | summarize data             |
| Drill-down | detailed view              |
| Slice      | filter one dimension       |
| Dice       | filter multiple dimensions |

Example:

Roll-up:

```text
City → State → Country
```

Drill-down:

```text
Year → Month → Day
```

---

# 1️⃣2️⃣ CUBE OPERATION

Creates multidimensional view

Example:

```text
Sales by
Region
Time
Product
```

Query example:

```sql
SELECT region, product, SUM(sales)
FROM sales
GROUP BY region, product;
```

---

# 1️⃣3️⃣ DATA WAREHOUSE vs DATABASE DIFFERENCE

| Feature | Database     | Data Warehouse |
| ------- | ------------ | -------------- |
| Purpose | transactions | analytics      |
| Data    | current      | historical     |
| Schema  | normalized   | star/snowflake |
| Users   | applications | analysts       |

---

# 1️⃣4️⃣ REAL INTERVIEW DESIGN QUESTION

Question:

Design warehouse for **Retail Sales Analytics**

Solution:

Fact table:

```sql
sales_fact(
product_id,
store_id,
time_id,
sales_amount
)
```

Dimension tables:

```sql
product_dim
store_dim
time_dim
customer_dim
```

Schema:

```text
Star Schema
```

Reason:

```text
Fast reporting queries required
```

---

# 🎯 MOST IMPORTANT DATA WAREHOUSE EXAM TRAPS

Yaad rakhna:

```text
OLTP = transaction systems
OLAP = analytics systems
Fact table = numeric values
Dimension table = descriptive attributes
Star schema = denormalized
Snowflake schema = normalized
ETL = Extract Transform Load
Roll-up = summarize
Drill-down = detail
Data mart = subset of warehouse
```


✅ **CAP Theorem + Distributed Systems Basics + Consistency Models** — jo Hadoop, HBase, Kafka, NoSQL interviews me directly poocha jaata hai.
