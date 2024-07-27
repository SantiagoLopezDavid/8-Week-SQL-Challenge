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

---
### Data Exploration

**1. What day of the week is used for each week_date value?**

```sql
```

**Explanation:**

**Results and Analysis:**

**2. What range of week numbers are missing from the dataset?**

```sql
```

**Explanation:**

**Results and Analysis:**

**3. How many total transactions were there for each year in the dataset?

```sql
```

**Explanation:**

**Results and Analysis:**

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
