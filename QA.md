What are your risk areas? Identify and describe them.
1) Tables, columns, or rows with completely nulled out, irrelevant, and duplicate data.
2) Column formatting which can include inncorrect data types or incorrect and inconsistent formats.

<br>The QA process to identify problems with the data is described below with SQL queries that were used to execute it.

---
## Initial Inspection
### Null
Upon importing the .csv files into the database, the initial analysis led to the inspection of columns that had only null values, irrelevant data, and duplicate data columns.

In the 'all_sessions' table, the columns suspected to be completely nulled out were:
  - search_keyword  
  - product_refund_amount
  - item_quantity
  - item_revenue  

In the 'analytics' table, the column suspected to be completely nulled out was:
  - user_id

For these columns, the following example query could be used to further investigate.
```sql
--SELECT and FROM: retrieve columns from 'all_sessions' table
select search_keyword 
from all_sessions 
--WHERE: filter rows returned by SELECT, in this case, where the column's rows are not empty/null
where search_keyword is not null 
```
Once the suspicions are confirmed, we can remove the [null data](cleaning_data.md/#Null-Data).

### <br>Duplicate
In the 'all_sessions' table, the column suspected of being a duplicate was:
  - transaction_revenue 

In the 'analytics' table, the column suspected of being a duplicate was:
  - visit_start_time

The following example query was used to further investigate the 'all_sessions' table.
```sql
select total_transaction_revenue, transactions, product_revenue, transaction_revenue
from all_sessions
where transaction_revenue is not null
```

![SQL-Project1-Pic_new1](https://github.com/DylJFern/lighthouse-labs-ds/assets/128000630/0d2ae970-83d3-4a08-93d0-a8b700ccf07d)

We can conclude that the 'transaction_revenue' column only has 4 records (or rows) where it has records of non-null values that match those of the 'total_transaction_revenue' column. Therefore, the 'transaction_revenue' column is a duplicate column and can be [removed](cleaning_data.md/#NDuplicate-Data).

Note: all the data was initialized as varchar to prevent data loss as mentioned in the [README.md](README.md).

### <br>Irrelevant
In the 'analytics' table, the following column is not needed:
  - social_engagement_type

It does not provide any unique or helpful information for the analysis being conducted. All of the column's rows consists strings with the value "Not Socially Engaged". This can be confirmed by running two queries one for all the rows in the 'analytics' table and another specifying rows containing that particular string.
```sql
select social_engagement_type
from analytics
```
```sql
select social_engagement_type
from analytics
where social_engagement_type = 'Not Socially Engaged'
```

![SQL-Project1-Pic_new2](https://github.com/DylJFern/lighthouse-labs-ds/assets/128000630/8e666170-9b2a-404f-8792-933127f2792b)

---
## Clarity and Readability
Improving the clarity of a database is useful to as it helps eliminate any confusion and makes things much more easily identifiable. For example, after importing all the .csv file data into the ecommerce database, one may notice that the tables 'sales_by_sku', 'sales_report', and 'all_sessions' all have a column named "product_sku" (refer to initial generated [ERD](README.md) for database).

While it may be obvious that they could be potential foreign keys to the primary key in the 'products' table with column name 'sku'. When performing the analysis, it may not be directly apparent. In addition, although minor, when wanting to access queries across multiple tables at once that require a join, each identifier has to be specified as opposed being performed on a single identifier.  

For instance, to join 'products' with 'sales_by_sku' utilzing the ON clause we must specify two identifiers,
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
or by utilizing the USING clause we can utilize a single identifier.
```sql
from products as p
join sales_by_sku as sbs
---USING: shorthand notation that allows the join to be performed where both sides of the join use the same name for the joining column(s), and removes the duplicate column in the joined result unlike ON clause
	using(product_sku)
```
---
## Simple Data Validation Check


After performing an initial analysis and removing unwanted data, the next step taken was to format columns of every table to more appropriate data types (based on column header). Before proceeding, there is a check that can be performed (for reassurance); however, in this case, since all the data is of type varchar, the compiler used by PostgreSQL will automatically throw an error if the data cannot be converted. For example, when attempting to convert a column that contains characters and numbers (such as 'a7cdz0395') to an integer type.

<br>The following checks or variations of them are only applicable to strings (a sequence of characters) and can be used prior to converting the data, but  they will be even more useful in the next two sections when making modifications to the data.
```sql
select column_name
from table_name
--REGEXP_LIKE: function that checks whether a match of a POSIX regular expression pattern occurs within a string
--[[:digit:]] is a shorthand bracket expression used match characters of a string with any digit ranging from [0-9]
where regexp_like(column_name, '[[:digit:]]')
```
and
```sql
select column_name
from table_name
--[[:alpha:]] is a shorthand bracket expression used match characters of a string with any characters ranging from [a-z, A-Z] 
where regexp_like(column_name, '[[:alpha:]]')  
```

<br>Example 1: checking for digits
```sql
select country
from all_sessions
where regexp_like(column_name, '[[:alpha:]]')
```
![SQL-Project1-Pic3](https://github.com/DylJFern/lighthouse-labs-ds/assets/128000630/c34ec80a-4c8e-4942-8bf9-f6365431cf16)

Example 2: checking for digits
```sql
select country
from all_sessions
where regexp_like(column_name, '[[:digit:]]')
```
![SQL-Project1-Pic4](https://github.com/DylJFern/lighthouse-labs-ds/assets/128000630/bbf8f354-b2ad-478c-a7fc-7fc447a4f969)

In example 1, the result returned 15,134 rows which is the same as the total number of rows in the 'all_sessions' table. In example 2, the result returned 0 rows meaning the column 'country' does not have any records containing digits.
