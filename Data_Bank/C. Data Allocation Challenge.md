# 👩‍💻 - Data Bank

### 🏦 A. Customer Nodes Exploration

To test out a few different hypotheses - the Data Bank team wants to run an experiment where different groups of customers would be allocated data using 3 different options:

* Option 1: data is allocated based off the amount of money at the end of the previous month
* Option 2: data is allocated on the average amount of money kept in the account in the previous 30 days
* Option 3: data is updated real-time
For this multi-part challenge question - you have been requested to generate the following data elements to help the Data Bank team estimate how much data will need to be provisioned for each option:

* running customer balance column that includes the impact each transaction

**Option 1: Data is allocated based off the amount of money at the end of the previous month**

How much data would have been required on a monthly basis?

```sql
WITH transaction_amt_cte AS(
SELECT 
        *,
        month(txn_date) AS txn_month,
        SUM(CASE
                WHEN txn_type="deposit" THEN txn_amount
                ELSE -txn_amount
            END) AS net_transaction_amt
FROM customer_transactions
GROUP BY 1,2
ORDER BY 1,2
),
running_customer_balance_cte AS(
SELECT 
        customer_id,
        txn_date,
        txn_month,
        txn_type,
        txn_amount,
        sum(net_transaction_amt) over(PARTITION BY customer_idORDER BY txn_month ROWS BETWEEN UNBOUNDED preceding AND CURRENT ROW) AS running_customer_balance
FROM transaction_amt_cte
),
     month_end_balance_cte AS(
SELECT 
        *,
        last_value(running_customer_balance over(PARTITION BY customer_id, txn_month ORDER BY txn_month) AS month_end_balance
FROM running_customer_balance_cte),
customer_month_end_balance_cte AS(
SELECT 
        customer_id,
        txn_month,
        month_end_balance
FROM month_end_balance_cte
GROUP BY 1,2
)
SELECT 
        txn_month,
        sum(month_end_balance) AS data_required_per_month
FROM customer_month_end_balance_cte
GROUP BY 1
ORDER BY 1
```

**Solution :**

txn_month | data_required_per_month
----------|----------------
1|126091
2|-34350
3|-194916
4|-180855

****