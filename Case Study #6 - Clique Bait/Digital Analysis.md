#### 1. How many users are there?
```Mysql
SELECT COUNT(DISTINCT user_id)
FROM clique_bait.users;
```

#### 2.How many cookies does each user have on average?
```Mysql
SELECT ROUND(COUNT(cookie_id)/COUNT(DISTINCT user_id),2) AS avg_cookies
FROM clique_bait.users;
```

#### 3.What is the unique number of visits by all users per month?
```Mysql
SELECT MONTH(event_time) AS Month, COUNT(DISTINCT visit_id) AS unique_visits
FROM clique_bait.events
GROUP BY Month;
```

#### 4.What is the number of events for each event type?
```Mysql
SELECT e.event_type, ei.event_name, COUNT(e.event_type) AS number_of_events
FROM clique_bait.events e
INNER JOIN clique_bait.event_identifier ei ON e.event_type = ei.event_type
GROUP BY e.event_type, ei.event_name;
```

#### 5.What is the percentage of visits which have a purchase event?
```Mysql
SELECT e.event_type, ei.event_name,
ROUND(100* COUNT(distinct e.visit_id) / (SELECT COUNT(distinct e.visit_id) FROM clique_bait.events e ), 2) AS visits_percentage
FROM clique_bait.events e
INNER JOIN clique_bait.event_identifier ei ON e.event_type = ei.event_type
WHERE e.event_type ='3'
GROUP BY e.event_type, ei.event_name;
```

#### 6.What is the percentage of visits which view the checkout page but do not have a purchase event?
```Mysql
WITH checkout_page_view AS (
    SELECT COUNT(visit_id) AS visits
    FROM clique_bait.events AS e
        JOIN clique_bait.event_identifier AS ei ON e.event_type = ei.event_type
        JOIN clique_bait.page_hierarchy AS p ON e.page_id = p.page_id
    WHERE p.page_name = 'Checkout' AND ei.event_name != 'Purchase')

SELECT ROUND((100 * visits / (SELECT (e.visit_id)
FROM clique_bait.events e)), 2) AS checkout_no_purchase_percentage
FROM checkout_page_view;
```

#### 7.What are the top 3 pages by number of views?
```Mysql
SELECT e.page_id,ph.page_name,SUM(CASE WHEN e.event_type ='1' then 1 else 0 end) AS views
FROM clique_bait.events e
INNER JOIN clique_bait.page_hierarchy ph ON e.page_id = ph.page_id
GROUP BY e.page_id,ph.page_name
ORDER BY views DESC
LIMIT 3;
```

#### 8.What is the number of views and cart adds for each product category?
```Mysql
SELECT ph.product_category,
SUM(CASE WHEN e.event_type ='1' then 1 else 0 end) AS views,
SUM(CASE WHEN e.event_type ='2' then 1 else 0 end) AS cart_adds
FROM clique_bait.page_hierarchy ph
INNER JOIN clique_bait.events e ON ph.page_id = e.page_id
Where ph.product_category is not null
GROUP BY ph.product_category
ORDER BY views DESC;
```

#### 9.What are the top 3 products by purchases?
```Mysql
WITH purchase_cte AS(
SELECT visit_id
FROM clique_bait.events
WHERE event_type ='3')

SELECT ph.page_name, COUNT(*) AS total_purchases
FROM clique_bait.page_hierarchy ph
INNER JOIN clique_bait.events e ON ph.page_id = e.page_id
INNER JOIN purchase_cte p ON e.visit_id = p.visit_id
WHERE event_type ='2'
GROUP BY ph.page_name
ORDER BY total_purchases DESC
LIMIT 3;
```
