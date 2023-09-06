# Transforming and Analyzing Data with SQL

## Project/Goals
The objective of this project is to combine and practice SQL skills involved with extracting data from an SQL database, cleaning, transforming and analyzing data, loading data into a database, as well as developing and implementing a QA process to validate transformed data against raw data.

## <br>Process
### Create a new PostgreSQL database called "*ecommerce*"
Before we can start working with the data, we must import it into a database. To do this, we can create tables with the same name as our .csv files.
```sql
--create and specify the table name
create table sales_by_sku (
--define the column name, its data type and length, as well as any column constraints
product_sku varchar(50)
total_ordered varchar(50)
)
```
This query creates a table called 'sales_by_sku' with two columns 'product_sku' and 'total_ordered' both of type varchar(50). This procedure can be repeated in a similar manner for the remaining .csv files ('all_sessions', 'analytics', 'products', 'sales_report'). In the table creation process, all the column names were initially assigned the data type varchar(50) to prevent any data loss that may occur or errors assiociated with importing data to a particular format. For example, a column that is expected to contain only integers may have a row containing a combination of numbers and characters like 'abz3053zs' which would result in an error when attempting to import the data.

As previously mentioned, the table columns were all initialized to varchar(50) for consistency when importing the data. A problem with this approach that is worth mentioning is that the limiter (n = 50) allowed only 50 characters to be stored into the variable. While it was effective for a majority of the variables (column names), there were a few such as 'page_title' and 'v2_product_name' from the 'all_sessions table that threw an error due to the imported data being larger than the variable size. For the problem variables, a simple temporary fix would be to increase the amount it can store.
```sql
alter table all_sessions
alter column page_title type varchar(1000)
```

### <br>Understanding the Data
The data can be manipulated, modified, and interpreted in many different ways, it depends on what one is trying to achieve with it. This section will provide a brief overview, but a more in-depth look into the step-by-step procedural methoodlogies and processes used can be found in [QA.md](QA.md) and [cleaning_data.md](cleaning_data.md).

The data was first explored and inspected (and initial findings were recorded for reference later on in the process) to become familiar with the tables and their columns, what data they should contain, how can the data contained within them be better formatted, what data is unnecessary and can be removed, as well as how the tables and/or columns can relate.

### <br>QA Process and Data Cleaning
When working through the data these two processes went hand-in-hand. The information gained from queries used to perform inspection and identify risk areas could also be used and implemented into modifications of queries used to clean the data. For example, the data in the 'city' column was suspected to have an uppercase character at the start of the string followed by a subsequent lowercase character. The following query was used to find data that does not conform to those rules using the NOT clause.
```sql
select city
from all_sessions
where not regexp_like(city,'^[[:upper:]][[:lower:]]')
```
The result returns "(not set)" and "not available in demo dataset" strings which then can be implemented into a query
```sql
update all_sessions
set city =
  (case
    when country in ('(not set)', 'not available in demo dataset') then null 
    else city
  end)
```
to replace values containing those string of characters with "null" value.

### <br>Formulating and Answering Questions
Further insight was gained of the database while attempting to work through and solve the [questions provided](starting_with_questions.md), as well as attempting to generate and [formulate questions](starting_with_data.md) that were not asked, but could have been through the use of queries.

### <br> Setting Up PKs and FKs for ERD 
Up to this point columns with null, irrelevant and duplicate data were deleted, validation checks were made and utilized accordingly to modify queries to clean the data, and data was converted to more appropriate types. 

In addition, a query to filter and remove duplicate records of unique identifieres was introduced; however, it was not applied directly to the database due to the fact it may or may have not caused a loss of potentially useful data. For instance, in the 'analytics' table, the columns 'visit_number, 'visit_id', 'visit_start_time', 'date', 'full_visitor_id' shared duplicate values but 'unit_price' tended to vary.

