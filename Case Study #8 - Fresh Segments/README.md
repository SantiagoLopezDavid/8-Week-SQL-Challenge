# 🍊 Case Study #8 - Fresh Segments

<img width="500" alt="image" src="https://github.com/user-attachments/assets/92ac910d-c4f7-43cb-99e6-1bb99ac1b605">

## Table of contents
1. Problem Statement.
2. Entity Relationship Diagram.
3. Solution.
  - A. Data Exploration and Cleansing.
  - B. Interest Analysis.
  - C. Segment Analysis.
  - D. Index Analysis.

## Problem Statement

Danny created Fresh Segments, a digital marketing agency that helps other businesses analyse trends in online ad click behaviour for their unique customer base.

Clients share their customer lists with the Fresh Segments team who then aggregate interest metrics and generate a single dataset worth of metrics for further analysis.

In particular - the composition and rankings for different interests are provided for each client showing the proportion of their customer list who interacted with online assets related to each interest for each month.

Danny has asked for your assistance to analyse aggregated metrics for an example client and provide some high level insights about the customer list and their interests.

## Entity Relationship Diagram

## Solution

### **A. Data Exploration and Cleansing**

1. Update the `fresh_segments.interest_metrics` table by modifying the `month_year` column to be a date data type with the start of the month.

```sql
ALTER TABLE interest_metrics 
ALTER COLUMN month_year TYPE DATE USING TO_DATE(month_year, 'MM-YYYY');
```

2. What is count of records in the `fresh_segments.interest_metrics` for each `month_year` value sorted in chronological order (earliest to latest) with the null values appearing first?.

```sql
SELECT month_year, COUNT(*) AS record_count 
FROM interest_metrics
GROUP BY month_year
ORDER BY month_year NULLS FIRST;
```

<img width="205" alt="image" src="https://github.com/user-attachments/assets/44432e2c-0191-4981-99b6-d53c4fab892e">

3. What do you think we should do with these null values in the `fresh_segments.interest_metrics`.

- Let's check what percentage of the entire table does the **NULL** values represent:

```sql
SELECT month_year, 
ROUND(100*(COUNT(*)::numeric/
		   (SELECT COUNT(*) FROM interest_metrics)),2) AS record_percentage 
FROM interest_metrics
GROUP BY month_year
ORDER BY month_year NULLS FIRST;
```
<img width="236" alt="image" src="https://github.com/user-attachments/assets/ac766bff-eede-481e-b0e7-d597be5e9745">

- Althought the **NULL** represent the highest percentage of the table with an 8.37%, it is still a not significant amount. We can create a clean version of `interest_metrics` without **NULL** values.

```sql
CREATE TABLE clean_interest_metrics AS
(SELECT * 
 FROM interest_metrics
 WHERE month_year IS NOT NULL);
```

- Let's check the **NULL** count in this new table:

```sql
SELECT 
COUNT(*) total_count,
SUM(CASE WHEN month_year IS NULL THEN 1 ELSE 0 END) AS null_count
FROM clean_interest_metrics;
```
<img width="186" alt="image" src="https://github.com/user-attachments/assets/563f28fe-0d17-4cbe-a0df-c2ea81a6bdb2">

- We have a total of 13079 rows and 0 **NULL** values.

4.. How many `interest_id` values exist in the `fresh_segments.interest_metrics` table but not in the `fresh_segments.interest_map` table? What about the other way around?

```sql
SELECT COUNT(interest_id) AS interest_id_count
FROM  clean_interest_metrics AS imt
LEFT JOIN interest_map AS im ON im.id = imt.interest_id::integer
WHERE im.id IS NULL;
```

<img width="126" alt="image" src="https://github.com/user-attachments/assets/ec81d3a4-baf3-424e-a5a9-6f5c21e42ab1">

- There is 0 `interest_id` in the `clean_interest_metrics` that do not exist in `interest_map`.

```sql
SELECT COUNT(im.id) AS interest_id_count
FROM  clean_interest_metrics AS imt
RIGHT JOIN interest_map AS im ON im.id = imt.interest_id::integer
WHERE imt.interest_id IS NULL;
```
<img width="130" alt="image" src="https://github.com/user-attachments/assets/bbd4499a-db67-4d54-863e-7f61adfe58b5">

- There are 7 `id` that are exist in the `interest_map` but not in the `clean_interest_metrics` table.

5.. Summarise the `id` values in the `fresh_segments.interest_map` by its total record count in this table.

```sql
SELECT COUNT(DISTINCT id) total_records_count
FROM interest_map;
```
<img width="141" alt="image" src="https://github.com/user-attachments/assets/da30408c-ea3b-435e-88c5-9d8999df4e0d">

- There are 1209 records in the `interest_map` table.

