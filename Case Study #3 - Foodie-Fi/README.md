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

**Results and Analysis:**

<img width="285" alt="image" src="https://github.com/user-attachments/assets/cbbf34e0-bfed-4897-904d-4daa62d3e6d6">


---

**4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?**

```sql
```
**Explanation:**

**Results and Analysis:**

---

**5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?**

```sql
```
**Explanation:**

**Results and Analysis:**

---

**6. What is the number and percentage of customer plans after their initial free trial?**

```sql
```
**Explanation:**

**Results and Analysis:**

---

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
