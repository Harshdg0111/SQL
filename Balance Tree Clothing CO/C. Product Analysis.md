# **🌲 - Balanced Tree Clothing Co.**

### **👚 C. Product Analysis**

**1. What are the top 3 products by total revenue before discount?**

```sql
select 
		pd.product_name,
		sum(s.price * s.qty) as total_revenue
from sales s 
join product_details pd on s.prod_id = pd.product_id
group by 1 
order by 2 desc 
limit 3;
```
**Solution :**

|product_name|	total_revenue|
|--------------|-----------------|
|Blue Polo Shirt - Mens|	217683|
|Grey Fashion Jacket - Womens|	209304|
|White Tee Shirt - Mens	|152000|

****

**2. What is the total quantity, revenue and discount for each segment?**

```sql
select 
	pd.segment_name,
	sum(s.qty) as total_quantity,
	sum(s.price * s.qty) as total_revenue,
	sum((s.qty * s.price)* s.discount/100) as total_discount
from sales s 
join product_details pd on s.prod_id = pd.product_id
group by 1 
order by 3 desc
```
**Solution :**

|segment_name|total_quantity|	total_revenue	|total_discount|
|---------------|----------------|------------------|-----------------|
|Shirt|	11265	|406143	|48082|
|Jacket|	11385	|366983	|42451|
|Socks	|11217|	307977	|35280|
|Jeans|	11349	|208350|	23673|

****

**3. What is the top selling product for each segment?**

```sql
with top_selling as( 
select 
	pd.segment_name,
	pd.product_name,
	sum(s.qty) as total_discount,
	rank() over(partition by pd.segment_name order by sum(s.qty)desc) as rnk
from sales s
join product_details pd on s.prod_id = pd.product_id
group by 1,2
)
select 
	segment_name,
	product_name,
	total_discount
from top_selling
where rnk = 1;
```
**Solution :**

|segment_name	|product_name|	total_discount|
|----------------|-------------|------------------|
|Jacket	|Grey Fashion Jacket - Womens	|3876|
|Jeans|	Navy Oversized Jeans - Womens|3856
|Shirt|	Blue Polo Shirt - Mens	|3819|
|Socks|	Navy Solid Socks - Mens|	3792|

****

**4. What is the total quantity, revenue and discount for each category?**

```sql
select 
	pd.category_name,
	sum(s.qty) as total_quantity,
	sum(s.price * s.qty) as total_revenue,
	sum((s.price * s.qty)* s.discount/100) as total_discount
from product_details pd 
join sales s on pd.product_id = s.prod_id
group by 1
order by 1;
```
**Solution :**

|category_name|	total_quantity	|total_revenue	|total_discount|
|---------------|------------------|-----------------|----------------|
|Mens|22482	|714120	|83362|
|Womens|	22734|	575333	|66124|

****

**5. What is the top selling product for each category?**

```sql
with top_selling as( 
select 
	pd.category_name,
	pd.product_name,
	sum(s.qty) as total_quantity,
	rank() over(partition by pd.category_name order by sum(s.qty)desc) as rnk
from sales s
join product_details pd on s.prod_id = pd.product_id
group by 1,2
)
select 
	category_name,
	product_name,
	total_quantity
from top_selling
where rnk = 1;
```
**Solution :**

|category_name|product_name	|total_quantity|
|------------------|--------------|----------------|
|Mens|Blue Polo Shirt - Mens|	3819|
|Womens|Grey Fashion Jacket -Womens|	3876|

****

**6. What is the percentage split of revenue by product for each segment?**

```sql
with segment_revenue as(
select 
	pd.segment_name,
	pd.product_name,
	sum(s.price * s.qty) as total_revenue
from sales s 
join product_details pd on s.prod_id = pd.product_id
group by 1,2
)
select 
	segment_name,
	product_name,
	round((100 * total_revenue:: decimal(10,2)
        /sum(total_revenue)over (partition by segment_name)),2) as segment_percent
from segment_revenue;
```
**Solution :**

|segment_name	|product_name	|segment_percent|
|---------------|---------------|-----------------|
|Jacket	|Indigo Rain Jacket - Womens|	19.45|
|Jacket	|Grey Fashion Jacket - Womens|	57.03|
|Jeans	|Navy Oversized Jeans - Womens|	24.06|
|Jeans	|Black Straight Jeans - Womens|	58.15|
|Jeans	|Cream Relaxed Jeans - Womens|	17.79|

****

**7. What is the percentage split of revenue by segment for each category?**

```sql
with category_revenue as(
select 
	pd.category_name,
	pd.segment_name,
	sum(s.price * s.qty) as total_revenue
from sales s 
join product_details pd on s.prod_id = pd.product_id
group by 1,2
)
select 
	category_name,
	segment_name,
	round((100 * total_revenue:: decimal(10,2)
	/sum(total_revenue)over (partition by category_name)),2) as category_percent
from category_revenue;
```
**Solution :**

|category_name	|segment_name	|category_percent|
|-----------------|---------------|------------------|
|Mens	|Socks|	43.13|
|Mens	|Shirt|	56.87|
|Womens	|Jeans|	36.21|
|Womens	|Jacket|	63.79|

****

**8. What is the percentage split of total revenue by category?**

```sql
with category_revenue as(
select 
	pd.category_name,
	sum(s.price * s.qty) as total_revenue
from sales s 
join product_details pd on s.prod_id = pd.product_id
group by 1
)
select 
	category_name,
	round((100 * total_revenue:: decimal(10,2)
	/sum(total_revenue)over ()),2) as category_percent
from category_revenue;
```
**Solution :**

|category_name	|category_percent|
|-----------------|------------------|
|Mens|	55.38|
|Womens|	44.62|

****

**9. What is the total transaction “penetration” for each product?**

```sql
with txn_penetration as(
select 
	distinct s.prod_id,
	pd.product_name,
	count(distinct s.txn_id) as unique_txn,
	(select count(distinct txn_id)
	from sales) as total_txn
from sales s 
join product_details pd on s.prod_id = pd.product_id
group by 1,2
)
select 
	product_name,
	round((100 * unique_txn::decimal(10,2) / total_txn),2) as total_penetration
from txn_penetration ;
```
**Solution :**

|product_name|total_penetration|
|---------------|-------------------|
|Khaki Suit Jacket - Womens|49.88|
|Pink Fluro Polkadot Socks - Mens|	50.32|
|Grey Fashion Jacket - Womens	|51.00|
|Teal Button Up Shirt - Mens	|49.68|
|White Striped Socks - Mens|	49.72|

****

**10. What is the most common combination of at least 1 quantity of any 3 products in a 1 single transaction?**

```sql
WITH products_txn AS (
SELECT 
	s.txn_id,
	pd.product_id,
	pd.product_name,
	s.qty,
	COUNT(pd.product_id) OVER (PARTITION BY txn_id) AS product_cnt
FROM sales s
JOIN product_details pd ON s.prod_id = pd.product_id
),

combinations AS (
SELECT 
      STRING_AGG(product_id::VARCHAR,	', ') WITHIN GROUP (ORDER BY product_id)  AS product_ids,
      STRING_AGG(product_name::VARCHAR, ', ') WITHIN GROUP (ORDproduct_ids,
      product_names
FROM combination_cnt
WHERE similar_combinations = (SELECT MAX(common_combinations) 
		              FROM combination_count);
```
**Solution :**

|product_ids	|product_names|
|-------------|-------------|
|2a2353, 2feb6b, c4a632|	Blue Polo Shirt - Mens, Pink Fluro Polkadot Socks - Mens, Navy Oversized Jeans - Womens
|c4a632, c8d436, e83aa3|	Navy Oversized Jeans - Womens, Teal Button Up Shirt - Mens, Black Straight Jeans - Womens
|b9a74d, c4a632, d5e9a6	|White Striped Socks - Mens, Navy Oversized Jeans - Womens, Khaki Suit Jacket - Womens
|5d267b, c4a632, e83aa3|	White Tee Shirt - Mens, Navy Oversized Jeans - Womens, Black Straight Jeans - Womens
|5d267b, c4a632, e31d39|	White Tee Shirt - Mens, Navy Oversized Jeans - Womens, Cream Relaxed Jeans - Womens

****