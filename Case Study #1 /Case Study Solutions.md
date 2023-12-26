#### Q1 What is the total amount each customer spent at the restaurant?

```SQL
SELECT s.customer_id, SUM(m.price) AS total_amount
FROM sales AS s
INNER JOIN menu AS m ON s.product_id = m.product_id
GROUP BY s.customer_id
ORDER BY s.customer_id ASC;
```

Output:
| customer_id| total_amount |
| --- | --- |
|A| 76 |
|B| 74 |
|C| 36 |

#### Q2 How many days has each customer visited the restaurant?

```SQL
SELECT customer_id, COUNT(DISTINCT order_date) As days
FROM sales
GROUP BY customer_id;
```
Output:
| customer_id| days |
| --- | --- |
|A| 4 |
|B| 6 |
|C| 2 |

#### Q3 What was the first item from the menu purchased by each customer?

```SQL
WITH order_date AS
( SELECT s.customer_id,s.order_date, m.product_name,
	DENSE_RANK() OVER (PARTITION BY s.customer_id ORDER BY s.order_date ) AS row_num
  FROM sales AS s
	INNER JOIN menu AS m ON s.product_id = m.product_id)

SELECT customer_id,product_name
FROM order_date
WHERE row_num =1
GROUP BY customer_id, product_name;
```

Output:
| customer_id|product_name|
| --- | --- |
|A| sushi  | 
|A| curry | 
|B| curry | 
|C| ramen | 


#### Q4 What is the most purchased item on the menu and how many times was it purchased by all customers?

```SQL
SELECT m.product_name, COUNT(s.product_id) AS times_purchased
FROM menu as m
INNER JOIN sales AS s ON m.product_id=s.product_id
GROUP BY m.product_name
ORDER BY times_purchased DESC
LIMIT 1;
```

Output:
| product_name|product_name|
| --- | times_purchased|
|ramen| 8 | 

### Q5 Which item was the most popular for each customer?

```SQL
WITH PopularItems AS (
  SELECT s.customer_id, m.product_name, COUNT(m.product_id) AS item_count,
    DENSE_RANK() OVER (PARTITION BY s.customer_id ORDER BY COUNT(s.customer_id) DESC) AS row_num
  FROM menu AS m
    INNER JOIN sales AS s ON m.product_id = s.product_id
  GROUP BY s.customer_id,m.product_name)

SELECT customer_id, product_name AS most_popular_item,item_count
FROM PopularItems
WHERE row_num = 1;
```
Output:
| customer_id|most_popular_item |item_count|
| --- | --- |--- |
|A| ramen | 3|
|B| curry | 2|
|B| sushi | 2|
|B| ramen | 2|
|C| ramen | 3|

#### Q6 Which item was purchased first by the customer after they became a member?

```SQL
WITH joined_date AS
(SELECT m.customer_id,s.product_id,
ROW_NUMBER() OVER (PARTITION BY m.customer_id ORDER BY s.order_date) AS row_num
FROM members as m
INNER JOIN sales as s ON m.customer_id = s.customer_id
WHERE s.order_date > m.join_date)

SELECT customer_id, product_name
FROM joined_date
INNER JOIN menu ON joined_date.product_id=menu.product_id 
WHERE row_num = 1
ORDER BY customer_id ASC;
```
Output:
| customer_id|product_name|
| --- | --- |
|A| ramen | 
|B| sushi | 

#### Q7 Which item was purchased just before the customer became a member?
```SQL
WITH joined_date AS
(SELECT m.customer_id,s.product_id,
ROW_NUMBER() OVER (PARTITION BY m.customer_id ORDER BY s.order_date DESC) AS row_num
FROM members as m
INNER JOIN sales as s ON m.customer_id = s.customer_id
WHERE s.order_date < m.join_date)

SELECT j.customer_id, menu.product_name
FROM joined_date AS j
INNER JOIN menu ON j.product_id=menu.product_id 
WHERE row_num = 1
ORDER BY j.customer_id ASC;
```

Output:
| customer_id|product_name|
| --- | --- |
|A| sushi | 
|B| sushi | 


#### Q8 What is the total items and amount spent for each member before they became a member?

```SQL
WITH joined_date AS
(SELECT s.customer_id, menu.product_id,menu.price, 
ROW_NUMBER() OVER (PARTITION BY s.customer_id ORDER BY s.order_date) AS row_num
FROM sales AS s
INNER JOIN menu ON s.product_id = menu.product_id
INNER JOIN members AS m ON s.customer_id = m.customer_id
WHERE s.order_date < m.join_date)

SELECT customer_id, COUNT(product_id) AS total_items, sum(price) AS total_amount
FROM joined_date
GROUP BY customer_id
ORDER BY customer_id ASC;
```

Output:
| customer_id|total_items |total_amount|
| --- | --- |--- |
|A| 2 | 25|
|B| 3 | 40|


#### Q9 If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
```SQL
WITH points_collected as
(SELECT s.customer_id,s.product_id,
    CASE
      WHEN menu.product_id = '1' THEN menu.price * 10 * 2
      ELSE menu.price * 10
    END AS points
FROM sales AS s 
INNER JOIN menu ON s.product_id = menu.product_id)

SELECT customer_id,SUM(points) AS total_points
FROM points_collected
GROUP BY customer_id;
 ```

Output:
| customer_id| days |
| --- | --- |
|A| 860 |
|B| 940 |
|C| 360 |
 
#### Q10 In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January? 
```SQL
WITH jan AS
(SELECT 
    customer_id, 
    join_date, 
    join_date + 6 AS valid_date,
    LAST_DAY('2021-01-31') AS last_day
  FROM members)

SELECT s.customer_id,SUM(CASE
      WHEN menu.product_id = '1' THEN 2 * 10 * menu.price
      WHEN s.order_date BETWEEN jan.join_date AND jan.valid_date THEN 2 * 10 * menu.price
      ELSE menu.price * 10 END) AS points
FROM sales AS s
INNER JOIN jan ON s.customer_id = jan.customer_id
  AND jan.join_date <= s.order_date
  AND s.order_date <= jan.last_day
INNER JOIN menu ON s.product_id = menu.product_id
GROUP BY s.customer_id
ORDER BY s.customer_id;
```
Output:
| customer_id| points |
| --- | --- |
|A| 1020 |
|B| 320 |

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


