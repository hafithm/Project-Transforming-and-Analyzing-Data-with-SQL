Answer the following questions and provide the SQL queries used to find the answer.

    
**Question 1: Which cities and countries have the highest level of transaction revenues on the site?**


SQL Queries:
```
SELECT
		country,
		city,
		SUM("totalTransactionRevenue") AS total_transaction_revenue

FROM
		all_sessions_clean

WHERE
		"totalTransactionRevenue" != 0
		AND  city <> 'not available in demo dataset'


GROUP BY 
		country,
		city

ORDER BY 
	total_transaction_revenue DESC;
```


Answer:

-- Calculating revenue by country and city using column "totalTransactionRevenue" 

-- USA is the country that has the highest level of transaction revenue

-- Cities include New York, Los Angeles and San Francisco

```
"country"	"city"	"total_transaction_revenue"
"United States"	"San Francisco"	1251130000
"United States"	"Sunnyvale"	872230000
"United States"	"Palo Alto"	608000000
"Israel"	"Tel Aviv-Yafo"	602000000
"United States"	"New York"	530360000
"United States"	"Los Angeles"	479480000
"United States"	"Mountain View"	474380000
"United States"	"Chicago"	449520000
```

**Question 2: What is the average number of products ordered from visitors in each city and country?**


SQL Queries:
```
SELECT s.city, s.country, round(avg(sk.total_ordered), 2) as Rounded_Avg

FROM all_sessions_clean s 

JOIN 
		sales_report_clean sr ON sr."productSKU" = s."productSKU"
JOIN 
		sales_by_sku sk ON sr."productSKU" = sr."productSKU"
  
WHERE 		city <> 'not available in demo dataset' and city <> '(not set)'

GROUP BY 	s.city, s.country, s."fullvisitorId"

ORDER BY	 avg(sk.total_ordered) desc;

```

Answer:

```
“city”	“country”	“rounded_avg”
“San Francisco”	“United States”	167.00
“Palo Alto”	“United States”	112.00
“New York”	“United States”	112.00
“San Francisco”	“United States”	112.00
“Los Angeles”	“United States”	112.00
```


**Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**


SQL Queries:
```
SELECT 		s.city, 
			s.country,
			s.productname, 
			s."fullvisitorId" 

FROM 		all_sessions_clean s

JOIN 		sales_report_clean sr ON sr."productSKU" = s."productSKU" 

JOIN 		sales_by_sku_clean sk ON sr."productSKU" = sr."productSKU"
			
WHERE 		city <> '(not set)' and s.productname <> '(not set)' 

group by 		s.city, 
				s.country,
				s.productname, 
				s."fullvisitorId"
order by	 	s.country, 
				s.city

```

 ```
"city"	"country"	"productname"	"fullvisitorId"
"not available in demo dataset"	"Albania"	"22 oz YouTube Bottle Infuser"	"5832725431264256293"
"not available in demo dataset"	"Albania"	"Google Women's Lightweight Microfleece Jacket"	"6675422547201751957"
"not available in demo dataset"	"Albania"	"YouTube Men's Vintage Tank"	"6775659825894576935"
"not available in demo dataset"	"Algeria"	"YouTube Twill Cap"	"21308687109528016"
"Buenos Aires"	"Argentina"	"Google Men's Skater Tee Charcoal"	"7218311731838701761"
"Buenos Aires"	"Argentina"	"Rocket Flashlight"	"9752731596862613394"
"Rosario"	"Argentina"	"Google Men's Vintage Badge Tee Black"	"4894343382296033805"
"Santa Fe"	"Argentina"	"SPF-15 Slim & Slender Lip Balm"	"6556394846497485581"

```
Answer:

- This query hee retrieves data from multiple tables, joins them based on specific conditions, filters the results based on fullVisitorId values, 
- We can notice that in this data, the presence of Nest products suggests an interest in home automation and technology. Also there is a consistent presence of electronics, indicating a strong interest in technology and gadgets among visitors. Both men's and women's apparel are ordered, suggesting a diverse range of clothing products that are popular.
- Various accessories such as headgear, stickers, and housewares are also present, showcasing a demand for personalization and unique items. 


**Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**


SQL Queries:

-- The CTE is used to calculate the most popular product for each city and country.

WITH top_products AS (

  SELECT
    		p.product_name,
   			s.city,
    		s.country,
    		ROW_NUMBER() OVER (PARTITION BY s.city, s.country ORDER BY COUNT(*) DESC) AS popular_product

      -- assigns a row number to each combination of city and country, ordered by the count of occurrences in descending order. 
      -- The row number represents the popularity rank of the product within each city and country.
      
  FROM 	
			analytics_clean a
    JOIN 
		    all_sessions_clean s ON a."visitId" = s."visitId"
    JOIN 
			sales_report_clean sr ON sr."productSKU" = s."productSKU"
    JOIN 
			products_clean p ON sr."stockLevel" = p."stockLevel"
-- 	WHERE
--     		s.country = 'Denmark' -- to filter out countries and what they tend to purchase.
-- 	  p.name = 'Mens Vintage Henley' -- to filter what products those countries tend to buy.

  GROUP BY
    		p.product_name, s.city, s.country
)
			SELECT
 				    product_name,
  					city,
  					country
			FROM
  					top_products
			WHERE
 					popular_product = 1
					
-- represents the most popular product when popular_prouct = 1 
--  indicates a pattern of products that are the most popular. 

 


Answer:
-- I noticed that in certain regions closer to the equator such as Brazil, Columbia etc... tend to buy lighter clothing

--  such as short sleeve Tee's and Cap's 

-- On the contrary, I realized that countries in northern Europe and in the West such as the US and Canada buy onesie and jacket's. 




**Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:
-- Using a temp table because it provides a flexible and efficient way to store and manipulate data.
-- changing the null values in revenue column to equate to unitprice * orderedQuantity using a temp table for efficiency


CREATE TEMPORARY TABLE revenue_table AS

SELECT s.country, 

		s.city, 
		s."fullvisitorId", 
		p.sku, 
		SUM(a.unit_price * p."orderedQuantity") AS revenue
  
FROM 
		analytics_clean a
  
JOIN 
		all_sessions_clean  s ON a."visitId" = s."visitId"
  
JOIN 
		sales_report_clean  sr ON sr."productSKU" = s."productSKU"
  
JOIN 
		products_clean  p ON sr."stockLevel" = p."stockLevel"
  
WHERE 
		a.revenue IS NULL
  
GROUP BY 

		s.country,
  		s.city, 
		s."fullvisitorId", 
		p.sku
  
limit 100
		
SELECT DISTINCT 	revenue, country, city


FROM 			revenue_table


order by 		revenue desc


Answer:

-- I noticed revenue is at his highest in the USA.

-- Also the most populated cities and countries such as New York and USA have the highest revenues.