With all this information in mind, we can now assign primary and foreign keys to the tables. Primary keys and foreign keys can be made in table creation or done after, our case is the latter.
```sql
alter table products
add primary key (product_sku)
```
Simarly, we can add a foreign key constraint to the primary key in the 'product_sku' column of the 'products' table.
```sql
alter table sales_report 
add constraint sr_fk_product_sku
foreign key (product_sku) 
references products (product_sku)
```
We also notice that tables 'sales_by_sku' and 'all_sessions' both have a column 'product_sku' (with distinct rows, in other words no duplicates); however, when attempting to add the foreign key constraint to 'sales_by_sku' we receive an error.

*ERROR:  Key (product_sku)=(GGOEGEVB070699) is not present in table "products".insert or update on table "all_sessions" violates foreign key constraint "s_fk_product_sku"*

*ERROR:  insert or update on table "all_sessions" violates foreign key constraint "s_fk_product_sku"
SQL state: 23503
Detail: Key (product_sku)=(GGOEGEVB070699) is not present in table "products".*

When we run the following query joining the 'products' and 'sales_by_sku' tables,
```sql
select p.product_sku, sbs.product_sku, sbs.total_ordered
from products as p
full outer join sales_by_sku as sbs
  on sbs.product_sku = p.product_sku
where p.product_sku is null
```
we realize that the 'sales_by_sku' table contains obsolete product sku's no longer used in the 'products' table. Yet, it also contains a column with non-null values for some records.

![SQL-Project1-Pic23_error2](https://github.com/DylJFern/lighthouse-labs-ds/assets/128000630/0c9a836d-1a71-47d6-9020-4ff44974f4ee).

Likewise, the 'all_sessions' table shares similar problems between it and the 'products' table, as well it and the 'analytics' table. Based on this and our want to preserve the duplicate rows of data we are left with the resulting [ERD schema](schema.png).

## <br>Results
### Data Analysis
Our initial ERD schema consisted of only varchar(n) of specified length n.

![Initial-ERD-varchar](https://github.com/DylJFern/lighthouse-labs-ds/assets/128000630/6c02bf3d-539f-4dc9-a1b6-9925143c5ed0)

Through the process of data transformation, development of methodology for checking, filtering, converting and cleaning the data, as well as interpretation of how the data could be used, we can rework the schema as follows.

![schema](https://github.com/DylJFern/lighthouse-labs-ds/assets/128000630/67161884-f4e3-464d-900a-54e8c5ca2089)

### <br>Data Insights
  - The purchases and types of purchases made based on geographical locations. From the data it could be seen that the United States was the top purchasing country, producing more than half of site visits and sales compare to all the other countries combine. The questions that can then be posed are whether the advertisements cater more towards the Western cultures, what kind and brand of apparel is more popular and how does it vary with gender, and so on.
  - Site visits and behaviour can be observed through the joining of 'analytics' and 'all_sessions' tables. For instance, it can be helpful to determine how often, how long, and the number of times visitor's access the site because it provides information as to what aspects can be improved (if any); aspects such navigation, visual design, content, and information accessibility.
  
## <br>Challenges 
  -  Time constraint: unable to fully explore, analyze, and clean the data.
  -  No clear direction on how to deal with duplicate rows as it was more or less up to interpretation.
      - Affected how primary and foreign keys were set up.
  -  Developing procedures not just specific to a particular problem, but one that serves as a general use case and can be applied for almost all the data.

## <br>Future Considerations
  - Choosing more appropriate data types to better reflect the column.
  - Further explore the tables for any issues (e.g. in the 'all_sessions' table, one of the cities listed for the country Canada was "New York").
      - Clean the database more thoroughly.
      - Develop a general case procedure (e.g. more advanced queries) for checking formatting for instance, as opposed to ones specific to particular problem.
  - Find resolution to whether it is better to keep or remove the duplicate rows.
  - Gain additional insights into other tables (as most of the analysis was done between the 'all_sessions' and 'products' table).
