# **🌲 - Balanced Tree Clothing Co.**

### **📈 A. High Level Sales Analysis**

**1. What was the total quantity sold for all products?**

``` sql
SELECT	
	pd.product_name,
	SUM(s.qty) AS total_quantity
FROM sales s
JOIN product_details pd ON s.prod_id = pd.product_id
GROUP BY 1;
```
**Solution :**

|product_name|total_quantity|
|--------------|----------------|
|White Tee Shirt - Mens|	3800|
|Navy Solid Socks - Mens|	3792|
|Grey Fashion Jacket - Womens|	3876|
|Navy Oversized Jeans - Womens	|3856|
|Pink Fluro Polkadot Socks - Mens	|3770|

**** 

**2. What is the total generated revenue for all products before discounts?**

```sql
SELECT
	pd.product_name,
	sum(s.price * s.qty) as total_revenue
from sales s 
join product_details pd on s.prod_id = pd.product_id
group by 1;
```
**Solution :**

|product_name	|total_revenue|
|---------------|---------------|
|White Tee Shirt - Mens|	152000|
|Navy Solid Socks - Mens|	136512|
|Grey Fashion Jacket - Womens|	209304|
|Navy Oversized Jeans - Womens|	50128|
|Pink Fluro Polkadot Socks - Mens|	109330|

****

**3. What was the total discount amount for all products?**

```sql
select 
	pd.product_name,
	um(s.discount) as total_discount
from sales s 
join product_details pd on s.prod_id = pd.product_id
group by 1;
```
**Solution :**

|product_name|	total_discount|
|--------------|------------------|
|White Tee Shirt - Mens|15487|
|Navy Solid Socks - Mens	|15646|
|Grey Fashion Jacket - Womens|	15500|
|Navy Oversized Jeans - Womens	|15418|
|Pink Fluro Polkadot Socks - Mens|	14946|

****

