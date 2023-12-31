What are your risk areas? Identify and describe them.

One of the main risk areas is in the product category column in the all_sessions table. I have cleaned as much as I can to get viable information to do some analysis, but despite that, there are still several one-off or infrequent categories that are too difficult or time-consuming to clean.
 

QA Process:
Describe your QA process and include the SQL queries used to execute it.
In the process of answering some of the questions. I noticed that the Products table had the most up-to-date records for the units_sold column. This was not only in the product quantity data but also in the number of non-null records. I joined the productsku and units_sold (which I renamed total_ordered) columns to both the sales_by_sku and then sales_reports tables to find out how the records compared. From that comparison, I drew my conclusion that the products table was the most reliable.

```sql
DROP VIEW IF EXISTS v_sales_by_sku;
CREATE VIEW v_sales_by_sku AS (
	SELECT 	*
	FROM	sales_by_sku
			)

UPDATE v_sales_by_sku
SET total_ordered = vp.total_ordered
FROM v_products vp
WHERE v_sales_by_sku.productsku = vp.productsku;

UPDATE v_sales_reports
SET total_ordered = vp.total_ordered
FROM v_products vp
WHERE v_sales_reports.productsku = vp.productsku;
```
This query finds the true duplicates and the second one removes them from the db. As I was exploring, I found that in the all_sessions table there are 5 instances (10 rows total) that have 'true duplicates'
```sql

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
