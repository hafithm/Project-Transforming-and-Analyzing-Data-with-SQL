What are your risk areas? Identify and describe them.
A risk area could be disaster recovery. It's crucial to make sure you can access the ecommerce database. To prevent this it is important to have backup tables to ensure

that doesn't happen. This should minimize the the risk of losing data and can run smoothly when running the busniess. 


2. Some columns such as transactions have null values. This can mean incomplete data or missing info throughout the column. It's important to handle them accordingly by

either setting them as 0 or just removing the column entirely. 


3. Important to handle null values because it lead to inconsistency and poor presentation which can be handleded better. In some cases you can also remove the column. 

QA Process:
Describe your QA process and include the SQL queries used to execute it.

1. -- create back_up files for all tables in the database for future references just in case of a mess up 

create table analytics_backup as
select * from analytics


create table all_sessions_backup as
select * from all_sessions 


create table products_backup as
select * from product 


create table sales_by_sku_backup as
select * from sales_by_sku 


create table sales_report_backup as
select * from sales_report 

-- contains a copy of the data from the all original tables. This backup table can be used for disaster recovery in case of any data loss from cleaning.

2. Query to identify incomplete data:-

-- 
SELECT COUNT(*) AS IncompleteDataCount
FROM all_sessions
WHERE transactions IS NULL;

3. Query to change null values in those specific columns in the all_sessions table to either n/a or 0
   This ensures we can have more consistency and also better presentation in the data

   SELECT  COALESCE("searchKeyword", 'N/A') AS searchKeyword,
           COALESCE("transactions", 0) AS transactions
   FROM    all_sessions_clean;




