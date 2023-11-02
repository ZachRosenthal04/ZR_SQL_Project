What issues will you address by cleaning the data?

I will be adjusting the product category column to get more manageable and insightful data. I will make views of all the original tables so that the original db is not adversely affected. I'm checking for true duplicates and getting rid of them and I am making sure that when there are products sold and a price, that product revenue is filled in. I right-padded 0's in both the fullvisitorid and product sku columns in all the views I made to standardize the format when applicable (i.e when the product SKU was in the 'GGOE%' format).  



Queries:

### QUESTION 1: 
```SQL
--Creating a view to not effect the raw db
DROP VIEW IF EXISTS v_all_sessions;
CREATE VIEW v_all_sessions AS ( --This will be the copy I work on so I don't change the original db
	SELECT 	*
	FROM	all_sessions
		);```
```SQL
--ALL_SESSIONS TABLE:
--1. Make the prices in the proper format
UPDATE v_all_sessions --This divides all the pricing data by 1M thanks to the hint
SET productprice = (productprice / 1000000.0)::NUMERIC (10,2),
	totaltransactionrevenue = (totaltransactionrevenue / 1000000.0)::NUMERIC (10,2),
	productrevenue = (productrevenue / 1000000.0)::NUMERIC (10,2),
	transactionrevenue = (transactionrevenue / 1000000.0)::NUMERIC (10,2);

--Add missing product revenue when possible:
UPDATE 	v_all_sessions
SET	productrevenue = (productquantity * productprice)::NUMERIC (10, 2);

--0 padding on right to make fullvisitorid the same format (19 chars)
UPDATE v_all_sessions
SET fullvisitorid = RPAD(fullvisitorid, 19, '0');

--update to make the 'GGOE%' product skus into the 14 char format when they have the GGOE beginning
UPDATE v_all_sessions 
SET productsku = 
		CASE WHEN productsku LIKE 'GGOE%' THEN RPAD(productsku, 14, '0')
		ELSE productsku END;

--3.Add NULLS where data is '(not set)' or 'not available in demo dataset'
UPDATE 	v_all_sessions 
SET 	country =  
		CASE WHEN country LIKE '(not set)' THEN NULL 
		ELSE country END,
	city = 
		CASE WHEN city IN ('(not set)', 'not available in demo dataset') THEN NULL
		ELSE city END,
	productvariant = 
		CASE WHEN productvariant LIKE '(not set)' THEN NULL
		ELSE productvariant END;

--Changed the 'v2' column headers to match other tables
ALTER VIEW v_all_sessions
RENAME COLUMN v2productname TO productname;

ALTER VIEW v_all_sessions
RENAME COLUMN v2productcategory TO productcategory;

--Found 'true' duplicates and removed them from the view/table
SELECT	COUNT(*) AS count_productsku, --5 rows are 'true duplicates'
		productsku,
		productname,
		visitid,
		fullvisitorid
FROM	v_all_sessions
GROUP 	BY  productsku, productname, visitid, fullvisitorid
HAVING	COUNT(*) > 1

DELETE FROM v_all_sessions --removed the true duplicates from a_s_products_clean1
WHERE visitid IN
    (SELECT visitid
    FROM 
        (SELECT visitid,
         ROW_NUMBER() OVER( 
			 PARTITION BY productsku, productname, visitid, fullvisitorid
        	ORDER BY  visitid ) AS row_num
        FROM v_all_sessions ) vas
        WHERE vas.row_num > 1 );
```

### QUESTION 2:
Since the cleaning for question 1 was done on a view, it has been completed for question 2.

\ANALYTICS TABLE
----------------------
I created a View of the analytics table to not affect the raw db.
I also created a temp table with just the distinct elements of the analytics table since it is so large and cumbersome to use.
```SQL
DROP IF EXISTS v_analytics;
CREATE VIEW v_analytics AS (
		SELECT 	*
		FROM	analytics
		)

DROP TABLE IF EXISTS distinct_analytics;
CREATE TEMP TABLE distinct_analytics AS (
		SELECT DISTINCT	* 
		FROM	analytics
	);
```

Below is similar cleaning to question 1's:

```SQL
--make sure fullvisitorid is formatted correctly
UPDATE distinct_analytics
SET fullvisitorid = RPAD(fullvisitorid, 19, '0');

UPDATE distinct_analytics --This divides all the pricing data by 1M thanks to the hint
SET unit_price = (unit_price / 1000000.0)::NUMERIC (10,2),
	revenue = (revenue / 1000000.0)::NUMERIC (10,2);
```
____________________________________________________
### QUESTION 2: PRODUCTS TABLE	
I changed the name of columns to match other tables
```SQL
ALTER TABLE products
RENAME COLUMN sku to productsku;

ALTER TABLE products
RENAME COLUMN name to productname;

ALTER TABLE products
RENAME COLUMN orderedquantity TO total_ordered;


```sql
SELECT 	COUNT(*) AS count_products, --shows that every productsku and productname pair are unique 
		productsku, productname
