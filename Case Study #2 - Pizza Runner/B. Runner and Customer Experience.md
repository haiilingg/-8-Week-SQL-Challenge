### B. Runner and Customer Experience
#### Q1 How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)
``` MySQL
SELECT 1 + DATEDIFF(registration_date, MAKEDATE(YEAR(registration_date), 1)) DIV 7 AS signed_up_week, COUNT(runner_id) AS no_of_runners
FROM runners
GROUP BY signed_up_week
ORDER BY signed_up_week;
```

#### Q2 What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
``` MySQL
SELECT r.runner_id, AVG(TIMESTAMPDIFF(MINUTE, c.order_time, r.pickup_time)) AS avg_pickup_time_minutes
FROM runner_orders AS r
INNER JOIN customer_orders AS c ON r.order_id = c.order_id
GROUP BY r.runner_id;
```

#### Q3 Is there any relationship between the number of pizzas and how long the order takes to prepare?
``` MySQL
SELECT c.order_id,COUNT(*) AS order_count,TIMEDIFF(r.pickup_time, c.order_time) AS time_difference
FROM runner_orders r
JOIN customer_orders c ON r.order_id = c.order_id
WHERE COALESCE(r.cancellation, '') NOT LIKE '%Cancellation%'
GROUP BY c.order_id,r.pickup_time, c.order_time
ORDER BY time_difference DESC;
```

#### Q4 What was the average distance travelled for each customer?
``` MySQL
SELECT c.customer_id, AVG(r.distance) AS avg_distance
FROM customer_orders AS c
INNER JOIN runner_orders AS r ON c.order_id = r.order_id
WHERE r.duration != 0
GROUP BY c.customer_id;
```

#### Q5 What was the difference between the longest and shortest delivery times for all orders?
``` MySQL
WITH delivery_time_diff AS (
SELECT MAX(CASE WHEN duration NOT IN ('null', '') AND duration IS NOT NULL THEN duration END) AS max_delivery_time,
MIN(CASE WHEN duration IS NOT NULL THEN duration END) AS min_delivery_time
FROM runner_orders)

SELECT (max_delivery_time - min_delivery_time) AS time_diff
FROM delivery_time_diff;
```

#### Q6 What was the average speed for each runner for each delivery and do you notice any trend for these values?
``` MySQL
SELECT runner_id, AVG(DISTANCE), AVG (distance / TIME_TO_SEC(duration) * 60 )AS average_speed
FROM runner_orders 
WHERE distance != 0
GROUP BY runner_id
ORDER BY average_speed DESC;
```

#### Q7 What is the successful delivery percentage for each runner?
``` MySQL
SELECT runner_id,
(COUNT(CASE WHEN COALESCE(cancellation, '') NOT LIKE '%Cancellation%' THEN 1 END) / COUNT(order_id))*100 AS successful_percentage
FROM runner_orders
GROUP BY runner_id;
```
