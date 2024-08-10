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

All the information regarding the case study has been sourced from [here](https://8weeksqlchallenge.com/case-study-8/)

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

- We can perform an **INNER JOIN** between the tables `interest_map` and `clean_interest_metrics`.

```sql
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
SELECT interest_id,
COUNT(DISTINCT month_year) month_count
FROM clean_interest_metrics
GROUP BY interest_id
HAVING COUNT(DISTINCT month_year) = 
	(SELECT COUNT(DISTINCT month_year) FROM clean_interest_metrics);
```

**Explanation:**

- Using a subquery, get the total number of unique `month_year` values.
- Group by all `interest_id` and get the **COUNT** of unique `month_year` for all of them.
- Using the **HAVING** clause, filter the resulting `interest_id` to those which `month_count` is equal to the total unique `month_year`.

**Results and Analysis:**

Partial screenshot from resulting table:

<img width="146" alt="image" src="https://github.com/user-attachments/assets/dc62e8be-2ed1-49af-b7d0-05da926ef7da">

There are a total of 480 interest that are present in all 14 `month_year` dates.

**2. Using this same `total_months` measure - calculate the cumulative percentage of all records starting at 14 months - which `total_months` value passes the 90% cumulative percentage value?**

```sql
WITH total_months_cte AS 
	(SELECT interest_id,
	COUNT(DISTINCT month_year) AS total_months
	FROM clean_interest_metrics
	GROUP BY interest_id
	ORDER BY 2),
interest_count_cte AS
	(SELECT 
	total_months, 
	COUNT(interest_id) interest_count
	FROM total_months_cte
	GROUP BY total_months
	ORDER BY total_months)
SELECT total_months, interest_count,
ROUND((100 * SUM(interest_count) OVER(ORDER BY total_months DESC)/
	SUM(interest_count) OVER()),2) AS cum_percentage
FROM interest_count_cte
GROUP BY total_months, interest_count;
```
**Explanation:**

- Use one **CTE** to **COUNT** the number of unique `month_year` for all `interest_id`.
- Use the first **CTE** to **GROUP BY** the results by the **total_months**.
- As a final query, use the **interest_count_cte** to calculate the `cum_percentage` in descending order.

**Results and Analysis:**

<img width="344" alt="image" src="https://github.com/user-attachments/assets/dea5db24-a141-436e-ae01-41c35f69f8a5">

- Starting from the 14th month down, its at the sixth month that the cummulative percentage passes the 90% value.

**3. If we were to remove all `interest_id` values which are lower than the `total_months` value we found in the previous question - how many total data points would we be removing?**

```sql
SELECT COUNT(*) AS data_points_count
FROM clean_interest_metrics
WHERE interest_id in
	(SELECT interest_id
	FROM clean_interest_metrics
	GROUP BY interest_id
	HAVING COUNT(month_year) < 6);
```

**Explanation:**
- Using a subquery, get all the `interest_id` where the **COUNT** of `month_year` is less than 6.
- In the main query, **COUNT** all the rows of `interest_id` that correspond to those in the subquery.

**Results and Analysis:**

<img width="135" alt="image" src="https://github.com/user-attachments/assets/56401354-80a3-4f35-9a99-504362bfe93f">

- We would be removing a total of 400 data points.

**4. Does this decision make sense to remove these data points from a business perspective? Use an example where there are all 14 months present to a removed `interest` example for your arguments - think about what it means to have less months present from a segment perspective.**

It does makes sense to remove this data points from the data set. 
- First of all, the 400 data points represent 3% of all the data set. It is a very small percentage to change the final outcome or analysis.
- Second, `interest_id`s that have not been present in more than 6 months of the year might not represent a valuable trend but just random topics of interest of consumers. The client could be more interested in `interest_id` with a bigger presence during the year.

From a segment perspective, less months means being able to focus on what actually will bring more value to the client. It not just having less data to analyse, but having more meaningful data to draw insights from.

Let's create a new table `interest_removed` with the `interest_id` removed:

```sql
CREATE TABLE interest_removed AS
(SELECT *
FROM clean_interest_metrics
WHERE interest_id not in
	(SELECT interest_id
	FROM clean_interest_metrics
	GROUP BY interest_id
	HAVING COUNT(month_year) < 6));
```

**5. After removing these interests - how many unique interests are there for each month?**

```sql
SELECT month_year,
COUNT(DISTINCT interest_id) AS interest_count
FROM interest_removed
GROUP BY 1;
```

**Explanation:**
- From the new table `interest_removed`, **COUNT** all unique `interest_id`.
- **GROUP BY** `month_year`.

**Results and Analysis:**

<img width="211" alt="image" src="https://github.com/user-attachments/assets/b2658ec5-852c-4b6f-a860-599ffc723a7f">

---
### **C. Segment Analysis**

**1. Using our filtered dataset by removing the interests with less than 6 months worth of data, which are the top 10 and bottom 10 interests which have the largest composition values in any `month_year`? Only use the maximum composition value for each interest but you must keep the corresponding `month_year`.**

- **Largest composition**

```sql
SELECT month_year, interest_id,
MAX(composition) AS max_composition
FROM interest_removed
GROUP BY month_year, interest_id
ORDER BY 3 DESC
LIMIT 10;
```

**Explanation:**

- **GROUP BY** `month_year` and `interest_id` and get the **MAX** value for `composition` for each `interest_id`.
- **ORDER BY** the **MAX** value in descending order.

**Results and Analysis:**

<img width="369" alt="image" src="https://github.com/user-attachments/assets/c871201c-df05-4c21-8676-cc77abc9de99">


- **Smallest composition**

```sql
SELECT month_year, interest_id,
MAX(composition) AS max_composition
FROM interest_removed
GROUP BY month_year, interest_id
ORDER BY 3 
LIMIT 10;
```
**Explanation:**

- **ORDER BY** the **MAX** value in ascending order.

**Results and Analysis:**

<img width="367" alt="image" src="https://github.com/user-attachments/assets/41f5acba-40b2-4b69-8d8f-1b3b7f3039a6">


**2. Which 5 interests had the lowest average `ranking` value?**

```sql
SELECT interest_id , interest_name,
ROUND(AVG(ranking),2) avg_ranking
FROM interest_removed
JOIN interest_map ON interest_map.id = interest_removed.interest_id::integer
GROUP BY interest_id, interest_name
ORDER BY 3 
LIMIT 5;
```

**Explanation:**

- Calculate the **AVG** of `ranking` for all `interest_id`.
- Limit the results to 5 rows.

**Results and Analysis:**

<img width="426" alt="image" src="https://github.com/user-attachments/assets/c4014690-6f03-41db-b821-6c826d40ddfc">

**3. Which 5 interests had the largest standard deviation in their `percentile_ranking` value?**

```sql
SELECT interest_id,
ROUND(STDDEV(percentile_ranking)::numeric, 2) AS std_dev
FROM interest_removed
GROUP BY interest_id
ORDER BY 2 DESC
LIMIT 5;
```

**Explanation:**

- Calculate the **STDDEV** of the `percentile_ranking` value for all `interest_id`.
- Order the results based on the `std_dev` in descending order.
- Limit the results to 5 rows.

**Results and Analysis:**

<img width="226" alt="image" src="https://github.com/user-attachments/assets/33af7659-0e8a-432a-b9f1-5b82e5ea571c">


**4. For the 5 interests found in the previous question - what was minimum and maximum `percentile_ranking` values for each interest and its corresponding `year_month` value? Can you describe what is happening for these 5 interests?**

```sql
WITH cte AS 
	(SELECT interest_id,
	MIN(percentile_ranking) AS min_percentile,
	MAX(percentile_ranking) AS max_percentile
	FROM interest_removed
	WHERE interest_id IN 
		(SELECT interest_id
		FROM interest_removed
		GROUP BY interest_id
		ORDER BY STDDEV(percentile_ranking) DESC
		LIMIT 5)
	GROUP BY interest_id)
SELECT cte.interest_id, 
cte.min_percentile,
MIN(t1.month_year) AS month_year_min,
cte.max_percentile,
MIN(t2.month_year) AS month_year_max
FROM cte 
LEFT JOIN interest_removed AS t1 ON cte.min_percentile = t1.percentile_ranking
LEFT JOIN interest_removed AS t2 ON cte.max_percentile = t2.percentile_ranking
GROUP BY 1, 2, 4
ORDER BY 1;
```

**Results and Analysis:**

<img width="644" alt="image" src="https://github.com/user-attachments/assets/d6997958-e8ae-41e3-97c1-d9a42b20e800">

**5. How would you describe our customers in this segment based off their composition and ranking values? What sort of products or services should we show to these customers and what should we avoid?**

**Analysis:**

---
### **D. Index Analysis**

The `index_value` is a measure which can be used to reverse calculate the average composition for Fresh Segments’ clients.

Average composition can be calculated by dividing the `composition` column by the `index_value` column rounded to 2 decimal places.

**1. What is the top 10 interests by the average composition for each month?**

```sql
WITH cte AS 
	(SELECT *,
	ROUND((composition/index_value)::numeric , 2) AS avg_composition,
	RANK() OVER(PARTITION BY month_year 
				ORDER BY ROUND((composition/index_value)::numeric , 2) DESC) AS rnk
	FROM interest_removed)
SELECT month_year, interest_id, avg_composition, rnk
FROM cte
WHERE rnk <= 10;
```

**Explanation:**

- Using a **CTE** calculate the `avg_composition` based on the given formula.
- Use the **RANK** window function to order the `interest_id` based on their `avg_composition` per each `month_year`.
- Use the **CTE** to extract only those `interest_id` which `rnk >= 10`.

**Results and Analysis:**

**The results in the following screenshot are only of the first two months. The entire table has 142 rows.**

<img width="450" alt="image" src="https://github.com/user-attachments/assets/f4252f92-3242-42fe-94c0-23af5d3e4eee">

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
