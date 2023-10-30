Question 1:  
Find the most frequently viewed page.

SQL Queries: 

SELECT pageviews, COUNT(*) AS viewCount

FROM all_sessions_clean

GROUP BY pageviews

ORDER BY viewCount DESC

LIMIT 1;

Answer: Product 1 3297 times



Question 2: 
Find the average session duration for the top 3 visitor.


SQL Queries:
SELECT "fullvisitorId", AVG("timeOnSite") AS avgSessionDuration

FROM all_sessions_clean

GROUP BY "fullvisitorId"

limit 3

Answer: 
-- first has 798  seconds
-- second has 504 seconds
-- third has 92 seconds




Question 3: 
How many products have a stock level greater than 1?


SQL Queries:
SELECT "productSKU", "stockLevel"

FROM sales_report_clean

WHERE "stockLevel" > 1

Answer:
the answer is 367 products (rows)



Question 4:  
What is the max numer of session time for the first visitor?

SQL Queries:
SELECT "visitId", MAX("timeOnSite") AS maxSessionTime

FROM all_sessions_clean

GROUP BY "visitId"

Answer:
VisitId 1496925644 has max of 18.  


Question 5: 

SQL Queries:

Answer:
