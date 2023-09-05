What issues will you address by cleaning the data?  
1) Remove completely nulled out, irrelevant, and duplicate columns.
2) Match column names between tables.
4) Column formatting to more appropriate structure and data types for analysis.
5) Fix structural errors (e.g. rows in columns with instances of unrelated data or missing data), number (e.g. number of significiant digits) and string formatting (character capitalization and replacement),
6) Filter and remote duplicate records of unique identifiers
---
Queries: below, SQL queries needed to clean the data will be provded.
#
After importing the .csv files into the database, the initial analysis led to the inspection of columns that had only null values, irrelevant data, and duplicate columns.
### <br>Null Data  
#### In table: *all_sessions*  
The columns suspected to be completely nulled out were: 
- search_keyword  
- product_refund_amount
- item_quantity
- item_revenue
#### In table: *analytics*  
The column suspected to be completely nulled out was:  
- user_id    

<br>The following example query was used to investigate further,  
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
### <br>Duplicate Data
#### In table: *all_sessions* 
The column suspected of being a duplicate was:
- transaction_revenue
#### In table: *analytics*  
The column suspected of being a duplicate was:
- visit_start_time   

<br>The following example query was used to investigate the 'all_sessions' table further,
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
 
<br>In the "analytics" table the column 'visit_start_time' matches 'visit_id' row-by-row; however, instead of removing it, it is kept as another example of data cleaning to be subsequently discussed.

### <br>Irrelevant Data
#### In table: *analytics*  
The following column is not needed:
- social_engagement_type

<br>It does not provide any unique information for the analysis. All of the column's rows contain the string "Not Socially Engaged". This can be confirmed by running two queries, one for all the rows in the table and another specifying rows containing that string.
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

<br>For instance, join 'products' with 'sales_by_sku' utilzing the ON clause:
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

<br>To update the column name in the 'products' table we can use:
```sql
alter table products
--RENAME: give the column a new name
rename column sku to product_sku
```
#
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

