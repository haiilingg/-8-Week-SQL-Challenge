### 1. How many unique transactions were there?
``` MYSQL
SELECT COUNT(DISTINCT txn_id)
FROM balanced_tree.sales;
```

### 2. What is the average unique products purchased in each transaction?
``` MYSQL
WITH unique_products_cte AS
(SELECT COUNT(DISTINCT prod_id) AS unique_products
    FROM balanced_tree.sales
    GROUP BY txn_id)

SELECT ROUND(AVG(unique_products), 2) AS average_unique_products
FROM unique_products_cte;

```

### 3. What are the 25th, 50th and 75th percentile values for the revenue per transaction?



### 4. What is the average discount value per transaction?
``` MYSQL
SELECT SUM(discount/100 * qty*price)/COUNT(DISTINCT txn_id) AS average_discount
FROM balanced_tree.sales;
```


### 5. What is the percentage split of all transactions for members vs non-members?
```MYSQL
SELECT member,
ROUND(100* COUNT(member) / (SELECT COUNT(txn_id) 
FROM balanced_tree.sales), 2) AS member_percentage  
FROM balanced_tree.sales
GROUP BY member;
```


### 6. What is the average revenue for member transactions and non-member transactions
```MYSQL
SELECT member,
ROUND(SUM((price*qty)*(1-(discount)/100)),2) AS average_revenue  
FROM balanced_tree.sales
GROUP BY member;
```
