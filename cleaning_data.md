What issues will you address by cleaning the data?  
1) Removing completely nulled out, irrelevant, and duplicate columns.
2) Matching column names between tables.
4) Column formatting to more appropriate structure and data types for analysis.
5) Consistent formatting for number of significant digits, character capitalization and replacement.
6) Fixing structural errors, e.g. rows (or records) for columns with instances of unrelated data.  
#
Queries: below, SQL queries needed to clean the data will be provded.
#
After importing the .csv files into the database, the initial analysis led to the inspection of columns that had only null values, irrelevant data, and duplicate columns.  
### Null Data  
#### In table: *all_sessions*  
The columns suspected to be completely nulled out were: 
- search_keyword  
- product_refund_amount
- item_quantity
- item_revenue
#### In table: *analytics*  
The column suspected to be completely nulled out was:  
- user_id    

The following example query was used to investigate further,  
```sql
--SELECT and FROM: retrieve columns from 'all_sessions' table
select search_keyword 
from all_sessions 
--WHERE: filter rows returned by SELECT, in this case, where the column's rows are not empty/null
where search_keyword is not null  
```  
once the suspicions were confirmed, the columns were deleted using  
```sql
--ALTER: add, remove, or modify columns in an existing table
alter table all_sessions
--DROP: delete/remove e.g. existing tables or columns based on specifier 
drop column search_keyword,  
drop column product_refund_amount,  
drop column item_quantity,  
drop column item_revenue  
``` 
and  
```sql  
alter table analytics  
drop column user_id
```
### Duplicate Data
#### In table: *all_sessions* 
The column suspected of being a duplicate was:
- transaction_revenue

The following example query was used to investigate further,
```sql
select total_transaction_revenue, transactions, product_revenue, transaction_revenue
from all_sessions
where transaction_revenue is not null
```
![SQL-Project1-Pic1](https://github.com/DylJFern/lighthouse-labs-ds/assets/128000630/0e7f9321-defb-4d4b-b66d-ed62d77b0e2c)

and based on analysis, we can conclude that 'transaction_revenue' only has 4 records (or rows) where it has non-null values and those records happen to match those of 'total_transaction_revenue'. Therefore, 'transaction_revenue' is a duplicate column and can be removed. 
```sql
alter table all_sessions
drop column transaction_revenue
```
Note: all data was initialized as varchar to prevent data loss as mentioned in the [README.md](README.md) and will later be converted [later on](cleaning_data.md/#Irrelevant-Data).
#### In table: *analytics*  
The column suspected of being a duplicate was:
- visit_start_time    

In the "analytics" table the column 'visit_start_time' matches 'visit_id' row-by-row; however, instead of removing it, it is kept as another example of data cleaning to be subsequently discussed.
### Irrelevant Data
#### In table: *analytics*  
The following column is not needed:
- social_engagement_type

It does not provide any unique information for the analysis. All of the column's rows contain the string "Not Socially Engaged". This can be confirmed by running two queries, one for all the rows in the table and another specifying rows containing that string.
```sql
select *
from analytics
```
and 
```sql
select *
from analytics
where social_engagement_type = 'Not Socially Engaged'
```
Both queries return the following result,

![SQL-Project1-Pic2](https://github.com/DylJFern/lighthouse-labs-ds/assets/128000630/a1e79dc3-2dc6-49b9-8bdc-ad693d1afe06)

as such, since all rows contain the same values, it does not provide any additional insight to the data and can be removed.
#
Although, it might not be considered part of the data cleaning process, matching corresponding column names between tables can be useful to improve clarity. For example, after importing all the data into the ecommerce database, one may notice that 'sales_by_sku', 'sales_report', and 'all_sessions' have a column named "product_sku" (refer to initial generated [ERD](README.md) for database).

While it may be obvious that they are foreign keys to the primary key in the 'products' table with column 'sku'. When performing the analysis, it may not be directly apparent. In addition, although minor, when wanting to access queries across multiple tables at once that require a join, each identifier has to be specified as opposed to it being performed on a single identifier.  

For instance, join 'products' with 'sales_by_sku' utilzing the ON clause:
```sql
select *
--"...as p": creates an alias "p" for the 'products' table; it can be created for CTE's, subqueries, SELECT statement's, and so on
from products as p
--there are different types of joins, e.g. INNER JOIN (default when type not specified), LEFT JOIN, RIGHT JOIN, and FULL OUTER JOIN
join sales_by_sku as sbs
--ON: specifies the join condition (defines how the tables should be joined, in other words, how the records should be matched)
	on sbs.product_sku = p.sku
--without the alias name we would write:
--	on sales_by_sku.product_sky = products.sku
```
Or by utilizing the USING clause:
```sql
from products as p
join sales_by_sku as sbs
---USING: shorthand notation that allows the join to be performed where both sides of the join use the same name for the joining column(s), and removes the duplicate column in the joined result unlike ON clause
	using(product_sku)
```

To update the column name in the 'products' table we can use:
```sql
alter table products
--RENAME: give the column a new name
rename column sku to product_sku
```
#
