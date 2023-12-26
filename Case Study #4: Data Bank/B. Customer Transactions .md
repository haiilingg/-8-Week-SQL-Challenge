### B. Customer Transactions 
#### Q1 What is the unique count and total amount for each transaction type?

``` MYSQL
SELECT txn_type, COUNT(*) AS unique_count, SUM(txn_amount) AS total_amount
FROM customer_transactions
GROUP BY txn_type;
```

Output:
|txn_type|unique_count|total_amount|
| --- | --- | --- |
|deposit|2671|1359168|
|withdrawal|1580| x |
|purchase|1617| x |

#### Q2 What is the average total historical deposit counts and amounts for all customers?

``` MYSQL
SELECT COUNT(*)AS avg_deposit_count, AVG(txn_amount) AS avg_deposit_amount 
FROM customer_transactions
WHERE txn_type='deposit';
```
Output:
|avg_deposit_count|avg_deposit_amount|
| --- | --- | 
|2671|508.8611| 

#### Q3 For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?

``` MYSQL
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
```
Output:
|month|customer_count|
| --- | --- | 
|1|168| 
|2|181|
|3|192|
|4|70|

#### Q4 What is the closing balance for each customer at the end of the month?

``` MYSQL
SELECT MONTH(txn_date) AS month,customer_id,SUM(txn_amount * CASE WHEN txn_type IN ('withdrawal', 'purchase') THEN -1 ELSE 1 END) AS total_net_balance
FROM customer_transactions
GROUP BY month,customer_id
ORDER BY month;
 ```
Output(only showing top 5, refer to SQL script for full results):
|month|customer_id|total_net_balance|
| --- | --- | --- | 
|1|1|312| 
|1|2|549|
|1|3|144|
|1|4|848|
|1|5|954|

#### Q5 What is the percentage of customers who increase their closing balance by more than 5%?

``` MYSQL
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

Output:
|month|percentage_increase|
| --- | --- | 
|1|68.6000| 
|2|41.3187|
|3|37.7193|
|4|40.4531|
