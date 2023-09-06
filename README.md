# Transforming and Analyzing Data with SQL

## Project/Goals
The objective of this project is to combine and practice SQL skills involved with extracting data from an SQL database, cleaning, transforming and analyzing data, loading data into a database, as well as developing and implementing a QA process to validate transformed data against raw data.

## Process
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
```
update all_sessions
set city =
  (case
    when country in ('(not set)', 'not available in demo dataset') then null 
    else city
  end)
```
to replace values containing those string of characters with "null" value.

### <br>Formulating and Answering Questions
Further insight was gained of the database while attempting to work through and solve the [questions provided](starting_with_questions.md), as well as attempting to generate and [formulate questions](starting_with_data.md) that were not asked, but could have been through the user of queries.

## <br>Results
(fill in what you discovered this data could tell you and how you used the data to answer those questions)

## Challenges 
(discuss challenges you faced in the project)

## Future Goals
(what would you do if you had more time?)
