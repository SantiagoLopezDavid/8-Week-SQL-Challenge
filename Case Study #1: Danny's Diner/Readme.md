# 🍜 Case Study #1: Danny's Diner

<img width="500" alt="image" src="https://github.com/SantiagoLopezDavid/8-Week-SQL-Challenge/assets/170588432/bef724dc-4dfa-45f0-90de-29adf2de8b0f">

## Table of contents
1. Problem Statement.
2. Entity Relationship Diagram.
3. Questions and Solutions.

All the information regarding the case study has been sourced from [here](https://8weeksqlchallenge.com/case-study-1/)

## Problem Statement
The main objectives are to determine the most **frequent customers**, analyze **spending patterns**, and understand **customer preferences** over time. The challenge involves using `SQL` to query and analyze the data to answer specific business questions posed by Danny, such as identifying the highest-spending customers and the most popular menu items.

## Entity Relationship Diagram
<img width="500" alt="image" src="https://github.com/SantiagoLopezDavid/8-Week-SQL-Challenge/assets/170588432/fad4c1b3-01aa-45fd-bf09-4d5fb451376a">

## Questions and Solutions

**1. What is the total amount each customer spent at the restaurant?**

```sql
SELECT customer_id, SUM(price) AS total_spent
FROM sales s
JOIN menu m ON s.product_id = m.product_id
GROUP BY customer_id
ORDER BY customer_id;
```
**Explanation:**
- We need to perfomr a **JOIN** since the information about the sales and price of each item are in two separate tables. In this case the `sales` and `menu` tables have the information we need and the column in common is `product_id`.
- Use the **SUM()** function to calculate the `total_spent` by each `customer_id`.
- Finally use the **GROUP BY** clause with `customer_id`.

**Results and Analysis:**
|customer_id|total_spent|
|---|---| 
|A|76| 
|B|74|
|C|36|

* Customer A spent a total of $76.
* Customer B spent a total of $74.
* Customer C spent a total of $36.

---

**2. How many days has each customer visited the restaurant?**

```sql
SELECT customer_id, 
COUNT(DISTINCT order_date) AS visits_count
FROM sales
GROUP BY customer_id;
```
**Explanation:**
- From the `sales` table, we can use the column `order_date` as an indicator for visits. But we are only interested in counting each day once, we don't want multiple orders to count as two different visits.
- For this purpose we can use **COUNT(DISTINCT `order_date`)**

**Results and Analysis:**
|customer_id|visits_count|
|---|---|
|A|4|
|B|6|
|C|2|

- Customer A has visited the restaurant a total of 4 separate times.
- Customer B has visited the restaurant a total of 6 separate times.
- Customer C has visited the restaurant a total of 2 separate times.

---

**3. What was the first item from the menu purchased by each customer?**
```sql
SELECT customer_id, product_name FROM
	(SELECT *,
	DENSE_RANK() OVER(PARTITION BY customer_id ORDER BY order_date ASC) AS rnk
	FROM sales) AS x
JOIN menu ON x.product_id = menu.product_id
WHERE rnk = 1
GROUP BY customer_id, product_name
```
**Asumptions:**

For this question, multiple `customer_id` could have bought more than one item in their first visit. There is not a way of knowing which one was first based on the dataset provided. As a result we could have multiple `product_name` for each `customer_id`.

**Explanation:**

- Create a subquery named `x` to create a new column `rnk` and calculate the row number using **DENSE_RANK()** window function. The **PARTITION BY** clause divides the data by `customer_id`, and the **ORDER BY** clause orders the rows within each partition by `order_date`.
- In the main query use **SELECT** the desire columns and use the **WHERE** clause to filter the data by only those rows with a `rnk = 1`.
- Use the **GROUP BY** clause to group the result by `customer_id` and `product_name`.

**Results and Analysis:**

|customer_id|product_name|
|---|---|
|A|curry|
|A|sushi|
|B|curry|
|C|ramen|

- Customer A purchased Curry and Sushi as his/her first order.
- Customer B purchased Curry as his/her first order.
- Customer C purchased Ramen as his/her first order.

---

**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**
```sql
SELECT sales.product_id, product_name, COUNT(*) AS count_purchases
FROM sales 
JOIN menu ON sales.product_id = menu.product_id
GROUP BY sales.product_id, product_name
ORDER BY count_purchases DESC
LIMIT 1
```
**Explanation:**
- Use a **COUNT** aggregation on all rows and use a **GROUP BY** clause with `sales.product_id` and `product_name` to narrow the count.
- Order `count_purchases` in descending order and apply **LIMIT 1** clause to filter and retrieve the highest number of purchased items.

**Results and Analysis:**
|product_id|product_name|count_purchases|
|---|---|---|
|3|ramen|8|

- The most purchased item on the menu is Ramen with 8 counts.

---

**5. Which item was the most popular for each customer?**
```sql
SELECT customer_id, product_name, count_product FROM
	(SELECT customer_id,product_id,count(product_id) as count_product,
	RANK() OVER(PARTITION BY customer_id ORDER BY COUNT(product_id) DESC) AS rnk
	FROM sales
	GROUP BY customer_id, product_id
	ORDER BY customer_id, COUNT(product_id)) AS x
JOIN menu ON menu.product_id = x.product_id
WHERE rnk = 1
ORDER BY customer_id, count_product

```
**Explanation:**
- Create a subquery named `x`, and within this subquery use a **COUNT()** aggregation on `product_id` and utilize the **RANK()** function to calculate the ranking of each `customer_id` partition based on **COUNT(product_id)** in descending order.
- In the main query **JOIN** the tables `x` and `menu` on the `product_id` column.
- Filter the resulting table using the **WHERE** clause by only those rows with a `rnk = 1`
- 
**Results and Analysis:**
|customer_id|product_name|count_product|
|---|---|---|
|A|ramen|3|
|B|sushi|2|
|B|curry|2|
|B|ramen|2|
|C|ramen|3|


- Customers A and C share Ramen as their favorite product of the menu. Each with 3 counts.
- Customer B has order all items in the menu an equal number or times.

---
**6. Which item was purchased first by the customer after they became a member?**
```sql
SELECT customer_id, order_date, product_name, join_date from
	(SELECT s.customer_id, order_date, product_id, join_date,
	RANK() OVER (PARTITION BY s.customer_id ORDER BY order_date - join_date) AS rnk
	FROM sales s
	JOIN members m ON s.customer_id = m.customer_id
	WHERE order_date - join_date > 0 ) AS x
JOIN menu ON x.product_id = menu.product_id
WHERE rnk = 1
ORDER BY customer_id
```
**Explanation:**


**Results and Analysis:**

|customer_id|order_date|product_name|join_date|
|---|---|---|---|
|A|2021-01-10|ramen|2021-01-07|
|B|2021-01-11|sushi|2021-01-09|

- The first product that the customer A purchased after becoming a member was Ramen.
- The first product that the customer B purchased after becoming a member was Sushi.
  
---

**7. Which item was purchased just before the customer became a member?**
```sql
SELECT customer_id, order_date, product_name, join_date FROM
	(SELECT s.customer_id, order_date, product_id, join_date,
	order_date - join_date AS date_diff,
	RANK() OVER (PARTITION BY s.customer_id ORDER BY order_date - join_date DESC) AS rnk
	FROM sales s
	JOIN members m ON s.customer_id = m.customer_id
	WHERE order_date - join_date < 0) AS x
JOIN menu ON x.product_id = menu.product_id
WHERE rnk = 1
ORDER BY customer_id
```
**Explanation:**


**Results and Analysis:**

|customer_id|oder_date|product_name|join_date|
|---|---|---|---|
|A|2021-01-01|sushi|2021-01-07|
|A|2021-01-01|curry|2021-01-07|
|B|2021-01-04|sushi|2021-01-09|

- The items that customer A purchased just before becoming a member were Sushi and Curry.
- The item that customer B purchased just before becoming a member was Sushi.

---

**8. What is the total items and amount spent for each member before they became a member?**
```sql
SELECT customer_id, SUM(price) AS total_spent FROM
	(SELECT s.customer_id, order_date, s.product_id, price
	FROM sales s
	JOIN members m ON s.customer_id = m.customer_id
	JOIN menu ON menu.product_id = s.product_id
	WHERE order_date < join_date
	ORDER BY s.customer_id) AS x
GROUP BY customer_id
```
**Explanation:**


**Results and Analysis:**

|customer_id|total_spent|
|---|---|
|A|25|
|B|40|

- Customer A has spent a total of $25 before becoming a member.
- Customer B has spent a total of $40 before becoming a member.

---

**9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**
```sql
SELECT customer_id,
SUM(CASE 
	WHEN product_name = 'sushi' THEN price* 20
	ELSE price*10
	END) AS total_points
FROM sales
JOIN menu ON menu.product_id = sales.product_id
GROUP BY customer_id
ORDER BY customer_id
```
**Explanation:**


**Results and Analysis:**
|customer_id|total_points|
|---|---|
|A|860|
|B|940|
|C|360|

- Customer A has a total of 860 points based on his/hers expenses.
- Customer B has a total of 940 points based on his/hers expenses.
- Customer C has a total of 360 points based on his/hers expenses.

---

**10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**

```sql
SELECT customer_id, SUM(total_points) AS total_points FROM
	(SELECT s.customer_id,
	CASE 
		WHEN order_date BETWEEN join_date AND join_date+7 THEN price* 20
		ELSE price*10
		END AS total_points
	FROM sales s
	JOIN menu ON menu.product_id = s.product_id
	JOIN members m ON m.customer_id = s.customer_id
	WHERE order_date < '2021-01-30') AS x
GROUP BY customer_id
ORDER BY customer_id
```
**Explanation:**


**Results and Analysis:**
|customer_id|total_points|
|---|---|
|A|1270|
|B|840|

- Customer A has a total of 860 points based on his/hers expenses.
- Customer B has a total of 940 points based on his/hers expenses.

---
## Bonus Questions
**Join All The Things**

Create a table with: customer_id, order_date, product_name, price, member (Y/N)

```sql
SELECT s.customer_id, order_date, product_name, price,
CASE
	WHEN order_date < join_date THEN 'N'
	WHEN order_date >= join_date THEN 'Y'
	ELSE 'N'
	END AS member
FROM sales s
LEFT JOIN menu m ON m.product_id = s.product_id
LEFT JOIN members mem ON mem.customer_id = s.customer_id
ORDER BY customer_id, order_date
```
**Explanation:**


**Results and Analysis:**
|customer_id|order_date|product_name|price|member|
|---|---|---|---|---|
|A|2021-01-01|sushi|10|N|
|A|2021-01-01|curry|15|N|
|A|2021-01-07|curry|15|Y|
|A|2021-01-10|ramen|12|Y|
|A|2021-01-11|ramen|12|Y|
|A|2021-01-11|ramen|12|Y|
|B|2021-01-01|curry|15|N|
|B|2021-01-02|curry|15|N|
|B|2021-01-04|sushi|10|N|
|B|2021-01-11|sushi|10|Y|
|B|2021-01-16|ramen|12|Y|
|B|2021-02-01|ramen|12|Y|
|C|2021-01-01|ramen|12|N|
|C|2021-01-01|ramen|12|N|
|C|2021-01-07|ramen|12|N|

---

**Rank All The Things***

Danny also requires further information about the `ranking` of customer products, but he purposely does not need the `ranking` for non-member purchases so he expects null `ranking` values for the records when customers are not yet part of the loyalty program.

```sql
WITH cte AS 
	(SELECT s.customer_id, order_date, product_name, price,
	CASE
		WHEN order_date < join_date THEN 'N'
		WHEN order_date >= join_date THEN 'Y'
		ELSE 'N'
		END AS member
	FROM sales s
	LEFT JOIN menu m ON m.product_id = s.product_id
	LEFT JOIN members mem ON mem.customer_id = s.customer_id
	ORDER BY customer_id, order_date)
SELECT *, 
CASE
	WHEN member = 'N' THEN NULL
   	ELSE RANK() OVER(PARTITION BY customer_id, member ORDER BY order_date) 
	END AS ranking
FROM cte;
```
**Explanation:**


**Results and Analysis:**
|customer_id|order_date|product_name|price|member|ranking|
|---|---|---|---|---|---|
|A|2021-01-01|sushi|10|N|NULL|
|A|2021-01-01|curry|15|N|NULL|
|A|2021-01-07|curry|15|Y|1|
|A|2021-01-10|ramen|12|Y|2|
|A|2021-01-11|ramen|12|Y|3|
|A|2021-01-11|ramen|12|Y|3|
|B|2021-01-01|curry|15|N|NULL|
|B|2021-01-02|curry|15|N|NULL|
|B|2021-01-04|sushi|10|N|NULL|
|B|2021-01-11|sushi|10|Y|1|
|B|2021-01-16|ramen|12|Y|2|
|B|2021-02-01|ramen|12|Y|3|
|C|2021-01-01|ramen|12|N|NULL|
|C|2021-01-01|ramen|12|N|NULL|
|C|2021-01-07|ramen|12|N|NULL|



