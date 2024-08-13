# 🥑 Case Study #3: Foodie-Fi
<img width="500" alt="image" src="https://github.com/user-attachments/assets/dff6f33b-c433-436d-a4a5-010b7bc372c9">

## Table of contents
1. Problem Statement.
2. Entity Relationship Diagram.
3. Solution.

All the information regarding the case study has been sourced from [here](https://8weeksqlchallenge.com/case-study-3/)

## Problem Statement

Foodie-Fi is a subscription-based streaming service for food enthusiasts. Our task is to analyze customer and subscription data to gain insights into user behavior, subscription plans, and revenue trends. We will need to create and run SQL queries to answer various business questions and provide actionable insights based on the data provided.

## Entity Relationship Diagram
<img width="500" alt="image" src="https://github.com/user-attachments/assets/5b8e1159-73cd-4196-b2ea-6b9f1ac03c22">

## Solution

**Customer Journey**

Based off the 8 sample customers provided in the sample from the `subscriptions` table, write a brief description about some customer’s onboarding journey.

```sql
SELECT s.customer_id, s.plan_id, plan_name, price, s.start_date
FROM subscriptions s
LEFT JOIN plans ON plans.plan_id = s.plan_id
WHERE s.customer_id IN (1,2,11,13,15,16,18,19)
ORDER BY s.customer_id, s.plan_id
```
**Table 8 Sample Customers**

<img width="500" alt="image" src="https://github.com/user-attachments/assets/1229384e-a8d6-44ba-8b14-184af10d2dc8">

- **Customer 1:**
  - Customer 1 started his trial plan on '2020-08-01' and then change his plan to the basic monthly once the trial period was over.

<img width="500" alt="image" src="https://github.com/user-attachments/assets/0e6d7227-1ae6-4422-aadc-e8a3d4f9b6c2">

- **Customer 11:**
  - Customer 11 started his trial plan on '2020-11-19' and then decided not to continue once his trial period was over.

<img width="500" alt="image" src="https://github.com/user-attachments/assets/50aca6fd-ea54-4939-9dd1-6991497c25d7">

- **Customer 15:**
  - Customer 15 started his trial on '2020-03-17' and once his trial period was over he continue with the pro_monthly plan. He continue for one month before cancelling his subscription the '2020-04-29'

<img width="500" alt="image" src="https://github.com/user-attachments/assets/74302fdf-d5d4-416b-abd9-1e5585cd4dfd">

- **Customer 19:**
  - Customer 19 started hs trial on '2020-06-22' and continue with the pro-monthly plan after the trial period was over. A couple months after purchased the pro-annual plan.

<img width="500" alt="image" src="https://github.com/user-attachments/assets/857d129d-b361-4054-b8c2-d8abdf4c1e4b">

--- 
**1. How many customers has Foodie-Fi ever had?**
```sql
SELECT COUNT(DISTINCT customer_id) customer_count
FROM subscriptions;
```
**Explanation:**

- **COUNT** the **DISTINCT** `customer_id` in the table `subscriptions`.

**Results and Analysis:**

<img width="124" alt="image" src="https://github.com/user-attachments/assets/e0eb79d3-ff63-4537-9f06-b368c056e85c">

- Foodie-Fi has had a total of 1000 unique customers.

---

**2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value**

```sql
SELECT
EXTRACT(month FROM start_date) AS month_num,
TO_CHAR(start_date,'Month') AS month_str,
COUNT(plan_id) AS plan_count
FROM subscriptions
WHERE plan_id = 0
GROUP BY month_num,month_str
ORDER BY month_num;
```
**Explanation:**

- Use the **EXTRACT** function to get the 'month' field as a number from `start_date`.
- Use the **TO_CHAR** function to convert the `start_date` into the **'Month'** string.
- **COUNT** the number of `plan_id` per `month_num` and `month_str`.

**Results and Analysis:**

<img width="285" alt="image" src="https://github.com/user-attachments/assets/7cda9ae7-93d0-455b-b8ac-1baa9c37efa7">

- The month with the highest trial plan start date is March with a count of 94.
- The month with the lowest trial plan start date is February with a count of 68.

---

**3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name**

```sql
SELECT s.plan_id, plan_name,
COUNT(*) plan_count
FROM subscriptions s
JOIN plans on s.plan_id = plans.plan_id
WHERE EXTRACT(year FROM start_date) > 2020
GROUP BY s.plan_id, plan_name
ORDER BY s.plan_id;
```
**Explanation:**

- **JOIN** the tables `subscriptions` and `plans` to get the `plan_name` for each `plan_id`.
- Use the **WHERE** clause to filter the resulting table to those `start_date` that have a year greater than 2020.
- **COUNT** each of the results for each `plan_id` and `plan_name`.

**Results and Analysis:**

<img width="285" alt="image" src="https://github.com/user-attachments/assets/cbbf34e0-bfed-4897-904d-4daa62d3e6d6">

- Plan 1 or Basic Monthly has 8 subscriptions after the year 2020. The lowest of the year 2021.
- Plan 2 or Pro Monthly has 60 subscriptions after the year 2020.
- Plan 3 or Pro Annual has 63 subscriptions after the year 2020.
- Plan 4 of churn has a count of 71 customers. Churn is the highest count for the year 2021.
---

**4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?**

```sql
SELECT ROUND((count_churn::numeric/customer_count::numeric)*100,1) AS churn_percentage,
count_churn
FROM
	(SELECT 
	SUM(CASE WHEN plan_id = 4 THEN 1 END) AS count_churn,
	COUNT(DISTINCT customer_id) AS customer_count
	FROM subscriptions) AS x
```
**Explanation:**

- Use a subquery to **COUNT** the total of unique `customer_id` and **SUMM** the total of `plan_id = 4` from the database.
- In the main query, use the two variables from the subquery to calculate the `churn_percentage`.

**Results and Analysis:**

<img width="233" alt="image" src="https://github.com/user-attachments/assets/b47b7528-4108-457f-a0f6-85318091ed67">

- The customer churn count is 307 in total. Representing a 30.7% of the total count of unique customers.
---

**5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?**

```sql
WITH cte_churn AS
	(SELECT *
	FROM subscriptions
	WHERE plan_id = 4),
cte_trial AS
	(SELECT *
	 FROM subscriptions
	 WHERE plan_id=0),
cte_churn_after_trial AS 
	(SELECT 
	COUNT(cc.customer_id) AS customer_count
	FROM cte_churn cc
	JOIN cte_trial ct ON ct.customer_id = cc.customer_id
	WHERE cc.start_date - ct.start_date = 7)
SELECT customer_count,
ROUND(customer_count::numeric/COUNT(DISTINCT customer_id::numeric)*100,2) AS percentage_churn
FROM subscriptions, cte_churn_after_trial
GROUP BY customer_count;
```
**Explanation:**
- Create a **CTE** to get the `customer_id` where the `plan-id = 4`. Meaning they cancelled the subscription.
- Create a **CTE** to get the `customer_id` where the `plan-id = 0`. Meaning they started the free trail.
- Both these **CTEs** will have the `start_date` for the plans. We can use another **CTE** to calculate the difference between these dates and using the **WHERE** clause, filter the `customer_id` which difference between the `start_date` from their free trail and the date they decided to cancelled.
- Finally, use this `customer_count` to calculate the `percentage_churn` with all the unique `customer_id` from the table `subscriptions`.

**Results and Analysis:**

<img width="256" alt="image" src="https://github.com/user-attachments/assets/64290e0f-5c9a-4d03-a8e1-b17092f8b6af">

- There are 92 customers who cancelled their subscriptions right after their free trail period ended. This number is only 9.2% from the total count of customers.

---

**6. What is the number and percentage of customer plans after their initial free trial?**

```sql
WITH cte AS 
	(SELECT customer_id, plan_id, start_date,
	LEAD(plan_id) OVER(PARTITION BY customer_id ORDER BY plan_id) AS next_plan_id
	FROM subscriptions)
SELECT 
next_plan_id,
COUNT(customer_id) AS customer_count,
ROUND(COUNT(customer_id)::numeric/
	  (SELECT COUNT(DISTINCT customer_id) FROM subscriptions)*100,2) AS percentage
FROM cte
WHERE plan_id = 0
GROUP BY next_plan_id
ORDER BY next_plan_id;
```
**Explanation:**

- With a **CTE**, get every `next_plan_id` form every row.
- Using that `CTE` count the number of `customer_id` per group of `next_plan_id`. This would calculate the amount of customers converted after their free trail period.
- Calculate the `percentage` of each grup.
- Using the **WHERE** clause, filter the resulting table only by those rows that had the `next_plan_id` right after the `plan_id = 0`.

**Results and Analysis:**

<img width="320" alt="image" src="https://github.com/user-attachments/assets/3d54482f-903c-489d-9228-092e7766474a">

- There is a total of 1000 unique customers.
- After the initial free trial:
  - 546 customers stay in the basic monthly plan, which represents 54.6% of the total.
  - 325 customers stay in the pro monthly plan, which represents 32.5% of the total.
  - 37 customers stay in the pro annual plan, which represents 3.7% of the total.
  - 92 customers decided to cancelled their plan after the free trail, which represents 9.2% of the total.
- Most of the customer stay in the basic monthly plan, which meand that more than half the customers enjoy the service and decided to keep a plan but not the pro monthly.
- The next big group is customer who decided to keep their subscription as pro monthly. This could be because their did not bother to change their plan.
  
**7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?**

```sql
WITH cte AS 
	(SELECT customer_id,
	MAX(plan_id) AS final_plan
	FROM subscriptions
	WHERE start_date < '2020-12-31'
	GROUP BY customer_id
	ORDER BY customer_id)
SELECT final_plan AS plan_id,
COUNT(customer_id) AS customer_count,
ROUND(COUNT(customer_id)::numeric/
	  (SELECT COUNT(DISTINCT customer_id) FROM subscriptions)*100,2) AS percentage
FROM cte
GROUP BY plan_id
ORDER BY plan_id;
```
**Explanation:**

- With a **CTE**, get the **MAX** `plan_id` for each customer. This would be the last plan per customer for the year 2020.
- Using that `CTE` count the number of `customer_id` per group of `plan_id`. This would calculate the amount of customers per plan group by the end of the year.
- Calculate the `percentage` of each grup.

**Results and Analysis:**

<img width="298" alt="image" src="https://github.com/user-attachments/assets/573a2881-b3ce-4d9a-847a-92a26be11603">

- At the end of the year 2020:
  - 19 customers are in a free trail plan.
  - 224 customers are in the basic monthly plan. This represents a 22.4% of the total.
  - 327 customer are in the pro monthly plan. The highest percentage for the year, 32.7%.
  - 195 customers purchased the pro annual plan.
  - Finally 235 customers finished the year cancelling their plans.
---

**8. How many customers have upgraded to an annual plan in 2020?**

```sql
SELECT COUNT(customer_id) AS customer_count
FROM
	(SELECT customer_id, start_date,
	MAX(plan_id)
	FROM subscriptions
	GROUP BY customer_id, start_date
	HAVING MAX(plan_id) = 3 AND start_date < '2020-12-31'
	ORDER BY customer_id) AS x;
```
**Explanation:**

- Use a subquery to filter the table `subscriptions` by only those `customer_id` that have a `plan_id = 3` by the end of the year 2020.
- In the main query **COUNT** the number of `customer_id`.

**Results and Analysis:**

<img width="125" alt="image" src="https://github.com/user-attachments/assets/1afc2e88-2493-48b6-952e-8548d9b12034">

- There is a total of 195 customer who upgraded to the pro annual plan in the year 2020.

---

**9. How many days on average does it take for a customer to change to an annual plan from the day they join Foodie-Fi?**

```sql
WITH cte_trail AS
	(SELECT *
	FROM subscriptions
	WHERE plan_id = 0),
cte_annual AS 
	(SELECT *
	FROM subscriptions
	WHERE plan_id = 3)
SELECT 
ROUND(AVG(ca.start_date - ct.start_date)::numeric,2) AS date_diff_avg
FROM cte_trail ct
JOIN cte_annual ca ON ct.customer_id = ca.customer_id;
```
**Explanation:**

- Using two separate **CTEs** get separate the `customer_id` and `start_date` for each of the two plans to be used.
- Using the aggregate functon **AVG** calculate the average of the difference between the `start_date` of `plan_id = 3` and the `start_date` of `plan_id = 0`.

**Results and Analysis:**

<img width="109" alt="image" src="https://github.com/user-attachments/assets/d8fee030-69b6-4cbb-96b5-89c2734d83df">

- On average it takes ~105 days for a customer to change to an annual plan from their free trail plan. 

---

**10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)**

