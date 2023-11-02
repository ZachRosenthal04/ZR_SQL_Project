Answer the following questions and provide the SQL queries used to find the answer.

    
**Question 1: Which cities and countries have the highest level of transaction revenues on the site?**


SQL Queries:
```SQL
--FINDING THE VALID TRANSACTION DATA TO ANSWER QUESTION 1:
SELECT 	totaltransactionrevenue, 
		productprice, 
		productquantity, 	
		productrevenue
FROM 	all_transactions 
WHERE 	productrevenue <> totaltransactionrevenue

			
UPDATE all_transactions
SET totaltransactionrevenue = 
			CASE 
				WHEN totaltransactionrevenue <> productrevenue THEN productrevenue	
				ELSE totaltransactionrevenue END;

--72 Valid Transaction Data
SELECT 	fullvisitorid,
		visitid,
		city,
		country,
		totaltransactionrevenue,
		transactions,
		productquantity,
		productprice,
		productrevenue
FROM	all_transactions
WHERE	NOT (transactions IS NULL OR 
		productprice > totaltransactionrevenue OR
		city IS NULL AND country IS NULL);
		
		
DROP TABLE IF EXISTS good_transactions;
CREATE TEMP TABLE good_transactions AS (
	SELECT 	* 
	FROM	all_transactions
	WHERE	NOT (transactions IS NULL OR 
			productprice > totaltransactionrevenue OR
			city IS NULL AND country IS NULL));
```


Answer:
```SQL
--ANSWER QUESTION 1: COUNTRIES
--United States most transaction revenue
SELECT 	country,
		SUM(totaltransactionrevenue) AS transaction_revenue
FROM 	good_transactions
GROUP	BY country
ORDER BY SUM(totaltransactionrevenue) DESC```

```SQL
--ANSWER QUESTION 1: CITIES
--The highest transaction city is a city somewhere in the US, that needs further study
SELECT 	COALESCE (city, country, 'N/A'),
		SUM(totaltransactionrevenue)
FROM	good_transactions
GROUP	BY country, city
ORDER	BY SUM(totaltransactionrevenue) DESC```



**Question 2: What is the average number of products ordered from visitors in each city and country?**


Answer:



**Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**


SQL Queries:
```sql
DROP TABLE IF EXISTS vas_productcategories;
CREATE TEMP TABLE vas_productcategories AS (
	SELECT	vas.fullvisitorid,
		vas.visitid, 
		vas.country,
		vas.city, 
		vas.date,
		vas.productsku AS productsku,  
		vas.productname AS as_productname, 
		vp.total_ordered,
		vas.productcategory,
		vas.pagetitle AS page_title
	FROM	v_all_sessions vas
	FULL JOIN v_products vp
	ON	vas.productsku = vp.productsku
	WHERE	NOT(vp.total_ordered IS NULL OR
			vas.city IS NULL AND 
			vas.country IS NULL)
					);

```
```sql
--Page title is a much better represenation of the product category so that will be used 
--to update the productcategory column

SELECT 	as_productname, --No instances where their info does not align
	productcategory, 
	page_title
FROM	as_p_productcategories
WHERE	(productcategory IS NOT NULL AND
	page_title IS NULL) OR 
	(productcategory IS NULL AND 
	 page_title IS NOT NULL);
```
```sql
--CLEANING THE PRODUCT CATEGORY COLUMN:
UPDATE as_p_productcategories
SET productcategory = CASE WHEN productcategory LIKE '(not set)' THEN NULL
						ELSE page_title END;

UPDATE as_p_productcategories
SET 	page_title = CASE WHEN page_title LIKE 'Store search results' THEN NULL
						ELSE page_title END;
						
UPDATE as_p_productcategories
SET		productcategory = CASE WHEN productcategory LIKE 'Pet%' THEN 'Pet'
							ELSE productcategory END;
UPDATE as_p_productcategories
SET		productcategory = CASE 
							WHEN productcategory LIKE 'Drinkware%' THEN 'Drinkware'
							WHEN productcategory LIKE '%Drinkware%' THEN 'Drinkware'
							ELSE productcategory END;
UPDATE as_p_productcategories
SET		productcategory = CASE 
							WHEN productcategory LIKE 'Bag%' THEN 'Bags'
							WHEN productcategory LIKE '%Bag%' THEN 'Bags'
							ELSE productcategory END;
						
SELECT 	 	COUNT(*), --Looking at all the Apparel Categories
			productcategory
FROM 	 	as_p_productcategories
WHERE		productcategory LIKE '%Apparel%'
GROUP BY	productcategory

UPDATE as_p_productcategories
SET		productcategory = CASE
					WHEN productcategory LIKE 'Apparel%' THEN 'Apparel'
					WHEN productcategory LIKE 'Infant%' THEN 'Kid''s Apparel'
					WHEN productcategory LIKE '%Infant%' THEN 'Kid''s Apparel'
					WHEN productcategory LIKE 'Kids%' THEN 'Kid''s Apparel'
					WHEN productcategory LIKE '%Kid''s%' THEN 'Kid''s Apparel'
					WHEN productcategory LIKE 'Men''s%' THEN 'Men''s Apparel'
					WHEN productcategory LIKE '%Men''s%' THEN 'Men''s Apparel'
					WHEN productcategory LIKE 'Toddler%' THEN 'Kid''s Apparel'
					WHEN productcategory LIKE 'Women%' THEN 'Women''s Apparel'
					WHEN productcategory LIKE '%Women''s' THEN 'Women''s Apparel'
					WHEN productcategory LIKE 'Youth%' THEN 'Kid''s Apparel'
					WHEN productcategory LIKE '%Youth''s' THEN 'Kid''s Apparel'
					WHEN productcategory LIKE 'Headgear%' THEN 'Headgear'
					ELSE productcategory END;
							
--Focusing on only cleaning categories with meaningful representation
SELECT 	COUNT(*), productcategory 
FROM	as_p_productcategories
WHERE	productcategory IS NOT NULL AND
		LENGTH(productcategory) > 15
GROUP 	BY productcategory 
HAVING COUNT(*) > 25

UPDATE as_p_productcategories
SET productcategory = CASE
			WHEN productcategory LIKE '%Electronics%' THEN 'Electronics'
			WHEN productcategory LIKE 'Electronics%' THEN 'Electronics'
			WHEN productcategory LIKE 'Fun%' THEN 'Accessories'
			WHEN productcategory LIKE 'Stickers' THEN 'Accessories'
			WHEN productcategory LIKE 'Office%' THEN 'Office'
			WHEN productcategory LIKE '%Office%' THEN 'Office'
			WHEN productcategory LIKE 'Sports & Fitness%'THEN 'Lifestyle'
			WHEN productcategory LIKE 'Housewares%' THEN 'Housewares'
			WHEN productcategory LIKE 'Accessories%' THEN 'Accessories'
			ELSE productcategory END;

SELECT 	as_productname,
	productcategory, --No instances where their info does not allign
	page_title
FROM	as_p_productcategories
WHERE	productcategory LIKE 'YouTube%'

UPDATE as_p_productcategories
SET productcategory = CASE 
			WHEN productcategory = 'YouTube' THEN 'YouTube Merch'
			WHEN productcategory LIKE 'YouTube%%' THEN 'YouTube Merch'
			WHEN productcategory = 'Google' THEN 'Google Merch'
			WHEN productcategory LIKE 'Google%' THEN 'Google Merch'
			WHEN productcategory LIKE '%Google%' THEN 'Google Merch'
			ELSE productcategory END;
						
UPDATE as_p_productcategories
SET productcategory = CASE WHEN productcategory LIKE 'Store search results' THEN NULL
						ELSE productcategory END;
```