6.. What sort of table join should we perform for our analysis and why? Check your logic by checking the rows where `interest_id = 21246` in your joined output and include all columns from `fresh_segments.interest_metrics` and all columns from `fresh_segments.interest_map` except from the `id` column.

- We can perfomr a **INNER JOIN** between the tables `interest_map` and `clean_interest_metrics`.

``sql
SELECT 
imt._month, imt._year, imt.month_year, imt.interest_id, imt.composition, 
imt.index_value, imt.ranking, imt.percentile_ranking, im.interest_name, 
im.interest_summary, im.created_at, im.last_modified
FROM clean_interest_metrics AS imt
JOIN interest_map AS im ON im.id = imt.interest_id::integer;
```

7. Are there any records in your joined table where the `month_year` value is before the `created_at` value from the `fresh_segments.interest_map` table? Do you think these values are valid and why?

```sql
WITH CTE AS 
	(SELECT 
	imt._month, imt._year, imt.month_year, imt.interest_id, imt.composition, 
	imt.index_value, imt.ranking, imt.percentile_ranking, im.interest_name, 
	im.interest_summary, im.created_at, im.last_modified
	FROM clean_interest_metrics AS imt
	JOIN interest_map AS im ON im.id = imt.interest_id::integer)
SELECT COUNT(*) count_records
FROM cte
WHERE month_year < created_at;
```

<img width="114" alt="image" src="https://github.com/user-attachments/assets/09a57c8f-e6a1-402e-ac59-66c17f6b3df1">

- There are 188 records where the `month_year` value is before the `created_at` value. These values can be consider valid since when we change the data type for `month_year`, we set the date at the beginning of each month. As long as the `_month` and `_year` are correct, the value should be valid.

---
### **B. Interest Analysis**

**1. Which interests have been present in all `month_year` dates in our dataset?**

```sql
```

**Explanation:**

**Results and Analysis:**


**2. Using this same `total_months` measure - calculate the cumulative percentage of all records starting at 14 months - which `total_months` value passes the 90% cumulative percentage value?**

```sql
```

**Explanation:**

**Results and Analysis:**

**3. If we were to remove all `interest_id` values which are lower than the `total_months` value we found in the previous question - how many total data points would we be removing?**

```sql
```

**Explanation:**

**Results and Analysis:**

**4. Does this decision make sense to remove these data points from a business perspective? Use an example where there are all 14 months present to a removed `interest` example for your arguments - think about what it means to have less months present from a segment perspective.**

```sql
```

**Explanation:**

**Results and Analysis:**

**5. After removing these interests - how many unique interests are there for each month?**

```sql
```

**Explanation:**

**Results and Analysis:**

---
### **C. Segment Analysis**

**1. Using our filtered dataset by removing the interests with less than 6 months worth of data, which are the top 10 and bottom 10 interests which have the largest composition values in any `month_year`? Only use the maximum composition value for each interest but you must keep the corresponding `month_year`.**

```sql
```

**Explanation:**

**Results and Analysis:**

**2. Which 5 interests had the lowest average `ranking` value?**

```sql
```

**Explanation:**

**Results and Analysis:**

**3. Which 5 interests had the largest standard deviation in their `percentile_ranking` value?**

```sql
```

**Explanation:**

**Results and Analysis:**

**4. For the 5 interests found in the previous question - what was minimum and maximum `percentile_ranking` values for each interest and its corresponding `year_month` value? Can you describe what is happening for these 5 interests?**

```sql
```

**Explanation:**

**Results and Analysis:**

**5. How would you describe our customers in this segment based off their composition and ranking values? What sort of products or services should we show to these customers and what should we avoid?**

```sql
```

**Explanation:**

**Results and Analysis:**

---
### **D. Index Analysis**

The `index_value` is a measure which can be used to reverse calculate the average composition for Fresh Segments’ clients.

Average composition can be calculated by dividing the `composition` column by the `index_value` column rounded to 2 decimal places.

**1. What is the top 10 interests by the average composition for each month?**

```sql
```

**Explanation:**

**Results and Analysis:**

**2. For all of these top 10 interests - which interest appears the most often?**

```sql
```

**Explanation:**

**Results and Analysis:**

**3. What is the average of the average composition for the top 10 interests for each month?**

```sql
```

**Explanation:**

**Results and Analysis:**

**4. What is the 3 month rolling average of the max average composition value from September 2018 to August 2019 and include the previous top ranking interests in the same output shown below.**

```sql
```

**Explanation:**

**Results and Analysis:**


**5. Provide a possible reason why the max average composition might change from month to month? Could it signal something is not quite right with the overall business model for Fresh Segments?**

```sql
```

**Explanation:**

**Results and Analysis:**

---
