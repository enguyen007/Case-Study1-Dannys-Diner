# 🍜 🍥 Case Study #1: Danny's Diner

### **1. What is the total amount each customer spent at the restaurant?**
``` sql
SELECT s.customer_id, SUM(price) AS total_sales
FROM dannys_diner.sales AS s
JOIN dannys_diner.menu AS m
   ON s.product_id = m.product_id
GROUP BY customer_id; 
```

Answer:
| customer_id	  | total_sales |
| ------------- | ------------- |
| A  | 76  |
| B  | 74  |
| C  | 36  |

Steps:
- Use JOIN to merge sales and menu tables as customer_id is from the sales table and price is from the menu table.
- Use SUM and GROUP BY to find out total_sales contributed by each customer.


### **2. How many days has each customer visited the restaurant?** 
``` sql
SELECT customer_id, COUNT(DISTINCT(order_date)) AS visit_count
FROM dannys_diner.sales
GROUP BY customer_id; 
```

Answer:
| customer_id	  | visit_count |
| ------------- | ------------- |
| A  | 4  |
| B  | 6  |
| C  | 2  |

Steps: 
- The nature of the question implies that we need to use the DISTINCT function because we don't want to count repeated order days for any customer.

### **3. What was the first item from the menu purchased by each customer?**
```sql
WITH ordered_sales_cte AS
(
   SELECT customer_id, order_date, product_name,
      DENSE_RANK() OVER(PARTITION BY s.customer_id
      ORDER BY s.order_date) AS rank
   FROM dannys_diner.sales AS s
   JOIN dannys_diner.menu AS m
      ON s.product_id = m.product_id
)

SELECT customer_id, product_name
FROM ordered_sales_cte
WHERE rank = 1
GROUP BY customer_id, product_name;
```

Answer:
| customer_id	  | product_name |
| ------------- | ------------- |
| A  | curry  |
| A  | sushi  |
| B  | curry  |
| C  | ramen  |

Steps:
- Use Windows function with DENSE_RANK and temp table order_sales_cte to create a new column rank order by order_date.
- Instead of ROW_NUMBER or RANK, use DENSE_RANK as order_date is not time-stamped. There is no sequence as to which item is ordered first if 2 or more items are ordered on the same day.
- Then GROUP BY all columns to show rank = 1 only.

### **4. What is the most purchased item on the menu and how many times was it purchased by all customers?** 
```sql
SELECT (COUNT(s.product_id)) AS most_purchased, product_name
FROM dannys_diner.sales AS s
JOIN dannys_diner.menu AS m
   ON s.product_id = m.product_id
GROUP BY s.product_id, product_name
ORDER BY most_purchased DESC
LIMIT 1;
```

Answer:
| most_purchased	| product_name |
| ------------- | ------------- |
| 8  | ramen  |

Steps:
- COUNT number of product_id and ORDER BY most_purchased by descending order.
- LIMIT 1 to query the item with the highest number of purchases

### **5. Which item was the most popular for each customer?**
``` sql
WITH fav_item_cte AS
(
   SELECT s.customer_id, m.product_name, COUNT(m.product_id) AS order_count,
      DENSE_RANK() OVER(PARTITION BY s.customer_id
      ORDER BY COUNT(s.customer_id) DESC) AS rank
   FROM dannys_diner.menu AS m
   JOIN dannys_diner.sales AS s
      ON m.product_id = s.product_id
   GROUP BY s.customer_id, m.product_name
)

SELECT customer_id, product_name, order_count
FROM fav_item_cte 
WHERE rank = 1;
```

Answer:
| customer_id	  | product_name | order_count |
| ------------- | ------------- |------------- |
| A  | ramen  | 3 |
| B  | sushi  | 2 |
| B  | curry  | 2 |
| B  | ramen  | 2 |
| C  | ramen  | 3 |

Steps:
- Within the sub query, fav_item_cte, use DENSE RANK to rank the order count for each customer, then put them in descending order.
- Since we are querying for the most popular product(s) for each customer, filter by rank = 1.

### **6. Which item was purchased first by the customer after they became a member?**
```sql
WITH member_sales_cte AS 
(
   SELECT s.customer_id, m.join_date, s.order_date, s.product_id,
      DENSE_RANK() OVER(PARTITION BY s.customer_id
      ORDER BY s.order_date) AS rank
   FROM dannys_diner.sales AS s
   JOIN dannys_diner.members AS m
      ON s.customer_id = m.customer_id
   WHERE s.order_date >= m.join_date
)

SELECT s.customer_id, s.order_date, m2.product_name 
FROM member_sales_cte AS s
JOIN dannys_diner.menu AS m2
   ON s.product_id = m2.product_id
WHERE rank = 1;
```

Answer:
| customer_id	  | order_date | product_name |
| ------------- | ------------- |------------- |
| A  | 2021-01-07  | curry |
| B  | 2021-01-11  | sushi |

