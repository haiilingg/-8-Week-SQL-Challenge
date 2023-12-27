### D. Pricing and Ratings
#### Q1 If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?

``` MySQL
WITH price_of_pizza AS
(Select pizza_id, pizza_name, 
CASE WHEN pizza_id = '1' THEN 12 WHEN pizza_id = '2' THEN 10 ELSE 0 END AS pizza_price
FROM pizza_names)

SELECT pp.pizza_id, pp.pizza_name, COUNT(C.order_id) AS total_pizzas,SUM(pp.pizza_price) AS total_income
FROM price_of_pizza AS pp
INNER JOIN customer_orders AS c ON pp.pizza_id = c.pizza_id
INNER JOIN runner_orders AS r ON c.order_id = r.order_id
WHERE COALESCE(r.cancellation, '') NOT LIKE '%Cancellation%'
GROUP BY pp.pizza_id, pp.pizza_name;
```

Output:
|pizza_id|pizza_name|total_pizzas|total_income|
| --- | --- |--- |--- |
|1| Meatlovers |9| 108|
|2| Vegetarian |3| 30|

#### Q2 What if there was an additional $1 charge for any pizza extras? Add cheese is $1 extra
``` MySQL
WITH price_of_pizza AS
(Select pizza_id, pizza_name, 
CASE WHEN pizza_id = '1' THEN 12 WHEN pizza_id = '2' THEN 10 ELSE 0 END AS pizza_price
FROM pizza_names)

SELECT pp.pizza_id, pp.pizza_name,
SUM(pp.pizza_price + CASE WHEN c.extras IS NOT NULL AND c.extras NOT IN ('null', '') THEN 1 ELSE 0 END) AS total_income
FROM price_of_pizza AS pp
INNER JOIN customer_orders AS c ON pp.pizza_id = c.pizza_id
INNER JOIN runner_orders AS r ON c.order_id = r.order_id
WHERE COALESCE(r.cancellation, '') NOT LIKE '%Cancellation%'
GROUP BY pp.pizza_id, pp.pizza_name;
```

Output:
|pizza_id|pizza_name|total_income|
| --- | --- |--- |
|1| Meatlovers | 110|
|2| Vegetarian | 31|

#### Q3 The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.
``` MySQL
 CREATE TABLE ratings (
  order_id INTEGER,
  customer_id INTEGER,
  runner_id INTEGER,
  pizza_id INTEGER,
  duration VARCHAR(10),
  cancellation VARCHAR(23),
  rating_given INTEGER);

INSERT INTO ratings
  (order_id, customer_id, runner_id, pizza_id, duration,cancellation,rating_given)
VALUES
  ('1', '101', '1', '1','32 mins', 'N/A','5'),
  ('2', '101', '1','1','27 mins', 'N/A','4'),
  ('3', '102', '1','1','20 mins', 'N/A','4' ),
  ('3', '102', '1','2','20 mins', 'N/A','4'),
  ('4', '103', '2','1','40 mins', 'N/A','5' ),
  ('4', '103', '2', '1','40 mins', 'N/A','5' ),
  ('4', '103', '2','2','40 mins', 'N/A', '5'),
  ('5', '104', '3','1','15 mins', 'N/A', '4'),
  ('6', '101', '3','2',NULL, 'Restaurant Cancellation',NULL),
  ('7', '105', '2','2', '25mins', 'N/A','3'),
  ('8', '102', '2', '1', '15 mins', 'N/A','2'),
  ('9', '103', '2','1',NULL, 'Customer Cancellation',NULL),
  ('10', '104','1','1','10mins', 'N/A','3'),
  ('10', '104','1', '1','10mins', 'N/A','3');

SELECT *
FROM ratings;
```

