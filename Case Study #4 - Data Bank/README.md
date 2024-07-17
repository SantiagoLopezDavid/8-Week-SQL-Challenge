# 🏛️ Case Study #4 - Data Bank

<img width="500" alt="image" src="https://github.com/user-attachments/assets/b246f5cd-162b-42f7-8ee5-4f6bb1865a29">

## Table of contents
1. Problem Statement.
2. Entity Relationship Diagram.
3. Solution.
  - A. Customer Nodes Exploration.
  - B. Customer Transactions.

All the information regarding the case study has been sourced from [here](https://8weeksqlchallenge.com/case-study-4/)

## Problem Statement

The management team at Data Bank want to increase their total customer base - but also need some help tracking just how much data storage their customers will need. This case study is all about calculating metrics, growth and helping the business analyse their data in a smart way to better forecast and plan for their future developments.

## Entity Relationship Diagram

<img width="500" alt="image" src="https://github.com/user-attachments/assets/b81d1d36-5b9d-49eb-92c2-e58e1fc43f3e">

## Solution

### **A. Customer Nodes Exploration**

**1. How many unique nodes are there on the Data Bank system?**

```sql
SELECT COUNT(DISTINCT node_id) AS node_count
FROM customer_nodes;
```

**Explanation:**

- **COUNT** all the **DISTINCT** `node_id` from the table `customer_nodes`.

**Results and Analysis:**

<img width="99" alt="image" src="https://github.com/user-attachments/assets/bf0f1a8a-3214-4ab3-a8ed-274131faafff">

- There are 5 unique nodes on the Data Bank System.

---

**2. What is the number of nodes per region?**

```sql
SELECT cn.region_id, r.region_name,
COUNT(cn.node_id) AS total_node_count,
COUNT(DISTINCT cn.node_id) AS unique_node_count
FROM customer_nodes cn
LEFT JOIN regions r ON cn.region_id = r.region_id
GROUP BY cn.region_id, r.region_name
ORDER BY cn.region_id;
```

**Explanation:**

- **COUNT** all the `node_id` per `region_id` and `region_name`.
- **COUNT** all the **DISTINCT** `node_id` per `region_id` and `region_name`.
- Group all results per `region_id` and `region_name`.

**Results and Analysis:**

<img width="500" alt="image" src="https://github.com/user-attachments/assets/e967c21d-080a-4dfd-9ba1-70e96536de02">

- All regions have a total of 5 unique nodes. All regions have all nodes.
  - There are a total of 770 nodes for region 1 or Australia.
  - There are a total of 735 nodes for region 2 or America.
  - There are a total of 714 nodes for region 3 or Africa.
  - There are a total of 665 nodes for region 4 or Asia.
  - There are a total of 616 nodes for region 5 or Europe.
---

**3. How many customers are allocated to each region?**

```sql
SELECT cn.region_id, r.region_name,
COUNT(DISTINCT cn.customer_id) AS unique_customer_count
FROM customer_nodes cn
LEFT JOIN regions r ON cn.region_id = r.region_id
GROUP BY cn.region_id, r.region_name
ORDER BY cn.region_id;
```

**Explanation:**

- **COUNT** the number of **DISTINCT** `customer_id` for each `region_id`.

**Results and Analysis:**

<img width="392" alt="image" src="https://github.com/user-attachments/assets/364ea440-c217-40be-ad1a-b3d334852898">

- There are a total of 110 customers for region 1 / Australia.
- There are a total of 105 customers for region 2 / America.
- There are a total of 102 customers for region 3 / Africa.
- There are a total of 95 customers for region 4 / Asia.
- There are a total of 88 customers for region 5 / Europe.
---

**4. How many days on average are customers reallocated to a different node?**

```sql
SELECT
ROUND(AVG(end_date - start_date)::numeric,2) AS avg_days
FROM customer_nodes
WHERE end_date <> '9999-12-31';
```

**Explanation:**

- Calculate the **AVG** of the difference between the `end_date` and `start_date` for each customer.
- Looking into the table, we can see that there are final instances where a `customer_id` will be assign to a node until the year '9999-12-31'. To get a correct calculation of the average, avoid using this rows. If we don't take this rows out of the calculation, the number will be too big and won't make sense.

**Results and Analysis:**

<img width="87" alt="image" src="https://github.com/user-attachments/assets/124b6365-ecf1-459c-95a5-06d79d5049b2">

- Customers are reallocated to a different node an average of 14.63 days.
---

**5. What is the median, 80th and 95th percentile for this same reallocation days metric for each region?**

```sql
SELECT region_id,
PERCENTILE_CONT(0.5) WITHIN GROUP(ORDER BY end_date - start_date) AS median,
PERCENTILE_CONT(0.8) WITHIN GROUP(ORDER BY end_date - start_date) AS percentile_80th,
PERCENTILE_CONT(0.95) WITHIN GROUP(ORDER BY end_date - start_date) AS percentile_95th
FROM customer_nodes
WHERE end_date <> '9999-12-31'
GROUP BY region_id;
```

**Explanation:**
- **PERCENTILE_CONT** calculates a percentile based on a continuous distribution of the column. So if you specify 0.5 as its argument, it returns the median using interpolation between two middle adjacent values if the number of observation is even (and returns just the middle value if odd).
- We use this same function for the 80th and 95th percentile.
- Avoid using the rows with a `end_date` as '9999-12-31'.
**Results and Analysis:**

<img width="454" alt="image" src="https://github.com/user-attachments/assets/714be678-71cb-4941-8e80-47ac5fde7c16">

- The median of days that customers are reallocated for each region is 15 days.
- For the 95th percentile, all regions have a value of 28 days.
- For the 80th percentile , values are between 23 and 24 days.

---
### **B. Customer Transactions**

**1. What is the unique count and total amount for each transaction type?**

```sql
```

**Explanation:**

**Results and Analysis:**

---

**2. What is the average total historical deposit counts and amounts for all customers?**

```sql
```

**Explanation:**

**Results and Analysis:**

---

**3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?**

```sql
```

**Explanation:**

**Results and Analysis:**

---

**4. What is the closing balance for each customer at the end of the month?**

```sql
```

**Explanation:**

**Results and Analysis:**

---

**5. What is the percentage of customers who increase their closing balance by more than 5%?**

```sql
```

**Explanation:**

**Results and Analysis:**

---
