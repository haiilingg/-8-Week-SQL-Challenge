### A. Customer Nodes Exploration
### Q1 How many unique nodes are there on the Data Bank system?

```
SELECT COUNT(DISTINCT node_id)
FROM customer_nodes;
```

-- Q2 What is the number of nodes per region? 
SELECT region_id,COUNT(node_id) AS num_of_nodes
FROM customer_nodes
GROUP BY region_id
ORDER BY region_id;

-- Q3 How many customers are allocated to each region?
SELECT region_id,COUNT(DISTINCT customer_id) As no_of_customer
FROM customer_nodes
GROUP BY region_id
ORDER BY region_id;

-- Q4 How many days on average are customers reallocated to a different node?
SELECT AVG(DATEDIFF(end_date, start_date)) AS avg_number_of_day
FROM customer_nodes
WHERE end_date != '9999-12-31';

-- Q5 What is the median, 80th and 95th percentile for this same reallocation days metric for each region? cmi
SET @rowindex := -1;
 
SELECT region_id,AVG(reallocation_days) AS Median
FROM (SELECT region_id,@rowindex:=@rowindex + 1 AS rowindex,
(DATEDIFF(end_date, start_date)) AS reallocation_days
    FROM customer_nodes
    WHERE end_date != '9999-12-31' AND region_id IN ('1','2','3','4','5')
    GROUP BY region_id,end_date, start_date
    ORDER BY region_id, reallocation_days ) AS d
WHERE d.rowindex IN (FLOOR(@rowindex / 2), CEIL(@rowindex / 2))
GROUP BY region_id;


-- B. Customer Transactions 
-- Q1 What is the unique count and total amount for each transaction type?
SELECT txn_type, COUNT(*) AS unique_count, SUM(txn_amount) AS total_amount
FROM customer_transactions
GROUP BY txn_type;

-- Q2 What is the average total historical deposit counts and amounts for all customers?
SELECT COUNT(*)AS avg_deposit_count, AVG(txn_amount) AS avg_deposit_amount 
FROM customer_transactions
WHERE txn_type='deposit';

-- Q3 For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?
WITH valid_transactions AS (
  SELECT customer_id, MONTH(txn_date) AS month,
    SUM(CASE WHEN txn_type = 'deposit' THEN 1 ELSE 0 END) AS deposit_count,
    SUM(CASE WHEN txn_type = 'purchase' THEN 1 ELSE 0 END) AS purchase_count,
    SUM(CASE WHEN txn_type = 'withdrawal' THEN 1 ELSE 0 END) AS withdrawal_count
  FROM customer_transactions
  GROUP BY customer_id, MONTH(txn_date))

SELECT month,COUNT(DISTINCT customer_id) AS customer_count
FROM valid_transactions
WHERE deposit_count > 1 AND (purchase_count >= 1 OR withdrawal_count >= 1)
GROUP BY month
ORDER BY month;

-- Q4 What is the closing balance for each customer at the end of the month?
SELECT MONTH(txn_date) AS month,customer_id,SUM(txn_amount * CASE WHEN txn_type IN ('withdrawal', 'purchase') THEN -1 ELSE 1 END) AS total_net_balance
FROM customer_transactions
GROUP BY month,customer_id
ORDER BY month;
 
-- Q5 What is the percentage of customers who increase their closing balance by more than 5%?
WITH CustomerBalances AS ( 
SELECT MONTH(txn_date) AS month,customer_id,
 SUM(txn_amount * CASE WHEN txn_type IN ('withdrawal', 'purchase') THEN -1 ELSE 1 END) AS total_net_balance
FROM customer_transactions
GROUP BY month, customer_id)

SELECT month,
AVG(CASE WHEN total_net_balance > 0.05 * ABS(total_net_balance) THEN 1 ELSE 0 END)*100 AS percentage_increase
FROM CustomerBalances
GROUP BY month
ORDER BY month;
