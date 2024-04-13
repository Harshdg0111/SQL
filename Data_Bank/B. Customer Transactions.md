# 👩‍💻 - Data Bank

### 🏦 B. Customer Transactions

**1. What is the unique count and total amount for each transaction type?**

```sql
/*
> FOP: txn_type, transaction_count, total_amount
> Conditions:
-- output unique count and total_amount of each txn_type
> PLAN:
-- use count() on customer_id
-- use sum on txn_amount
*/
SELECT 
        txn_type,
        count(*) AS unique_count,
        sum(txn_amount) AS total_amont
FROM customer_transactions
GROUP BY 1;
```
**Solution :**

txn_type|	unique_count	|total_amont|
|-----------|--------|-----------|
purchase	|1617	|806537
withdrawal	|1580|	793003
deposit	|2671	|1359168

****

**2. What is the average total historical deposit counts and amounts for all customers?**

```sql
/*
> FOP: customer_id, customer_amount
> Conditions:
-- output avg_dep_count, avg_dep_amount
> PLAN:
-- use cte for Customer_Deposite
-- use count() on *
-- use sum on txn_amount
*/
WITH customerDeposit AS (
SELECT 
    	customer_id,
    	txn_type,
    	COUNT(*) AS dep_count,
    	SUM(txn_amount) AS dep_amount
FROM customer_transactions
WHERE txn_type = 'deposit'
GROUP BY 1,2
)

SELECT
  		ROUND(AVG(dep_count)) AS avg_dep_count,
  		ROUND(AVG(dep_amount)) AS avg_dep_amount
FROM customerDeposit;
```
**Solution :**

average_deposit_count|	average_deposit_amount
|----------|-----
5	|508.86|

*****

**3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?**

```sql
/*
> FOP: Month, customer_id
> Conditions:
-- output month , customer_count
> PLAN:
-- use cte for Customer_transaction
-- use sum(case function) on txn_type
-- count (Custumber_id)
*/
WITH cte_transaction AS (
SELECT 
        customer_id,
        MONTH(txn_date) AS months,
        SUM(CASE WHEN txn_type = 'deposit' THEN 1 ELSE 0 END) AS deposit_count,
        SUM(CASE WHEN txn_type = 'purchase' THEN 1 ELSE 0 END) AS purchase_count,
        SUM(CASE WHEN txn_type = 'withdrawal' THEN 1 ELSE 0 END) AS withdrawal_count
FROM customer_transactions
GROUP BY 1,2
)
SELECT 
        months,
        COUNT(customer_id) AS customer_count
FROM cte_transaction
WHERE deposit_count > 1
AND (purchase_count = 1 OR withdrawal_count = 1)
GROUP BY 1;
```
**Solution :**

months|	customer_count
-------|--------
1	|115
2	|108
3	|113
4|	50

****

**4. What is the closing balance for each customer at the end of the month?**

```sql
/*
> FOP: Month, customer_id
> Conditions:
-- output month , customer_count
> PLAN:
-- use cte for Customer_transaction
-- use sum(case function) on txn_type
-- count (Custumber_id)
*/
WITH txn_monthly_balance_cte AS(
SELECT 
	 customer_id,
         txn_amount,
         EXTRACT(MONTH FROM txn_date) AS txn_month,
         SUM(CASE
                  WHEN txn_type= 'deposit' THEN txn_amount
                  ELSE -txn_amount
              END) AS net_transaction_amt
FROM customer_transactions
GROUP BY 1,2,3
ORDER BY 1
)
SELECT 
	customer_id,
       	txn_month,
       	net_transaction_amt,
       	sum(net_transaction_amt) over(PARTITION BY customer_id
                                     ORDER BY txn_month ROWS BETWEEN UNBOUNDED preceding AND CURRENT ROW) AS closing_balance
FROM txn_monthly_balance_cte;

```
**Solution :**

customer_id	|end_date|	transactions	|closing_balance|
|--------|------|--------|-------|
1	|2020-01-31	|312	|312|
1	|2020-02-29	|0	|312|
1|	2020-03-31	|-952	|-640|
1	|2020-04-30	|0	|-640
2|	2020-01-31	|549|	549

****

**5. What is the percentage of customers who increase their closing balance by more than 5%??**

```sql
/*
> FOP: customer_id,eomonth
> Conditions:
-- output percentage_of_customers
> PLAN:
--CTE 1: Monthly transactions of each customer (Q4)
--CTE 2: Increment last days of each month till they are equal to @maxDate (Q4)
-- CTE 3: Closing balance of each customer by monthly (Q4)
--CTE 4: CTE 3 & next_balance
--CTE 5: Calculate the increase percentage of closing balance for each customer
--Create a temporary table because of the error: Null value is eliminated by an aggregate or other SET operation
--Calculate the percentage of customers whose closing balance increasing 5% compared to the previous month
*/

WITH monthly_transactions AS (
SELECT
        customer_id,
        EOMONTH(txn_date) AS end_date,
        SUM(CASE WHEN txn_type IN ('withdrawal', 'purchase') THEN -txn_amount
             ELSE txn_amount 
        END) AS transactions
FROM customer_transactions
GROUP BY customer_id, EOMONTH(txn_date)
),
recursive_dates AS (
SELECT
        DISTINCT customer_id,
        CAST('2020-01-31' AS DATE) AS end_date
FROM customer_transactions
UNION ALL
SELECT 
        customer_id,
        EOMONTH(DATEADD(MONTH, 1, end_date)) AS end_date
FROM recursive_dates
WHERE EOMONTH(DATEADD(MONTH, 1, end_date)) <= @maxDate
),
customers_balance AS (
SELECT 
        r.customer_id,
        r.end_date,
        COALESCE(m.transactions, 0) AS transactions,
        SUM(m.transactions) OVER (PARTITION BY r.customer_id ORDER BY r.end_date 
        ROWS UNBOUNDED PRECEDING) AS closing_balance
FROM recursive_dates r
LEFT JOIN  monthly_transactions m
ON r.customer_id = m.customer_id
AND r.end_date = m.end_date
),
customers_next_balance AS (
SELECT *,
    LEAD(closing_balance) OVER(PARTITION BY customer_id ORDER BY end_date) AS next_balance
FROM customers_balance
),
pct_increase AS (
SELECT *,
    100.0*(next_balance-closing_balance)/closing_balance AS pct
FROM customers_next_balance
WHERE closing_balance ! = 0 AND next_balance IS NOT NULL
)
SELECT *
INTO #temp
FROM pct_increase;
SELECT 
        CAST(100.0*COUNT(DISTINCT customer_id) AS FLOAT)
      / (SELECT COUNT(DISTINCT customer_id) FROM customer_transactions) AS percentage_of_customers
FROM #temp
WHERE pct > 5;
```
**Solution :**

|percentage_of_customers|
|-------------------|
75.8

****
