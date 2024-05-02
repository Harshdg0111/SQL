![alt text](image-1.png) 
# Amazon Sales Analysis

**Amazon DataSet**
```sql
## showing first five results only
select
       *
from amazon_sales_report
```
**Solution :**

index	|order_id|	date|	status|fulfillment|	sales_channel	|ship_service_level|	style_id	|product_id|	category|	size|asin	|courier_status|	qty|currency|amount|	ship_city|	ship_state	|ship_postal_code|	ship_country|	b2b|
-----|-----|------|-----|---------|-------|----------|--------|-------|-------|--------|---------|--------------|--------|-------|--------|------|----------|--------|--------|--------|
0	|405-8078784-5731545|	2022-04-30	|Cancelled|Merchant|	Amazon.in|Standard	|SET389|	SET389-KR-NP-S|	Set	|S|B09KXVBD7Z|	Pending|	0	|INR|	647.62	|MUMBAI	|"MAHARASHTRA"|	"400081.0"|	"IN"|	false|
1|	171-9198151-1101146|	2022-04-30	|Shipped - Delivered to Buyer	|Merchant|	Amazon.in	|Standard	|JNE3781|	JNE3781-KR-XXXL	|kurta|	3XL	|B09K3WFS32|	Shipped|	1|	INR|	406|	BENGALURU	|KARNATAKA	|560085.0|	IN	|false|
2	|404-0687676-7273146|	2022-04-30|	Shipped|	Amazon|	Amazon.in	|Expedited|	JNE3371|	JNE3371-KR-XL|	kurta	|XL	|B07WV4JV4D	|Shipped|	1|	INR|	329	|NAVI MUMBAI|	MAHARASHTRA|	410210.0|	IN|	true|
3|	403-9615377-8133951|	2022-04-30|	Cancelled|	Merchant|	Amazon.in	|Standard	|J0341	|J0341-DR-L|	Western Dress|	L	|B099NRCT7B|	Pending	|0	|INR|	753.33|	PUDUCHERRY|	PUDUCHERRY	|605008.0	|IN	|false
4	|407-1069790-7240320|	2022-04-30	|Shipped	|Amazon|	Amazon.in	|Expedited|	JNE3671	|JNE3671-TU-XXXL|	Top	|3XL	|B098714BZP|	Shipped	|1	|INR|	574	|CHENNAI|TAMIL NADU	|600073.0|IN|	false

****

**1. Retrieve the total sales amount for each month of the year**
```sql
select 
	to_char(date,'month') as months,
	sum(qty * amount) as Total_Sales
from amazon_sales_report
group by 1
```
**Solution :**
months	|total_sales|
--------|-----------
april   | 	27840926
june     |	22757631
march    |	98261
may      |	25320057

****

**2. Identify the top 3 most profitable categories based on the total sales amount.**
```sql
select 
	category,
	sum(qty * amount) as Total_Sales
from amazon_sales_report
group by 1
order by 2 desc
limit 3
```
**Solution :**
category	|total_sales
------|---------
Set|	37925486
kurta|	20668481
Western Dress	|10707197

****

**3. Find the average sales amount per product for each style of product sold in the last month.**
```sql
SELECT 
	TO_CHAR(date, 'Month')	as months,
        Round(AVG((qty * amount)::numeric),2) AS Avg_sales
FROM amazon_sales_report
WHERE TO_CHAR(date, 'Month') = 
		(
		SELECT 
		        TO_CHAR(MAX(date) - INTERVAL '1 month', 'Month')
		FROM amazon_sales_report
		)
group by 1;
```
**Solution :**
months|	avg_sales
----|------
May      |	602.46
****

**4. Calculate the quantity sold for each size category.**
```sql
select 
	size,
	sum(qty) as Total_Quantity
from amazon_sales_report
group by 1
order by 2 desc

```
**Solution :**
size|total_quantity
------|-------
M|	20442
L|	19991
XL|	18920
XXL|	16513
S|15327
3XL	|13523
XS	|9942
6XL|	688
5XL|	513
4XL	|396
Free|	366

