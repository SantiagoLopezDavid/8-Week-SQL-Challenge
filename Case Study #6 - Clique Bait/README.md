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
```

**Explanation:**

**Results and Analysis:**

**5. What is the percentage of visits which have a purchase event?**

```sql
```

**Explanation:**

**Results and Analysis:**

**6. What is the percentage of visits which view the checkout page but do not have a purchase event?**

```sql
```

**Explanation:**

**Results and Analysis:**

**7. What are the top 3 pages by number of views?**

```sql
```

**Explanation:**

**Results and Analysis:**

**8. What is the number of views and cart adds for each product category?**

```sql
```

**Explanation:**

**Results and Analysis:**

**9. What are the top 3 products by purchases?**

```sql
```

**Explanation:**

**Results and Analysis:**

---
### **B. Product Funnel Analysis**

Using a single SQL query - create a new output table which has the following details:

- How many times was each product viewed?
- How many times was each product added to cart?
- How many times was each product added to a cart but not purchased (abandoned)?
- How many times was each product purchased?

Additionally, create another table which further aggregates the data for the above points but this time for each product category instead of individual products.

Use your 2 new output tables - answer the following questions:

**1. Which product had the most views, cart adds and purchases?**

```sql
```

**Explanation:**

**Results and Analysis:**

**2. Which product was most likely to be abandoned?**

```sql
```

**Explanation:**

**Results and Analysis:**

**3. Which product had the highest view to purchase percentage?**

```sql
```

**Explanation:**

**Results and Analysis:**

**4. What is the average conversion rate from view to cart add?**

```sql
```

**Explanation:**

**Results and Analysis:**

**5. What is the average conversion rate from cart add to purchase?**

```sql
```

**Explanation:**

**Results and Analysis:**

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


