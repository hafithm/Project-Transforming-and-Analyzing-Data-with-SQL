What issues will you address by cleaning the data?

1.Missing Data: One of the most common issues in datasets is missing data. This can occur due to various reasons such as data entry errors, system failures, or incomplete data collection. Cleaning the data involves identifying missing values and deciding how to handle them. You may choose to remove rows with missing values
2. Duplicates occur when there are multiple identical or very similar entries in the dataset. Inconsistent data entries can arise due to data entry errors, variations in data sources, or different data collection methods.
3. Data Type Conversion: In datasets, you may encounter data that is stored in the wrong data type. For example, numeric values may be stored as text or dates may be stored as strings.





-- ANALYTICS TABLE (CLEANING)

-- Look for null values

```
SELECT *
FROM analytics_backup
WHERE NOT (analytics IS NOT NULL);
```
```
"date"	"fullvisitorId"	"channelGrouping"	"socialEngagementType"	"pageviews"	"bounces"	"unit_price"	"visitNumber"	"visitId"	"visitStartTime"	"revenue"
"20170624"	"4033999852320230451"	"Social"	"Not Socially Engaged"	"3"		6990000				
"20170624"	"4033999852320230451"	"Social"	"Not Socially Engaged"	"3"		4990000				
"20170624"	"4033999852320230451"	"Social"	"Not Socially Engaged"	"3"		10990000
```

-- Drop the columns that are not needed
```
ALTER TABLE analytics_backup

DROP COLUMN units_sold,

DROP COLUMN userid,

DROP COLUMN timeonsite
```

-- Updated the analytics_backup table to insert values in the revenue column by multiplying unit_ptice by orderedQuanity. 
```
UPDATE analytics_backup AS ab

SET revenue = (ab.unit_price::numeric * cp."orderedQuantity"::numeric, 0

FROM (


  SELECT ab."visitId"
  
  FROM analytics_backup ab
  
  JOIN all_sessions s ON ab."visitId" = s."visitId"	
  
   LIMIT 100
 
) AS sub

JOIN all_sessions s ON sub."visitId" = s."visitId"

JOIN sales_report sr ON sr."productSKU" = s."productSKU"

JOIN clean_products cp ON sr."stockLevel" = cp."stockLevel"

```


-- Look for duplicate visitNumber's

```
ALTER TABLE analytics_clean  RENAME TO analytics_backup;
```
```
SELECT DISTINCT "visitNumber", count(*)

FROM analytics_backup 


GROUP BY "visitNumber"


HAVING count(*) > 1
```
-- Deleting duplicate visitNumer's
```

WITH remove_duplicates AS (
  
SELECT "visitNumer" , count(*) AS row_count
  
ROW_NUMBER() over (Particition BY "visitNumer" ORDER BY "visitNumer") AS rows
  
FROM analytics_clean 

GROUP BY "visitNumber"

HAVING count (*)
)
```
```
DELETE FROM analytics_backup
WHERE ("visitNumber" , remoove_duplicate) IN (
	SELECT "visitNumber" , remoove_duplicate 
 FROM remoove_duplicate
 WHERE rows > 1 
 );

DELETE FROM analytics_backup

WHERE id NOT IN (

  SELECT MIN(id)
  
  FROM analytics_backup
  
  GROUP BY "visitNumber"
  
  HAVING COUNT(*) > 1
  
);
```
```
ALTER TABLE analytics_backup 

RENAME TO analytics_clean;


UPDATE analytics_clean

SET bounces = 0;

-- This query will update all rows in the analytics_clean table and set bounces columns to 0.
```
-- Change unit_price column to decimal and divide it by 100000 for readability
```
update analytics
ALTER TABLE analytics
ALTER COLUMN unit_price TYPE DECIMAL;
ALTER TABLE analytics
RENAME COLUMN unit_price TO unitprice;
UPDATE analytics
SET unitprice =
CASE
WHEN unitprice IS NOT NULL THEN (unitprice / 1000000.0): :DECIMAL (10,2)
ELSE unitprice
END;
```


-- PRODUCTS TABLE (CLEANING)

-- Look for null values
```
SELECT *

FROM products

WHERE NOT (products IS NOT NULL);

-- only sentimentScore and sentimentMagnitude have null values.
```
-- Look for duplicate skus
```
SELECT sku, count(*)

FROM products

GROUP BY sku

HAVING count(*) > 1

-- TRIM sku and name columns for any extra space

-- Fill out with 0 null values from columns sentiment_score and sentiment_magnitude

-- Order table by sku
```
```
--Create a view of the cleaned table

CREATE OR REPLACE VIEW working_products AS 
    SELECT 
    	
      	TRIM(sku) AS sku,
	
        TRIM(name) AS product_name,
	
        "orderedQuantity",
	
        "stockLevel",
	
        "restockingLeadTime",
        CASE 
            WHEN "sentimentScore" IS NULL THEN 0
            ELSE "sentimentScore"
        END AS "sentimentScore",
	
        CASE
            WHEN "sentimentMagnitude" IS NULL THEN 0
            ELSE "sentimentMagnitude"
        END AS "sentimentMagnitude"
		
		
    FROM products
    
    ORDER BY sku;
	
-- Does not recognize columns 

-- Use double quotes for them

-- Make an empty copy of the original table 
```

