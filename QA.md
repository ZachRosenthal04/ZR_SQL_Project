What are your risk areas? Identify and describe them.

Despite my cleaning efforts one of my main risk areas is in the product category column. I have cleaned as much as I can to get viable information to do some analysis, but despite that, there are still several one-off or infrequent categories that are too difficult or time-consuming to clean.

Additionally, there are a lot of city records that are NULL that need to filled in with 

QA Process:
Describe your QA process and include the SQL queries used to execute it.
In process of answering some of the questions. I noticed that the Products table had the most up-to-date records for the units_sold column. This was not only in quantity, but also in number of non-null records. I joined the productsku and units_sold (which I renamed total_ordered) columns to both the sales_by_sku and then sales_reports tables to find out how the records compared. Though sales_by_sku and sales_reports tables were fairly similar, as mentioned, I decided the products table had the most reliable records for the units_sold/total_ordered columns. 

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
