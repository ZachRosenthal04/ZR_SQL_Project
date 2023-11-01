# Final-Project-Transforming-and-Analyzing-Data-with-SQL

## Project/Goals
My project goals were to find which tables had the most reliable sources of data for each column and be able to transfer that reliable data to the gaps wherever possible. An example is that from my analysis the products table had the most reliable data based on quantities purchased compared to the all_sessions, sales_reports, analytics, and sales_by_sku tables, so the quantities from that table were used to explore and answer (as best as possible) my questions as well as the required questions  

## Process
### First Step
My first step was to go over the tables to try and better situate myself amidst the data. I tried to get some feel for what it was trying to convey. From advice of some mentors, I then decided it not make changed to the raw data and the original database since they said this was not usually something one could do in the real world. IN lieu of this, I created a view version of all the tables and did my cleaning from there. 
### Answering the Required Questions
My next decision was that I  would be best served to clean and answer the required questions first since those questions were designed based on the e-commerce database we created before cleaning and exploring. Being able to clean with those questions in mind was very helpful in getting me better situated with the data. It also helped me to target my cleaning which I think was a fruitful strategy. 

## Results
(fill in what you discovered this data could tell you and how you used the data to answer those questions)

## Challenges 
This project was very challenging because of how unclean the data is. Many tables have and had conflicting records, columns with illogical records i.e. having a total transaction revenue less than a product price and additionally, there were many unnecessary columns with data that was not useful for any exploratory research. 
Adding to the difficulty was a lack of proper organization. The tables were far from normalized and their interrelationship was unclear.
Another challenge (arguably the biggest of all) though unrelated to the assignment itself is that my pgAdmin4 fatally crashed and lost much of my work and lost A LOT of time trying reinstall it and recreate the tables. It has been a technical-difficulty-ridden few days, but I think the worst is behind me now. 
## Future Goals
(what would you do if you had more time?)
If I had more time, I would really spend time normalizing the tables. I would breakdown the large all_sessions table into parts relating to transactions, product details, and then add the analytics portion tot he analytics table. I would do the same for the other tables as well.
