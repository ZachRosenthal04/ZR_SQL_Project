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



Answer:





**Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:



Answer:







