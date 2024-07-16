# 🥑 Case Study #3: Foodie-Fi
<img width="500" alt="image" src="https://github.com/user-attachments/assets/dff6f33b-c433-436d-a4a5-010b7bc372c9">

## Table of contents
1. Problem Statement.
2. Entity Relationship Diagram.
3. Solution.

All the information regarding the case study has been sourced from [here](https://8weeksqlchallenge.com/case-study-3/)

## Problem Statement


## Entity Relationship Diagram
<img width="500" alt="image" src="https://github.com/user-attachments/assets/5b8e1159-73cd-4196-b2ea-6b9f1ac03c22">

## Solution
---
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

- **COUNT** the **DISTINCT** `customer_id` in the table `subscriptions`

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
	(SELECT 
	COUNT(*) AS total_customer_count,
	SUM(CASE
		WHEN final_plan=1 
		THEN 1 END) AS basic_monthly,
	SUM(CASE
		WHEN final_plan=2 
		THEN 1 END) AS pro_monthly,
	SUM(CASE
		WHEN final_plan=3 
		THEN 1 END) AS pro_annual,
	SUM(CASE
		WHEN final_plan=4 
		THEN 1 END) AS churn
	FROM
		(SELECT customer_id,
		MIN(plan_id) AS trail_plan,
		MAX(plan_id) AS final_plan
		FROM subscriptions
		GROUP BY customer_id
		ORDER BY customer_id) AS x)
SELECT total_customer_count,
basic_monthly,
ROUNG(basic_monthly::numeric/total_customer_count::numeric * 100,2) AS basic_monthly_percentage,
pro_monthly,
ROUND(pro_monthly::numeric/total_customer_count::numeric * 100,2) AS pro_monthly_percentage,
pro_annual,
ROUND(pro_annual::numeric/total_customer_count::numeric * 100,2) AS pro_annual_percentage,
churn,
ROUND(churn::numeric/total_customer_count::numeric *100,2) AS churn_percentage
FROM cte;
```
**Explanation:**



**Results and Analysis:**
<img width="1174" alt="image" src="https://github.com/user-attachments/assets/88b0c85d-430a-4d87-bd0e-73a60950f766">

- There is a total of 1000 unique customers.
- After the initial free trial:
  - 125 customers stay in the basic monthly plan, which represents 12.5% of the total.
  - 316 customers stay in the pro monthly plan, which represents 31.6% of the total.
  - 252 customers stay in the pro annual plan, which represents 25.2% of the total.
  - 307 customers decided to cancelled their plan after the free trail, which represents 30.7% of the total.
- Most of the customer stay in the pro monthly plan, which could mean that they enjoy the service and did not bother to change the plan after the free trail.
- The next big group is customer who decided to cancelled their plan as soon as the free trail plan was over. 

**7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?**

```sql
```
**Explanation:**

**Results and Analysis:**

---

**8. How many customers have upgraded to an annual plan in 2020?**

```sql
```
**Explanation:**

**Results and Analysis:**

---

**9. How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?**

```sql
```
**Explanation:**

**Results and Analysis:**

---

**10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)**

```sql
```
**Explanation:**

**Results and Analysis:**

---
**11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?**

```sql
```
**Explanation:**

**Results and Analysis:**

---
