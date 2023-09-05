Answer the following questions and provide the SQL queries used to find the answer.
    
**Question 1: Which cities and countries have the highest level of transaction revenues on the site?**

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
  from all_sessions
  where total_transaction_revenue is not null
)

select city, country, max(total_transaction_revenue_millions) as max_tts_millions
from all_sessions_clean_data
group by city, country
order by max_tts_millions desc
```

Answer:
SQL-Project1-Pic_ans1

**Question 2: What is the average number of products ordered from visitors in each city and country?**

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

Answer: 
SQL-Project1-Pic_ans2

**Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**

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

Answer:
SQL-Project1-Pic_ans3

We can apply additional filters and limits to examine different patterns, for example, 
```sql
...
having v2_product_category is not null
  and city is not null
  and v2_product_category like '%Apparel/%'
order by v2_product_category_count desc --or "asc" to see who least visits the site
limit 10
```
We notice that the "Apparel" product category is most visited by people from the United States and is least visited by prople from countries like Argentina and Australia.

**Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**

SQL Queries:
```

```

Answer:
SQL-Project1-Pic_ans4

**Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:
```

```

Answer:
SQL-Project1-Pic_ans5
