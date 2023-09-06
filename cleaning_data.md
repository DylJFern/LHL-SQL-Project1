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
## Converting Data Types
  - Character type columns will remain as **VARCHAR(50)**, but if the column corresponds to a name, description, or some sort of directory (like a web address or a folder structure) it will be converted to **TEXT**.
    - Either VARCHAR (without a specified variable length) or TEXT data types could be used and according to [PostgreSQL documentation](https://www.postgresql.org/docs/8.1/datatype-character.html) there is no significant difference between them. But, when applying string functions like TRIM() the data will convert the CHARACTER type to TEXT type.
  - Numeric type columns will be represented as either **INTEGER** (e.g. id, time, and number of transactions) or **NUMERIC** (revenue, costs, and product price). Ideally, columns like 'ecommerce_action_type' and 'ecommerce_action_type' would be represented as a SMALLINT, and columns that deal with currency like 'total_transaction_reveue' and 'product_price' would be represented by MONEY type, but for simplicity's sake they were not considered.
  - Date/Time type columns will be represented by multiple types.

### <br>Example: Convert to Numeric
```sql
--full_visitor_id was cast to numeric since its value was too large for INTEGER (and even BIGINT)
alter table all_sessions
alter column full_visitor_id type numeric
--CAST(): convert a value of one data type to another
using cast(full_visitor_id as numeric)
--we can also use expression::type, such as full_visitor_id::numeric
```
The columns 'time' and 'time_on_site' were converted to integer types, in the next section we will see a format modification that can be performed on them.

### <br>Example: Convert to Integer
```sql
alter table all_sessions
alter column time type integer 
using cast(time as integer)
```

### <br>Example: Convert to Date
```sql
alter table all_sessions
alter column date type date
using cast(date as date)
```

### <br>Example: Convert to Text
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

### <br>Example: Convert to Time
As [previously mentioned](cleaning_data.md/#Duplicate-Data) the 'visit_start_time' column was found to be a duplicate of 'visit_id', but instead of removing it we can convert it.
```sql
alter table analytics
alter column visit_start_time type time
using (time '00:00:00' + visit_start_time * interval '1 second') --multiply 'visit_start_time' by 1 second intervals and display it as 24-hour clock time 
```
For instance, we can write a simple query showcasing this.
```sql
select (time '00:00:00' + 60 * interval '1 second') as time --where 60*('1 second' intervals) = 60 seconds or 1 minute
```
![SQL-Project1-Pic5](https://github.com/DylJFern/lighthouse-labs-ds/assets/128000630/5e07b7a7-d7e5-496d-b87b-04efb5e67d6f)

---
## String and Number Format Specific Data Cleaning Techniques
In this section we will handle issues that exist within the columns of the data such as unexpected or missing data, as well as the formatting of numbers and strings. We are going to be examining the 'all_sessions' table only (since it has the most columns), but the procedures shown can be applied to others.

### Strings
#### Consistent Length: Example 1
The 'product_sku' records in the 'sales_by_sku' table do not have consistent string lengths.
```sql
select *
from sales_by_sku
order by total_ordered desc
```

![SQL-Project1-Pic_new6](https://github.com/DylJFern/lighthouse-labs-ds/assets/128000630/5d5b864d-6bc9-434e-a972-c5c541036a32)

This can be fixed and updated using the following query,
```sql
update sales_by_sku
set product_sku = lpad(cast(product_sku AS char(14)), 14, '0')
```
and re-running the initial query produces and updated output. The process for determining the equation set to the 'product_sku' column can be found in [QA.md](QA.md/#Consistent-Length).

![SQL-Project1-Pic_new7](https://github.com/DylJFern/lighthouse-labs-ds/assets/128000630/b3a9c9f7-1074-4eb3-9351-9140cc461ccd)

#### <br>Consistent Format: Example 1
Based on the [example provided](QA.md/#Consistent-Format), we can also update the column 'v2_product_category'.
```sql
update all_sessions
set v2_product_category = translate(v2_product_category, '/', '>')
```

#### <br> Unexpected Data: Example 1
There are 5 rows displayed with "(Other)" in the 'channel_grouping' column of the 'all_sessions' table which can be removed and updated.
```sql
--UPDATE & SET: modify data in a table with new values that are specified
update all_sessions
--NULLIF(arg1, arg2): returns a null value if arg1 equals arg2, otherwirse it returns arg1
set channel_grouping = nullif(channel_grouping, '(Other)')
where channel_grouping like '%(%)%'
```
The resulting column now displays "null" in place of "(Other)" for those 5 rows of the total 15,134.

![SQL-Project1-Pic_new9](https://github.com/DylJFern/lighthouse-labs-ds/assets/128000630/877e122a-4801-47e8-9f29-ef1fa8ab7c11)

<br>On the other hand, we may have records that already contain a null value, but would like to replace them with more appropriate text. In particular, the 'ecommerce_action_option' column consists of "null", "Review", "Billing and Shipping", and "Payment". In this case, we would like to replace "null" with "N/A" based on the context of the column header.
```sql
update all_sessions
--COALESCE(arg1, arg2, ...): accepts an unlimited number of arguments; it evalutes arguments from left to right (arg2, ...) until it finds the first non-null argument (otherwise returns null), remaining arguments are not evaluated
set ecommerce_action_option = coalesce(ecommerce_action_option, 'N/A')
```

#### <br> Unexpected Data: Example 2
Similar to 'channel_grouping', we can repeat the process for the column 'country'.
```sql
select country
from all_sessions
where country like '(%)'
```
The query returns 31 rows containing a combination of "(not set)", "Macedonia (FYROM)", and "Myanmar (Burma)".

![SQL-Project1-Pic_new10](https://github.com/DylJFern/lighthouse-labs-ds/assets/128000630/5df3bfd4-e511-4e28-8e25-a58538b8b787)

We can update the table using a CASE statement.
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
In the [QA.md](QA.md/#Unexpected-Data-(Part-2)) we walkthrough an example in which we implement the use of a REGEXP_LIKE() function using the POSIX regular expression anchor. It allows us to uncover unexpected data in many ways just through different manipulations and variations of the function patterns and flags.
```sql
--REGEXP_LIKE(string, pattern, flags)
regexp_like(city, '^[[:upper:]][[:lower:]]')
```
Using the procedure, we were able to discover the two strings "(not set)" and "${escCatTitle}", and to remove them we can use a similar CASE statement but with the IN condition instead of LIKE.
```sql
update all_sessions
set v2_product_category =
  (case
--expression IN(value1, value2,....): test the expression against the values, if they match then the condition evalutes to true (e.g. "null" in this case)
    when country in ('(not set)', '${escCatTitle}') then null 
    else v2_product_category
  end)
```

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

<br>Another alternative would be to use INITCAP() to convert the first letter of each word in a string expression to uppercase with the remaining characters in lowercase (a test case can be seen in [QA.md](QA.md/#Character-Case)).

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
