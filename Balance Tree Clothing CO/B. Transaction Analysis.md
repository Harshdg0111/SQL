# **🌲 - Balanced Tree Clothing Co.**

### **🧾 B. Transaction Analysis**

**1. How many unique transactions were there?**

```sql
select 
	count(distinct txn_id) as unique_transaction
from sales;
```
**Solution :**

|unique_transaction|
|--------------------|
|2500|

****

**2. What is the average unique products purchased in each transaction?**

```sql
select  
	round(avg(total_quantity)) as avg_unique_products
from (
	select 
	txn_id, 
	sum(qty) as total_quantity
	from sales s
	group by  txn_id)
	as total_quantities;
```
**Solution :**

|avg_unique_products|
|---------------------|
|18|

****

**3. What are the 25th, 50th and 75th percentile values for the revenue per transaction?**

```sql
select 
	percentile_cont(0.25) within group (order by x1.total_revenue) as percentile_25,
	percentile_cont(0.50) within group (order by x1.total_revenue) as percentile_50,
	percentile_cont(0.75) within group (order by x1.total_revenue) as percentile_75
from 
	(select txn_id,
	sum(price * qty) as total_revenue
	from sales
	group by 1) x1;
```
**Solution :**

|percentile_25|	percentile_50|	percentile_75|
|--------------|-----------|----------|
|375.75	|509.5|	647|

****

**4. What is the average discount value per transaction?**

```sql
select 
	round(avg(x1.discount_amt),2) as avg_discount
from 
	(select 
	txn_id,
	sum(qty * price * discount / 100) as discount_amt
	from sales
	group by 1) x1;
```
**Solution :**

|avg_discount|
|--------------|
|59.79|

****

**5. What is the percentage split of all transactions for members vs non-members?**

```sql
with unique_txn as(
select 
	member,
	count(distinct txn_id) as unique_transaction
from sales  
group by 1 )
select 
	member,
	unique_transaction,
	round(100* unique_transaction /
			 (select sum(unique_transaction)
			  from unique_txn))
from unique_txn
group by 1,2;
```
**Solution :**

|member|unique_transaction|round|
|---------|--------------------|---------|
|false|	995|	40|
|true |1505|	60|

****

**6. What is the average revenue for member transactions and non-member transactions?**

```sql
with txn_revenue as(
select 
	member,
	txn_id,
	sum(price * qty) as total_revenue
from sales 
group by 1,2 
)
select 
	member,
	round(avg(total_revenue),2) as avg_revenue
from txn_revenue
group by 1;
```
**Solution :**

|member|	avg_revenue|
|--------|---------------|
|false	|515.04|
|true	|516.27|

****