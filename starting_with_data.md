Question 1: find the total number of unique visitors.

SQL Queries: 
```sql
select count(distinct(full_visitor_id)) as unique_visitors
from all_sessions
```
#### Answer: 

![SQL-Project1-Pic_ans6](https://github.com/DylJFern/lighthouse-labs-ds/assets/128000630/f40352fd-29cf-443e-b372-09523433adfa)

<br>Question 2: find the number of times the visitors from the United States viewed the site more than twice after the year 2016.

SQL Queries:
```sql
select full_visitor_id, country, date, count(*)
from all_sessions
where extract(year from date) > 2016 and country = 'United States'
group by full_visitor_id, country, date
having count(*) > 2
order by date
```

#### Answer:

![SQL-Project1-Pic_ans7](https://github.com/DylJFern/lighthouse-labs-ds/assets/128000630/6612fa2e-22bc-49c2-bb0b-3066454bf40b)

<br>Question 3: determine the previously ordered and sold products that are not considered obsolete.

SQL Queries:
```sql
select p.product_sku as product_sku_current, sbs.product_sku as product_sku_obsolete, sbs.total_ordered
from products as p
full outer join sales_by_sku as sbs
  on sbs.product_sku = p.product_sku
where p.product_sku is null and sbs.total_ordered != 0
```
or
```sql
select p.product_sku as product_sku_current, sbs.product_sku as product_sku_obsolete, sbs.total_ordered
from sales_by_sku as sbs
left join products as p
  on sbs.product_sku = p.product_sku
where p.product_sku is null and sbs.total_ordered != 0
```

#### Answer:

![SQL-Project1-Pic_ans8](https://github.com/DylJFern/lighthouse-labs-ds/assets/128000630/cecab3d1-9e8e-41ac-9c17-9f62c08347ff)

<br>Question 4: 

SQL Queries:

#### Answer:

<br> Question 5: 

SQL Queries:

#### Answer:
