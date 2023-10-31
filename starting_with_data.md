Question 1:  
Find the most frequently viewed page.

SQL Queries: 
```
SELECT pageviews, COUNT(*) AS viewCount

FROM all_sessions_clean

GROUP BY pageviews

ORDER BY viewCount DESC

LIMIT 1;
```

Answer: Product 1 3071 times



Question 2: 
Find the average session duration for the top 3 visitor.


SQL Queries:
```
SELECT "fullvisitorId", ROUND(AVG("timeOnSite"), 3) AS avgSessionDuration

FROM all_sessions_clean

GROUP BY "fullvisitorId"

LIMIT 3;
```
Answer: 
```
"fullvisitorId"	"avgsessionduration"
"8174358870162124945"	798.000
"5050299044657579745"	504.000
"9885808834292655397"	92.000

```


Question 3: 
How many products have a stock level greater than 1?


SQL Queries:

```
SELECT "productSKU", "stockLevel"

FROM sales_report_clean

WHERE "stockLevel" > 1
```
Answer:
```

"GGOEGAAX0081"	740
"GGOENEBQ084699"	1142
"GGOEGAAX0325"	104
"GGOEYDHJ056099"	1944
"GGOEADHH073999"	283
"GGOEWAEA083899"	20
```

Question 4:  
What is the max numer of session time for the first visitor?

SQL Queries:
```
SELECT "visitId", MAX("timeOnSite") AS maxSessionTime

FROM all_sessions_clean

GROUP BY "visitId"
```
Answer:
```
"visitId"	"maxsessiontime"
1496925644	18
1483063620	245
1491261761	143
1496686413	
1478860149	1813
1475766320	61
```
Question 5: 

SQL Queries:

Answer:
