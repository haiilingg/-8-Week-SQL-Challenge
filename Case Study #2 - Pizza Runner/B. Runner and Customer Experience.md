### B. Runner and Customer Experience
#### Q1 How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)
``` MySQL
SELECT 1 + DATEDIFF(registration_date, MAKEDATE(YEAR(registration_date), 1)) DIV 7 AS signed_up_week, COUNT(runner_id) AS no_of_runners
FROM runners
GROUP BY signed_up_week
ORDER BY signed_up_week;
```
signed_up_week| no_of_runners|
| --- | --- |
|1|2|
|2|1|
|3|1|

#### Q2 What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
``` MySQL
SELECT r.runner_id, AVG(TIMESTAMPDIFF(MINUTE, c.order_time, r.pickup_time)) AS avg_pickup_time_minutes
FROM runner_orders AS r
INNER JOIN customer_orders AS c ON r.order_id = c.order_id
GROUP BY r.runner_id;
```
runner_id|avg_pickup_time_minutes|
| --- | --- |
|1|15.3333|
|2|23.4000|
|3|10.0000|

#### Q3 Is there any relationship between the number of pizzas and how long the order takes to prepare?
``` MySQL
SELECT c.order_id,COUNT(*) AS order_count,TIMEDIFF(r.pickup_time, c.order_time) AS time_difference
FROM runner_orders r
JOIN customer_orders c ON r.order_id = c.order_id
WHERE COALESCE(r.cancellation, '') NOT LIKE '%Cancellation%'
GROUP BY c.order_id,r.pickup_time, c.order_time
ORDER BY time_difference DESC;
```
| ORDER_ID | ORDER_COUNT | TIME_DIFFERENCE |
|----------|-------------|-----------------|
| 4        | 3           | 29:17.0         |
| 3        | 2           | 21:14.0         |
| 8        | 1           | 20:29.0         |
| 10       | 2           | 15:31.0         |
| 1        | 1           | 10:32.0         |
| 5        | 1           | 10:28.0         |
| 7        | 1           | 10:16.0         |
| 2        | 1           | 10:02.0         |


Answer: Yes, the more quantity of pizzas ordered, the longer the preparation time is.

#### Q4 What was the average distance travelled for each customer?
``` MySQL
SELECT c.customer_id, AVG(r.distance) AS avg_distance
FROM customer_orders AS c
INNER JOIN runner_orders AS r ON c.order_id = r.order_id
WHERE r.duration != 0
GROUP BY c.customer_id;
```

| customer_id | avg_distance |
|-------------|--------------|
| 101         | 20           |
| 102         | 16.73        |
| 103         | 23.4         |
| 104         | 10           |
| 105         | 25           |

#### Q5 What was the difference between the longest and shortest delivery times for all orders?
``` MySQL
WITH delivery_time_diff AS (
SELECT MAX(CASE WHEN duration NOT IN ('null', '') AND duration IS NOT NULL THEN duration END) AS max_delivery_time,
MIN(CASE WHEN duration IS NOT NULL THEN duration END) AS min_delivery_time
FROM runner_orders)

SELECT (max_delivery_time - min_delivery_time) AS time_diff
FROM delivery_time_diff;
```
| time_diff |
|-------------|
| 40       |


#### Q6 What was the average speed for each runner for each delivery and do you notice any trend for these values?
``` MySQL
SELECT runner_id, AVG(DISTANCE), AVG (distance / TIME_TO_SEC(duration) * 60 )AS average_speed
FROM runner_orders 
WHERE distance != 0
GROUP BY runner_id
ORDER BY average_speed DESC;
```

| runner_id | AVG(DISTANCE)| average_speed |
|----------|-------------|-----------------|
| 2        | 23.93           | 62.9       |
| 1        | 15.85           | 45.5361        |
| 3        | 10          | 40      |

Answer: The further the average distance, the longer the average speed is.

#### Q7 What is the successful delivery percentage for each runner?
``` MySQL
SELECT runner_id,
(COUNT(CASE WHEN COALESCE(cancellation, '') NOT LIKE '%Cancellation%' THEN 1 END) / COUNT(order_id))*100 AS successful_percentage
FROM runner_orders
GROUP BY runner_id;
```
| runner_id | successful_percentage|
|----------|-------------|
| 2        |100.00           |
| 1        | 75.00           | 
| 3        | 50.00          | 

