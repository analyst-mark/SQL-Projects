
 

Each of the following case study questions can be answered using a single SQL statement:

1. What is the total amount each customer spent at the restaurant?

```sql
   SELECT customer_id, sum(price) total
   from sales s
   join menu m
   using (product_id)
   group by customer_id
   order by 2 desc
   

```

![](src/Pasted%20image%2020230607115445.png)

1. How many days has each customer visited the restaurant?

```sql
SELECT customer_id, count(distinct order_date) total_visits
from sales
GROUP by 1
order by 2 desc

```

![](src/Pasted%20image%2020230607115813.png)


1. What was the first item from the menu purchased by each customer?

```sql

SELECT customer_id , min(order_date), product_name
from sales  
join menu
using(product_id)
group by customer_id

```

![](src/Pasted%20image%2020230607120300.png)


1. What is the most purchased item on the menu and how many times was it purchased by all customers?

```sql
-- get he most purchased item
SELECT product_name, COUNT(*) AS total_purchases
FROM sales
JOIN menu
USING(product_id)
GROUP BY product_name
ORDER BY total_purchases DESC
LIMIT 1;


--- ramen sales breakdown by customers

SELECT customer_id, COUNT(*) AS total_purchases
FROM sales
JOIN menu
USING(product_id)
WHERE product_name = 'ramen'
GROUP by customer_id

```

![](src/Pasted%20image%2020230607120940.png)

![](src/Pasted%20image%2020230607121202.png)


1. Which item was the most popular for each customer?

```sql
--rank items by customer id, and product name with their total purchases
SELECT customer_id, product_name,COUNT(*) AS total_purchases,
      RANK() OVER (PARTITION BY customer_id ORDER BY count(*) DESC) AS rank
FROM sales
JOIN menu
USING(product_id)
GROUP by customer_id,product_name

--use a CTE to get all the rank number 1 since they have tie

WITH most_pop
     AS (SELECT customer_id,
                product_name,
                Count(*)   AS total_purchases,
                Rank()
                  OVER (
                    partition BY customer_id
                    ORDER BY Count(*) DESC) AS rank
         FROM   sales
                JOIN menu using(product_id)
         GROUP  BY customer_id,
                   product_name)
SELECT *
FROM   most_pop
WHERE  rank = 1 
```

![](src/Pasted%20image%2020230607125155.png)

![](src/Pasted%20image%2020230607125215.png)



1. Which item was purchased first by the customer after they became a member?

```sql
-- show join date

SELECT customer_id , join_date
from sales
join members
using(customer_id)
gROUP by customer_id

--get the min order date by comparing the join date and order date

SELECT customer_id , join_date, min( order_date),product_name
from sales
join members
using (customer_id)
join menu
USING (product_id)
where order_date >= join_date
gROUP by customer_id
order by customer_id

```

![](src/Pasted%20image%2020230607125539.png)

![](src/Pasted%20image%2020230607130559.png)

1. Which item was purchased just before the customer became a member?

```sql
-- shows orders before joined date and their rank

SELECT customer_id , join_date, order_date,product_name,
		rank() over (partition by customer_id order by order_date desc) rank
from sales
join members
using (customer_id)
join menu
USING (product_id)
where order_date < join_date
order by customer_id


-- use cte to extract the orders with rank 1 ,because some item have the same rank

with max_ord as (


SELECT customer_id , join_date, order_date,product_name,
		rank() over (partition by customer_id order by order_date desc) rank
from sales
join members
using (customer_id)
join menu
USING (product_id)
where order_date < join_date
order by customer_id)
SELECT *
from max_ord
where rank=1


```

![](src/Pasted%20image%2020230607133915.png)


![](src/Pasted%20image%2020230607134214.png)


1. What is the total items and amount spent for each member before they became a member?

```sql
SELECT   customer_id, sum(price)
from sales
join members
using (customer_id)
join menu
USING (product_id)
where order_date < join_date
GROUP by customer_id


```

![](src/Pasted%20image%2020230607135531.png)


1. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

```sql

--calculate each points, price * multiplier
select customer_id , price, product_name,
	case 
		when product_name = 'sushi' then price*20
        else price* 10
     end as points
from sales s
join menu m
on  s.product_id = m.product_id

-- group by to get the sum

select customer_id ,sum(
	case 
		when product_name = 'sushi' then price*20
        else price* 10
     end) as points
from sales s
join menu m
on  s.product_id = m.product_id
GROUP by customer_id


```


![](src/Pasted%20image%2020230607142828.png)

![](src/Pasted%20image%2020230607142930.png)



1. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

```sql
select customer_id, SUM(
	case 
		when product_name = 'sushi' then price*20
        when order_date BETWEEN join_date and DATE(join_date, '+7 days') then price*20          else price* 10
     end) as points
from sales s
join menu m
on  s.product_id = m.product_id
join members
USING(customer_id)
WHERE order_date < '2021-02-01'
GROUP by customer_id

--  when order_date BETWEEN join_date and DATE(join_date, '+7 days') then price*20  check whether order is made within the 1 week 2x multiplier
-- WHERE order_date < '2021-02-01' check whether order is before end of january

```




guide:
![](src/Pasted%20image%2020230607224611.png)




![](src/Pasted%20image%2020230607225527.png)



## Bonus Questions

### Join All The Things

The following questions are related creating basic data tables that Danny and his team can use to quickly derive insights without needing to join the underlying tables using SQL.

![](src/Pasted%20image%2020230607225742.png)



```sql

select customer_id,order_date,product_name,price,
		CASE 
			when order_date >= join_date then 'Y'
            else 'N'
 		END as member
from sales
left join menu
USING(product_id)
left join members
 USING(customer_id)

```


### Rank All The Things

Danny also requires further information about the `ranking` of customer products, but he purposely does not need the ranking for non-member purchases so he expects null `ranking` values for the records when customers are not yet part of the loyalty program.

![](src/Pasted%20image%2020230607230925.png)


```sql

with data as (SELECT customer_id,order_date,product_name,price,
		CASE
        	WHEN order_date >= join_date then 'Y'
            else 'N'
         end as member
from sales
LEFT join members USING (customer_id)
INNER join menu USING (product_id))

select *,
		CASE
        	WHEN member = 'N' then NULL
            else rank() over (partition by customer_id,member
                              order by order_date)
        end as ranking
FROM data

```
