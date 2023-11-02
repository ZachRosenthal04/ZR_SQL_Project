Question 1: The United States dominates this table in total number of products ordered and in the amount of total revenue generated from the product. Since the average number of products purchased by country yielded a top 5 without the US. I want to know what were the product categories for the top 5 average orders.  

SQL Queries:
```sql
WITH country_product_quantities_CTE AS (
	SELECT 	country,
			productcategory,
			ROUND(AVG(total_ordered), 2) AS avg_products_purchased,
			RANK() OVER (
			PARTITION BY country
			ORDER BY AVG(total_ordered) DESC) AS rank
	FROM	vas_productcategories
	WHERE	productrevenue IS NOT NULL AND
			total_ordered IS NOT NULL
	GROUP	BY country, productcategory
	)
SELECT	country,
		productcategory AS top_product,
		avg_products_purchased
FROM	country_product_quantities_CTE
WHERE	rank = 1
ORDER	BY avg_products_purchased DESC
LIMIT 	10;
```
Answer:
The highest average products purchased for Chile was in the Lifestyle category. They were followed by Drinkware for Italy, Accessories for Portugal, Lifestyle for Russia, and Drinkware for Singapore. 

Question 2: Which countries order the most and least Men's Apparel,

SQL Queries:
```sql
WITH most_ordered_mens_apparel_CTE AS (
    SELECT	country,
        	productcategory,
        	SUM(total_ordered) AS most_total_mens_apparel_ordered
    FROM 	vas_productcategories
    WHERE	total_ordered > 0 AND
        	productcategory = 'Men''s Apparel'
    GROUP BY country, productcategory
    ORDER BY most_total_mens_apparel_ordered DESC
    LIMIT 5
	),
least_ordered_mens_apparel_CTE AS (
    SELECT	country,
        	productcategory,
        	SUM(total_ordered) AS least_total_mens_apparel_ordered
    FROM 	vas_productcategories
    WHERE	total_ordered > 0 AND
        	productcategory = 'Men''s Apparel'
    GROUP BY country, productcategory
    ORDER BY least_total_mens_apparel_ordered ASC
    LIMIT 5
	)
SELECT	mto.country,
    	mto.most_total_mens_apparel_ordered,
		lto.country,
    	lto.least_total_mens_apparel_ordered
FROM 	most_ordered_mens_apparel_CTE mto
FULL OUTER JOIN least_ordered_mens_apparel_CTE lto
ON 		mto.country = lto.country AND mto.productcategory = lto.productcategory;
```
Answer:
The top 5 countries that ordered the most men's apparel are: USA, India, Canada, UK, and Japan.
The bottom 5 countries that ordered the least men's apparel are: Paraguay, Serbia, Haiti, Cambodia, and Puerto Rico. 


Question 3: Which countries ordered the most and least Women's Apparel

SQL Queries:
```sql
WITH most_ordered_womens_apparel_CTE AS (
    SELECT	country,
        	productcategory,
        	SUM(total_ordered) AS most_total_womens_apparel_ordered
    FROM 	vas_productcategories
    WHERE	total_ordered > 0 AND
        	productcategory = 'Women''s Apparel'
    GROUP BY country, productcategory
    ORDER BY most_total_womens_apparel_ordered DESC
    LIMIT 5
	),
least_ordered_womens_apparel_CTE AS (
    SELECT	country,
        	productcategory,
        	SUM(total_ordered) AS least_total_womens_apparel_ordered
    FROM 	vas_productcategories
    WHERE	total_ordered > 0 AND
        	productcategory = 'Women''s Apparel'
    GROUP BY country, productcategory
    ORDER BY least_total_womens_apparel_ordered ASC
    LIMIT 5
	)
SELECT	mto.country,
    	mto.most_total_womens_apparel_ordered,
		lto.country,
    	lto.least_total_womens_apparel_ordered
FROM 	most_ordered_womens_apparel_CTE mto
FULL OUTER JOIN least_ordered_womens_apparel_CTE lto
ON 		mto.country = lto.country AND mto.productcategory = lto.productcategory;
```
Answer:
The top 5 countries with the most orders for women's apparel are: USA, Canada, UK, India, Australia.
The 5 countries that ordered the least women's apparel are: Belarus, Dominican, Republic, Macedonia, Serbia, Honduras. 


Question 4: Which countries ordered the most and least Kid's Apparel?

SQL Queries:
```sql

WITH most_ordered_kids_apparel_CTE AS (
    SELECT	country,
        	productcategory,
        	SUM(total_ordered) AS most_total_kids_apparel_ordered
    FROM 	vas_productcategories
    WHERE	total_ordered > 0 AND
        	productcategory = 'Kid''s Apparel'
    GROUP BY country, productcategory
    ORDER BY most_total_kids_apparel_ordered DESC
    LIMIT 5
	),
least_ordered_kids_apparel_CTE AS (
    SELECT	country,
        	productcategory,
        	SUM(total_ordered) AS least_total_kids_apparel_ordered
    FROM 	vas_productcategories
    WHERE	total_ordered > 0 AND
        	productcategory = 'Kid''s Apparel'
    GROUP BY country, productcategory
    ORDER BY least_total_kids_apparel_ordered ASC
    LIMIT 5
	)
SELECT	mto.country,
    	mto.most_total_kids_apparel_ordered,
		lto.country,
    	lto.least_total_kids_apparel_ordered
FROM 	most_ordered_kids_apparel_CTE mto
FULL OUTER JOIN least_ordered_kids_apparel_CTE lto
ON 		mto.country = lto.country AND mto.productcategory = lto.productcategory

```
Answer:
The US dominates the world in their kid's apparel ordered by a very large margin. They ordered approximately 200% more than the next second placed top ordered Canada. The top orderers are: United States, Canada, Singapore, India, Australia.
The bottom 5 kid's apparel ordering countries all ordered similar total quantities. They are: Indonesia, Greece, Russia, Sweden, Algeria.


Question 5: 

SQL Queries:

Answer:
