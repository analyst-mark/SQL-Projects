
## Case Study Questions
---

1. What is the total amount each customer spent at the restaurant?
2. How many days has each customer visited the restaurant?
3. What was the first item from the menu purchased by each customer?
4. What is the most purchased item on the menu and how many times was it purchased by all customers?
5. Which item was the most popular for each customer?
6. Which item was purchased first by the customer after they became a member?
7. Which item was purchased just before the customer became a member?
8. What is the total items and amount spent for each member before they became a member?
9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?


# Answers
---

-  What is the total amount each customer spent at the restaurant?

```sql
   SELECT customer_id, sum(price) total
   from sales s
   join menu m
   on s.product_id = m.product_id
   group by customer_id
   order by 2 desc

```

![](src/Pasted%20image%2020230615113543.png)

- What was the first item from the menu purchased by each customer?

```sql
WITH first_product AS (
    SELECT s.customer_id, s.order_date, p.product_name,
           ROW_NUMBER() over (PARTITION BY s.customer_id ORDER BY s.order_date) AS rank_
    FROM sales s
    JOIN menu p ON s.product_id = p.product_id
    WHERE s.order_date = (
        SELECT MIN(order_date)
        FROM sales
        WHERE customer_id = s.customer_id
    )
)
SELECT customer_id, order_date, product_name
FROM first_product
WHERE rank_ = 1;



```

- It creates a Common Table Expression (CTE) named `first_product`.
- The CTE selects the `customer_id`, `order_date`, and `product_name` from the `sales` and `menu` tables.
- The CTE also calculates a rank for each row using the `ROW_NUMBER()` function.
- The rank is calculated by partitioning the data by `customer_id` and ordering by `order_date`.
- The final query selects the `customer_id`, `order_date`, and `product_name` from the CTE where the rank is equal to 1.
- This returns the first product ordered by each customer.

![](src/Pasted%20image%2020230615120140.png)

 - What is the most purchased item on the menu and how many times was it purchased by all customers?

```sql
SELECT top 1 product_name, COUNT(*) AS total_purchases
FROM sales s 
JOIN menu m
ON s.product_id = m.product_id
GROUP BY product_name
ORDER BY total_purchases DESC


---

SELECT customer_id, COUNT(*) AS total_purchases
FROM sales s 
JOIN menu m
ON s.product_id = m.product_id
where product_name = 'ramen'
GROUP BY customer_id
ORDER BY total_purchases DESC

```

![](src/Pasted%20image%2020230615120451.png)  ![](src/Pasted%20image%2020230615120656.png)

-  Which item was the most popular for each customer?

```sql

with rank_cte as 
(SELECT customer_id, product_name,COUNT(*) AS total_purchases,
      dense_RANK() OVER (PARTITION BY customer_id ORDER BY count(*) DESC) AS rank
FROM sales s 
JOIN menu m
ON s.product_id = m.product_id
GROUP by customer_id,product_name)
select customer_id,product_name,total_purchases
from rank_cte
where rank =1

```

![](src/Pasted%20image%2020230615121200.png)

![](src/Pasted%20image%2020230615121214.png)

- Which item was purchased first by the customer after they became a member?

```sql
-- show join date

with purchased_first as (
select s.customer_id,order_date,join_date,product_name,
	ROW_NUMBER() over (partition by s.customer_id order by order_date) rank
from sales s
join menu m
on s.product_id = m.product_id
join members ms
on ms.customer_id = s.customer_id
where order_date >= join_date)
select  *
from purchased_first
where rank = 1

```


![](src/Pasted%20image%2020230615122317.png)

![](src/Pasted%20image%2020230615122609.png)

- Which item was purchased just before the customer became a member?

```sql

with purchased_first as (
select s.customer_id,order_date,join_date,product_name,
	ROW_NUMBER() over (partition by s.customer_id order by order_date desc) rank
from sales s
join menu m
on s.product_id = m.product_id
join members ms
on ms.customer_id = s.customer_id
where order_date < join_date
)
select  *
from purchased_first
where rank = 1

```

![](src/Pasted%20image%2020230615122943.png)

![](src/Pasted%20image%2020230615122952.png)

-  What is the total items and amount spent for each member before they became a member?

```sql
select s.customer_id,sum(price) total
from sales s
join menu m
on s.product_id = m.product_id
join members ms
on s.customer_id = ms.customer_id
where join_date > order_date
group by s.customer_id


```

![](src/Pasted%20image%2020230615123445.png)


- If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

```sql

with total_points as (
select s.customer_id, product_name,price,
		case	
			when product_name = 'sushi' then price * 20 
			else price * 10
		end as points
from sales s
left join menu m
on s.product_id = m.product_id
left join members ms
on s.customer_id = ms.customer_id
)
select customer_id,sum(points) as total_points
from total_points
group by customer_id



```

![](src/Pasted%20image%2020230615123923.png)  ![](src/Pasted%20image%2020230615123947.png)

- In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

```sql
SELECT s.customer_id,
       SUM(CASE
               WHEN product_name = 'sushi' THEN price * 20
               WHEN order_date BETWEEN join_date AND DATEADD(day, 7, join_date) THEN price * 20
               ELSE price * 10
           END) AS points
FROM sales s
JOIN menu m ON s.product_id = m.product_id
JOIN members  ON s.customer_id = members.customer_id
WHERE order_date < '2021-02-01'
GROUP BY s.customer_id;

```

![](src/Pasted%20image%2020230615124443.png)


---

## Bonus Questions

### Join All The Things

The following questions are related creating basic data tables that Danny and his team can use to quickly derive insights without needing to join the underlying tables using SQL.

![](src/Pasted%20image%2020230607225742.png)


```sql
select s.customer_id,order_date,product_name,price,
		CASE 
			when order_date >= join_date then 'Y'
            else 'N'
 		END as member
from sales s
left join menu m
 on s.product_id  = m.product_id
left join members ms
on s.customer_id = ms.customer_id
```


### Rank All The Things

Danny also requires further information about the `ranking` of customer products, but he purposely does not need the ranking for non-member purchases so he expects null `ranking` values for the records when customers are not yet part of the loyalty program.

![](src/Pasted%20image%2020230607230925.png)

```sql
with data as (
select s.customer_id,order_date,product_name,price,
		CASE 
			when order_date >= join_date then 'Y'
            else 'N'
 		END as member
from sales s
left join menu m
 on s.product_id  = m.product_id
left join members ms
on s.customer_id = ms.customer_id
)
select *,
		CASE
        	WHEN member = 'N' then NULL
            else 
            rank() over (partition by customer_id,member order by order_date)
        end as ranking
FROM data

```