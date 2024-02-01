#### Q1 What are the top 3 products by total revenue before discount?
``` MySQL
SELECT s.prod_id,pd.product_name, SUM(s.qty * s.price) AS revenue
FROM balanced_tree.sales s
INNER JOIN balanced_tree.product_details pd ON s.prod_id = pd.product_id
GROUP BY s.prod_id,pd.product_name
ORDER BY revenue DESC
LIMIT 3;
```

#### Output:
|prod_id|product_name|revenue|
|--|--|--|
|2a2353|Blue Polo Shirt - Mens	|217683|
|9ec847|Grey Fashion Jacket - Womens	|209304|
|5d267b|	White Tee Shirt - Mens	|152000|

#### Q2 What is the total quantity, revenue and discount for each segment?
```mysql
SELECT pd.segment_id,pd.segment_name, SUM(s.qty) AS total_quantity,
SUM(s.qty * s.price) AS revenue, SUM(s.qty * s.price* (s. discount/100)) AS total_discount
FROM balanced_tree.sales s
INNER JOIN balanced_tree.product_details pd ON s.prod_id = pd.product_id
GROUP BY pd.segment_id,pd.segment_name;
```

#### Output:
|segment_id|segment_name|total_quantity|revenue|total_discount|
|---|---|---|---|---|
|3|Jeans|11349|208350|25343.97|
|5|Shirt|11265|406143|49594.27|
|6|Socks|11217|307977|37013.44|
|4|Jacket|11385|366983|44277.46|

#### Q3 What is the top selling product for each segment?
``` mysql
WITH top_selling_product AS (
SELECT pd.segment_id,pd.segment_name, s.prod_id,pd.product_name, SUM(s.qty) AS total_quantity,
RANK() OVER (PARTITION BY pd.segment_id ORDER BY SUM(s.qty) DESC) AS row_num
FROM balanced_tree.sales s
INNER JOIN balanced_tree.product_details pd ON s.prod_id = pd.product_id
GROUP BY pd.segment_id,pd.segment_name,s.prod_id,pd.product_name)

SELECT segment_id,segment_name,prod_id,product_name,total_quantity
FROM top_selling_product
WHERE row_num =1;
```

#### Output:
|segment_id|segment_name|prod_id|product_name|total_quantity|
|---|---|---|---|---|
|3|Jeans|c4a632|Navy Oversized Jeans - Womens|3856|
|4|Jacket|9ec847|Grey Fashion Jacket - Womens|3876|
|5|Shirt|2a2353|Blue Polo Shirt - Mens|3819|
|6|Socks|f084eb|Navy Solid Socks - Mens|3792|

#### Q4 What is the total quantity, revenue and discount for each category?
``` Mysql
SELECT pd.category_id,pd.category_name, SUM(s.qty) AS total_quantity,
SUM(s.qty * s.price) AS revenue, SUM(s.qty * s.price* (s. discount/100)) AS total_discount
FROM balanced_tree.sales s
INNER JOIN balanced_tree.product_details pd ON s.prod_id = pd.product_id
GROUP BY pd.category_id,pd.category_name;
```

#### Output:
|category_id|category_name|total_quantity|revenue|total_discount|
|---|---|---|---|---|
|1|Womens|22734|575333|69621.43|
|2|Mens|22482|714120|86607.71|

#### Q5 What is the top selling product for each category?
``` Mysql
WITH top_selling_product AS (
SELECT pd.category_id,pd.category_name, s.prod_id,pd.product_name, SUM(s.qty) AS total_quantity,
RANK() OVER (PARTITION BY pd.category_id ORDER BY SUM(s.qty) DESC) AS row_num
FROM balanced_tree.sales s
INNER JOIN balanced_tree.product_details pd ON s.prod_id = pd.product_id
GROUP BY pd.category_id,pd.category_name,s.prod_id,pd.product_name)

SELECT category_id,category_name,prod_id,product_name,total_quantity
FROM top_selling_product
WHERE row_num =1;
```

#### Output:
|category_id|category_name|prod_id|product_name|total_quantity|
|--|--|--|--|--|
|1|Womens|9ec847|Grey Fashion Jacket - Womens|3876|
|2|Mens	|2a2353	|Blue Polo Shirt - Mens	|3819|

#### Q6 What is the percentage split of revenue by product for each segment?
```Mysql
WITH split_cte AS
(SELECT pd.segment_id,pd.segment_name,s.prod_id,pd.product_name, SUM(s.qty * s.price) AS revenue
 FROM balanced_tree.sales s
INNER JOIN balanced_tree.product_details pd ON s.prod_id = pd.product_id
GROUP BY pd.segment_id,pd.segment_name,s.prod_id,pd.product_name)

SELECT *, ROUND(100* revenue / (SUM(revenue) OVER (PARTITION BY segment_id)), 2) AS revenue_percentage
FROM split_cte 
ORDER BY segment_id;
```

#### Output:

#### Q7 What is the percentage split of revenue by segment for each category?
```Mysql
WITH split_cte AS
(SELECT pd.category_id,pd.category_name,pd.segment_id,pd.segment_name, SUM(s.qty * s.price) AS revenue
 FROM balanced_tree.sales s
INNER JOIN balanced_tree.product_details pd ON s.prod_id = pd.product_id
GROUP BY pd.category_id,pd.category_name,pd.segment_id,pd.segment_name)

SELECT *, ROUND(100* revenue / (SUM(revenue) OVER (PARTITION BY category_id)), 2) AS revenue_percentage
FROM split_cte 
ORDER BY category_id;
```
#### Output:

#### Q8 What is the percentage split of total revenue by category?
```mysql
SELECT pd.category_id,pd.category_name,
ROUND(100* SUM(s.qty * s.price) / (SELECT SUM(s.qty * s.price) AS revenue FROM balanced_tree.sales s ), 2) AS revenue_percentage  
FROM balanced_tree.sales s
INNER JOIN balanced_tree.product_details pd ON s.prod_id = pd.product_id
GROUP BY pd.category_id,pd.category_name;
```

#### Output:

#### Q9 What is the total transaction “penetration” for each product? (hint: penetration = number of transactions where at least 1 quantity of a product was purchased divided by total number of transactions)
```mysql
SELECT s.prod_id, pd.product_name,ROUND(COUNT(s.txn_id)/ (SELECT COUNT(DISTINCT s.txn_id) 
FROM balanced_tree.sales s), 3) AS penetration
FROM balanced_tree.sales s
INNER JOIN balanced_tree.product_details pd ON s.prod_id = pd.product_id
GROUP BY s.prod_id,pd.product_name;
```

#### Output:

#### Q10 What is the most common combination of at least 1 quantity of any 3 products in a 1 single transaction?
```mysql
SELECT s.prod_id,pd.product_name,s1.prod_id,pd1.product_name, s2.prod_id, pd2.product_name,COUNT(*) AS frequency
FROM balanced_tree.sales s
JOIN balanced_tree.sales s1 ON s1.txn_id = s.txn_id
AND s.prod_id < s1.prod_id 
JOIN balanced_tree.sales s2 ON s2.txn_id = s1.txn_id
AND s1.prod_id < s2.prod_id
INNER JOIN balanced_tree.product_details pd ON s.prod_id = pd.product_id
INNER JOIN balanced_tree.product_details pd1 ON s1.prod_id = pd1.product_id
INNER JOIN balanced_tree.product_details pd2 ON s2.prod_id = pd2.product_id
GROUP BY s.prod_id,s1.prod_id, s2.prod_id,pd.product_name,pd1.product_name, pd2.product_name
ORDER BY 7 DESC
LIMIT 1;
```

#### Output:


