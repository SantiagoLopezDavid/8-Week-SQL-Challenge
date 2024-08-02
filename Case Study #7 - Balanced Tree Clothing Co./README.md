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
```

**Explanation:**

**Results and Analysis:**

**2. What is the average unique products purchased in each transaction?**

```sql
```

**Explanation:**

**Results and Analysis:**

**3. What are the 25th, 50th and 75th percentile values for the revenue per transaction?**

```sql
```

**Explanation:**

**Results and Analysis:**

**4. What is the average discount value per transaction?**

```sql
```

**Explanation:**

**Results and Analysis:**

**5. What is the percentage split of all transactions for members vs non-members?**

```sql
```

**Explanation:**

**Results and Analysis:**

**6. What is the average revenue for member transactions and non-member transactions?**

```sql
```

**Explanation:**

**Results and Analysis:**

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
