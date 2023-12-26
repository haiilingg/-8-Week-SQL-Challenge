### A. Customer Nodes Exploration
#### Q1 How many unique nodes are there on the Data Bank system?

``` MYSQL
SELECT COUNT(DISTINCT node_id) AS unique_nodes
FROM customer_nodes;
```

Output:
| unique_nodes|
| --- | 
|5| 

#### Q2 What is the number of nodes per region? 

``` MYSQL
SELECT region_id,COUNT(node_id) AS num_of_nodes
FROM customer_nodes
GROUP BY region_id
ORDER BY region_id;
```
Output:
|region_id|num_of_nodes|
| --- | --- | 
|1|1540| 
|2|1470|
|3|1428|
|4|1330|
|5|1232|

#### Q3 How many customers are allocated to each region?

``` MYSQL
SELECT region_id,COUNT(DISTINCT customer_id) As no_of_customer
FROM customer_nodes
GROUP BY region_id
ORDER BY region_id;
```

Output:
|region_id|no_of_customer|
| --- | --- | 
|1|110| 
|2|105|
|3|102|
|4|95|
|5|88|


#### Q4 How many days on average are customers reallocated to a different node?

``` MYSQL
SELECT AVG(DATEDIFF(end_date, start_date)) AS avg_number_of_day
FROM customer_nodes
WHERE end_date != '9999-12-31';
```
Output:
|avg_number_of_day|
| --- | 
|14.6340|