```sql
WITH cte_trail AS
	(SELECT *
	FROM subscriptions
	WHERE plan_id = 0),
cte_annual AS 
	(SELECT *
	FROM subscriptions
	WHERE plan_id = 3),
cte_bins AS 
	(SELECT ct.start_date AS trail_sd, 
	ca.start_date AS annual_sd,
	ca.start_date - ct.start_date AS date_diff,
	WIDTH_BUCKET(ca.start_date - ct.start_date, 0, 365, 12) AS bin
	FROM cte_trail AS ct
	JOIN cte_annual AS ca ON ct.customer_id = ca.customer_id
	ORDER BY date_diff)
SELECT
((bin - 1) * 30 || ' - ' || bin * 30 || ' days') AS period, 
COUNT(*) AS customer_count
FROM cte_bins
GROUP BY bin
ORDER BY bin;
```
**Explanation:**

- Using two separate **CTEs** get separate the `customer_id` and `start_date` for each of the two plans to be used.
- A third **CTE** will be used with the function **WIDTH_BUCKET** to separate the `date_diff` of each customer into 12 different bins/buckets.
- Using the last **CTE** , **COUNT** the amount of `customer_id` for each bin.

**Results and Analysis:**

<img width="224" alt="image" src="https://github.com/user-attachments/assets/04af1601-e67a-4ee2-81e6-bcb77550870c">