FROM	products
GROUP	BY productsku, productname
HAVING	COUNT(*) > 1
```


Below, provide the SQL queries you used to clean your data.
--CLEANING FOR SQL
PROJECT QUESTION 1:

```SQL
--1. This query creates a temporary table with the useful data from the all_sessions tables
--regarding transactions
DROP TABLE transactions
CREATE TEMP TABLE transactions AS (
SELECT	*
FROM 	all_sessions
WHERE	transactions IS NOT NULL OR --81 rows
		transactionrevenue IS NOT NULL OR --81 rows
		totaltransactionrevenue IS NOT NULL --81 rows
);


--2. GETTING THE APPROPRIATE PRICE FORMAT
UPDATE transactions --This divides all the pricing data by 1M thanks to the hint
SET productprice = (productprice / 1000000.0)::NUMERIC (10,2),
	totaltransactionrevenue = (totaltransactionrevenue / 1000000.0)::NUMERIC (10,2),
	productrevenue = (productrevenue / 1000000.0)::NUMERIC (10,2),
	transactionrevenue = (transactionrevenue / 1000000.0)::NUMERIC (10,2);

--3.GETTING RID OF THE LONG STRINGS IN THE CITY COLUMN
UPDATE transactions --I changed the "not available..." because I thought it would be easier to deal with
SET	city = CASE
			WHEN city LIKE 'not available in demo dataset' THEN NULL
			ELSE city
			END;
			

--4. Now I'm going to check for duplicates in all the id columns as well as the date and time columns
--to see if these are really duplicates.

SELECT 	COUNT(*), --Since the result is an empty table, I can be reasonably sure that there are no		
		fullvisitorid, --exact duplicates in identifier columns of the transactions table. 
		visitid,		--therefore, the three id columns can represent a composite key for the transactions table
		transactionid
FROM 	transactions
GROUP BY fullvisitorid,
		visitid, 
		transactionid
HAVING COUNT(*) > 1



--Exploring the duplicated records.
--it is duplicated, the two records have the same visitid and fullvisitorid but different transaction ids 
--and product skus so  think the primary key is probably a composite key. Since a primary key cannot be NUL,
--it probably a combination of fullvisitorid, visitid


--5. FINDING THAT ONLY TOTALTRANSACTIONREVENUE is necessary for the output
SELECT 	totaltransactionrevenue, --totaltransactionrevenue = transactionrevenue.   
		transactionrevenue		 --transactionrevenue doesn't need to be in the answer's output
FROM	transactions
WHERE	totaltransactionrevenue <> transactionrevenue

--6.checking if the totaltransactionrevenue is valid. 
--	If the productprice > than totaltransactionrevenue than it isn't valid

 --10 rows have invalid totaltransctionrevenues. I then added OR city is null because it doesn't
-- help me answer Question 1.
SELECT	* --output is 32 rows.
FROM	transactions
WHERE	productprice > totaltransactionrevenue OR
	city IS NULL
--I'm going to try and use this output as a way to filter the main table's 81 rows to get a 
--table with only valid data to find out the answer to question 1.

--7. make the above query a temp table so I can subtract transactions from bad_transactions
DROP TABLE 
CREATE TEMP TABLE bad_transactions AS ( 
SELECT	* --output is 32 rows.
FROM	transactions
WHERE	productprice > totaltransactionrevenue OR
		city IS NULL
		);
--Sanity check for the bad_transactions table. 
SELECT totaltransactionrevenue, productprice --empty table 
FROM bad_transactions 
WHERE productprice < totaltransactionrevenue AND CITY IS NOT NULL

SELECT	* --expected output 49 rows. transactions - bad_transactions
FROM	transactions
WHERE	NOT(productprice > totaltransactionrevenue OR
		city IS NULL)

--8. Creating another temp table with only the good transactions data as: good_transactions
DROP TABLE good_transactions
CREATE TEMP TABLE good_transactions AS (
	SELECT	* --expected output 49 rows. transactions - bad_transactions
	FROM	transactions
	WHERE	NOT(productprice > totaltransactionrevenue
		OR city IS NULL)
);
```
 

### Cleaning Product Categories for Question 3, 4, 5:
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
