
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

customer_id| order_date|product_name|price|member
| --- | --- | --- | --- | --- | 
|A|01/01/2021|sushi|10|N|
|A|01/01/2021|curry|15|N|
|A|07/01/2021|curry|15|Y|
|A|10/01/2021|ramen|12|Y|
|A|11/01/2021|ramen|12|Y|
|A|11/01/2021|ramen|12|Y|
|B|01/01/2021|curry|15|N|
|B|02/01/2021|curry|15|N|
|B|04/01/2021|sushi|10|N|
|B|11/01/2021|sushi|10|Y|
|B|16/01/2021|ramen|12|Y|
|B|01/02/2021|ramen|12|Y|
|C|01/01/2021|ramen|12|N|
|C|01/01/2021|ramen|12|N|
|C|07/01/2021|ramen|12|N|


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
customer_id| order_date|product_name|price|member|ranking
| --- | --- | --- | --- | --- | --- | 
|A|01/01/2021|sushi|10|N|NULL|
|A|01/01/2021|curry|15|N|NULL|
|A|07/01/2021|curry|15|Y|1|
|A|10/01/2021|ramen|12|Y|2|
|A|11/01/2021|ramen|12|Y|3|
|A|11/01/2021|ramen|12|Y|3|
|B|01/01/2021|curry|15|N|NULL|
|B|02/01/2021|curry|15|N|NULL|
|B|04/01/2021|sushi|10|N|NULL|
|B|11/01/2021|sushi|10|Y|1|
|B|16/01/2021|ramen|12|Y|2|
|B|01/02/2021|ramen|12|Y|3|
|C|01/01/2021|ramen|12|N|NULL|
|C|01/01/2021|ramen|12|N|NULL|
|C|07/01/2021|ramen|12|N|NULL|
