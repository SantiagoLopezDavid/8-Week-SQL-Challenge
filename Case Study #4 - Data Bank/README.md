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
SELECT txn_type,
SUM(txn_amount) AS total_amount
FROM customer_transactions
GROUP BY txn_type;
```

**Explanation:**

- Using the aggregrate function **SUM**, calculate the `total_amount` for each `txn_type`.

**Results and Analysis:**

<img width="259" alt="image" src="https://github.com/user-attachments/assets/ef6e85c1-1605-4e10-847b-ec341fe8faf2">

- There are three unique type of transacitions : Purhcase, Withdrawal and Deposit.
- The total amount for purchase is $806.537.
- The total amount for withdrawal is $793.003.
- The total amount for deposit is $1.359.168.
---

**2. What is the average total historical deposit counts and amounts for all customers?**

```sql
WITH cte_deposit AS
	(SELECT customer_id,
	COUNT(txn_type) AS deposit_count,
	ROUND(AVG(txn_amount)::numeric,2) AS avg_amount
	FROM customer_transactions
	WHERE txn_type = 'deposit'
	GROUP BY customer_id
	ORDER BY customer_id)
SELECT 
ROUND(AVG(deposit_count)::numeric,2) AS avg_deposit,
ROUND(avg(avg_amount)::numeric,2) AS avg_deposit
FROM cte_deposit;
```

**Explanation:**

- Create a **CTE** to calculate the `deposit_count` and the `avg_amount` per `customer_id`.
- Use the **WHERE** clause to filter results to only those with `txn_type = 'deposit'`
- Finally calculate the overall average of `deposit_count` and `avg_amount`.

**Results and Analysis:**

<img width="201" alt="image" src="https://github.com/user-attachments/assets/6a1be208-67ee-4d22-b63c-bcdb0574e729">

- In average customers make 5.34 deposits with an average value of $508.61.
---

**3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?**

```sql
WITH cte AS 
	(SELECT
	EXTRACT(MONTH FROM txn_date) AS month_num,
	TO_CHAR(txn_date,'Mon') AS month_str,
	customer_id,
	SUM(CASE WHEN txn_type = 'deposit' THEN 1 END) AS deposit_count,
	SUM(CASE WHEN txn_type = 'withdrawal' THEN 1 END) AS withdrawal_count,
	SUM(CASE WHEN txn_type = 'purchase' THEN 1 END) AS purchase_count
	FROM customer_transactions
	GROUP BY month_num,month_str,customer_id
	ORDER BY month_num,month_str,customer_id)
SELECT month_num, month_str,
SUM(CASE
	WHEN deposit_count > 1 AND (withdrawal_count >=1 OR purchase_count >=1)
	THEN 1
	end) AS customer_count
FROM cte
GROUP BY month_num, month_str
ORDER BY month_num, month_str;
```

**Explanation:**

- Create a **CTE** to count the times each customer makes a 'deposit','withdrawal' and 'purchase'.
- **GROUP BY** the 'month' to get the count for each customer per each month.
- Using the **CTE** count the number of customer who fit the desire condition.

**Results and Analysis:**

<img width="312" alt="image" src="https://github.com/user-attachments/assets/923014ac-3f1e-48c3-9d56-fc1689fc8ed9">

- In January there are 168 customers who made more than 1 deposit and either 1 purchase or 1 withdrawal.
- In February there are 181 customers who made more than 1 deposit and either 1 purchase or 1 withdrawal.
- In March there are 192 customers who made more than 1 deposit and either 1 purchase or 1 withdrawal.
- In April there are 70 customers who made more than 1 deposit and either 1 purchase or 1 withdrawal.
---

**4. What is the closing balance for each customer at the end of the month?**

```sql
WITH cte_current_month AS 
	(SELECT customer_id,
	EXTRACT(month FROM txn_date) AS txn_month,
	TO_CHAR(txn_date,'Mon') AS txn_month_str,
	SUM(CASE WHEN txn_type = 'deposit' THEN txn_amount
		ELSE txn_amount*-1
		END) OVER(PARTITION BY customer_id,EXTRACT(month FROM txn_date)) AS end_month
	FROM customer_transactions
	ORDER BY customer_id, txn_month, txn_month_str)
SELECT customer_id, txn_month, txn_month_str,
SUM(end_month) OVER(PARTITION BY customer_id ORDER BY txn_month) AS running_total
FROM cte_current_month
GROUP BY customer_id, txn_month, txn_month_str, end_month
ORDER BY customer_id, txn_month, txn_month_str;
```

**Explanation:**

- Use a **CTE** to calculate the amount of funds in the account for each `customer_id`. This `end_month` will be independent for each month.
- Using the past **CTE**, calculate a `running_total` for each `customer_id`. This **SUM** window function will take into account the amounts for the past month to calculate the current balance and the total at the end of the month.

**Results and Analysis:**

<img width="410" alt="image" src="https://github.com/user-attachments/assets/e82ed9a2-dfd8-4606-8345-f34d1728e197">

- The previous table shows the results for the running total for customers 1,2,3,4,5 for each of the months where they had any transaction.
---

**5. What is the percentage of customers who increase their closing balance by more than 5%?**
