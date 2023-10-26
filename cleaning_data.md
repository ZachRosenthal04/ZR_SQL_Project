What issues will you address by cleaning the data?





Queries:
Below, provide the SQL queries you used to clean your data.
--CLEANING FOR SQL PROJECT QUESTION 1:

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

--checking individually for sanity:
--full visitor id
SELECT COUNT(*), fullvisitorid -- fullvisitorid "3764227345226401562" has a duplicate
FROM	transactions
GROUP BY fullvisitorid
HAVING COUNT(*) > 1

--visit id:
SELECT COUNT(*), visitid -- visitid "1490046065" has a duplicate
FROM	transactions
GROUP BY visitid
HAVING COUNT(*) > 1

--transaction id:
SELECT COUNT(*), transactionid -- transactionid has no duplicates except for the 72 NULLs 3 not null
FROM	transactions
WHERE	transactionid IS NOT NULL
GROUP BY transactionid
HAVING COUNT(*) > 1


--Exploring the duplicated records.
--it is duplicated, the two records have the same visitid and fullvisitorid but different transaction ids 
--and product skus so  think the primary key is probably a composite key. Since a primary key cannot be NUL,
--it probably a combination of fullvisitorid, visitid, and productsku (but maybe transactionid)
SELECT 	*
FROM	transactions
WHERE	fullvisitorid = '3764227345226401562' AND
		visitid = '1490046065'


--5. FINDING THAT ONLY TOTALTRANSACTIONREVENUE is necessary for the output
SELECT 	totaltransactionrevenue, --totaltransactionrevenue = transactionrevenue.   
		transactionrevenue		 --transactionrevenue doesn't need to be in the answer's output
FROM	transactions
WHERE	totaltransactionrevenue <> transactionrevenue

--6.checking if the totaltransaction revenue is valid. 
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

--8. Creating another temp table with only the good transactions data. good_transactions
DROP TABLE good_transactions
CREATE TEMP TABLE good_transactions AS (
	SELECT	* --expected output 49 rows. transactions - bad_transactions
	FROM	transactions
	WHERE	NOT(productprice > totaltransactionrevenue OR
			city IS NULL)
);

--References
SELECT	* FROM transactions
SELECT	* FROM bad_transactions
SELECT 	* FROM good_transactions 
--///////////////////////////////

--QUESTION 1: Which cities and countries have the highest level of transaction revenues on the site?
--Query Showing the US is the country with the highest level of 
SELECT 	country, city,
		SUM(totaltransactionrevenue) AS city_transaction_revenue
FROM good_transactions
GROUP BY country, city
ORDER BY SUM(totaltransactionrevenue) DESC; ```
