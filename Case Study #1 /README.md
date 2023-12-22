


```SQL
SELECT s.customer_id, SUM(m.price) AS total_amount
FROM sales AS s
INNER JOIN menu AS m ON s.product_id = m.product_id
GROUP BY s.customer_id
ORDER BY s.customer_id ASC;
```
