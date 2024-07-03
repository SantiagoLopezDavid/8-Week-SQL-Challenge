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
- From the `sales` table, we can use the column `order_date` as an indicator for visits. But we are only interested in counting each day once, we don't want that multiple orders to count as two different visits.
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

For this question, multiple `customer_id` could have bought more than one item in their fisrt visit. There is not a way of knowing which one was first based on the dataset provided. As a result we could have multiple `product_name` for each `customer_id`.

**Explanation:**

- We will the subquery `x` to create a new column `rnk` and calculate the row number using **DENSE_RANK()** window function. The **PARTITION BY** clause divides the data by `customer_id`, and the **ORDER BY** clause orders the rows within each partition by `order_date`.
- In the main query we will **SELECT** the desire columns and use the **WHERE** clause to filter the data by only those row with a `rnk = 1`.
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
ORDER BY COUNT(*) DESC
LIMIT 1
```
**Explanation:**
-
-

**Results and Analysis:**
|product_id|product_name|count_purchases|
|---|---|---|
|3|ramen|8|

- The most purchased item on the menu is Ramen with 8 counts.

---


















