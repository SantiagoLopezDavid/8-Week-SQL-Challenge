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

- In average members and non-members make the company similar revenue. The difference between their average is 1.23.

---

### **C. Product Analysis**

**1. What are the top 3 products by total revenue before discount?**

```sql
SELECT prod_id, product_name, revenue
FROM 
	(SELECT s.prod_id, pd.product_name, 
	SUM(s.qty * s.price) AS revenue,
	RANK() OVER(ORDER BY SUM(s.qty * s.price) DESC) AS rnk
	FROM sales AS s
	JOIN product_details AS pd ON pd.product_id = s.prod_id
	GROUP BY s.prod_id, pd.product_name) AS x
WHERE rnk IN(1,2,3);
```

**Explanation:**

- Use a subquery to calculate the **SUM** of the `revenue` per `prod_id`.
- Use the **RANK** function to organize the results from highest to lowest based on the `revenue`.
- Use the subquery to filter the results only by those rows with `rnk = 1,2,3`

**Results and Analysis:**

<img width="418" alt="image" src="https://github.com/user-attachments/assets/f7fbb39d-8f13-4a91-970b-08c021bc8e4c">

- The top 3 products based on their revenue before discounts are **Blue Polo Shirt - Man**, **Gret Fashion Jacket - Womens** and **White Tee Shirt - Mens**.

**2. What is the total quantity, revenue and discount for each segment?**

```sql
SELECT pd.segment_id, pd.segment_name,
SUM(s.qty) AS sum_quantity,
SUM(s.qty * s.price) AS revenue,
ROUND(SUM(s.qty*s.price*(s.discount::numeric/100)),2) AS sum_discount
FROM sales AS s
JOIN product_details AS pd ON pd.product_id = s.prod_id
GROUP BY pd.segment_id, pd.segment_name
ORDER BY pd.segment_id;
```

**Explanation:**

- Use the aggregate function **SUM** to get the `sum_quantity`, `revenue` and `sum_discount`.
- Group results by their `segment_id` and `segment_name`.

**Results and Analysis:**

<img width="540" alt="image" src="https://github.com/user-attachments/assets/7dd7549e-3152-4d85-b96a-e45279e0ce28">

- The segment with the highest revenue before discounts is **Shirts**.

**3. What is the top selling product for each segment?**

```sql
WITH cte AS
	(SELECT pd.segment_id, pd.segment_name, pd.product_name,  
	SUM(s.qty) AS sum_quantity,
	RANK() OVER(PARTITION BY pd.segment_id, pd.segment_name ORDER BY SUM(s.qty) DESC) AS rnk
	FROM sales AS s
	JOIN product_details AS pd ON pd.product_id = s.prod_id
	GROUP BY pd.segment_id, pd.segment_name, pd.product_name
	ORDER BY pd.segment_id, pd.segment_name)
SELECT segment_id, segment_name, product_name, sum_quantity
FROM cte
WHERE rnk = 1
ORDER BY segment_id, segment_name;
```

**Explanation:**

- In a **CTE**, use the function **RANK** based on the `sum_quantity` to get the order of products within each `segment_id`.
- Get the results from the **CTE** where the `rnk = 1`.

**Results and Analysis:**

<img width="547" alt="image" src="https://github.com/user-attachments/assets/fc22c044-509f-4454-ad10-05420d8840af">

- The top selling product from all segments is **Grey Fashion Jacket - Womens** from the **Jacket** segment.

**4. What is the total quantity, revenue and discount for each category?**

```sql
SELECT pd.category_id, pd.category_name,
SUM(s.qty) AS sum_quantity,
SUM(s.qty * s.price) AS revenue,
ROUND(SUM(s.qty*s.price*(s.discount::numeric/100)),2) AS sum_discount
FROM sales AS s
JOIN product_details AS pd ON pd.product_id = s.prod_id
GROUP BY pd.category_id, pd.category_name
ORDER BY pd.category_id;
```

**Explanation:**

- We can use the same code from the previous questions. The only thing to change is `category_id` and `category_name`.

**Results and Analysis:**

<img width="540" alt="image" src="https://github.com/user-attachments/assets/4531c438-15c5-44b5-8ff2-e50a402b4348">

- **Mens** category makes more revenue before discounts than **Womens**

**5. What is the top selling product for each category?**

```sql
WITH cte AS
	(SELECT pd.category_id, pd.category_name, pd.product_name,  
	SUM(s.qty) AS sum_quantity,
	RANK() OVER(PARTITION BY pd.category_id, pd.category_name ORDER BY SUM(s.qty) DESC) AS rnk
	FROM sales AS s
	JOIN product_details AS pd ON pd.product_id = s.prod_id
	GROUP BY pd.category_id, pd.category_name, pd.product_name
	ORDER BY pd.category_id, pd.category_name)
SELECT category_id, category_name, product_name, sum_quantity
FROM cte
WHERE rnk = 1
ORDER BY category_id, category_name;
```

