# **🥑 - Foodie-Fi**

### **B. Data Analysis Questions**

**1. How many customers has Foodie-Fi ever had?**

```sql
/*
> FOP:
> Conditions:
> PLAN:
*/
SELECT 
	COUNT(DISTINCT customer_id) AS unique_customers
FROM subscriptions;
```
**Solution :**

|unique_customers|
|-----------------|
|1000|

**2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value**

```sql
/*
> FOP:month, distribution value
> Conditions:
-- output count(month) , count *
> PLAN:
-- join subcription on plan
*/
SELECT 
  	EXTRACT(MONTH FROM start_date) AS months,
  	COUNT(*) AS distribution_values
FROM subscriptions s
JOIN plans p ON s.plan_id = p.plan_id
WHERE p.plan_name = 'trial'
GROUP BY 1;
```
**Solution :**

|months	|distribution_values|
|----------|----------------|
3	|94
7	|89
2|	68
11|	75
12|	84
1|	88
4	|81
9	|87
5	|88
6	|79
10|	79
8	|88

****

**3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name**

```sql
/*
> FOP: plan id, plan name
> Conditions:
-- output count on customerid
> PLAN:
-- start date > 2021-01-01
*/
SELECT 
  	p.plan_id,
  	p.plan_name,
  	COUNT(s.customer_id) AS num_of_events
FROM subscriptions  s
JOIN plans p ON s.plan_id = p.plan_id
WHERE s.start_date >= '2021-01-01'
GROUP BY 1,2
ORDER BY 1;
```
**Solution :**

plan_id|	plan_name|	num_of_events|
|------|-----------|------------------|
1	|basic monthly|	8
2|	pro monthly|	60
3	|pro annual|63
4	|churn	|71

****

**4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?**

```sql
/*
> FOP: sum(case) , cast 
> Conditions:
-- output churn count churn pct
> PLAN:
-- join subcription on plan
-- case statement to on plan name = churn
*/
SELECT 
  	SUM(CASE WHEN p.plan_name = 'churn' THEN 1 END) AS churn_count,
  	CAST(100*SUM(CASE WHEN p.plan_name = 'churn' THEN 1 END) AS FLOAT(1)) 
              / COUNT(DISTINCT customer_id) AS churn_pct
FROM subscriptions s
JOIN plans p ON s.plan_id = p.plan_id;
```
**Solution :**

|churn_count|	churn_pct|
|-------------|------------|
307	|30.7|

****

**5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?**

```sql
/*
> FOP: cust_id, start_date, planname
> Conditions:
-- output churn after trail, percentage
> PLAN:
-- use cte 1 as nxt plan 
-- use lead for the new plan join s on p
*/
WITH nxt_plan AS (
SELECT 
        s.customer_id,
        s.start_date,
        p.plan_name,
        LEAD(p.plan_name) OVER(PARTITION BY s.customer_id ORDER BY p.plan_id) AS new_plan
FROM subscriptions s
JOIN plans p ON s.plan_id = p.plan_id
)
SELECT 
  	COUNT(*) AS churn_after_trial,
  	100*COUNT(*) / (SELECT COUNT(DISTINCT customer_id) FROM subscriptions) AS percentage
FROM nxt_plan
WHERE plan_name = 'trial' 
AND new_plan = 'churn';
```
**Solution :**

|churn_after_trial	|percentage|
|------------|----------------|
92	|9|

****

**6. What is the number and percentage of customer plans after their initial free trial?**

```sql
/*
> FOP: cust_id, planname
> Conditions:
-- output plan_name, cusotomer_count, customer_pct
> PLAN:
-- count(custmerid), uniq_cust * 100 /
-- subquery for percentage
*/
SELECT 
	plan_name,
        count(customer_id) customer_count,
        round(100 *count(DISTINCT customer_id) /
             (SELECT count(DISTINCT customer_id) AS unique_customers
              FROM subscriptions), 2) AS customer_pct
FROM subscriptions s
JOIN plans p ON s.plan_id = p.plan_id
WHERE plan_name != 'trial'
GROUP BY 1
ORDER BY 1;
```
**Solution :**

plan_name	|customer_count	|customer_pct|
|-----------|----------|-------------|
basic monthly|	546|	54.00|
churn	|307|	30.00|
pro annual	|258	|25.00|
pro monthly|	539	|53.00|

****

**7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?**

```sql
/*
> FOP: cust_id
> Conditions:
-- output plan_name, cusotomer_count, customer_pct
> PLAN:
-- count(custmerid), uniq_cust * 100 /
-- subquery for percentage
*/
WITH nxt_date AS (
SELECT
    	s.customer_id,
    	p.plan_id,
	p.plan_name,
  	s.start_date,
    	LEAD(start_date) OVER (PARTITION BY customer_id ORDER BY start_date) AS nxt_dates
FROM subscriptions s  
join plans p on s.plan_id = p.plan_id
WHERE start_date <= '2020-12-31'
)
SELECT
	plan_id,
	plan_name,
	COUNT(DISTINCT customer_id) AS customers,
  	ROUND(100.0 * COUNT(DISTINCT customer_id) /
	(SELECT COUNT(DISTINCT customer_id) 
      	 FROM subscriptions),1) AS pct
FROM nxt_date
WHERE nxt_dates IS NULL
GROUP BY 1,2;
```
**Solution :**

plan_id	 |plan_name|customers	|pct|
|----------|--------|-------------|--------|
0	|trial	|19	|1.9|
1	|basic monthly|224	|22.4|
2	|pro monthly	|326	|32.6|
3	|pro annual	|195	|19.5|
4	|churn	|236	|23.6|

****

**8. How many customers have upgraded to an annual plan in 2020?**

```sql
/*
> FOP: cust_id
> Conditions:
-- total_Customers
> PLAN:
-- count distinct cumstomers
*/
SELECT 
        COUNT(DISTINCT customer_id) AS Total_customers
FROM subscriptions
WHERE plan_id = 3
AND start_date <= '2020-12-31';
```
**Solution :**

|total_customers|
|---------|
195

****

**9. How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?**

```sql
/*
> FOP: cust_id, plane_name , startdate
> output:
-- avg_days to upgrage
> PLAN:
-- use cte for trial date and annual date 
*/
WITH trial_plan AS (
SELECT 
        customer_id, 
        start_date AS trial_date
FROM subscriptions
WHERE plan_id = 0
),
 annual_plan AS (
SELECT 
    customer_id, 
    start_date AS annual_date
FROM subscriptions
WHERE plan_id = 3
)
SELECT 
  	ROUND(AVG(a.annual_date - t.trial_date),0) AS avg_days_to_upgrade
FROM trial_plan AS t
JOIN annual_plan AS a
ON t.customer_id = a.customer_id;
```
**Solution :**

|avg_days_to_upgrade|
|-----------|
105

****

**10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)**

```sql
/*
> FOP: *
> output:
-- Window_30_days , Customer_Count
> PLAN:
-- use cte for next_plan_cte and window_details_cte
*/
WITH next_plan_cte AS
  (SELECT 
            *,
            lead(start_date, 1) over(PARTITION BY customer_id
                                      ORDER BY start_date) AS next_plan_start_date,
            lead(plan_id, 1) over(PARTITION BY customer_id
                                ORDER BY start_date) AS next_plan
FROM subscriptions),
window_details_cte AS(
SELECT
        *,
        datediff(next_plan_start_date, start_date) AS days,
        round(datediff(next_plan_start_date, start_date)/30) AS window_30_days
FROM next_plan_cte
WHERE next_plan=3)
SELECT 
        window_30_days,
        count(*) AS customer_count
FROM window_details_cte
GROUP BY window_30_days
ORDER BY window_30_days;
```
**Solution :**

| Window_30_days| Customer_Count|
|--------------|-----------|
0|44
1|32
2|41
3|43
4|38
5|36
6|24

****

**11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?**

```sql
/*
> FOP: cust_id, plane_name , startdate
> output:
-- pro to basic monthly 
> PLAN:
-- use cte for nxtplan
*/
WITH nxtplan AS (
SELECT 
    	s.customer_id,
    	s.start_date,
    	p.plan_name,
    	LEAD(p.plan_name) OVER(PARTITION BY s.customer_id ORDER BY p.plan_id) AS next_plan
FROM subscriptions s
JOIN plans p ON s.plan_id = p.plan_id
)
SELECT 
        COUNT(*) AS pro_to_basic_monthly
FROM nxtplan
WHERE plan_name = 'pro monthly'
AND next_plan = 'basic monthly'
AND EXTRACT(YEAR FROM start_date) = 2020;
```
**Solution :**

|pro_to_basic_monthly|
|-----|
0

****