**5. Calculate the total sales amount and quantity sold for category in each size  of the categor**
```sql
## Showing starting five values only
with t1 as(
select 
	size,
	category,
	sum(qty * amount) as Total_Sales,
	sum(qty) as Total_Quantity,
	row_number() over(partition by size order by sum(qty * amount)desc)
from amazon_sales_report
group by 1 ,2
order by 1 
)
select 
	size,
	category,
	Total_Sales,
	Total_Quantity
from t1 
order by 1 desc
```
**Solution :**

size|	category|	total_sales|	total_quantity|
----|--------|---------|-----------|
XXL|	Ethnic Dress|	95759|	143
XXL|	kurta|3145103	|6949
XXL	|Blouse|	66815|	117
XXL	|Bottom|	24244	|64
XXL	|Set|4634500|	5608

*****

**6. Calculate the status of the product by months**
```sql
## showing first five values only
with t1 as(
select 
	to_char(date,'month') as months,
	courier_status,
	count(order_id) as no_products,
	row_number() over(partition by to_char(date,'month') order by count(order_id)desc)
from amazon_sales_report
group by 1 ,2
order by 1 
)
select 
	months,
	courier_status,
	no_products
from t1 
order by 1 ,3 desc
```
**Solution :**

months|	courier_status|no_products
--------|-------|--------|
april    |	Shipped|	41829
april    |Pending	|2813
april    	|Cancelled|	2258
april    	|Unshipped|	2154
june     |	Shipped	|31403
********


**7. calculate the number of product sold in each state and city**
```sql
## showing first five values only
with t1 as(
select 
	ship_state,
	ship_city,
	count(order_id) as no_products,
	row_number() over(partition by ship_state order by count(order_id)desc)
from amazon_sales_report
group by 1 ,2
order by 1 
)
select 
	ship_state,
	ship_city,
	no_products
from t1
order by 1,3 desc
```
**Solution :**

ship_state|	ship_city	|no_products|
----------|--------|------------
ANDAMAN & NICOBAR 	|PORT BLAIR|	132
ANDAMAN & NICOBAR |	Port Blair	|27
ANDAMAN & NICOBAR 	|PROTHRAPUR|	20
ANDAMAN & NICOBAR 	|Port blair	|15
ANDAMAN & NICOBAR 	|FERRARGUNJ	|10

****

**8. calculate the number of product sold by state**
```sql
## showing first five results only
select 
	DISTINCT ship_state,
	count(order_id) as no_products
from amazon_sales_report
group by 1
order by 2 desc
```
**Solution :**

ship_state|	no_products
-----|-------
MAHARASHTRA|	22260
KARNATAKA|	17326
TAMIL NADU|	11483
TELANGANA|	11330
UTTAR PRADESH	|10638

****

**9. calculate the number of product sold by city**
```sql
## showing first five results only
select 
	ship_city,
	count(order_id) as no_products
from amazon_sales_report
group by 1
order by 2 desc 
```
**Solution :**

ship_city|	no_products
------|----------
BENGALURU	|11217
HYDERABAD|	8074
MUMBAI|	6126
NEW DELHI|	5795
CHENNAI|	5421

****

**10. calculate the product delivered  by which services**
```sql
select 
	ship_service_level as services,
	count(order_id) as no_products
from amazon_sales_report
group by 1
order by 2 desc
```
**Solution :**

services|	no_products
------|--------
Expedited|	88595
Standard|	40347

****

**11. calculate which sales channel sold the most product**
```sql
## showing first five results only
select 
	sales_channel as company,
	count(order_id) as no_product
from amazon_sales_report
group by 1 
order by 1
```
**Solution :**

company	|no_product|
-----|----------
Amazon.in|	128818
Non-Amazon|	124

******