**Explanation:**

- We can use the same code from the previous questions. The only thing to change is `category_id` and `category_name`.

**Results and Analysis:**

<img width="540" alt="image" src="https://github.com/user-attachments/assets/b8a409b7-aa0b-419e-9d99-e06144d84ac1">

- The product that sells the most for **Womans** is the **Grey Fashion Jacket**.
- The product that sells the most for **Mens** is the **Blue Polo Shirt**.

**6. What is the percentage split of revenue by product for each segment?**

```sql
WITH product_cte AS 
	(SELECT pd.segment_id, pd.segment_name, pd.product_name,
	SUM(s.qty * s.price) AS product_revenue
	FROM sales AS s
	JOIN product_details AS pd ON pd.product_id = s.prod_id
	GROUP BY pd.segment_id, pd.segment_name, pd.product_name
	ORDER BY pd.segment_id, pd.segment_name),
segment_cte AS 
	(SELECT pd.segment_id, pd.segment_name,
	SUM(s.qty * s.price) AS total_revenue
	FROM sales AS s
	JOIN product_details AS pd ON pd.product_id = s.prod_id
	GROUP BY pd.segment_id, pd.segment_name
	ORDER BY pd.segment_id, pd.segment_name)
SELECT pc.segment_id, pc.segment_name, pc.product_name, 
ROUND((pc.product_revenue::numeric/total_revenue)*100,2) AS percentage
FROM product_cte AS pc
JOIN segment_cte AS sc ON sc.segment_id = pc.segment_id;
```

**Explanation:**

- Use a **CTE** to calculate the **SUM** of `revenue` per `product_name` for each `segment_id`.
- Use a second **CTE** to get the `total_revenue` per `segment_id`.
- Use both results from the **CTEs** to calculate the `percentage` for each `product_name`.

**Results and Analysis:**

<img width="541" alt="image" src="https://github.com/user-attachments/assets/d7d1187b-e5cb-4ab1-b2c8-4001a6d997d8">

- The product with the highest percentage for **Jeans** is **Black Straight Jeans - Womens**.
- The product with the highest percetage for **Jacket** is **Grey Fashion Jacket - Womens**.
- The product with the highest percentage for **Shirt** is **Blue Polo Shirt - Mens**.
- The product with the highest percentage for **Socks** is **Navy Solid Socks - Mens**.

**7. What is the percentage split of revenue by segment for each category?**

```sql
WITH segment_cte AS 
	(SELECT pd.category_id, pd.category_name,pd.segment_id, pd.segment_name,
	SUM(s.qty * s.price) AS segment_revenue
	FROM sales AS s
	JOIN product_details AS pd ON pd.product_id = s.prod_id
	GROUP BY pd.category_id, pd.category_name,pd.segment_id, pd.segment_name
	ORDER BY pd.segment_id, pd.segment_name),
category_cte AS 
	(SELECT pd.category_id, pd.category_name,
	SUM(s.qty * s.price) AS total_revenue
	FROM sales AS s
	JOIN product_details AS pd ON pd.product_id = s.prod_id
	GROUP BY pd.category_id, pd.category_name
	ORDER BY pd.category_id, pd.category_name)
SELECT sc.category_id, sc.category_name, sc.segment_name, 
ROUND((sc.segment_revenue::numeric/total_revenue)*100,2) AS percentage
FROM segment_cte AS sc
JOIN category_cte AS cc ON sc.category_id = cc.category_id;
```

**Explanation:**

- We can use the same code from the previous question.

**Results and Analysis:**

<img width="484" alt="image" src="https://github.com/user-attachments/assets/f42f9301-fd90-4819-9e60-1eaad7366a3d">

- The segment with the highest percentage from the **Womens** category is **Jacket**.
- The segment with the highest percentage from the **Mens** category is **Shirt**.

**8. What is the percentage split of total revenue by category?**

```sql
SELECT pd.category_id, pd.category_name,
ROUND((SUM(s.qty::numeric * s.price::numeric)/
	   (SELECT SUM(qty*price) FROM sales))*100,2) AS percentage
FROM sales AS s
JOIN product_details AS pd ON pd.product_id = s.prod_id
GROUP BY pd.category_id, pd.category_name
ORDER BY pd.category_id, pd.category_name;
```

**Results and Analysis:**

<img width="339" alt="image" src="https://github.com/user-attachments/assets/1d00aafc-fc7d-45cb-9f01-989720cf18f8">

- **Mens** have the highest percentage split from the two categories.

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
