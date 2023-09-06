Answer the following questions and provide the SQL queries used to find the answer.

---
**Question 1: Which cities and countries have the highest level of transaction revenues on the site?**

SQL Queries: In this query, it is assumed no data cleaning has been done prior.
```sql
with all_sessions_clean_data as (
  select
    case
      when city in ('(not set)', 'not available in demo dataset') then null
      else city
    end as city,

    case
      when country in ('(not set)', 'not available in demo dataset') then null
      else country
    end as country,

    round(trunc(total_transaction_revenue/1000000, 3), 1) as total_transaction_revenue_millions
  from all_sessions
  where total_transaction_revenue is not null
)

select city, country, max(total_transaction_revenue_millions) as max_transaction_revenue_millions
from all_sessions_clean_data
group by city, country
having country is not null
order by max_transaction_revenue_millions desc
```

#### Answer:   
The city and its corresponding country will have its highest level of transaction revenue where 'total_transaction_revenue' is not null (returns 81 rows), then using a GROUP BY clause we can then determine of those grouped instances which city and its corresponding country have the maximum (highest) transaction revenue.

The CTE helps "null" out unexpected data in the city and country column and the combined function that implements ROUND() and TRUNC() is used to change the displayed output.

The HAVING clause also attempts to filter data for situations where 'country' is "null" (does not affect the resulting row count returned). 

![SQL-Project1-Pic_ans1](https://github.com/DylJFern/lighthouse-labs-ds/assets/128000630/9e723c4a-d041-4898-8805-ab885b6fd0c4)

**<br>Question 2: What is the average number of products ordered from visitors in each city and country?**

SQL Queries: In this query, we assume no data cleaning has been done prior.
```sql
select
  case
    when s.city in ('(not set)', 'not available in demo dataset') then null
    else s.city
  end as city,

  case
    when s.country in ('(not set)', 'not available in demo dataset') then null
    else s.country
  end as country,
	
  round(avg(p.ordered_quantity), 3) as average_product_quantity	
from all_sessions as s
inner join products as p
  on s.product_sku = p.product_sku
where s.country is not null --assumption: the city can be unknown, but the country cannot; if filter not used, query would return a total of 428 rows
group by s.country, s.city
order by average_product_quantity desc
```

#### Answer:  
In each city and its corresponding country, the average number of products from visitors can be found by performing an inner join between 'all_sessions' table and 'products' table using the 'product_sku' column as the join specifier. Using the aggregate function AVG() we can calculate the average product quantity for each country and city grouping. Again, we can apply a CTE to replace unexpected data with "null" and apply the ROUND() function to the average value of the ordered quantity.

![SQL-Project1-Pic_ans2](https://github.com/DylJFern/lighthouse-labs-ds/assets/128000630/21dad69b-e4e9-48fa-9494-14951525a3a8)

**<br>Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**

SQL Queries: In this query, we assume no data cleaning has been done prior.
```sql
with all_sessions_clean_data as (
  select
    case
      when city in ('(not set)', 'not available in demo dataset') then null
      else city
    end as city,

    case
      when country in ('(not set)', 'not available in demo dataset') then null
      else country
    end as country,

    case
      when v2_product_category in ('(not set)', '${escCatTitle}') then null
      else v2_product_category
    end as v2_product_category
  from all_sessions
)

select city, country, v2_product_category, count(v2_product_category) as v2_product_category_count
from all_sessions_clean_data
group by country, city, v2_product_category
having v2_product_category is not null
order by v2_product_category_count desc
```

#### Answer:
We can also apply additional filters and limits to examine different patterns, for example, 
```sql
...
having v2_product_category is not null
  and city is not null
  and v2_product_category like '%Apparel/%'
order by v2_product_category_count desc --or "asc" to see who least visits the site
limit 10
```
Based on the results output by the query, we notice that the "Apparel" product category is most visited by people from the United States and is least visited by prople from countries like Argentina and Australia.

![SQL-Project1-Pic_ans3 1](https://github.com/DylJFern/lighthouse-labs-ds/assets/128000630/8128e71e-01a3-408d-953f-1264da90196a)

**<br>Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**

SQL Queries: In this query, we assume no data cleaning has been done prior.
```sql
with all_sessions_clean_data as (
  select
    case
      when city in ('(not set)', 'not available in demo dataset') then null
      else city
    end as city,

    case
      when country in ('(not set)', 'not available in demo dataset') then null
      else country
    end as country,	
	
    s.v2_product_name,
	
    rank() over (partition by s.v2_product_name order by sum(p.ordered_quantity) desc) as ranking	
  from all_sessions as s
  join products as p
    on s.product_sku = p.product_sku
  group by s.v2_product_name, s.country, s.city
)

select *
from all_sessions_clean_data
where ranking = 1
```

#### Answer:
Of the 985 rows, the United States ordered 536 top-selling products, they are responsible for influencing 54% of which products are considered top-selling (accounting for majority of the sales). And of those 536 rows, 56% (302 rows) are missing 'city' data.

![SQL-Project1-Pic_ans4](https://github.com/DylJFern/lighthouse-labs-ds/assets/128000630/7911a8ce-952b-4299-b38f-2553f2cb6240)

**<br>Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries: In this query, we assume no data cleaning has been done prior.
```sql
with all_sessions_clean_data as (
  select
    case
      when city in ('(not set)', 'not available in demo dataset') then null
      else city
    end as city,

    case
      when country in ('(not set)', 'not available in demo dataset') then null
      else country
    end as country,
	
    round(trunc(total_transaction_revenue/1000000, 3), 1) as total_transaction_revenue_millions
  from all_sessions as s
	where total_transaction_revenue is not null
)

select city, country, sum(total_transaction_revenue_millions) as total_revenue_generated_millions
from all_sessions_clean_data
group by city, country
order by total_revenue_generated_millions desc
```

#### Answer:
We can summarize the impact of the revenue generated from each city and its corresponding country. The results indicate that it is predominantly generated by the United States (16 of the 21 rows).

![SQL-Project1-Pic_ans5](https://github.com/DylJFern/lighthouse-labs-ds/assets/128000630/a331cf20-bf29-407f-b9bb-707e6257dd04)
