# 🌲 Case Study #7 - Balanced Tree Clothing Co.

<img width="500" alt="image" src="https://github.com/user-attachments/assets/c2d57f23-adee-4bb2-9ee3-248615bc61fc">

## Table of contents
1. Problem Statement.
2. Entity Relationship Diagram.
3. Solution.
  - A. High Level Sales Analysis.
  - B. Transaction Analysis.
  - C. Product Analysis.

All the information regarding the case study has been sourced from [here](https://8weeksqlchallenge.com/case-study-7/)

## Problem Statement

## Entity Relationship Diagram

<img width="500" alt="image" src="https://github.com/user-attachments/assets/e606fde8-18b1-43d3-8b67-094cac3970ad">

## Solution

### **A. High Level Sales Analysis**

**1. What was the total quantity sold for all products?**

```sql
SELECT s.prod_id, pd.product_name,
SUM(s.qty) AS sum_quantity
FROM sales AS s
LEFT JOIN product_details AS pd ON pd.product_id = s.prod_id
GROUP BY s.prod_id, pd.product_name
ORDER BY 3 DESC;
```

**Explanation:**

- **SUM** the `qty` and use the **GROUP BY** clause to group results by `prod_id` and `product_name`.

**Results and Analysis:**

<img width="456" alt="image" src="https://github.com/user-attachments/assets/ff46ac1b-b342-4cda-8cac-7067afacceb7">

- The product with the highest number of quantity sales is **Grey Fashion Jacket - Womens**.

**2. What is the total generated revenue for all products before discounts?**

```sql
SELECT s.prod_id,pd.product_name,
SUM (s.qty * s.price) AS revenue
FROM sales AS s
LEFT JOIN product_details AS pd ON pd.product_id = s.prod_id
GROUP BY s.prod_id, pd.product_name
ORDER BY 3 DESC;
```

**Explanation:**

- **SUM** the results from the multiplication of `qty` and `price` for each `prod_id`.
- **GROUP BY** the results by `prod_id` and `product_name`.

**Results and Analysis:**

<img width="430" alt="image" src="https://github.com/user-attachments/assets/738892ae-65c1-447d-928c-01b296916700">

- The product that generated the most revenue is **Blue Polo Shirt - Mens**.

**3. What was the total discount amount for all products?**

```sql
SELECT s.prod_id,pd.product_name,
ROUND(SUM (s.qty * s.price * (s.discount::numeric/100)),2) AS total_discount
FROM sales AS s
LEFT JOIN product_details AS pd ON pd.product_id = s.prod_id
GROUP BY s.prod_id, pd.product_name
ORDER BY 3 DESC;
```

**Explanation:**

- **SUM** the discount for each sale made for each `prod_id`.

**Results and Analysis:**

<img width="460" alt="image" src="https://github.com/user-attachments/assets/26cd8019-f3a8-4633-944e-e5deb41668bf">

- The product with the highest discount amount is **Blue Polo Shirt - Mens**.
---

### **B. Transaction Analysis**

**1. How many unique transactions were there?**

```sql
SELECT COUNT(DISTINCT txn_id) AS count_transactions
FROM sales;
```

**Results and Analysis:**

<img width="140" alt="image" src="https://github.com/user-attachments/assets/4031ac27-3c70-42f8-9d8d-337f4de05f39">

- There are 2500 unique transactions.

**2. What is the average unique products purchased in each transaction?**

```sql
SELECT ROUND(AVG(product_count),2) AS avg_unique_products
FROM
	(SELECT txn_id, COUNT(prod_id) AS product_count
	FROM sales
	GROUP BY txn_id) AS x;
```

**Explanation:**

- Use a subquery to **COUNT** the `prod_id` for each `txn_id `.
- Use the aggregate function **AVG** to get the `avg_unique_products`.

**Results and Analysis:**

<img width="149" alt="image" src="https://github.com/user-attachments/assets/4065019b-e00e-4689-9782-a9a0ba23744c">

- In average, users buy around 6 unique products per transaction.

**3. What are the 25th, 50th and 75th percentile values for the revenue per transaction?**

```sql
SELECT 
PERCENTILE_CONT(0.25) WITHIN GROUP(ORDER BY revenue) AS percentile_25,
PERCENTILE_CONT(0.5) WITHIN GROUP(ORDER BY revenue) AS percentile_50,
PERCENTILE_CONT(0.75) WITHIN GROUP(ORDER BY revenue) AS percentile_75
FROM
	(SELECT txn_id, 
	SUM(qty*price) AS revenue
	FROM sales
	GROUP BY txn_id) AS x;
```

**Explanation:**

- Use a subquery to calculate the `revenue` per `txn_id`.
- Use the results from the subquery to calculate the percentiles with the **PERCENTILE_CONT** function.

**Results and Analysis:**

<img width="370" alt="image" src="https://github.com/user-attachments/assets/ba24d7b1-33f1-46cf-b4be-ddd6ea6d6f82">

- The value for percentile 25 is 375.75
- The value for percentile 50 is 509.5
- The value for percentile 75 is 647.

**4. What is the average discount value per transaction?**

```sql
WITH discount_cte AS 
	(SELECT txn_id, 
	ROUND(SUM(qty*price*(discount::numeric/100)),2) AS discount_value
	FROM sales
	GROUP BY txn_id)
SELECT ROUND(AVG(discount_value),2) AS avg_discount_value
FROM discount_cte;
```

**Explanation:**

- Using a **CTE** calculate the `discount_value` per `txn_id`.
- From the **CTE** calculate the **AVG** `discount_value`.

**Results and Analysis:**

<img width="142" alt="image" src="https://github.com/user-attachments/assets/e8f3c859-ffcb-44ea-bf30-1ee451a16501">

- The average discount value per transaction is 62.49.

**5. What is the percentage split of all transactions for members vs non-members?**

```sql
WITH member_cte AS
	(SELECT member,
	COUNT(DISTINCT txn_id) AS txn_count
	FROM sales
	GROUP BY member)
SELECT member, txn_count,
ROUND((txn_count::numeric/(SELECT COUNT(DISTINCT txn_id) FROM sales))*100,2) AS percentage
FROM member_cte;
```

**Explanation:**

- Using a **CTE**, **COUNT** the unique `txn_id` by `member`.
- Calculate the `percentage` base on the total of unique `txn_id` in the `sales` table for each `member`. 

**Results and Analysis:**

**True : Member
False : Non-Member**

<img width="265" alt="image" src="https://github.com/user-attachments/assets/cb7864ef-852e-4f17-97cf-ccd7090df769">

- **Members** have a bigger percentage split for all transactions compare to **Non-members**. 

**6. What is the average revenue for member transactions and non-member transactions?**

```sql
SELECT member, 
ROUND(AVG(revenue),2) AS avg_revenue
FROM 
	(SELECT txn_id, member, 
	SUM(qty * price) AS revenue
	FROM sales
	GROUP BY txn_id, member) AS x
GROUP BY member;
```

**Explanation:**

- Use a subquery to calculate the `revenue` for each individual `txn_id`.
- Use the result from the subquery to calculate the **AVG** from the `revenue` column per `member`.

**Results and Analysis:**

<img width="182" alt="image" src="https://github.com/user-attachments/assets/45e37d1b-9d29-40bc-acb7-a494081d8156">

- In average members and non-members make the company similar revenue. The difference between them is less than  



---

### **C. Product Analysis**

**1. What are the top 3 products by total revenue before discount?**

```sql
```

**Explanation:**

**Results and Analysis:**

**2. What is the total quantity, revenue and discount for each segment?**

```sql
```

**Explanation:**

**Results and Analysis:**

**3. What is the top selling product for each segment?**

```sql
```

**Explanation:**

**Results and Analysis:**

**4. What is the total quantity, revenue and discount for each category?**

```sql
```

**Explanation:**

**Results and Analysis:**

**5. What is the top selling product for each category?**

```sql
```

**Explanation:**

**Results and Analysis:**

**6. What is the percentage split of revenue by product for each segment?**

```sql
```

**Explanation:**

**Results and Analysis:**

**7. What is the percentage split of revenue by segment for each category?**

```sql
```

**Explanation:**

**Results and Analysis:**

**8. What is the percentage split of total revenue by category?**

```sql
```

**Explanation:**

**Results and Analysis:**

**9. What is the total transaction “penetration” for each product? (hint: penetration = number of transactions where at least 1 quantity of a product was purchased divided by total number of transactions)**

```sql
```

**Explanation:**

**Results and Analysis:**

**10. What is the most common combination of at least 1 quantity of any 3 products in a 1 single transaction?**

```sql
```

**Explanation:**

**Results and Analysis:**

---
