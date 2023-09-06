What issues will you address by cleaning the data?  
1) Remove completely nulled out, irrelevant, and duplicate columns.
2) Match column names between tables.
4) Column formatting to more appropriate structure and data types for analysis.
5) Fix structural errors (e.g. rows in columns with instances of unrelated data or missing data), number (e.g. number of significiant digits) and string formatting (character capitalization and replacement),
6) Filter and remote duplicate records of unique identifiers

<br>The SQL queries needed to clean the data can be seen below.

---
### Null Data  
The columns that contain only null data can be deleted using the query for the 'all_sessions' table,
```sql
--ALTER: add, remove, or modify columns in an existing table
alter table all_sessions
--DROP: delete/remove (e.g. existing tables or columns based on specifier)
drop column search_keyword,  
drop column product_refund_amount,  
drop column item_quantity,  
drop column item_revenue  
``` 
and for the 'analytics' table.
```sql  
alter table analytics  
drop column user_id
```

<br>Null data at times can be important as it helps identify areas that could be improved or reconsidered. For example, when conducting a survery the respondent(s) may leave fields as empty (or null), this can lead to questions as to why (e.g. was the terminology confusing) and of the total number of respondents how many left the field empty. However, our situation is different as the columns do not provide any helpful insight into the data.

### <br>Duplicate Data
Similarly to the previous queries, we can delete the duplicate column.
```sql
alter table all_sessions
drop column transaction_revenue
```
Note: all the data was initialized as varchar to prevent data loss and will later be converted [later on](cleaning_data.md/#Irrelevant-Data).
 
<br>In the 'analytics' table, the column 'visit_start_time' matches 'visit_id' row-by-row; however, instead of removing it, we will keep it to showcase a different example of data cleaning (to be subsequently discussed).

### <br>Irrelevant Data
Similarly to the previous queries, we can delete the irrelevant data column since it is common to all rows.
```sql
alter table analytics
drop column social_engagement_type
```
---
## Column Naming Convention
By changing the column headers we can help improve visual clarity, and although it might not be considered part of the data cleaning process, matching corresponding column names helps avoids any confusion, helps make performing joins easier, makes potential FKs or PKs more visible, and so on.

<br>To update the column name in the 'products' table a query to alter the table can be used.
```sql
alter table products
--RENAME: give the column a new name
rename column sku to product_sku
```
---

### Converting Data Types
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

#### <br>Consistent Format
Just like padding that can be used for consistent formatting of string length, we can use functions like TRANSLATE() and REGEXP_REPLACE() to format the text output of a given column.
```sql
--TRANSLATE(string, from, to): replace a set of characters (from) with a new set of characters (to)
select translate(v2_product_category, '/', '>') as v2_product_category_new,
  v2_product_category as v2_product_category_original
from all_sessions
```

![SQL-Project1-Pic20](https://github.com/DylJFern/lighthouse-labs-ds/assets/128000630/338e9e41-648f-441b-8d08-a0b319296436)


#### <br> Unexpected Data: Example 1
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

<br>On the other hand, we may have records that already contain a null value, but would like to replace them with more appropriate text. In particular, the 'ecommerce_action_option' column consists of "null", "Review", "Billing and Shipping", and "Payment". Although, in this scenario it may make more sense to replace "null" records with "N/A" based on the context of the column.
```sql
update all_sessions
--COALESCE(arg1, arg2, ...): accepts an unlimited number of arguments; it evalutes arguments from left to right (arg2, ...) until it finds the first non-null argument (otherwise returns null), remaining arguments are not evaluated
set ecommerce_action_option = coalesce(ecommerce_action_option, 'N/A')
```

#### <br> Unexpected Data: Example 2
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

#### <br> Unexpected Data: Example 3
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
The characters of a string can be manipulated in unique ways, especially if one were to implement POSIX regular expressions. Nevertheless, to keep things simple we will only explore three PostgreSQL functions.

All the rows in the 'type' column contain only uppercase characters.

![SQL-Project1-Pic17](https://github.com/DylJFern/lighthouse-labs-ds/assets/128000630/8b0ca571-0ec7-4799-b16d-7e158f8f7ebd)

The uppercase characters can all be changed to lowercase using,
```sql
select lower(type)
from all_sessions
```
and the table can be updated using,
```sql
update all_sessions
set type = lower(type)
```
but this will change its data type to text. This can be counteracted by casting it back to varchar(50) if one wanted to.
```sql
update all_sessions
set type = cast(lower(type) as varchar(50))
```

<br>Simarly, we can change all the characters in a string to uppercase with UPPER().

<br>Another alternative would be to use INITCAP() to convert the first letter of each word in a string expression to uppercase with the remaining characters in lowercase.
```sql
select initcap(v2_product_name) as v2_product_name_new, v2_product_name as v2_product_name_original
from all_sessions
```
This can be problematic without proper consideration of use, for example, "Google 22 oz Water Bottle" becomes "Google 22 Oz Water Bottle" but "Android Men’s Zip Hoodie" becomes "Android Men’S Zip Hoodie" with a capital "S" following the apostrophe.

![SQL-Project1-Pic18](https://github.com/DylJFern/lighthouse-labs-ds/assets/128000630/c58317c7-a44b-4f7a-862b-8531898c2a70)

### <br>Numbers
#### Formatting (Example 1)
We can format the column 'time' of integer type as hh:mm:ss (hours:minutes:seconds) using the following query,
```sql
--TO_CHAR(): converts a time stamp to string according to the given format (specified as 'hh:mm:ss AM' where "AM" is used to repesent it as a 12-hour clock format)
--TO_TIMESTAMP(): converts a number (e.g. time of interger type) or a string to a timestamp (by default, it is represented by "YYYY-MM-DD hh:mm:ss-(time_zone)" format
select to_char(to_timestamp(time), 'hh:mm:ss AM') as time
from all_sessions
```
which will convert the data type to text. For this specific analysis, 'time' and 'time_on_site' were left as an integer type (as previously mentioned) except for the 'visit_start_time' column in the 'analytics' table.

#### <br>Formatting (Example 2)
The columns that contain costs, revenue, sales, etc. can be formatted to display a certain number of significant digits. For instance 'product_price' can be divided by 1,000,000 to represent 'product_price' in millions, this data can also be either truncated or cast to a data type such as numeric that allows control of precision and scale.
```sql
--TRUNC(num, precision): returns a number (num) that is truncated to a specified amount of decimal places (precision)
select trunc(product_price/1000000, 2) as product_price_millions,
--NUMERIC(precision, scale): where the precision is the total count of significant digits in the whole number (on both sides of the decimal point) and scale is the count of decimal digits (to the right of the decimal point)
  (product_price/1000)::numeric(10,4) as product_price_thousands
from all_sessions
```
The ROUND(source, n) function can be applied to the output, where "n" determines the number of decimal places after round.

#### <br>Unexpected Data
Similar to strings, numbers can have data records with null value. For example, columns 'total_transaction_revenue', 'transactions', and 'page_views should only contain rows of real numbers.
```sql
update all_sessions
set total_transaction_revenue = coalesce(total_transaction_revenue, 0)
```
