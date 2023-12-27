### C. Ingredient Optimisation
#### Q1 What are the standard ingredients for each pizza?
``` MySQL
DELIMITER //

CREATE PROCEDURE ProcessPizzaToppings()
BEGIN
    DECLARE done INT DEFAULT FALSE;
    DECLARE orderId INT;
    DECLARE pizzaToppings VARCHAR(255);
    DECLARE cur CURSOR FOR
        SELECT pizza_id, toppings
        FROM pizza_recipes;

    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;

    CREATE TEMPORARY TABLE IF NOT EXISTS temp_toppings (pizza_id INT,topping_id INT);

    OPEN cur;

    read_loop: LOOP
        FETCH cur INTO orderId, pizzaToppings;
        IF done THEN LEAVE read_loop;
        END IF;

        -- Split toppings and insert into temporary table
        INSERT INTO temp_toppings (pizza_id, topping_id)
        SELECT orderId, CAST(TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(pizzaToppings, ',', n.digit + 1), ',', -1)) AS SIGNED) AS topping_id
        FROM (SELECT 0 AS digit UNION ALL SELECT 1 UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4 UNION ALL SELECT 5 UNION ALL SELECT 6 UNION ALL SELECT 7) n
        WHERE n.digit < LENGTH(pizzaToppings) - LENGTH(REPLACE(pizzaToppings, ',', '')) + 1;
    END LOOP;
    CLOSE cur;
```

#### Select combined toppings names
``` MySQL
    SELECT tt.pizza_id,GROUP_CONCAT(pt.topping_name ORDER BY tt.topping_id SEPARATOR ', ') AS Standard_toppings
    FROM temp_toppings tt
    INNER JOIN pizza_toppings pt ON tt.topping_id = pt.topping_id
    GROUP BY tt.pizza_id;
    DROP TEMPORARY TABLE IF EXISTS temp_toppings;
END //

DELIMITER ;

CALL ProcessPizzaToppings();
```

Output:
|pizza_id|Standard_toppings|
|------------|---------------|
|1	|Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami|
|2|Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce|

#### Q2 What was the most commonly added extra?
``` MySQL
SELECT p.topping_name, COUNT(c.extras) AS added_extra
FROM customer_orders AS c
INNER JOIN pizza_toppings AS p ON c.extras = p.topping_id
WHERE c.extras NOT IN ('null', '')
GROUP BY p.topping_name
ORDER BY added_extra DESC;
```
Output:
| topping_name  | added_extra |
|------------|---------------|
| Bacon     | 4              |
  
#### Q3 What was the most common exclusion?
``` MySQL
SELECT p.topping_name, COUNT(c.exclusions) AS exclusions_no
FROM customer_orders AS c
INNER JOIN pizza_toppings AS p ON c.exclusions = p.topping_id
WHERE c.exclusions NOT IN ('null', '')
GROUP BY p.topping_name
ORDER BY exclusions_no DESC
LIMIT 1;
```
Output:
| topping_name  | exclusions_no |
|------------|---------------|
| Cheese       | 4              |


#### Q4 Generate an order item for each record in the customers_orders table in the format of one of the following:
- Meat Lovers
``` MySQL
SELECT co.order_id,co.customer_id, co.pizza_id,co.exclusions,co.extras,co.order_time,
CASE WHEN pn.pizza_name IS NOT NULL THEN pn.pizza_name
ELSE 'Unknown Pizza'
END AS order_item
FROM customer_orders co
LEFT JOIN pizza_names pn ON co.pizza_id = pn.pizza_id;
```

Output:
| order_id | customer_id | pizza_id | exclusions | extras   | order_time           | order_item  |
|-----------|-------------|----------|------------|----------|----------------------|-------------|
| 1         | 101         | 1        |            |          | 01/01/2020 18:05     | Meatlovers  |
| 2         | 101         | 1        |            |          | 01/01/2020 19:00     | Meatlovers  |
| 3         | 102         | 1        |            |          | 02/01/2020 23:51     | Meatlovers  |
| 3         | 102         | 2        | NULL       |          | 02/01/2020 23:51     | Vegetarian  |
| 4         | 103         | 1        | 4          |          | 04/01/2020 13:23     | Meatlovers  |
| 4         | 103         | 1        | 4          |          | 04/01/2020 13:23     | Meatlovers  |
| 4         | 103         | 2        | 4          |          | 04/01/2020 13:23     | Vegetarian  |
| 5         | 104         | 1        | NULL       | 1        | 08/01/2020 21:00     | Meatlovers  |
| 6         | 101         | 2        | NULL       | NULL     | 08/01/2020 21:03     | Vegetarian  |
| 7         | 105         | 2        | NULL       | 1        | 08/01/2020 21:20     | Vegetarian  |
| 8         | 102         | 1        | NULL       | NULL     | 09/01/2020 23:54     | Meatlovers  |
| 9         | 103         | 1        | 4          | 1, 5     | 10/01/2020 11:22     | Meatlovers  |
| 10        | 104         | 1        | NULL       | NULL     | 11/01/2020 18:34     | Meatlovers  |
| 10        | 104         | 1        | 2, 6       | 1, 4     | 11/01/2020 18:34     | Meatlovers  |


- Meat Lovers - Exclude Beef
- Meat Lovers - Extra Bacon
- Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers

#### Q5 Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients. For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"
TBC


#### Q6 What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?
``` MySQL
WITH PizzaIngredients AS (SELECT c.pizza_id,
        CAST(TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(p.toppings, ',', 1), ',', -1)) AS UNSIGNED) AS ingredient1,
        CAST(TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(p.toppings, ',', 2), ',', -1)) AS UNSIGNED) AS ingredient2,
        CAST(TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(p.toppings, ',', 3), ',', -1)) AS UNSIGNED) AS ingredient3,
        CAST(TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(p.toppings, ',', -1), ',', -1)) AS UNSIGNED) AS last_ingredient
    FROM customer_orders AS c
    JOIN pizza_recipes AS p ON c.pizza_id = p.pizza_id
    JOIN runner_orders AS r ON c.order_id = r.order_id
    WHERE COALESCE(r.cancellation, '') NOT LIKE '%Cancellation%')

SELECT pt.topping_id,pt.topping_name, COUNT(pi.pizza_id) AS total_quantity
FROM pizza_toppings AS pt
JOIN PizzaIngredients AS pi ON pt.topping_id IN (pi.ingredient1, pi.ingredient2, pi.ingredient3, pi.last_ingredient)
GROUP BY pt.topping_id, pt.topping_name
ORDER BY total_quantity DESC;
```

Output:
| topping_id | topping_name  | total_quantity |
|------------|---------------|-----------------|
| 1          | Bacon         | 9               |
| 2          | BBQ Sauce     | 9               |
| 3          | Beef          | 9               |
| 10         | Salami        | 9               |
| 4          | Cheese        | 3               |
| 6          | Mushrooms     | 3               |
| 7          | Onions        | 3               |
| 12         | Tomato Sauce  | 3               |