Answer:
--This is query shows the total number of products ordered for each country based on product categories.
--The US has the 9/10 products ordered in unique categories.
SELECT	DISTINCT(country),
		productcategory,
		SUM(total_ordered) AS total_products_ordered  
FROM	vas_productcategories
WHERE	total_ordered  > 0 AND
		productcategory IS NOT NULL
GROUP BY country, productcategory
ORDER BY total_products_ordered DESC




**Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**


SQL Queries:
```sql
DROP TABLE IF EXISTS country_city_top_sellers;
CREATE TEMP TABLE country_city_top_sellers AS (
SELECT DISTINCT 
		acq.country,
		acq.city,
		acq.productsku, 
		acq.productname,
		acq.total_ordered,
		acq.productcategory, 
		acq.totaltransactionrevenue
FROM 
   	 	as_clean_q4 acq 
JOIN --Joining on a table I created (with 3 columns - country, city, MAX totalrevenue)
    (SELECT 	country, city, 
        		MAX(totaltransactionrevenue) AS max_revenue
    	FROM 	as_clean_q4
    	GROUP 	BY country, city) acq_b 
ON --Both columns need to match to satisfy the join constraints
    acq.country = acq_b.country AND
	acq.city = acq_b.city AND 
	acq.totaltransactionrevenue = acq_b.max_revenue 
	
WHERE 
    		NOT(acq.city IS NULL OR 
    		acq.country IS NULL) AND
    		acq.total_ordered > 0 
ORDER BY 	country, total_ordered DESC
					);
```
---------------------------------------------
### Answer:
```sql

SELECT	country, productname,
		SUM(total_ordered) AS country_total_ordered
FROM 	country_city_top_sellers	
GROUP	BY country, productname 
ORDER	BY SUM(total_ordered) DESC
LIMIT 10
```
The interesting thing is that much like the real world, the US had the most total orders out of 217 countries and their most purchased product is security systems.




**Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:

```sql
--The answer can be derived from the same table as Question 4.


DROP TABLE IF EXISTS country_city_top_sellers;
CREATE TEMP TABLE country_city_top_sellers AS (
SELECT DISTINCT 
		acq.country,
		acq.city,
		acq.productsku, 
		acq.productname,
		acq.total_ordered,
		acq.productcategory, 
		acq.totaltransactionrevenue
FROM 
   	 	as_clean_q4 acq 
JOIN --Joining on a table I created (with 3 columns - country, city, MAX totalrevenue)
    (SELECT 	country, city, 
        		MAX(totaltransactionrevenue) AS max_revenue
    	FROM 	as_clean_q4
    	GROUP 	BY country, city) acq_b 
ON --Both columns need to match to satisfy the join constraints
    acq.country = acq_b.country AND
	acq.city = acq_b.city AND 
	acq.totaltransactionrevenue = acq_b.max_revenue 
	
WHERE 
    		NOT(acq.city IS NULL OR 
    		acq.country IS NULL) AND
    		acq.total_ordered > 0 
ORDER BY 	country, total_ordered DESC
				);
```
### Answer:

```sql

--Question 5: 
--Can we summarize the impact of revenue generated from each city/country?

SELECT		country,
		SUM(totaltransactionrevenue) AS total_revenue
FROM		country_city_top_sellers ccts
GROUP BY 	country
ORDER BY	SUM(totaltransactionrevenue) DESC

```
There were 2 South American countries (El Salvador and Paraguay) had the least revenue
And interestingly, 3/5 top revenue generating companies were from North America. 
The United States in first, Canada in third, and Mexico in 5th. 




