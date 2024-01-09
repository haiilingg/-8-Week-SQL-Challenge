### B. Data Analysis Questions 
#### 1. How many customers has Foodie-Fi ever had?
``` MySQL
SELECT COUNT(DISTINCT customer_id) AS no_of_customers
FROM subscriptions;
```
| no_of_runners|
| --- |
|1000|

#### 2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value
``` MySQL
SELECT MONTH(start_date) AS start_month, COUNT(*) AS distribution
FROM subscriptions
WHERE plan_id = 0
GROUP BY start_month
ORDER BY start_month;
```

start_month| distribution|
| --- | --- |
1|88
2|68
3|94
4|81
5|88
6|79
7|89
8|88
9|87
10|79
11|75
12|84

#### 3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name
``` MySQL
SELECT s.plan_id, p.plan_name,COUNT(*) AS count
FROM subscriptions AS s
INNER JOIN plans AS p ON s.plan_id = p.plan_id
WHERE YEAR(s.start_date) > 2020
GROUP BY s.plan_id, p.plan_name
ORDER BY s.plan_id;
```

| plan_id | plan_name | count |
|----------|-------------|-----------------|
1|basic monthly|8
2|pro monthly|60
3|pro annual|63
4|churn|71

#### 4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?

``` MySQL
SELECT COUNT(DISTINCT customer_id) AS no_of_customers, 
ROUND(COUNT(DISTINCT customer_id)*100/(SELECT COUNT(DISTINCT customer_id) FROM subscriptions),1) AS percentage_of_customers
FROM subscriptions 
WHERE plan_id = 4;
```

|no_of_customers	|percentage_of_customers|
| --- | --- |
|307|	30.7|
