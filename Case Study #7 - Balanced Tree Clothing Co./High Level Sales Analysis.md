### Q1 What was the total quantity sold for all products?

``` MYSQL
SELECT sum(qty) AS total_quantity_sold
FROM balanced_tree.sales ;
```
#### Output:
|total_quantity_sold| 
|--|
|45216|

### Q2 What is the total generated revenue for all products before discounts?
``` MYSQL
SELECT SUM(price*qty) AS total_revenue  
FROM balanced_tree.sales; 
```
#### Output:
|total_revenue| 
|--|
|1289453|

### Q3 What was the total discount amount for all products?
``` MYSQL
SELECT ROUND(SUM(price*qty*discount/100),2) AS total_discount 
FROM balanced_tree.sales ;
```

#### Output:
|total_discount| 
|--|
|156229.14|
