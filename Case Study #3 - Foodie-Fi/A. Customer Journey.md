### Question: 
Based off the 8 sample customers provided in the sample from the subscriptions table, write a brief description about each customer’s onboarding journey.

#### Code:
``` MySQL
SELECT s.customer_id, s.plan_id, p.plan_name,s.start_date
FROM subscriptions AS s
INNER JOIN plans AS p ON s.plan_id = p.plan_id
WHERE s.customer_id IN (1,2,11,13,15,16,18,19);
```

#### Output:
customer_id	|plan_id	|plan_name	|start_date
--|--|--|--|
1|0|trial|01/08/2020
1|1|basic monthly|08/08/2020
2|0|trial|20/09/2020
2|3|pro annual|27/09/2020
11|0|trial|19/11/2020
11|4|churn|26/11/2020
13|0|trial|15/12/2020
13|1|basic monthly|22/12/2020
13|2|pro monthly|29/03/2021
15|0|trial|17/03/2020
15|2|pro monthly|24/03/2020
15|4|churn|29/04/2020
16|0|trial|31/05/2020
16|1|basic monthly|07/06/2020
16|3|pro annual|21/10/2020
18|0|trial|06/07/2020
18|2|pro monthly|13/07/2020
19|0|trial|22/06/2020
19|2|pro monthly|29/06/2020
19|3|pro annual|29/08/2020
