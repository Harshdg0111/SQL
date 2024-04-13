# 👩‍💻 - Data Bank

### 🏦 A. Customer Nodes Exploration

**1. How many unique nodes are there on the Data Bank system?**

```sql
/*
> FOP: node_id
> Conditions:
-- output unique nodes_id
> PLAN:
-- use count(distinct) on nodes_id
*/
SELECT 
        COUNT(DISTINCT node_id) AS unique_nodes
FROM customer_nodes;
```
**Solution :**

|unique_nodes|
|------------|
5

****

**2. What is the number of nodes per region**

```sql
/*
> FOP: region_id, region_name, node_id
> Conditions:
-- output unique nodes_id 
-- group by region_id , region_name
> PLAN:
-- use count() on nodes_id
*/
SELECT 
        r.region_id,
        r.region_name,
        COUNT(n.node_id) AS nodes
FROM customer_nodes n
JOIN regions r
ON n.region_id = r.region_id
GROUP BY 1,2
ORDER BY 1;
```
**Solution :**

|region_id	|region_name|nodes|
|----------|------------|-------------|
1	|Australia	|770|
2	|America	|735|
3	|Africa	|714
4|Asia|665
5	|Europe|616

****

**3. How many customers are allocated to each region?**

```sql
/*
> FOP: region_id, customer_id
> Conditions:
-- output unique customer_id
> PLAN:
-- use count() on customer_id
*/
SELECT 
        r.region_id,
        r.region_name,
        COUNT(DISTINCT n.customer_id) AS customers
FROM customer_nodes n
JOIN regions r
ON n.region_id = r.region_id
GROUP BY 1,2
ORDER BY 1;
```
**Solution :**
region_id|	region_name|customers
|----------|----------|----|
1	|Australia|	110|
2	|America|	105
3	|Africa|	102
4	|Asia|	95
5|	Europe|	88

****

**4. How many days on average are customers reallocated to a different node?**

```sql
/*
> FOP: region_id, customer_id
> Conditions:
-- output unique customer_id
> PLAN:
-- use count() on customer_id
*/
WITH node_days AS (
SELECT 
        customer_id, 
        node_id,
        end_date - start_date AS days_in_node
FROM customer_nodes
WHERE end_date != '9999-12-31'
GROUP BY 1,2, start_date, end_date
) , 
total_node_days AS (
SELECT 
        customer_id,
        node_id,
        SUM(days_in_node) AS total_days_in_node
FROM node_days
GROUP BY 1,2
)
SELECT ROUND(AVG(total_days_in_node)) AS avg_node_reallocation_days
FROM total_node_days;
```
**Solution :**

|avg_node_reallocation_days|
|----------------|
24

****

**5. What is the median, 80th and 95th percentile for this same reallocation days metric for each region?**

```sql
/*
> FOP: region_id, customer_id
> Conditions:
-- output unique customer_id
> PLAN:
-- use count() on customer_id
*/
WITH customerDates AS (
SELECT 
        customer_id,
        region_id,
        node_id,
        MIN(start_date) AS first_date
FROM customer_nodes
GROUP BY 1,2,3
),
reallocation AS (
SELECT
        customer_id,
        region_id,
        node_id,
        first_date,
        DATEDIFF(DAY, first_date, 
                LEAD(first_date) OVER(PARTITION BY customer_id 
                                   ORDER BY first_date)) AS moving_days
FROM customerDates
)
SELECT 
        DISTINCT r.region_id,
        rg.region_name,
        PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY r.moving_days) OVER(PARTITION BY r.region_id) AS median,
        PERCENTILE_CONT(0.8) WITHIN GROUP (ORDER BY r.moving_days) OVER(PARTITION BY r.region_id) AS percentile_80,
        PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY r.moving_days) OVER(PARTITION BY r.region_id) AS percentile_95
FROM reallocation r
JOIN regions rg ON r.region_id = rg.region_id
WHERE moving_days IS NOT NULL;
```
**Solution :**

region_id|	region_name|	percentile_50	|percentile_80|	percentile_95|
|----------|-----------|-------|----------|---|
1	|Australia|	22	|31	|54
2	|America	|21	|33.2	|57
3	|Africa	|21	|33.2|	58.8
4	|Asia	|22	|32.4	|49.85
5|	Europe	|22|	31|	54.3

****