CREATE TABLE clean_products AS
```
TABLE products

WITH NO DATA;
```
-- Insert clean data from the view table 
INSERT INTO clean_products
```
SELECT * FROM working_products;

-- Rename column name

ALTER TABLE clean_products
    RENAME COLUMN name TO product_name;
	
ALTER TABLE clean_products 

RENAME TO products_clean;	
```

-- create back_up files for all tables in the database for future references just in case of a mess up 
```
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

select * from all_sessions
```

-- Cleaning all_sessions_backup table

ALTER TABLE all_sessions_backup 


RENAME TO all_sessions_clean;

```
SELECT *

FROM all_sessions_backup

WHERE NOT (all_sessions_backup IS NOT NULL);

```
-- Looking for duplicates in fullvisitorId
```
SELECT DISTINCT "fullvisitorId", count(*)

FROM all_sessions_backup 

GROUP BY "fullvisitorId"

HAVING count(*) > 1
```
-- Remove duplicates in fullvisitorId
```
DELETE FROM all_sessions_backup

WHERE "fullvisitorId" IN (

    SELECT "fullvisitorId"
    
    FROM all_sessions_backup
    
    GROUP BY "fullvisitorId"
    
    HAVING COUNT(*) > 1
);
```
-- Removing null values and updating with default values for "totalTransactionRevenue":
```
UPDATE all_sessions_backup
SET "totalTransactionRevenue" = 0
WHERE "totalTransactionRevenue" IS NULL;
```
-- Removing null values and updating with default values for "productRefundAmount":
```
UPDATE all_sessions_backup
SET "productRefundAmount" = 0
WHERE productRefundAmount IS NULL;
```
-- Removing null values and updating with default values for "productQuantity":
```
UPDATE all_sessions_backup
SET "productQuantity" = 0
WHERE productQuantity IS NULL;
```
-- Removing null values and updating with default values for "productRevenue":
```
UPDATE all_sessions_backup
SET "productRevenue" = 0
WHERE "productRevenue" IS NULL;
```
-- Removing null values and updating with default values for "itemQuantity":
```
UPDATE all_sessions_backup
SET "itemQuantity" = 0
WHERE "itemQuantity" IS NULL;
```
-- Removing null values and updating with default values for "itemRevenue":
```
UPDATE all_sessions_backup
SET "itemRevenue" = 0
WHERE "itemRevenue" IS NULL;
```
-- Removing null values and updating with default values for "transactionRevenue":
```
UPDATE all_sessions_backup
SET "transactionRevenue" = 0
WHERE "transactionRevenue" IS NULL;
```
-- Removing null values and updating with default values for "transactionld":
```
UPDATE all_sessions_backup
SET "transactionId" = 0
WHERE "transactionId" IS NULL;
```
-- Changing names for easier read
```
ALTER TABLE all_sessions_backupf
RENAME COLUMN "v2ProductName" TO ProductName;
ALTER TABLE all_sessions_backup
RENAME COLUMN "v2ProductCategory" TO ProductCategory;
```
-- removing redundant columns that represent the same or similar to other columns in the all_sessions table 
```
ALTER TABLE all_sessions_clean
DROP COLUMN "productRefundAmount", --similar to productRevenue by representing the total revenue generated by a product, including refunds. 
DROP COLUMN "transactionId", --  similar to transactionRevenue by 
DROP COLUMN "eCommerceAction_option"; -- similar to searchKeyword by 
```
-- updating the all_sessions_clean table to include 0 for transactions column and n/a for searchKeyword columm

--  This ensures we can have more consistency and also better presentation in the data

```
SELECT COALESCE("searchKeyword", 'N/A') AS searchKeyword,
       COALESCE("transactions", 0) AS productRefundAmount
FROM all_sessions_clean;
```


UPDATE all_sessions_clean
```
SET "searchKeyword" = 'N/A', 
    "transactions" = 0
WHERE "searchKeyword" IS NULL OR "transactions" IS NULL;

```
-- Cleaning sales_report_backup table
```
-- looking for null values
SELECT *
FROM sales_report_backup
WHERE NOT (sales_report_backup IS NOT NULL);
```
- removing null values 
UPDATE sales_report_backup
SET ratio = 0
WHERE ratio IS NULL;
```

- Looking for duplicates in productSKU
```
SELECT DISTINCT "productSKU", count(*)
FROM sales_report_backup 
GROUP BY "productSKU"
HAVING count(*) > 1
```
- There is no duplicates therefore no need to further clean the column 
```
ALTER TABLE sales_report_backup 
RENAME TO sales_report_clean;

- Cleaning sales_by_sku_backup table

- looking for null values
```
SELECT *
FROM x
WHERE NOT (sales_by_sku_backup IS NOT NULL);

-- No null values in the table
```

- Looking for duplicates in productSKU
```
SELECT DISTINCT "productSKU", count(*)
FROM sales_by_sku_backup 
GROUP BY "productSKU"
HAVING count(*) > 1
```
- There are no duplicates
```
ALTER TABLE sales_by_sku_backup 
RENAME TO sales_by_sku_clean;
```
