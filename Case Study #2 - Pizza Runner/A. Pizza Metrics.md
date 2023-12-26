### A. Pizza Metrics
#### Q1 How many pizzas were ordered?

``` MYSQL
SELECT count(pizza_id)
FROM customer_orders;
```

#### Q2 How many unique customer orders were made?
``` MYSQL
SELECT COUNT(DISTINCT order_id)
FROM customer_orders;
 ```
 
#### Q3 How many successful orders were delivered by each runner?
``` MYSQL
SELECT runner_id, COUNT(order_id) AS successful_orders
FROM runner_orders
WHERE COALESCE(cancellation, '') NOT LIKE '%Cancellation%'
GROUP BY runner_id;
```

#### Q4 How many of each type of pizza was delivered?
``` MYSQL
SELECT c.pizza_id,p.pizza_name, COUNT(c.order_id) AS pizza_delivered
FROM customer_orders AS c
INNER JOIN runner_orders AS r ON c.order_id = r.order_id
INNER JOIN pizza_names AS p ON c.pizza_id = p.pizza_id
WHERE COALESCE(r.cancellation, '') NOT LIKE '%Cancellation%'
GROUP BY c.pizza_id,p.pizza_name;
```

#### Q5 How many Vegetarian and Meatlovers were ordered by each customer?
``` MYSQL
SELECT c.customer_id, p.pizza_name, COUNT(c.order_id) AS pizza_ordered
FROM customer_orders AS c
INNER JOIN runner_orders AS r ON c.order_id = r.order_id
INNER JOIN pizza_names AS p ON c.pizza_id = p.pizza_id
GROUP BY c.customer_id, p.pizza_name
ORDER BY c.customer_id;
```

#### Q6 What was the maximum number of pizzas delivered in a single order?
``` MYSQL
SELECT c.order_id,COUNT(pizza_id) AS pizza_delivered
FROM customer_orders AS c
INNER JOIN runner_orders AS r ON c.order_id = r.order_id
WHERE COALESCE(r.cancellation, '') NOT LIKE '%Cancellation%'
GROUP BY c.order_id
ORDER BY pizza_delivered DESC
LIMIT 1;
```

#### Q7 For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
``` MYSQL
WITH PizzaChanges AS ( 
SELECT customer_id, order_id,
CASE WHEN exclusions NOT IN ('null', '') OR extras NOT IN ('null', '') OR  extras IS NOT NULL THEN 'at least 1 change' ELSE 'no changes' END AS changes
FROM customer_orders)

SELECT changes,COUNT(customer_id) AS count
FROM PizzaChanges
INNER JOIN runner_orders AS r ON PizzaChanges.order_id = r.order_id
WHERE COALESCE(r.cancellation, '') NOT LIKE '%Cancellation%'
GROUP BY changes;
```

#### Q8 How many pizzas were delivered that had both exclusions and extras?
``` MYSQL
SELECT COUNT(c.order_id) AS pizza_delivered
FROM customer_orders AS c
RIGHT JOIN runner_orders AS r ON c.order_id = r.order_id
WHERE exclusions NOT IN ('null', '') AND extras NOT IN ('null', '') AND extras IS NOT NULL 
AND COALESCE(r.cancellation, '') NOT LIKE '%Cancellation%';
```

#### Q9 What was the total volume of pizzas ordered for each hour of the day?
``` MYSQL
SELECT HOUR(order_time) AS order_hour,COUNT(order_id) AS pizzas_ordered
FROM customer_orders
GROUP BY order_hour
ORDER BY order_hour;
```

#### Q10 What was the volume of orders for each day of the week?
``` MYSQL
SELECT DAYNAME(order_time) AS order_day,COUNT(order_id) AS pizzas_ordered
FROM customer_orders
GROUP BY order_day
ORDER BY pizzas_ordered DESC;
```
