# 🍕 Case Study #2 Pizza Runner

<img width="500" alt="image" src="https://github.com/SantiagoLopezDavid/8-Week-SQL-Challenge/assets/170588432/69c54f8b-a0aa-4aca-9890-81b3595e07b4">

## Table of contents
1. Problem Statement.
2. Entity Relationship Diagram.
3. Solution.
   - Data Cleaning and Transformation
   - Pizza Metrics
   - Runner and Customer Experience
   - Ingredient Optimisation
   - Pricing and Ratings
   - Bonus DML Challenges (DML = Data Manipulation Language)
  
All the information regarding the case study has been sourced from [here](https://8weeksqlchallenge.com/case-study-2/)
  
## Problem Statement


## Entity Relationship Diagram

<img width="700" alt="image" src="https://github.com/SantiagoLopezDavid/8-Week-SQL-Challenge/assets/170588432/8a566a31-b587-44db-b966-1764b8f37929">

## Solution

### Data Cleaning and Transformation

**1. Table: customer_orders**
- By looking at the `customer_orders` table, we can notice that the columns `exclusions` and `extras` have missing/blank spaces ' ' and **NULL** values.
<img width="700" alt="image" src="https://github.com/SantiagoLopezDavid/8-Week-SQL-Challenge/assets/170588432/8327930f-eb37-4fd1-b3bf-56bb00ec6524">

- To clean the table we need to:
  - Create the temporaty table `customer_orders_temp` from `customer_orders` original table.
  - Replace **NULL** values and 'null' strings with ' ' for both columns `exclusions` and `extras`.
 
```sql
CREATE TEMP TABLE customer_orders_temp AS
	SELECT order_id, customer_id, pizza_id,
	CASE
		WHEN exclusions IS NULL OR exclusions LIKE 'null' THEN ''
		ELSE exclusions
		END AS exclusions,
	CASE
		WHEN extras IS NULL OR extras LIKE 'null' THEN ''
		ELSE extras
		END AS extras,
	order_time
FROM pizza_runner.customer_orders;
```
- This is how the clean `customers_orders_temp` table looks like and we will use this table to run all the following queries.

<img width="700" alt="image" src="https://github.com/SantiagoLopezDavid/8-Week-SQL-Challenge/assets/170588432/3763a2c7-21e7-4d8f-acd9-3321fe0c89fa">


**2. Table: runner_orders**
- By looking at the `runner_orders` table, we can notice that the columns `pickup_time`, `distance`, `duration` and `cancellation` have missing/blank spaces ' ', **NULL** values, bad formatting and wrong data type.
  
<img width="700" alt="image" src="https://github.com/SantiagoLopezDavid/8-Week-SQL-Challenge/assets/170588432/9b84f3db-5d0f-44c8-951a-9f82dfa89dd1">

- To clean the table we need to:
  - Create the temporaty table `runner_orders_temp` from `runner_orders` original table.
  - Replace **NULL** values and 'null' strings with ' ' for the column `cancellation`.
  - Replace 'null' strings with **NULL** values for the columns `pickup_time`, `distance` and `duration`.
  - In `distance` column, remove "km".
  - In `duration` column, remove "minutes", "minute" and "mins".
 
```sql
CREATE TEMP TABLE runner_orders_temp AS
	SELECT order_id, runner_id,
	CASE
		WHEN pickup_time LIKE 'null' THEN NULL
		ELSE pickup_time
		END AS pickup_time,
	CASE
		WHEN distance LIKE '%km' THEN TRIM(TRAILING 'km' FROM distance)
		WHEN distance LIKE 'null' THEN NULL
		ELSE distance
		END AS distance,
	CASE
		WHEN duration LIKE '%minutes' THEN TRIM(TRAILING 'minutes' FROM duration)
		WHEN duration LIKE '%mins' THEN TRIM(TRAILING 'minutes' FROM duration)
		WHEN duration LIKE '%minute' THEN TRIM(TRAILING 'minutes' FROM duration)
		WHEN duration LIKE 'null' THEN NULL
		ELSE duration
		END AS duration,
	CASE 
		WHEN cancellation IS NULL OR cancellation LIKE 'null' THEN ''
		ELSE cancellation
		END AS cancellation
FROM pizza_runner.runner_orders;
```
- This is how the clean `runner_orders_temp` table looks like and we will use this table to run all the following queries.
  
<img width="700" alt="image" src="https://github.com/SantiagoLopezDavid/8-Week-SQL-Challenge/assets/170588432/78745985-2e88-459e-905f-0cca9974db45">

- Then, we change the data type of the columns `pickup_time`, `distance` and `duration`.
- **PostgreSQL** doesn’t allow the implicit type casting from **VARCHAR/TEXT** to **INTEGER**, **FLOAT** or **TIMESTAMP** data type. In PostgreSQL, the **USING** clause is used to cast the **VARCHAR/TEXT** data type these other data types.

```sql
ALTER TABLE runner_orders_temp
ALTER COLUMN pickup_time TYPE TIMESTAMP USING pickup_time::TIMESTAMP WITHOUT time zone,
ALTER COLUMN distance TYPE FLOAT USING distance::FLOAT,
ALTER COLUMN duration TYPE INT USING duration::INTEGER;
```
<img width="700" alt="image" src="https://github.com/SantiagoLopezDavid/8-Week-SQL-Challenge/assets/170588432/729baf61-fc8a-4f62-8369-d540910d2a2d">

---
### Pizza Metrics

**1. How many pizzas were ordered?**

```sql
SELECT COUNT(pizza_id) AS pizza_count
FROM customer_orders_temp;
```

**Explanation:**
- Use the **COUNT** aggregate function to get the total number of `pizza_id` ordered.
  
**Results and Analysis:**
|pizza_count|
|---|
|14|

- There is a total of 14 pizzas ordered.
  

**2. How many unique customer orders were made?**
```sql
SELECT COUNT(DISTINCT order_id) AS unique_order 
FROM customer_orders_temp;
```

**Explanation:**
- Use the **COUNT** aggregate function to get the total number of **DISTINCT** `order_id` ordered.

**Results and Analysis:**
|unique_order|
|---|
|10|

- There is a total of 10 unique `order_id`.

**3. How many successful orders were delivered by each runner?**

```sql
SELECT runner_id, COUNT(order_id) AS count_orders
FROM runner_orders_temp
WHERE cancellation = ''
GROUP BY runner_id;
```

**Explanation:**
- Use the **COUNT** aggregate function to get the total number of `order_id` per group of `runner_id`.
- Filter the resulting table using the **WHERE** clause by only those rows with a `cancellation = ''`. If the `cancellation` row has any value, it means that the order was cancelled.

**Results and Analysis:**
|runner_id|count_orders|
|---|---|
|1|4|
|2|3|
|3|1|

- Runner 1 has a total of 4 sucessful orders delivered.
- Runner 2 has a total of 3 sucessful orders delivered.
- Runner 3 has a total of 1 sucessful orders delivered.

**4. How many of each type of pizza was delivered?**

```sql
SELECT ct.pizza_id, pizza_name, COUNT(*) AS pizza_id_count
FROM runner_orders_temp rt
JOIN customer_orders_temp ct ON ct.order_id = rt.order_id
JOIN pizza_names pn ON pn.pizza_id = ct.pizza_id
WHERE cancellation = ''
GROUP BY ct.pizza_id, pizza_name;
```

**Explanation:**

- Use the **COUNT** aggregate function to get the total number of rows per group of `pizza_id` and `pizza_name`.
- **JOIN** the tables `runner_orders_temp` , `customer_orders_temp` and `pizza_names` in order to get the right columns.
- Filter the resulting table using the **WHERE** clause by only those rows with a `cancellation = ''.
- Use the **GROUP BY** clause with `pizza_id` and `pizza_name`.

**Results and Analysis:**

|pizza_id|pizza_name|pizza_id_count|
|---|---|---|
|1|Meatlovers|9|
|2|Vegetarian|3|

- Meatlovers pizza with an id of 1 has a tota of 9 orders delivered.
- Vegetarian pizza with an id of 2 has a tota of 3 orders delivered.

**5. How many Vegetarian and Meatlovers were ordered by each customer?**

**Explanation:**
**Results and Analysis:**

**6. What was the maximum number of pizzas delivered in a single order?**

**Explanation:**
**Results and Analysis:**

**7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?**

**Explanation:**
**Results and Analysis:**

**8. How many pizzas were delivered that had both exclusions and extras?**

**Explanation:**
**Results and Analysis:**

**9. What was the total volume of pizzas ordered for each hour of the day?**

**Explanation:**
**Results and Analysis:**

**10. What was the volume of orders for each day of the week?**

**Explanation:**
**Results and Analysis:**

---
### Runner and Customer Experience

---
### Ingredient Optimisation

---
### Pricing and Ratings

---
### Bonus DML Challenges (DML = Data Manipulation Language)

---