Steps:
- Within member_sales_cte, use windows function to partition by customer_id and order order_date in ascending order. Then, filter order_date to be on or after join_date.
- We want to find the first purchased item on or after join_date, filter by rank = 1.

### **7. Which item was purchased just before the customer became a member?**
```sql
WITH prior_member_purchased_cte AS 
(
   SELECT s.customer_id, m.join_date, s.order_date, s.product_id,
         DENSE_RANK() OVER(PARTITION BY s.customer_id
         ORDER BY s.order_date DESC) AS rank
   FROM dannys_diner.sales AS s
   JOIN dannys_diner.members AS m
      ON s.customer_id = m.customer_id
   WHERE s.order_date < m.join_date
)

SELECT s.customer_id, s.order_date, m2.product_name 
FROM prior_member_purchased_cte AS s
JOIN dannys_diner.menu AS m2
   ON s.product_id = m2.product_id
WHERE rank = 1;
```

Answer:
| customer_id	  | order_date | product_name |
| ------------- | ------------- |------------- |
| A  | 2021-01-01  | sushi |
| A  | 2021-01-01  | curry |
| B  | 2021-01-04  | sushi |

Steps:
- Within prior_member_purchased_cte, use windows function to partition by customer_id and order order_date in descending order. Then, as opposite to the last problem, filter order_date to be before join_date.
- We want to find the last purchased item before join_date, filter by rank = 1.

### **8. What is the total items and amount spent for each member before they became a member?**
```sql
SELECT s.customer_id, COUNT(DISTINCT s.product_id) AS unique_menu_item, 
   SUM(mm.price) AS total_sales
FROM dannys_diner.sales AS s
JOIN dannys_diner.members AS m
   ON s.customer_id = m.customer_id
JOIN dannys_diner.menu AS mm
   ON s.product_id = mm.product_id
WHERE s.order_date < m.join_date
GROUP BY s.customer_id;
```
Answer:
| customer_id	  | unique_menu_item	 | product_name |
| ------------- | ------------- |------------- |
| A  | 2 | 25 |
| B  | 2 | 40 |

Steps:
- Join all sales, member, menu tables together as we need information on all 3 tables. Filter order_date before join_date then GROUP BY customer_id.
- The nature of the question implies that we need to use the DISTINCT function to count unique product_id and then SUM the total amount spent.

### **9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier — how many points would each customer have?**
```sql
WITH price_points_cte AS
(
   SELECT *, 
      CASE
         WHEN product_id = 1 THEN price * 20
         ELSE price * 10
      END AS points
   FROM dannys_diner.menu
)

SELECT s.customer_id, SUM(p.points) AS total_points
FROM price_points_cte AS p
JOIN dannys_diner.sales AS s
   ON p.product_id = s.product_id
GROUP BY s.customer_id;
```

Answer:
| customer_id		| total_points |
| ------------- | ------------- |
| A | 860  |
| B | 940  |
| C | 360  |

Steps:
- If $1 = 10 points but for sushi, it's $1 = 20 points (because it's 2x the normal amount). We need to use CASE WHEN to create our conditions that if it's any other product, it's $1 = 10 points but if it's sushi (product_id = 1) then $1 = 20 points. We then create price_points_cte using this method.
- Use SUM to calculate the points from price_points_cte for each customer.

### **10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi — how many points do customer A and B have at the end of January?**
```sql
WITH dates_cte AS 
(
   SELECT *, 
      DATEADD(DAY, 6, join_date) AS valid_date, 
      EOMONTH('2021-01-31') AS last_date
   FROM dannys_diner.members AS m
)

SELECT d.customer_id, s.order_date, d.join_date, d.valid_date, d.last_date, m.product_name, m.price,
   SUM(CASE
      WHEN m.product_name = 'sushi' THEN 2 * 10 * m.price
      WHEN s.order_date BETWEEN d.join_date AND d.valid_date THEN 2 * 10 * m.price
      ELSE 10 * m.price
      END) AS points
FROM dates_cte AS d
JOIN dannys_diner.sales AS s
   ON d.customer_id = s.customer_id
JOIN dannys_diner.menu AS m
   ON s.product_id = m.product_id
WHERE s.order_date < d.last_date
GROUP BY d.customer_id, s.order_date, d.join_date, d.valid_date, d.last_date, m.product_name, m.price;
```
 
Answer:
| customer_id		| total_points |
| ------------- | ------------- |
| A | 1370  |
| B | 820   |

Steps:
- In dates_cte, we need to find out customer’s valid_date (6 days after join_date, including join_date) and last_day of Jan 2021 (which is ‘2021–01–31’).
- Assuming: 
- From Day-X to Day 1 (customer becomes member on Day 1 join_date), each $1 spent is 10 points and for sushi, each $1 spent is 20 points.
- From Day 1 join_date to Day 7 valid_date, each $1 spent for all items is 20 points.
- From Day 8 to last_day of Jan 2021, each $1 spent is 10 points and sushi is 2x points.
































