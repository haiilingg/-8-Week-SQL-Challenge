
### BONUS
#### Join all tables
```SQL
SELECT s.customer_id,s.order_date,m.product_name,m.price,
CASE WHEN b.join_date <= s.order_date THEN 'Y'
ELSE 'N' END AS member
FROM sales AS s
INNER JOIN menu AS m ON s.product_id = m.product_id
LEFT JOIN members AS b ON s.customer_id = b.customer_id
ORDER BY customer_id;
```

#### with ranking
```SQL
WITH joins AS
(SELECT s.customer_id,s.order_date,m.product_name,m.price,
CASE WHEN b.join_date <= s.order_date THEN 'Y'
ELSE 'N' END AS member
FROM sales AS s
INNER JOIN menu AS m ON s.product_id = m.product_id
LEFT JOIN members AS b ON s.customer_id = b.customer_id
ORDER BY customer_id)

SELECT *, CASE WHEN member = 'N'THEN 'NULL'
ELSE RANK () OVER (PARTITION BY customer_id,member
ORDER BY order_date) END AS ranking 
FROM joins;
```
