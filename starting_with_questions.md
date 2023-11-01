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


SQL Queries:



Answer:





**Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**


SQL Queries:



Answer:





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
The answer can be derived from the same table as Question 4.
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
### Answer:







