# **🥑 - Foodie-Fi**

### **A. Customer Journey**

**Based off the 8 sample customers provided in the sample from the subscriptions table, write a brief description about each customer’s onboarding journey.**

****
**Distinct customer_id in the dataset**

```sql
SELECT 
        COUNT(DISTINCT(customer_id)) AS unique_customers
FROM subscriptions;
```
**Solution :**

|unique_customers|
|-----------------|
|1000|
****

Based off the 8 sample customers provided in the sample subscriptions table below,
write a brief description about each customer’s onboarding journey.
customer_id's : 83,55,23,63,72,27,12,84

**customer 83**
```sql
SELECT 
	s.customer_id,
        p.plan_id,
       	p.plan_name,
       	s.start_date
FROM subscriptions s
JOIN plans p on s.plan_id = p.plan_id
WHERE customer_id = 83;
```
**Solution :**

|customer_id	|plan_id	|plan_name|start_date|
|---------------|-----------|-------------|------------|
|83	|0	|trial|	2020-05-18|
|83	|1|basic monthly	|2020-05-25|
|83	|2	|pro monthly|	2020-10-29|
|83	|3	|pro annual|	2021-04-29|

• Customer 83 started the trail on 18 june 2020 
• Cusotmer upgraded	the plan from trail to basic montly on 25 june 2020 after 7 days
• Customer then again upgarded the plane form basic to pro on 29 october 2020
• The latest plan for customer 83 is the pro annual plan which started on 29 april 2021

****

**customer 55**
```sql
SELECT 
	s.customer_id,
       	p.plan_id,
       	p.plan_name,
       	s.start_date
FROM subscriptions s
JOIN plans p on s.plan_id = p.plan_id
WHERE customer_id = 55;
```
**Solution :**

|customer_id	|plan_id	|plan_name	|start_date|
|-------------|---------|----------|------------|
55|	0	|trial|	2020-10-22
55	|1	|basic monthly|	2020-10-29|
55	|3	|pro annual	|2021-03-01|

• Customer 83 started the trail on 22 october 2020
• Cusotmer upgraded	the plan from trail to basic montly on 29 october 2020 after 7 days from the trail plan
• The latest plan for customer 83 is the pro annual plan which started on 01 march 2021

****

**customer 23**
```sql
SELECT 
	s.customer_id,
       	p.plan_id,
       	p.plan_name,
       	s.start_date
FROM subscriptions s
JOIN plans p on s.plan_id = p.plan_id
WHERE customer_id = 23;
```
**Solution :**

|customer_id|	plan_id|	plan_name|	start_date|
|-------------|----------|-------------|--------------|
|23	|0|	trial	|2020-05-13|
|23	|3	|pro annual|	2020-05-20|

• Customer strated the free trail plan from 13 june 2020 and continued for a week
• Then they upgraged the plan to pro-annual on 20 june 2020

****

**customer 63**
```sql
SELECT 
	s.customer_id,
       	p.plan_id,
       	p.plan_name,
       	s.start_date
FROM subscriptions s
JOIN plans p on s.plan_id = p.plan_id
WHERE customer_id = 63;
```
**Solution :**

|customer_id	|plan_id|	plan_name|	start_date|
|--------------|-------|------------|-------------|
|63|	0|	trial	|2020-05-28|
|63	|1	|basic monthly|	2020-06-04|
|63	|4	|churn	|2020-06-18|

• Customer 63	started the free trial plan from 28 may 2020 and continued for a week
• Then they upgraded to basic monthly plan after a week from trail plan 
• Then they cancel the subscription and churned on 18 june 2020 may

****

**customer 72**
```sql
SELECT 
	s.customer_id,
       	p.plan_id,
       	p.plan_name,
       	s.start_date
FROM subscriptions s
JOIN plans p on s.plan_id = p.plan_id
WHERE customer_id = 72;
```
**Solution :**

|customer_id	|plan_id	|plan_name|	start_date|
|--------------|-----------|-------------|------------|
|72|	0	|trial|2020-12-10|
|72	|2	|pro monthly	|2020-12-17|
|72	|4	|churn|	2021-02-01|

• Customer 72 started the free subscription trial plan from 10 december 2020 and continued for a week
• Then they upgraded the subcription plan to pro monthly plan on 17 december 2020 after a week from free trial plan
• They they cancel the subcription and churned on 1st february 2021 , 2 months after the pro monthly plan

****

**customer 27**
```sql
SELECT 
	s.customer_id,
       	p.plan_id,
       	p.plan_name,
       	s.start_date
FROM subscriptions s
JOIN plans p on s.plan_id = p.plan_id
WHERE customer_id = 27;
```
**Solution :**

|customer_id|	plan_id	|plan_name	|start_date|
|-------------|------------|------------|----------|
|27|	0	|trial	|2020-08-24|
|27|	2	|pro monthly|	2020-08-31|

• Customer 27 started the free subcription trail plan from 24 september 2020 and continued for a week 
• The they upgraded the plan to pro monthly subcription plan on 31 september 2020 after a week from free trail plan and continued till today

****

**customer 12**
```sql
SELECT 
	s.customer_id,
       	p.plan_id,
       	p.plan_name,
       	s.start_date
FROM subscriptions s
JOIN plans p on s.plan_id = p.plan_id
WHERE customer_id = 12;
```
**Solution :**

|customer_id|	plan_id	|plan_name|	start_date|
|------------|------------|----------|-------------|
|12|	0	|trial|	2020-09-22|
|12	|1	|basic monthly|2020-09-29|

• Customer 12 started the free trail subcription plan from 22 october 2020  and continued for a week
• They then upgraded the free trial to basic montly subcriptition plan to on 29 october 2020 continued till today

****

**customer 84**
```sql
SELECT 
	s.customer_id,
       	p.plan_id,
       	p.plan_name,
       	s.start_date
FROM subscriptions s
JOIN plans p on s.plan_id = p.plan_id
WHERE customer_id = 84;
```
**Solution :**

|customer_id	|plan_id|	plan_name	|start_date|
|----------------|-----|------------|------------|
|84|	0	|trial|	2020-06-14|
84	|1	|basic monthly|	2020-06-21|
84	|4|churn|2020-07-07|

• Customer purchased the first free trail subsciption plan on 14 june 2020 
• Then the upgraded the plan to basic monthly plan on 21 june 2020 and then cancel it after 2 weeks 

****