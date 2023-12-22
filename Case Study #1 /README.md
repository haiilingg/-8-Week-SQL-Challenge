![casestudy1](https://github.com/haiilingg/-8-Week-SQL-Challenge/assets/130296433/5504c966-aa6b-4b71-a4eb-c13e9f572374)

### Introduction
Danny seriously loves Japanese food so in the beginning of 2021, he decides to embark upon a risky venture and opens up a cute little restaurant that sells his 3 favourite foods: sushi, curry and ramen.

Danny’s Diner is in need of your assistance to help the restaurant stay afloat - the restaurant has captured some very basic data from their few months of operation but have no idea how to use their data to help them run the business.

### ER Diagram
<img width="731" alt="Screenshot 2023-12-22 at 3 59 38 PM" src="https://github.com/haiilingg/-8-Week-SQL-Challenge/assets/130296433/d4558290-6610-45f0-9183-8d89f224658a">

Tools used: MySQL 

Source: [https://8weeksqlchallenge.com/case-study-1/](https://8weeksqlchallenge.com/case-study-1/)

#### Q1 What is the total amount each customer spent at the restaurant?

```SQL
SELECT s.customer_id, SUM(m.price) AS total_amount
FROM sales AS s
INNER JOIN menu AS m ON s.product_id = m.product_id
GROUP BY s.customer_id
ORDER BY s.customer_id ASC;
```

Output:

![image](https://github.com/haiilingg/-8-Week-SQL-Challenge/assets/130296433/36d3c7ea-2c46-4042-882e-ad6b5a4cf6d0)


#### Q2 How many days has each customer visited the restaurant?

```SQL
SELECT customer_id, COUNT(DISTINCT order_date)
FROM sales
GROUP BY customer_id;
```

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
#### Q4 What is the most purchased item on the menu and how many times was it purchased by all customers?

```SQL
SELECT m.product_name, COUNT(s.product_id) AS times_purchased
FROM menu as m
INNER JOIN sales AS s ON m.product_id=s.product_id
GROUP BY m.product_name
ORDER BY times_purchased DESC
LIMIT 1;
```

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

#### Q8 What is the total items and amount spent for each member before they became a member?

```SQL
WITH joined_date AS
(SELECT s.customer_id, menu.product_id,menu.price, 
ROW_NUMBER() OVER (PARTITION BY s.customer_id ORDER BY s.order_date) AS row_num
FROM sales AS s
INNER JOIN menu ON s.product_id = menu.product_id
INNER JOIN members AS m ON s.customer_id = m.customer_id
WHERE s.order_date < m.join_date)

SELECT customer_id, COUNT(product_id), sum(price)
FROM joined_date
GROUP BY customer_id
ORDER BY customer_id ASC;
```

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


