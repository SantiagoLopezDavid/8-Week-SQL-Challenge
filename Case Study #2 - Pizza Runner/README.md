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

---
### Runner and Customer Experience

---
### Ingredient Optimisation

---
### Pricing and Ratings

---
### Bonus DML Challenges (DML = Data Manipulation Language)

---
