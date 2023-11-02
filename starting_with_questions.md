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
--CLEANING THE PRODUCT CATEGORY COLUMN for Question 3, 4, and 5:
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


UPDATE as_p_productcategories
SET productcategory = CASE
			WHEN productcategory LIKE '(not set)' THEN NULL
						ELSE page_title END;

UPDATE as_p_productcategories
SET 	page_title = CASE
			WHEN page_title LIKE 'Store search results' THEN NULL
						ELSE page_title END;
						
UPDATE as_p_productcategories
SET		productcategory = CASE
					WHEN productcategory LIKE 'Pet%' THEN 'Pet'
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
SET productcategory = CASE
			WHEN productcategory LIKE 'Store search results' THEN NULL
						ELSE productcategory END;
```

### Answer:
```sql
--This query shows the total number of products ordered for each country based on product categories.

SELECT	DISTINCT(country),
	productcategory,
	SUM(total_ordered) AS total_products_ordered  
FROM	vas_productcategories
WHERE	total_ordered  > 0 AND
	productcategory IS NOT NULL
GROUP 	BY country, productcategory
ORDER 	BY total_products_ordered ASC
LIMIT 	10
```
One of the interesting patterns is that the United States has the most products purchased in 9/10 most purchased categories. Another interesting product category element is that none of the 3 apparel categories features in the top 10 categories with the most products purchased. 


### **Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**


SQL Queries:
```sql
--QUESTION 4: Top-Selling product (or category) for each city and country
WITH country_product_quantities_CTE AS (
	SELECT 	country,
			productcategory,
			SUM(total_ordered) AS product_qty_sold,
			RANK() OVER (
			PARTITION BY country
			ORDER BY SUM(total_ordered) DESC) AS rank
	FROM	vas_productcategories
	WHERE	productrevenue IS NOT NULL
	GROUP	BY country, productcategory
	)
SELECT	country,
		productcategory AS top_product,
		product_qty_sold
FROM	country_product_quantities_CTE
WHERE	rank = 1;
```
---------------------------------------------
### Answer:
For Country:
```sql
WITH country_product_quantities_CTE AS (
	SELECT 	country,
			productcategory,
			SUM(total_ordered) AS product_qty_sold,
			RANK() OVER (
			PARTITION BY country
			ORDER BY SUM(total_ordered) DESC) AS rank
	FROM	vas_productcategories
	WHERE	productrevenue IS NOT NULL
	GROUP	BY country, productcategory
	)
SELECT	country,
		productcategory AS top_product,
		product_qty_sold
FROM	country_product_quantities_CTE
WHERE	rank = 1
ORDER	BY product_qty_sold DESC
LIMIT 	10;
```
YouTube Merch was the top-selling category 9/10 of the top-selling countries. The countries were in order from most to least: United States, United Kingdom, India, Germany, Canada, Australia, France, Taiwan, Netherlands, and Italy. 

For City:
```sql
WITH city_product_quantities_CTE AS (
	SELECT 	city,
		productcategory,
		SUM(total_ordered) AS product_qty_sold,
		RANK() OVER (
		PARTITION BY city
		ORDER BY SUM(total_ordered) DESC) AS rank
	FROM	vas_productcategories
	WHERE	productrevenue IS NOT NULL AND 
		city IS NOT NULL
	GROUP	BY city, productcategory
	)
SELECT	city,
		productcategory AS top_product,
		product_qty_sold
FROM	city_product_quantities_CTE
WHERE	rank = 1
ORDER	BY product_qty_sold DESC
LIMIT 	10;
```
Conversely, the top 6/10 selling cities are in the United States and 4/5 of the top-selling products for cities are security products. Unsurprisingly, these cities are all in the United States.     

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