Output of new schema:
| order_id | customer_id | runner_id | pizza_id | duration | cancellation              | rating_given |
|----------|-------------|-----------|----------|----------|---------------------------|--------------|
| 1        | 101         | 1         | 1        | 32 mins  | N/A                       | 5            |
| 2        | 101         | 1         | 1        | 27 mins  | N/A                       | 4            |
| 3        | 102         | 1         | 1        | 20 mins  | N/A                       | 4            |
| 3        | 102         | 1         | 2        | 20 mins  | N/A                       | 4            |
| 4        | 103         | 2         | 1        | 40 mins  | N/A                       | 5            |
| 4        | 103         | 2         | 1        | 40 mins  | N/A                       | 5            |
| 4        | 103         | 2         | 2        | 40 mins  | N/A                       | 5            |
| 5        | 104         | 3         | 1        | 15 mins  | N/A                       | 4            |
| 6        | 101         | 3         | 2        | NULL     | Restaurant Cancellation | NULL         |
| 7        | 105         | 2         | 2        | 25 mins  | N/A                       | 3            |
| 8        | 102         | 2         | 1        | 15 mins  | N/A                       | 2            |
| 9        | 103         | 2         | 1        | NULL     | Customer Cancellation    | NULL         |
| 10       | 104         | 1         | 1        | 10 mins  | N/A                       | 3            |
| 10       | 104         | 1         | 1        | 10 mins  | N/A                       | 3            |


#### Q4 Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?(customer_id,order_id,runner_id,rating,order_time,pickup_time,Time between order and pickup, Delivery duration,Average speed,Total number of pizzas)
``` MySQL
SELECT c.customer_id,c.order_id,r.runner_id,ra.rating_given,c.order_time,r.pickup_time,(TIMESTAMPDIFF(MINUTE, c.order_time, r.pickup_time)) AS order_vs_pickup_time, r.duration, AVG(r.distance / TIME_TO_SEC(r.duration) * 60 ) AS avg_speed,COUNT(c.order_id) AS no_of_pizza
FROM customer_orders AS c
INNER JOIN runner_orders AS r ON c.order_id = r.order_id
INNER JOIN ratings AS ra ON c.order_id = ra.order_id
WHERE COALESCE(r.cancellation, '') NOT LIKE '%Cancellation%'
GROUP BY c.customer_id,c.order_id,r.runner_id,ra.rating_given,c.order_time,r.pickup_time,r.duration;
```

Output:
| customer_id | order_id | runner_id | rating_given | order_time | pickup_time | order_vs_pickup_time | duration | avg_speed | no_of_pizza |
|-------------|----------|-----------|--------------|------------|-------------|----------------------|----------|-----------|-------------|
| 101         | 1        | 1         | 5            | 01/01/2020 18:05 | 01/01/2020 18:15 | 10 | 32 minutes | 37.5 | 1 |
| 101         | 2        | 1         | 4            | 01/01/2020 19:00 | 01/01/2020 19:10 | 10 | 27 minutes | 44.44 | 1 |
| 102         | 3        | 1         | 4            | 02/01/2020 23:51 | 03/01/2020 0:12 | 21 | 20 mins | 40.2 | 4 |
| 103         | 4        | 2         | 5            | 04/01/2020 13:23 | 04/01/2020 13:53 | 29 | 40 | 35.1 | 9 |
| 104         | 5        | 3         | 4            | 08/01/2020 21:00 | 08/01/2020 21:10 | 10 | 15 | 40 | 1 |
| 105         | 7        | 2         | 3            | 08/01/2020 21:20 | 08/01/2020 21:30 | 10 | 25 mins | 60 | 1 |
| 102         | 8        | 2         | 2            | 09/01/2020 23:54 | 10/01/2020 0:15 | 20 | 15 minute | 93.6 | 1 |
| 104         | 10       | 1         | 3            | 11/01/2020 18:34 | 11/01/2020 18:50 | 15 | 10 minutes | 60 | 4 |


#### Q5 If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?
``` MySQL
WITH price_of_pizza AS
(Select pizza_id, pizza_name, 
CASE WHEN pizza_id = '1' THEN 12 WHEN pizza_id = '2' THEN 10 ELSE 0 END AS pizza_price
FROM pizza_names)

SELECT SUM(pp.pizza_price),SUM(AVG_distance) * 0.30 AS cost, SUM(pp.pizza_price) - SUM(AVG_distance * 0.30) AS net_income
FROM price_of_pizza AS pp
INNER JOIN customer_orders AS c ON pp.pizza_id = c.pizza_id
INNER JOIN (SELECT order_id, AVG(distance) AS AVG_distance
    FROM runner_orders
    WHERE COALESCE(cancellation, '') NOT LIKE '%Cancellation%'
    GROUP BY order_id
) AS r ON c.order_id = r.order_id;
```

Output:
|pizza_price|cost|net_income|
| --- | --- |--- |
|138| 64.62 | 73.38|
