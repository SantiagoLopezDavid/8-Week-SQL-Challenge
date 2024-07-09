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

**1.Table: Customer_orders**
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