### <br>Changing Data Types
  - Character type columns will remain as **VARCHAR(50)**, but if the column corresponds to a name, description, or some sort of directory (like a web address, or a folder structure) they will be converted to **TEXT**.
    - Either a VARCHAR (without a specified variable length) or TEXT data types could be used and according to [PostgreSQL documentation](https://www.postgresql.org/docs/8.1/datatype-character.html) there is no significant difference between them. But, when applying string functions like TRIM() it converts the character type to TEXT.
  - Numeric type columns will be represented as either **INTEGER** (e.g. id, time, number of transactions) or **NUMERIC** (revenue, costs, product price). Ideally, columns like 'ecommerce_action_type' and 'ecommerce_action_type' would be represented as a SMALLINT, and columns that involve currency like 'total_transaction_reveue' and 'product_price' would be represented by 'MONEY' type, but for simplicity were not considered.
  - Date/Time type columns will be represents by multiple types.
#### Example: Convert to Numeric
```sql
--full_visitor_id was cast to numeric since its value was too large was INTEGER (and even BIGINT)
alter table all_sessions
alter column full_visitor_id type numeric
--CAST(): convert a value of one data type into another
using cast(full_visitor_id as numeric)
--we can also use expression::type, e.g. using (full_visitor_id::numeric) 
```
The columns 'time' and 'time_on_site' were converted to integer types, in the next section we will see a format modification of it.
#### Example: Convert to Integer
```sql
alter table all_sessions
alter column time type integer 
using cast(time as integer)
```
#### Example: Convert to Date
```sql
alter table all_sessions
alter column date type date
using cast(date as date)
```
#### Example: Convert to Text
```sql
alter table all_sessions
alter column v2_product_name type text
using cast(v2_product_name as text)
```
or
```sql
--better alternate to the previous query as the imported data can have whitespace
alter table products
alter column name type text
--TRIM(): remove unwanted characters from a string; by default, it removes leading, trailing, or spaces (' ') on both sides
using trim(cast(name as text))
```
#### Example: Convert to Time
As [previously mentioned](cleaning_data.md/#Duplicate-Data) the 'visit_start_time' column was found to be a duplicate of 'visit_id', but instead of removing it we can convert it.
```sql
alter table analytics
alter column visit_start_time type time
using (time '00:00:00' + visit_start_time * interval '1 second') --multiply 'visit_start_time' by 1 second intervals and display it as 24-hour clock time 
```
For instance, using a simple example query
```sql
select (time '00:00:00' + 60 * interval '1 second') as time --where 60*('1 second' intervals) = 60 seconds or 1 minute
```
![SQL-Project1-Pic5](https://github.com/DylJFern/lighthouse-labs-ds/assets/128000630/5e07b7a7-d7e5-496d-b87b-04efb5e67d6f)
#
In this section we will handle issues that exist within the columns of the data such as unexpected or missing data, as well as the formatting of numbers and strings. We are going to be examining the 'all_sessions' table only (since it has the most columns), but the procedures shown can be applied to others.

### Strings
#### Consistent Length
For strings that do not have an equal amount of characters displayed across the rows, we can pad the string. In this example, we are going to use 'full_visitor_id' (a numeric type) and (temporarily) cast it to a string to apply a LPAD() function.

![SQL-Project1-Pic6](https://github.com/DylJFern/lighthouse-labs-ds/assets/128000630/8e13af68-5e71-4be7-a2d2-129ecb0640c0)

To apply LPAD() we need to determine the maximum length of 'full_visitor_id' as a string
```sql
--LENGTH(): return the number of characters that make up the string
--MAX(): an aggregate function that returns the maximum value in a set of values (e.g. column of 'full_visitor_id')
select max(length(full_visitor_id::varchar)) --convert full_visitor_id of numeric type to varchar type, obtain the number of characters in the string, and return the value of a string (in the column) with the largest length 
from all_sessions
```
or, we can write it as:
```sql
select max(length(cast(full_visitor_id as varchar)))
from all_sessions
```

Now, to display all 'full_visitor_id' rows with a length of 19 with the padding, we can use
```sql
--LPAD(string, length, fill): the "string" that should be padded on the left with "fill" to a "length"
--1) (temporarily) convert full_visitor_id of numeric type to a string length of 19; if it is less than 19 then it will had whitespace and if it is 19 it will do nothing
--2) fill the whitespace with 0's up to a string length of 19
--3) the result will be a column called 'full_visitor_id' of text type
select lpad(cast(full_visitor_id AS char(19)), 19, '0') as full_visitor_id
from all_sessions
```

![SQL-Project1-Pic7](https://github.com/DylJFern/lighthouse-labs-ds/assets/128000630/d7f0f7c5-3bdb-4543-9d76-9f16d99e4a42)

However, it should be known that this is a temporary column result based on the query, if you were to retrieve the 'full_visitor_id' column from the 'all_sessions' table, it will display it as numeric type without the string padding.

#### <br> Example 1: Unexpected Data
In the database, columns can contain data that is unexpected. For instance, when viewing the 'channel_grouping' column and performing the following query
```sql
select *
from all_sessions
--%: a wildcard; used to represent 0, 1, or many characters or numbers
--LIKE(): used to match text values against a pattern using wildcards
--ILIKE(): similar to LIKE() except its results include strings that are case-insensitive
where channel_grouping like '%(%)%'
```

![SQL-Project1-Pic8](https://github.com/DylJFern/lighthouse-labs-ds/assets/128000630/4fe53478-3696-4dfe-920e-8faa6dddd256)

We notice there are 5 rows displayed with '(Other)' and know that the total number of rows in the 'all_sessions' table is 15,134. Running the previous query with NOT clause in the WHERE statement alongside previously mentioned checkes returns 15,129 (= 15,134 - 5) rows which confirms there are only 5 rows of unexpected data. To remove them and update the column we can use
```sql
--UPDATE & SET: modify data in a table with new values that are specified
update all_sessions
--NULLIF(arg1, arg2): returns a null value if arg1 equals arg2, otherwirse it returns arg1
set channel_grouping = nullif(channel_grouping, '(Other)')
where channel_grouping like '%(%)%'
```
Re-running the query, we get the resulting output

![SQL-Project1-Pic9](https://github.com/DylJFern/lighthouse-labs-ds/assets/128000630/eb05aa21-870e-4d0b-b34d-7c649a1e4397)

#### <br> Example 2: Unexpected Data
Similar to 'channel_grouping', we can repeat the process for the column 'country'
```sql
select country
from all_sessions
where country like '%(%)%'
```
which returns 31 rows containing "(not set)", "Macedonia (FYROM)", and "Myanmar (Burma)"

![SQL-Project1-Pic10](https://github.com/DylJFern/lighthouse-labs-ds/assets/128000630/77845a09-736b-4696-898a-9d5cb5ce1b96)

and can update the table using a CASE statement 
```sql
update all_sessions
--CASE: like aa IF/ELSE statement in other programming languages; when a condition evalutes to false, the CASE expression evalutes the next condition from the top to bottom until it fines a condition that evaluates to true and returns the correspond result
set country =
  (case
     when country like ('(not set)') then null
     when country like ('Macedonia (FYROM)') then 'Macedonia'
	   when country like ('Myanmar (Burma)') then 'Myanmar'
     else country
  end)
```

#### <br> Example 3: Unexpected Data
From a quick analysis we notice names in the 'city' column begin with a uppercase character followed by a lowercase character. We can use the following to test our hypothesis (knowing the total row count is 15,134).
```sql
select city
from all_sessions
--similar to [[:alpha:]] and [[:digit:]], [[:upper:]] and [[:lower:]] search for a consecutive uppercase and lowercase character, respectively (e.g. if the uppercase character is the 3rd character in the string, the 4th character must be lowercase)
--the ^ is known as a POSIX regular expression anchor, this character ensures that the search only happens at the start of the string 
where regexp_like(city, '^[[:upper:]][[:lower:]]')
```

![SQL-Project1-Pic11](https://github.com/DylJFern/lighthouse-labs-ds/assets/128000630/0b56f134-c802-44ff-8143-8d9cf0b34d81)

returns 6,478 rows and using the NOT clause
```sql
select city
from all_sessions
where not regexp_like(city,'^[[:upper:]][[:lower:]]')
```

![SQL-Project1-Pic12](https://github.com/DylJFern/lighthouse-labs-ds/assets/128000630/042628e8-caa8-4bb2-a498-d54ae938d062)

returns 8,656 rows for a total of 15,134. And just to further confirm, we can determine the number results returned when including the previously determined data in our filter when using the WHERE clause.
```sql
select city
from all_sessions
where not (regexp_like(city, '^[[:upper:]][[:lower:]]') 
	or city in ('not available in demo dataset', '(not set)'))
```

![SQL-Project1-Pic13](https://github.com/DylJFern/lighthouse-labs-ds/assets/128000630/11474f1a-83f7-438a-9dab-bd0b6d568f24)

It returns 0 rows, and we can conclude our assessment was correct. We can update the table in a similar way used for the 'country' column.

<br>The importance of the anchor (^) character can be better seen when examining the 'v2_product_category' column. We can running a basic query search to get an idea of the format of the data present
```sql
select v2_product_category
from all_sessions
```

![SQL-Project1-Pic14](https://github.com/DylJFern/lighthouse-labs-ds/assets/128000630/2296a35d-c911-459a-bb6e-855aac620f4a)

and apply the REGEXP_LIKE() with the resulting output in a more condensed format.
```sql
--GROUP BY: divides the rows returned from the SELECT statement into groups
--COUNT(*): counts the number of rows in these newly formed groups
select v2_product_category , count(*)
from all_sessions
where not regexp_like(v2_product_category, '[[:upper:]][[:lower:]]')
group by v2_product_category
```

![SQL-Project1-Pic15](https://github.com/DylJFern/lighthouse-labs-ds/assets/128000630/d691edd7-370a-4d22-922a-baa092e33c74)

It appears the only unexpected data is "(not set)"; however, if we re-run the same query with the anchor character,
```sql
select v2_product_category , count(*)
from all_sessions
where not regexp_like(v2_product_category, '^[[:upper:]][[:lower:]]')
group by v2_product_category
```

![SQL-Project1-Pic16](https://github.com/DylJFern/lighthouse-labs-ds/assets/128000630/dd361f4f-7929-4ae5-a12e-7098fdfc6c93)

the result changes and now displays another row "${escCatTitle}" that is displayed 19 times in the 'v2_product_category' column. We immediately notice, it has the format of "...Ca..." which is an uppercase character followed by a lowercase character which is why it would not appear without the use of the anchor character.

#### <br>Character Case
