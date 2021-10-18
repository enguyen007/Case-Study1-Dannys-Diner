## 🍜 Case Study #1: Danny's Diner

**1. What is the total amount each customer spent at the restaurant?**
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
- Use SUM and GROUP BY to find out total_sales contributed by each customer.
- Use JOIN to merge sales and menu tables as customer_id and price are from both tables.