- Most of the customers who change their plans from the free trail to annual do it within the first 30 days of starting the trail period.
- There is a lower chance for a customer to change from a free trail to an annual plan if more than 210 days have gone by.

---
**11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?**

```sql
WITH cte_basic AS
	(SELECT *
	FROM subscriptions
	WHERE plan_id = 1),
cte_pro AS 
	(SELECT *
	FROM subscriptions
	WHERE plan_id = 2)
SELECT COUNT(*) AS customer_count
FROM cte_basic cb
JOIN cte_pro cp ON cb.customer_id = cp.customer_id
WHERE cb.start_date - cp.start_date > 0
AND cb.start_date < '2020-12-31' 
AND cp.start_date < '2020-12-31';
```
**Explanation:**

- Using two separate **CTEs** get separate the `customer_id` and `start_date` for each of the two plans to be used.
- Filter the table in the **WHERE** clause by those rows in which the difference between the `start_date` of 'basic monthly' and 'pro monthly' is grater than 0 (if its greater than 0 it meand that the `start_date` for the 'basic monthly' is later than the 'pro monthly' plan). Also, filter by those `start_date` before '2020-12-31'.

**Results and Analysis:**

<img width="123" alt="image" src="https://github.com/user-attachments/assets/aa095fed-453f-44c6-9c9a-bf8879e28573">

- There are 0 customers who dowgraded from a pro monthly to a basic montly plan in 2020.

---
