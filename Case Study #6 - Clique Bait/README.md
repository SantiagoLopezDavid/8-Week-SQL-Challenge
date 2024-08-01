# 🎣 Case Study #6 - Clique Bait

<img width="500" alt="image" src="https://github.com/user-attachments/assets/330c40f2-84da-499f-86b6-455e91f6e3ea">

## Table of contents
1. Problem Statement.
2. Entity Relationship Diagram.
3. Solution.
  - A. Digital Analysis.
  - B. Product Funnel Analysis.
  - C. Campaigns Analysis.

All the information regarding the case study has been sourced from [here](https://8weeksqlchallenge.com/case-study-6/)

## Problem Statement

**Clique Bait** is not like your regular online seafood store.

In this case study - you are required to support the founder and CEO Danny's vision and analyse his dataset and come up with creative solutions to calculate funnel fallout rates for the Clique Bait online store.

## Entity Relationship Diagram

<img width="500" alt="image" src="https://github.com/user-attachments/assets/9ff89359-9b58-4a78-bc00-5d6d5889cacf">

## Solution

### **A. Digital Analysis**

**1. How many users are there?**

```sql
SELECT 
COUNT(DISTINCT(user_id)) AS user_count
FROM users;
```

**Explanation:**
- **COUNT** the unique number of `user_id` from the table `users`.

**Results and Analysis:**

<img width="98" alt="image" src="https://github.com/user-attachments/assets/526fe3fc-2b92-419a-b8f0-87c126903670">

- There are **500** unique `user_id`.

**2. How many cookies does each user have on average?**

```sql
SELECT
ROUND(SUM(cookie_count)::numeric/500,2) AS avg_cookie
FROM
	(SELECT user_id, COUNT(cookie_id) AS cookie_count
	FROM users
	GROUP BY user_id) AS x;
```

**Explanation:**
- Use a subquery to **COUNT** the number of `cookie_id` per `user_id`.
- Using the subquery results, **SUM** the `cookie_count` and divide by the total number of unique `user_id` (500).

**Results and Analysis:**

<img width="96" alt="image" src="https://github.com/user-attachments/assets/4d4b9330-3ced-4c92-bf69-2077bb1f1e35">

- On average every `user_id` has 3.56 `cookie_id`.

**3. What is the unique number of visits by all users per month?**

```sql
SELECT
EXTRACT(month FROM event_time) AS month_num,
TO_CHAR(event_time, 'Month') AS month_str,
COUNT(DISTINCT visit_id) AS visit_count
FROM events
GROUP BY 1,2
ORDER BY 1,2;
```

**Explanation:**
- **EXTRACT** the month number from `event_time`.
- Format the month from `event_time` as a string for each month.
- **COUNT** each unique `visit_id` for the variables in the **GROUP BY** clause.

**Results and Analysis:**

<img width="282" alt="image" src="https://github.com/user-attachments/assets/66c24f38-7674-46b2-8b8a-442ff0b9ba8e">

- The month with the least number of unique visits is **May**.
- The month with the highest number of unique visits is **February**.

**4. What is the number of events for each event type?**

```sql
SELECT e.event_type,
ei.event_name,
COUNT(e.event_type) AS event_count
FROM events e
JOIN event_identifier ei ON e.event_type = ei.event_type
GROUP BY e.event_type, ei.event_name
ORDER BY e.event_type;
```

**Explanation:**
- **COUNT** the number of rows for each `event_type`.
- Use a **JOIN** statement to bring the `event_name` into the table.
- 
**Results and Analysis:**

<img width="348" alt="image" src="https://github.com/user-attachments/assets/260c3c31-6535-46a5-96ee-1c36bfa276d6">

- The event with the highest number of appearances is **Page View** with almost 21.000.
- The rest of the events have less than 10.000 appearances. The second most frecuent event is **Add to Cart**.

**5. What is the percentage of visits which have a purchase event?**

```sql
WITH purchase_cte AS 
	(SELECT COUNT(event_type) purchase_count
	FROM events
	WHERE event_type = 3),
total_visits_cte AS
	(SELECT COUNT(distinct visit_id) AS total_event
	FROM events)
SELECT 
ROUND((purchase_count::numeric/total_event)*100,2) AS purchase_percentage
FROM purchase_cte, total_visits_cte;
```

**Explanation:**
- Create two separate **CTE**. In the first one, **COUNT** the `event_type` where there is a purchase. In the second, **COUNT** the unique `visit_id` from the table (`total_event` or visits)
- With these two **CTE** results, calculate the `purchase_percentage`.

**Results and Analysis:**

<img width="153" alt="image" src="https://github.com/user-attachments/assets/c7632075-2cf0-464f-bf4f-e542f3468564">

- The **49.86%** of visits have a purchase event.

**6. What is the percentage of visits which view the checkout page but do not have a purchase event?**

```sql
WITH cte AS 
	(SELECT COUNT(visit_id) AS visit_count
	FROM events
	WHERE page_id = 12 AND visit_id NOT IN
		(SELECT visit_id
		FROM events
		WHERE event_type = 3))
SELECT
ROUND(visit_count::numeric/(SELECT COUNT(DISTINCT visit_id) FROM events)*100,2) AS visit_percentage
FROM cte;
```

**Explanation:**
- **COUNT** the `visit_id` that have a `page_id = 12` but are not included in the group of `visit_id` that have a purchase event.
- Calculate the `visit_percentage` taking into account all unique visits to the website.

**Results and Analysis:**

<img width="124" alt="image" src="https://github.com/user-attachments/assets/ae679cd1-eaca-4a60-bfdd-603a05202460">

- There is only a **9.15%** of visits that view the checkout page but did not follow up with the purchase.

**7. What are the top 3 pages by number of views?**

```sql
SELECT events.page_id,page_name,
COUNT(events.event_type) AS view_count
FROM events
JOIN page_hierarchy ON events.page_id = page_hierarchy.page_id
WHERE event_type = 1
GROUP BY events.page_id,page_name
ORDER BY 3 DESC
LIMIT 3;
```

**Explanation:**
- **COUNT** the `event_type` for each `page_id`.
- Filter the results by only those `page_id` where the `event_type = 1`.
- **ORDER BY** the `view_count` in descending order.
- **LIMIT** the result to only the top 3.

**Results and Analysis:**

<img width="328" alt="image" src="https://github.com/user-attachments/assets/084f0101-019f-444c-b58a-bc0b21a1dd16">

- The page with the most views is **All Products** with a total of 3174 views.
- The second most viewed page is **Checkout** with 2103 views.
- The least viewed page is the **Home Page** with 1782 views.

**8. What is the number of views and cart adds for each product category?**

```sql
SELECT
product_category,
SUM(CASE WHEN event_type = 1 THEN 1 ELSE 0 END) AS view_count,
SUM(CASE WHEN event_type = 2 THEN 1 ELSE 0 END) AS cart_add_count
FROM events AS e
JOIN page_hierarchy AS ph ON ph.page_id = e.page_id
WHERE product_category IS NOT NULL
GROUP BY product_category
ORDER BY 2 DESC;
```

**Explanation:**

- Use **SUM** and a **CASE** statement to count the number of views and card-adds per `product_category`.
- **JOIN** the tables `events` and `page_hierarchy` to get the `product_category` based on its `page_id`.
- Filter the resulting tables in the **WHERE** clause by omiting any **NULL** values for `product_category`.

**Results and Analysis:**

<img width="361" alt="image" src="https://github.com/user-attachments/assets/a2385a68-16c7-489f-b824-227ae5e339a9">

- The product category with the most views and card-adds is **Shellfish**.
- The product category with the least views and card-add is **Luxury**.

**9. What are the top 3 products by purchases?**

```sql
SELECT
page_name,
SUM(CASE WHEN event_type = 2 THEN 1 ELSE 0 END) AS purchased_count
FROM events AS e
JOIN page_hierarchy AS ph ON ph.page_id = e.page_id
WHERE e.page_id NOT IN (1,2,12,13) AND
visit_id IN (SELECT visit_id FROM events WHERE event_type = 3)
GROUP BY page_name
ORDER BY 2 DESC
LIMIT 3;
```

**Explanation:**
- Since we cannot directly count the number of purchases per product, we have to assume that every product that was added to the cart was purchased if the user has an event 3.
- Use **SUM** and a **CASE** statement to count the number of card-adds per `product_category`.
- Filter the data in the **WHERE** clause to omit the pages 1,2,12,13 and to take into account only those `visit_id` were there was actually a purchase.
- **LIMIT** the results to the top 3.

**Results and Analysis:**

<img width="284" alt="image" src="https://github.com/user-attachments/assets/889cfd0b-ad57-456d-b163-c6f437c14350">

- The top three products by purchases are **Lobster**, **Oyster** and **Crab**.
- The product with the highest number of purchases is **Lobster** with 754.

---
### **B. Product Funnel Analysis**

Using a single SQL query - create a new output table which has the following details:

- How many times was each product viewed?
- How many times was each product added to cart?
- How many times was each product added to a cart but not purchased (abandoned)?
- How many times was each product purchased?

Additionally, create another table which further aggregates the data for the above points but this time for each product category instead of individual products.

---
**1. Create new `products` table:**

```sql
CREATE TABLE products AS
WITH w AS (
WITH cte_views_cart AS
	(SELECT ph.page_name,
	SUM(CASE WHEN e.event_type = 1 THEN 1 ELSE 0 END) AS view_count,
	SUM(CASE WHEN e.event_type = 2 THEN 1 ELSE 0 END) AS add_cart_count
	FROM events AS e
	JOIN page_hierarchy AS ph ON e.page_id = ph.page_id
	WHERE e.page_id NOT IN (1,2,12,13)
	GROUP BY ph.page_name 
	ORDER BY 2 DESC),
abandoned_cte AS 
	(SELECT
	page_name,
	SUM(CASE WHEN event_type = 2 THEN 1 ELSE 0 END) AS abandoned_count
	FROM events AS e
	JOIN page_hierarchy AS ph ON ph.page_id = e.page_id
	WHERE e.page_id NOT IN (1,2,12,13) AND
	visit_id NOT IN (SELECT visit_id FROM events WHERE event_type = 3)
	GROUP BY page_name),
purchased_cte AS 
	(SELECT
	page_name,
	SUM(CASE WHEN event_type = 2 THEN 1 ELSE 0 END) AS purchased_count
	FROM events AS e
	JOIN page_hierarchy AS ph ON ph.page_id = e.page_id
	WHERE e.page_id NOT IN (1,2,12,13) AND
	visit_id IN (SELECT visit_id FROM events WHERE event_type = 3)
	GROUP BY page_name)
SELECT 
cvc.page_name as product,
cvc.view_count,
cvc.add_cart_count,
ac.abandoned_count,
pc.purchased_count
FROM cte_views_cart AS cvc
JOIN abandoned_cte AS ac ON ac.page_name = cvc.page_name
JOIN purchased_cte AS pc ON pc.page_name = cvc.page_name
)
SELECT * FROM w;
```

**The resulting table:**

<img width="500" alt="image" src="https://github.com/user-attachments/assets/03e1fdca-25d9-442e-8e16-926b44a18a3d">


**2. Create new `product_category` table:**

```sql
CREATE TABLE product_category AS
WITH w AS (
with cte_views_cart AS
	(SELECT ph.product_category,
	SUM(CASE WHEN e.event_type = 1 THEN 1 ELSE 0 END) AS view_count,
	SUM(CASE WHEN e.event_type = 2 THEN 1 ELSE 0 END) AS add_cart_count
	FROM events AS e
	JOIN page_hierarchy AS ph ON e.page_id = ph.page_id
	WHERE e.page_id NOT IN (1,2,12,13)
	GROUP BY ph.product_category 
	ORDER BY 2 DESC),
abandoned_cte AS 
	(SELECT
	product_category,
	SUM(CASE WHEN event_type = 2 THEN 1 ELSE 0 END) AS abandoned_count
	FROM events AS e
	JOIN page_hierarchy AS ph ON ph.page_id = e.page_id
	WHERE e.page_id NOT IN (1,2,12,13) AND
	visit_id NOT IN (SELECT visit_id FROM events WHERE event_type = 3)
	GROUP BY product_category),
purchased_cte AS 
	(SELECT
	product_category,
	SUM(CASE WHEN event_type = 2 THEN 1 ELSE 0 END) AS purchased_count
	FROM events AS e
	JOIN page_hierarchy AS ph ON ph.page_id = e.page_id
	WHERE e.page_id NOT IN (1,2,12,13) AND
	visit_id IN (SELECT visit_id FROM events WHERE event_type = 3)
	GROUP BY product_category)
SELECT 
cvc.product_category,
cvc.view_count,
cvc.add_cart_count,
ac.abandoned_count,
pc.purchased_count
FROM cte_views_cart AS cvc
JOIN abandoned_cte AS ac ON ac.product_category = cvc.product_category
JOIN purchased_cte AS pc ON pc.product_category = cvc.product_category
)
SELECT * FROM w;
```

**The resulting table:**

<img width="500" alt="image" src="https://github.com/user-attachments/assets/ce8dbe8b-03c6-46be-b45c-f2f95844ca49">

---
Use your 2 new output tables - answer the following questions:

**1. Which product had the most views, cart adds and purchases?**

```sql
SELECT product, view_count
FROM products
ORDER BY view_count DESC
LIMIT 1;
---
SELECT product, add_cart_count
FROM products
ORDER BY add_cart_count DESC
LIMIT 1;
---
SELECT product, purchased_count
FROM products
ORDER BY purchased_count DESC
LIMIT 1;
```
**Results and Analysis:**

1. Most views:

<img width="250" alt="image" src="https://github.com/user-attachments/assets/4d962c52-4185-4601-9944-89a755d69fc4">

- **Oyster** has the most views with 1568.

2. Most cart adds and purchases: 

<img width="272" alt="image" src="https://github.com/user-attachments/assets/4338d1fd-2db7-4d3d-ac5b-77409f058d22">

<img width="280" alt="image" src="https://github.com/user-attachments/assets/11866b48-95cc-48ca-b1fa-a591072571ee">

- **Lobster** has the most cart-add with 968 and purchases with 754.

**2. Which product was most likely to be abandoned?**

```sql
SELECT product, abandoned_count
FROM products
ORDER BY abandoned_count DESC
LIMIT 1;
```

**Results and Analysis:**

<img width="287" alt="image" src="https://github.com/user-attachments/assets/0427a89b-9018-434b-87ba-0273bdfa30ea">

- **Russian Caviar** is most likely to be abandoned with 249 counts.

**3. Which product had the highest view to purchase percentage?**

```sql
SELECT product,
ROUND((purchased_count::numeric/view_count)*100,2) AS view_purchase_percentage
FROM products
ORDER BY 2 DESC
LIMIT 1;
```

**Results and Analysis:**

<img width="333" alt="image" src="https://github.com/user-attachments/assets/a1e6ec16-0870-4fad-beaa-9f5b2693abfa">

- Almost half of the time an user views the product **Lobster**, they end up purchasing it.

**4. What is the average conversion rate from view to cart add?**

```sql
SELECT ROUND(AVG(conversion_rate),2) AS avg_conversion_rate
FROM
	(SELECT product,
	ROUND((add_cart_count::numeric/view_count)*100,2) AS conversion_rate
	FROM products) AS x;
```

**Results and Analysis:**

<img width="145" alt="image" src="https://github.com/user-attachments/assets/bf639251-956f-4d30-a777-8201e46877b7">

- The average conversion rate from view to cart add is **60.95%**.

**5. What is the average conversion rate from cart add to purchase?**

```sql
SELECT ROUND(AVG(conversion_rate),2) AS avg_conversion_rate
FROM
	(SELECT product,
	ROUND((purchased_count::numeric/add_cart_count)*100,2) AS conversion_rate
	FROM products) AS x;
```

**Results and Analysis:**

<img width="147" alt="image" src="https://github.com/user-attachments/assets/cc6d734b-0256-4137-8ec9-feee1093e8f4">

- The average conversion rate from cart add to purchase is **75.93%**.

---
### **C. Campaigns Analysis**

Generate a table that has 1 single row for every unique visit_id record and has the following columns:

- `user_id`
- `visit_id`
- `visit_start_time`: the earliest event_time for each visit
- `page_views`: count of page views for each visit
- `cart_adds`: count of product cart add events for each visit
- `purchase`: 1/0 flag if a purchase event exists for each visit
- `campaign_name`: map the visit to a campaign if the `visit_start_time` falls between the `start_date` and `end_date`
- `impression`: count of ad impressions for each visit
- `click`: count of ad clicks for each visit
- **(Optional column)** `cart_products`: a comma separated text value with products added to the cart sorted by the order they were added to the cart (hint: use the `sequence_number`)

Use the subsequent dataset to generate at least 5 insights for the Clique Bait team - bonus: prepare a single A4 infographic that the team can use for their management reporting sessions, be sure to emphasise the most important points from your findings.

Some ideas you might want to investigate further include:

- Identifying users who have received impressions during each campaign period and comparing each metric with other users who did not have an impression event
- Does clicking on an impression lead to higher purchase rates?
- What is the uplift in purchase rate when comparing users who click on a campaign impression versus users who do not receive an impression? What if we compare them with users who just an impression but do not click?
- What metrics can you use to quantify the success or failure of each campaign compared to eachother?

---


