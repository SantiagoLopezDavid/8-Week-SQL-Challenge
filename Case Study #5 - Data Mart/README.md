# 🛒 Case Study #5 - Data Mart

<img width="500" alt="image" src="https://github.com/user-attachments/assets/023fb868-588c-4467-ad00-608dcf49dfdd">

## Table of contents
1. Problem Statement.
2. Entity Relationship Diagram.
3. Solution:
  - Data Cleansing
  - Data Exploration
  - Before & After Analysis

## Problem Statement

Danny is asking for your support to analyse his sales performance on Data Mart, an international online supermarket that specialises in fresh produce. Since June 2020 all Data Mart products now use sustainable packaging methods in every single step from the farm all the way to the customer.

Danny needs your help to quantify the impact of this change on the sales performance for Data Mart and it’s separate business areas. The key business question he wants you to help him answer are the following:

- What was the quantifiable impact of the changes introduced in June 2020?
- Which platform, region, segment and customer types were the most impacted by this change?
- What can we do about future introduction of similar sustainability updates to the business to minimise impact on sales?

## Entity Relationship Diagram

<img width="320" alt="image" src="https://github.com/user-attachments/assets/0b9b33d5-b987-48ae-a2c3-047e44115223">


## Solution

### Data Cleansing

In a single query, perform the following operations and generate a new table in the `data_mart` schema named `clean_weekly_sales`:

- Convert the `week_date` to a **DATE** format.
- Add a `week_number` as the second column for each `week_date` value, for example any value from the 1st of January to 7th of January will be 1, 8th to 14th will be 2 etc.
- Add a `month_number` with the calendar month for each `week_date` value as the 3rd column.
- Add a `calendar_year` column as the 4th column containing either 2018, 2019 or 2020 values.
- Add a new column called `age_band` after the original `segment` column using the following mapping on the number inside the `segment` value.

  <img width="230" alt="image" src="https://github.com/user-attachments/assets/87bd07e7-85e3-4f31-9437-666479815c77">

- Add a new `demographic` column using the following mapping for the first letter in the `segment` values.

  <img width="230" alt="image" src="https://github.com/user-attachments/assets/e9b1b0a2-39aa-4624-9aa2-4ee67ea188de">

- Ensure all **NULL** string values with an "unknown" string value in the original `segment` column as well as the new `age_band` and `demographic` columns.
- Generate a new `avg_transaction` column as the `sales` value divided by `transactions` rounded to 2 decimal places for each record.

**Solution**

```sql
DROP TABLE IF EXISTS clean_weekly_sales;
CREATE TABLE clean_weekly_sales AS ( 
SELECT
TO_DATE(week_date,'DD/MM/YY') as week_date,
DATE_PART('week', to_date(week_date,'DD/MM/YY')) AS week_number,
DATE_PART('month', to_date(week_date,'DD/MM/YY')) AS month_number,
DATE_PART('year', to_date(week_date,'DD/MM/YY')) AS calendar_year,
region,
platform,
CASE
	WHEN segment = 'null' THEN 'unknown'
	ELSE segment
	END AS segment,
CASE
	WHEN SUBSTRING (segment, 2, 1) = '1' THEN 'Young Adults'
	WHEN SUBSTRING (segment, 2, 1) = '2' THEN 'Middle Aged'
	WHEN SUBSTRING (segment, 2, 1) IN  ('3','4') THEN 'Retirees'
	ELSE 'unknown'
	END AS age_band,
CASE
	WHEN SUBSTRING (segment, 1, 1) = 'C' THEN 'Couples'
	WHEN SUBSTRING (segment, 1, 1) = 'F' THEN 'Families'
	ELSE 'unknown'
	END AS demographic,
customer_type,
transactions,
sales,
ROUND(sales::numeric/transactions,2) AS avg_transaction
FROM weekly_sales
);
```
---
### Data Exploration

**1. What day of the week is used for each week_date value?**

```sql
SELECT 
DISTINCT(TO_CHAR(week_date,'DAY')) AS week_day_str
FROM clean_weekly_sales;
```

**Explanation:**
- Use the **TO_CHAR** function to formate the `week_date` as a string 'DAY'.
- Select only the **DISTINCT** values.

**Results and Analysis:**

<img width="110" alt="image" src="https://github.com/user-attachments/assets/416b75a4-0403-4621-bf3a-f002a98e53e1">

- The day of the week used for `week_date` is Monday.

**2. What range of week numbers are missing from the dataset?**

```sql
WITH cte AS 
	(SELECT generate_series(1,52) AS year_weeks)
SELECT distinct year_weeks
	FROM cte
	LEFT JOIN clean_weekly_sales ON cte.year_weeks = clean_weekly_sales.week_number
	WHERE week_number IS NULL
	ORDER BY 1;
```

**Explanation:**

- Create a **CTE** to generate a range of number from 1 to 52.
- **LEFT JOIN** the **CTE** with `clean_weekly_sales` on the `year_weeks`.
- In the **WHERE** clause, filter the table by only those row were `year_weeks IS NULL`.

**Results and Analysis:**

<img width="145" alt="image" src="https://github.com/user-attachments/assets/ed702b9d-cd34-4e67-bd64-ac10b0c43b36"> 
<img width="145" alt="image" src="https://github.com/user-attachments/assets/a0a863a0-24f7-401c-a7f0-5abe846ea270">

- The range of week numbers missing are (1,12) and (37,52).

**3. How many total transactions were there for each year in the dataset?**

```sql
SELECT calendar_year, 
COUNT(transactions) transaction_count
FROM clean_weekly_sales
GROUP BY calendar_year
ORDER BY calendar_year;
```

**Explanation:**
- **COUNT** the `transactions` and then **GROUP BY** each `calendar_year`.

**Results and Analysis:**

<img width="256" alt="image" src="https://github.com/user-attachments/assets/18b7f538-3b6e-43ae-af38-bdb28364ea1f">

- In 2018 there were a total of 5698 transactions.
- In 2019 there were a total of 5708 transactions.
- In 2020 there were a total of 5711 transactions.

**4. What is the total sales for each region for each month?**

```sql
```

**Explanation:**

**Results and Analysis:**


**5. What is the total count of transactions for each platform**

```sql
```

**Explanation:**

**Results and Analysis:**

**6. What is the percentage of sales for Retail vs Shopify for each month?**

```sql
```

**Explanation:**

**Results and Analysis:**

**7. What is the percentage of sales by demographic for each year in the dataset?**

```sql
```

**Explanation:**

**Results and Analysis:**

**8. Which age_band and demographic values contribute the most to Retail sales?**

```sql
```

**Explanation:**

**Results and Analysis:**

**9. Can we use the avg_transaction column to find the average transaction size for each year for Retail vs Shopify? If not - how would you calculate it instead?**

```sql
```

**Explanation:**

**Results and Analysis:**


---
### Before & After Analysis

This technique is usually used when we inspect an important event and want to inspect the impact before and after a certain point in time. Taking the `week_date` value of **2020-06-15** as the baseline week where the Data Mart sustainable packaging changes came into effect.

We would include all `week_date` values for **2020-06-15** as the start of the period after the change and the previous week_date values would be before. Using this analysis approach - answer the following questions:

- What is the total sales for the 4 weeks before and after **2020-06-15**? What is the growth or reduction rate in actual values and percentage of sales?
- What about the entire 12 weeks before and after?
- How do the sale metrics for these 2 periods before and after compare with the previous years in 2018 and 2019?
