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
The data was initialized as varchar to prevent any data loss, before converting the data to more appropriate types (e.g. relevant to column header and row data). There is a simple data validation check that can be performed for reassurance (which can also be used as a general use case for almost all situations). However, keep in mind that the compiler used by PostgreSQL will automatically throw an error if the data cannot be converted. For example, when attempting to convert a column that contains characters and numbers (such as 'a7cdz0395') to an integer type.

<br>The following checks (and any variations of them) are only applicable to strings (a sequence of characters) and can be used prior to converting the data, but they may prove more useful in later stages of data cleaning (e.g. when making modifications to the data itself). 
```sql
select column_name
from table_name
--REGEXP_LIKE: function that checks whether a match of a POSIX regular expression pattern occurs within a string
--[[:digit:]] is a shorthand bracket expression used match characters of a string with any digit ranging from [0-9]
where regexp_like(column_name, '[[:digit:]]')
```
```sql
select column_name
from table_name
--[[:alpha:]] is a shorthand bracket expression used match characters of a string with any characters ranging from [a-z, A-Z] 
where regexp_like(column_name, '[[:alpha:]]')  
```

<br>Example 1: checking for characters
```sql
select country
from all_sessions
where regexp_like(country, '[[:alpha:]]')
```

![SQL-Project1-Pic_new3](https://github.com/DylJFern/lighthouse-labs-ds/assets/128000630/ebf26a14-ef5a-4114-99b7-fc546974f874)

In example 1, the result returned 15,134 rows which is the same as the total number of rows in the 'all_sessions' table.

<br>Example 2: checking for digits
```sql
select country
from all_sessions
where regexp_like(country, '[[:digit:]]')
```

![SQL-Project1-Pic_new4](https://github.com/DylJFern/lighthouse-labs-ds/assets/128000630/15d02cd9-0158-4477-a472-aa6e00e51e61)

In example 2, the result returned 0 rows meaning the column 'country' does not have any rows containing digits.

---
## Format Specific Check
Based on the format, you may be required to perform a specific check. They are cruical and the query written should be done on a case-by-case basis. Without these specific checks, you may not get the correct data you are looking for or may query data that contains a combination of correct and incorrect data, and with larger databases containing thousands of rows of data in each column the query time can increase.
### Strings
#### Consistent Length
For strings that do not have an equal amount of characters displayed across the rows, we can pad the string. In this example, we are going to use 'full_visitor_id' (a numeric type) and (temporarily) cast it to a string to apply a LPAD() function.

![SQL-Project1-Pic6](https://github.com/DylJFern/lighthouse-labs-ds/assets/128000630/8e13af68-5e71-4be7-a2d2-129ecb0640c0)

To apply LPAD() we need to determine the maximum length of 'full_visitor_id' as a string.
```sql
--LENGTH(): return the number of characters that make up the string
--MAX(): an aggregate function that returns the maximum value in a set of values (e.g. column of 'full_visitor_id')
select max(length(full_visitor_id::varchar)) --convert full_visitor_id of numeric type to varchar type, obtain the number of characters in the string, and return the value of a string (in the column) with the largest length 
from all_sessions
```
Alternatively, it can be converted using the built-in CAST() function.
```sql
select max(length(cast(full_visitor_id as varchar)))
from all_sessions
```

Now, we can display all 'full_visitor_id' rows with a length of 19 with the padding.
```sql
--LPAD(string, length, fill): the "string" that should be padded on the left with "fill" to a "length"
--1) (temporarily) convert full_visitor_id of numeric type to a string with a length of 19; if it is less than 19 then the function will add whitespace, if it is less than 19 then the function will do nothing
--2) fill the whitespace with 0's up to a string length of 19
--3) the result will be a column called 'full_visitor_id' of text type
select lpad(cast(full_visitor_id AS char(19)), 19, '0') as full_visitor_id
from all_sessions
```

![SQL-Project1-Pic7](https://github.com/DylJFern/lighthouse-labs-ds/assets/128000630/d7f0f7c5-3bdb-4543-9d76-9f16d99e4a42)

However, as mentioned the conversion is performed to produce a temporary column result. If you were to run a query to retrieve the 'full_visitor_id' column from the 'all_sessions' table (after performing the LPAD() formatting), the display will be the original 'full_visitor_id' column with numeric type and without the string padding. An example showcasing a permanent result can be seen in the [cleaning_data.md](cleaning_data.md/#hConsistent-Length:-Example-1) file.

#### <br>Consistent Format
Similar to how padding can be used for formatting a consistent string length, the TRANSLATE() and REGEXP_REPLACE() functions can be used to format the text output of a given column.
```sql
--TRANSLATE(string, from, to): replace a set of characters (from) with a new set of characters (to)
select translate(v2_product_category, '/', '>') as v2_product_category_new,
  v2_product_category as v2_product_category_original
from all_sessions
```

![SQL-Project1-Pic20](https://github.com/DylJFern/lighthouse-labs-ds/assets/128000630/338e9e41-648f-441b-8d08-a0b319296436)

#### <br>Unexpected Data (Part 1)
In the database, columns can contain data that is unexpected. For instance, when viewing the 'channel_grouping' column.
```sql
select *
from all_sessions
--%: a wildcard; used to represent 0, 1, or many characters or numbers
--LIKE(): used to match text values against a pattern using wildcards
--ILIKE(): similar to LIKE() except its results include strings that are case-insensitive
where channel_grouping like '%(%)%' --or similar variations "...like '%(%'"
```

![SQL-Project1-Pic_new8](https://github.com/DylJFern/lighthouse-labs-ds/assets/128000630/ef76e088-05b9-40a8-bdcb-f0cfa0ac3a7e)

There are 5 rows displayed with "(Other)" in the 'channel_grouping' column. We know that the total number number of rows in the 'all_sessions' table is 15,134. Running the previous query with NOT clause in the WHERE statement returns 15,129 (= 15,134 - 5) rows which confirms there are only 5 rows of unexpected data. 

#### <br>Unexpected Data (Part 2)
From a quick analysis we also notice the names in the 'city' column begin with a uppercase character followed by a lowercase character. We can use the following to test our hypothesis (knowing the total row count is 15,134).
```sql
select city
from all_sessions
--similar to [[:alpha:]] and [[:digit:]], [[:upper:]] and [[:lower:]] search for a consecutive uppercase and lowercase character, respectively (e.g. if the uppercase character is the 3rd character in the string, the 4th character must be lowercase)
--the ^ is known as a POSIX regular expression anchor, this character ensures that the search only happens at the start of the string 
where regexp_like(city, '^[[:upper:]][[:lower:]]')
```

![SQL-Project1-Pic11](https://github.com/DylJFern/lighthouse-labs-ds/assets/128000630/0b56f134-c802-44ff-8143-8d9cf0b34d81)

The query returns 6,478 rows and using the NOT clause,
```sql
select city
from all_sessions
where not regexp_like(city,'^[[:upper:]][[:lower:]]')
```

![SQL-Project1-Pic12](https://github.com/DylJFern/lighthouse-labs-ds/assets/128000630/042628e8-caa8-4bb2-a498-d54ae938d062)

it returns 8,656 rows for a total of 15,134. And just to further confirm, we can determine the number results returned when including the previously determined data in our filter when using the WHERE clause.
```sql
select city
from all_sessions
where not (regexp_like(city, '^[[:upper:]][[:lower:]]') 
	or city in ('not available in demo dataset', '(not set)'))
```

![SQL-Project1-Pic13](https://github.com/DylJFern/lighthouse-labs-ds/assets/128000630/11474f1a-83f7-438a-9dab-bd0b6d568f24)

The result is 0 rows, as such we can conclude our assessment was correct. We can update the table in a similar way used for the 'country' column.

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

the result changes and now displays another row "${escCatTitle}" that is displayed 19 times in the 'v2_product_category' column. We immediately notice, it has the format of "...Ca..." which consists of an uppercase character followed by a lowercase character and is the reason why it would not appear without the use of the anchor character.

#### <br>Character Case
When using functions it is important to be mindful that the desired output is often not always the expected output. For instance, INITCAP() is function used to covert the first letter of each word in a string to uppercase. However, we examining the affect of it on the 'v2_product_name' column we can see that
```sql
select initcap(v2_product_name) as v2_product_name_new, v2_product_name as v2_product_name_original
from all_sessions
```
it becomes problematic when proper consideration of use is not taken into account. The string "Google 22 oz Water Bottle" becomes "Google 22 Oz Water Bottle" but "Android Men’s Zip Hoodie" becomes "Android Men’S Zip Hoodie" with a capital "S" following the apostrophe.

![SQL-Project1-Pic18](https://github.com/DylJFern/lighthouse-labs-ds/assets/128000630/c58317c7-a44b-4f7a-862b-8531898c2a70)
