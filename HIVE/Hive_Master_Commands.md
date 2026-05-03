# HIVE MASTER COMMANDS HANDBOOK (Beginner → Advanced → Admin → Exam Level)

This document contains structured Hive commands useful for CDAC / DBDA / Big Data interviews / Hadoop ecosystem practice.

---

## SECTION 1: STARTING HIVE ENVIRONMENT

# Start Hive shell

hive

# Start Beeline shell

beeline

# Connect Beeline to HiveServer2

beeline -u jdbc:hive2://localhost:10000

---

## SECTION 2: DATABASE COMMANDS

# Show all databases

show databases;

# Create database

create database company;

# Create database if not exists

create database if not exists company;

# Use database

use company;

# Show current database

set hive.current.database;

# Describe database

describe database company;

# Describe database extended info

describe database extended company;

# Drop database

drop database company;

# Drop database with contents

drop database company cascade;

---

## SECTION 3: TABLE CREATION COMMANDS

# Create table

create table employee(
id int,
name string,
salary float
);

# Show tables

show tables;

# Describe table

describe employee;

# Describe formatted table

describe formatted employee;

# Describe extended table

describe extended employee;

---

## SECTION 4: INTERNAL vs EXTERNAL TABLES

# Create internal table

create table emp_internal(
id int,
name string
);

# Create external table

create external table emp_external(
id int,
name string
)
row format delimited
fields terminated by ','
location '/hive/external/emp';

# Drop external table (data remains safe)

drop table emp_external;

---

## SECTION 5: LOAD DATA COMMANDS

# Load local data

load data local inpath '/home/data.txt'
into table employee;

# Load HDFS data

load data inpath '/input/data.txt'
into table employee;

# Overwrite table data

load data local inpath '/home/data.txt'
overwrite into table employee;

---

## SECTION 6: SELECT QUERY COMMANDS

# Select all records

select * from employee;

# Select specific columns

select name from employee;

# Select with condition

select * from employee where salary > 50000;

# Limit output

select * from employee limit 10;

---

## SECTION 7: FILTERING COMMANDS

# AND condition

select * from employee where salary > 30000 and id < 10;

# OR condition

select * from employee where salary > 30000 or id < 10;

# NOT condition

select * from employee where not salary > 30000;

---

## SECTION 8: SORTING COMMANDS

# Order by ascending

select * from employee order by salary;

# Order by descending

select * from employee order by salary desc;

# Sort within reducer

select * from employee sort by salary;

# Distribute rows

select * from employee distribute by id;

# Cluster data

select * from employee cluster by id;

---

## SECTION 9: GROUP BY COMMANDS

# Count employees per department

select dept, count(*)
from employee
group by dept;

# Sum salary

select dept, sum(salary)
from employee
group by dept;

---

## SECTION 10: JOIN COMMANDS

# Inner join

select a.id, b.dept
from emp a
join dept b
on a.id = b.id;

# Left join

select a.id, b.dept
from emp a
left join dept b
on a.id = b.id;

# Right join

select a.id, b.dept
from emp a
right join dept b
on a.id = b.id;

# Full outer join

select a.id, b.dept
from emp a
full outer join dept b
on a.id = b.id;

---

## SECTION 11: PARTITION TABLE COMMANDS

# Create partition table

create table sales(
id int,
amount int
)
partitioned by (year int);

# Load partition data

load data local inpath '/sales.txt'
into table sales partition(year=2024);

# Show partitions

show partitions sales;

---

## SECTION 12: BUCKETING COMMANDS

# Create bucket table

create table student(
id int,
name string
)
clustered by(id)
into 4 buckets;

---

## SECTION 13: VIEW COMMANDS

# Create view

create view emp_view as
select * from employee;

# Show views

show tables;

# Drop view

drop view emp_view;

---

## SECTION 14: INDEX COMMANDS

# Create index

create index emp_index
on table employee(name)
as 'compact'
with deferred rebuild;

# Rebuild index

alter index emp_index
on employee rebuild;

---

## SECTION 15: ALTER TABLE COMMANDS

# Rename table

alter table employee rename to emp;

# Add column

alter table emp add columns(age int);

# Replace columns

alter table emp replace columns(
id int,
name string
);

---

## SECTION 16: DROP TABLE COMMANDS

# Drop table

drop table emp;

# Drop if exists

drop table if exists emp;

---

## SECTION 17: SHOW COMMANDS

# Show databases

show databases;

# Show tables

show tables;

# Show partitions

show partitions sales;

# Show table properties

show tblproperties employee;

---

## SECTION 18: DESCRIBE COMMANDS

# Describe table

describe employee;

# Describe formatted

describe formatted employee;

# Describe extended

describe extended employee;

---

## SECTION 19: FILE FORMAT COMMANDS

# Create ORC table

create table orc_table(
id int,
name string
)
stored as orc;

# Create PARQUET table

create table pq_table(
id int,
name string
)
stored as parquet;

# Create TEXTFILE table

create table txt_table(
id int,
name string
)
stored as textfile;

---

## SECTION 20: PERFORMANCE TUNING COMMANDS

# Enable vectorization

set hive.vectorized.execution.enabled=true;

# Enable parallel execution

set hive.exec.parallel=true;

# Set reducer count

set mapreduce.job.reduces=4;

# Enable compression

set hive.exec.compress.output=true;
