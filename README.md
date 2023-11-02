# Final-Project-Transforming-and-Analyzing-Data-with-SQL

## Project/Goals
My project goals were to find which tables had the most reliable sources of data for each column and be able to transfer that reliable data to the gaps wherever possible. An example is that from my analysis the products table had the most reliable data based on quantities purchased compared to the all_sessions, sales_reports, analytics, and sales_by_sku tables, so the quantities from that table were used to explore and answer (as best as possible) my questions as well as the required questions  

## Process
### First Step
As a first step, I explored the data and tables to better situate myself amidst the data and some feel for what it conveyed. 
### Step 2
From the advice of some mentors, I then decided to not make changes to the raw data and the original database since they said this was not usually something one could do in the real world. In lieu of this, I created a view version of all the tables and started my cleaning from there. 
### Answering the Required Questions
My next decision was that I  would be best served to clean and answer the required questions first since those questions were designed based on the e-commerce database we created before cleaning and exploring. Being able to clean with those questions in mind was very helpful in getting me better situated with the data. It also helped me to target my cleaning which I think was a fruitful strategy. 

## Results
I figured out that (in my opinion) the products table had the most reliable information regarding products sold so I decided to use that as my measuring stick to join back into the all_sessions view/table to explore the information. Additionally, I noticed that there was a lot of missing information regarding the city but a fairly complete column concerning countries. Therefore, my personal questions for analysis are focused on the results from countries. I specifically focused on comparing the total products ordered for each major apparel category (Men's, Women's, and Kid's). I made tables with the top five and bottom five countries regarding the total products ordered in these categories.   

## Challenges 
This project was very challenging because the state of the raw data was very inconsistent. Many tables have conflicting records, or columns with illogical records i.e. having a transaction revenue less than a product price, and additionally, there were many unnecessary columns with data that was not useful for any exploratory research. One specific challenge regarding my focus was that the product category column was filled with all different kinds of categories that were a mix of both the page title column and product name column, and more. That being said, it was fun to try and take on that challenge to make some kind of intelligible and actionable grouping for analysis. Adding to the difficulty was a lack of proper organization. The tables were far from normalized and their interrelationship was unclear.
Another challenge (arguably the biggest of all) though unrelated to the project itself is that my pgAdmin4 fatally crashed and I lost much of my work and wasted a lot of time trying to reinstall and it back to working condition...but the show must go one!

## Future Goals
(what would you do if you had more time?)
If I had more time, I would like to normalize the tables. I would break down the large all_sessions table into parts relating to transactions, and product details, and then add the analytics portion to the analytics table and continue the process where appropriate. With more time (though I don't know if it's possible with the data we have) I would like to be able to disambiguate the city column. So that my category and product analysis can better measure the city data and its relationship to the country data. Additionally, while I made product categories for 'YouTube Merch' and 'Google Merch', I would like to further nuance those categories as well as the 'Nest-USA' category since there is a lot of security products being purchased which I think is really interesting to explore.
