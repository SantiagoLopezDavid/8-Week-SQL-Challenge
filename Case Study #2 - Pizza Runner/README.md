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

```sql
SELECT customer_id, 
coalesce(sum( case
	when pizza_id = 1 then 1
	end),0) as count_meatlovers,
coalesce(sum( case
	when pizza_id = 2 then 1
	end),0) as count_vegetarian
FROM customer_orders_temp
group by customer_id
order by customer_id;
```

**Explanation:**
- Using a **CASE** statement set a condition for `pizza_id = 1` (Meatlovers), then set a value of 1 for that row.
- Using a **SUM** aggregate function will get the total of `pizza_id = 1`. And in case if the `customer_id` hasn't order a type of pizza, we can use the **COALESCE** function to set the value to 0 instead of it being NULL.
- To create a column for Vegetarian pizza, used the same code but change the condition to `pizza_id = 2`.
- Use a **GROUP BY** clause by `customer_id`.
  
**Results and Analysis:**
|customer_id|count_meatlovers|count_vegetarian|
|---|---|---|
|101|2|1|
|102|2|1|
|103|3|1|
|104|3|0|
|105|0|1|

- Customer with id 101, has ordered a total of 2 Meatlovers pizzas and 1 Vegetarian.
- Customer with id 102, has ordered a total of 2 Meatlovers pizzas and 1 Vegetarian.
- Customer with id 103, has ordered a total of 3 Meatlovers pizzas and 1 Vegetarian.
- Customer with id 104, has ordered a total of 3 Meatlovers pizzas and 0 Vegetarian.
- Customer with id 105, has ordered a total of 0 Meatlovers pizzas and 1 Vegetarian.
  
**6. What was the maximum number of pizzas delivered in a single order?**
```sql
SELECT order_id, count_pizzas
FROM 
	(SELECT order_id, COUNT(pizza_id) AS count_pizzas,
	RANK() OVER(ORDER BY COUNT(pizza_id) DESC) AS rnk
	FROM customer_orders_temp
	WHERE order_id IN 
		(SELECT order_id
		FROM runner_orders_temp
		WHERE cancellation = '')
	GROUP BY order_id
	ORDER BY order_id) AS x
WHERE rnk = 1;
```

**Explanation:**
- Create a subquery for those `order_id` which were completed. Meaning that the row `cancellation = ''`.
- This subquery will be part of another subequery named `x`.
- Using the first subquery, we will use it in a **WHERE** clause to filter those `order_id` that are part of completed orders.
- Also, use the **COUNT** function to get the total of `pizza_id` for each `order_id`.
- Next, use the **RANK()** window function. **ORDER BY** the `COUNT(pizza_id)` in a descending order, since we are interested in the highest number.
- Finally, in the main query, which will use the subquery `x`, we will select the `order_id` and `count_pizzas` that has a `rnk = 1`.

**Results and Analysis:**
|order_id|count_pizzas|
|---|---|
|4|3|

- Order_id 4 has the highest number of pizzas delivered.

**7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?**
```sql
SELECT customer_id,
SUM(CASE 
	WHEN exclusions = '' AND extras = '' THEN 1
	ELSE 0
	END) AS no_changes,
SUM(CASE
	WHEN exclusions != '' OR extras != '' THEN 1
	ELSE 0
	END) AS at_least_1_change
FROM customer_orders_temp
WHERE order_id IN 
		(SELECT order_id
		FROM runner_orders_temp
		WHERE cancellation = '')
GROUP BY customer_id
ORDER BY customer_id;
```

**Explanation:**
- Create a subquery for those `order_id` which were completed. Meaning that the row `cancellation = ''`.
- Using the first subquery, we will use it in a **WHERE** clause to filter those `order_id` that are part of completed orders.
- Use a **CASE** statement for when `exclusions` and `extras` are empty and another **CASE** statement for the opposite situation.
- Use a **GROUP BY** clause to filter by `customer_id`.
  
**Results and Analysis:**
|customer_id|no_changes|at_least_1_change|
|---|---|---|
|101|2|0|
|102|3|0|
|103|0|3|
|104|1|2|
|105|0|1|

- Customer 101 has been delivered 2 pizzas with no changes.
- Customer 102 has been delivered 3 pizzas with no changes.
- Customer 103 has been delivered 3 pizzas with at least 1 change.
- Customer 104 has been delivered 1 pizza with no changes and 2 pizzas with at least 1 change.
- Customer 105 has been delivered 1 pizza with at least 1 change.

**8. How many pizzas were delivered that had both exclusions and extras?**
```sql
WITH cte AS (
	SELECT order_id,
	SUM(CASE
		WHEN exclusions != '' and extras != '' THEN 1
		ELSE 0
		END) AS exclusions_and_extras
	FROM customer_orders_temp
	WHERE order_id IN 
			(SELECT order_id
			FROM runner_orders_temp
			WHERE cancellation = '')
	GROUP BY order_id)
SELECT DISTINCT order_id FROM cte
WHERE exclusions_and_extras >= 1;
```

**Explanation:**
- Inside a **CTE** create a subquery for those `order_id` which were completed. Meaning that the row `cancellation = ''`.
- Using the first subquery, we will use it in a **WHERE** clause to filter those `order_id` that are part of completed orders.
- Use a **CASE** statement for when `exclusions` and `extras` are not empty or equal to ''.
- From the **CTE** filter the table with those **DISTINCT** `order_id` that `exclusions_and_extras >= 1`. We have to use **DISTINCT** as a single order could have multiple pizzas. 

**Results and Analysis:**
|order_id|
|10|

- Only order_id 10 has a pizza that has both exclusions and extras.

**9. What was the total volume of pizzas ordered for each hour of the day?**
```sql
SELECT
EXTRACT(hour FROM order_time) AS hour_order,
COUNT(pizza_id) AS count_pizza
FROM customer_orders_temp
GROUP BY hour_order
ORDER BY hour_order;
```

**Explanation:**
- Use the **EXTRACT** function to extract the 'hour' field form `order_date`.
- Use the **COUNT** function to get the total of `pizza_id` for each `hour_order`.
- Finally **GROUP BY** the `hour_order`.

**Results and Analysis:**
|hour_order|count_pizza|
|---|---|
|11|1|
|13|3|
|18|3|
|19|1|
|21|3|
|23|3|

- The hours with the highest number of pizzas ordered are 13(1pm), 18(6:00 pm), 21(9:00 pm) and 23(11:00 pm) all with 3 pizzas in total.
- The hours with the lowest volume of pizzas ordered are 11:00 am and 19(7:00 pm), each with only 1 order.

**10. What was the volume of orders for each day of the week?**
```sql
SELECT
TO_CHAR(order_time, 'Day') AS day_of_week,
COUNT(order_id) AS count_order
FROM customer_orders_temp
GROUP BY day_of_week
ORDER BY count_order DESC;
```

**Explanation:**
- Use the **TO CHAR** function to format the day of the week from `order_date` as a textual representation.
- Use the **COUNT** function to get the total of `order_id` for each `day_of_week`.
- Finally **GROUP BY** the `day_of_week` and **ORDER BY** `count_order` in a descending order to know which days of the week have the highest volume of orders.

**Results and Analysis:**
|day_of_week|count_order|
|---|---|
|Saturday|5|
|Wednesday|5|
|Thursday|3|
|Friday|1|

- The days of the week with the highest volume of orders is Wednesday and Saturday with a count of 5 orders each.
- Thursday has a total of 3 orders.
- Friday is the day of the week with the lowest order count, only 1.

---
### Runner and Customer Experience

**1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)**
```sql
SELECT
TO_CHAR(registration_date, 'MON') AS month,
TO_CHAR(registration_date, 'W') AS month_week,
COUNT(*) AS count_runners
FROM runners
GROUP BY month, month_week
ORDER BY month, month_week;
```

**Explanation:**
- Using the **TO_CHAR** function, change the formatting of `registration_date` to get both the `month` and the `month_week`. Using the format 'W' the first week starts on the first day of the month.
- **COUNT** the number of runner per group of `month` and `month_week`.

**Results and Analysis:**
|month|month_week|count_runners|
|---|---|---|
|JAN|1|2|
|JAN|2|1|
|JAN|3|1|

- The only month in the data base for the moment is January.
- The first week of the month 2 runners signed up.
- The second and third of the month, only 1 runner signed up.

**2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?**

```sql
SELECT rt.runner_id,
to_char(AVG(rt.pickup_time - ct.order_time), 'MI:SS') AS avg_time
FROM runner_orders_temp AS rt
JOIN customer_orders_temp AS ct ON ct.order_id = rt.order_id
GROUP BY rt.runner_id
ORDER BY avg_time;
```
**Explanation:**

- Calculate the average time of the substract between the `rt.pickup_time`and `ct.order_time`.
- Using the **TO_CHAR** function, change the formatting of `avg_time` to get a precision of two seconds.
- Use the **JOIN** clause to get both timestamps needed for this question.
  
**Results and Analysis:**
|runner_id|avg_time|
|---|---|
|3|10:28|
|1|15:40|
|2|23:43|

- The runner 3 is the fastest out of all runners with an average time of 10 minutes and 28 seconds.
- The runner 1 has an average time of 15 minutes and 40 seconds.
- The runner 2 is the slowest out of all runners with an average time of 23 minutes and 43 seconds.

**3. Is there any relationship between the number of pizzas and how long the order takes to prepare?**
```sql
WITH cte AS 
	(SELECT rt.order_id, COUNT(ct.pizza_id) AS pizza_count,
	rt.pickup_time - ct.order_time AS diff_time
	FROM runner_orders_temp AS rt
	JOIN customer_orders_temp AS ct ON ct.order_id = rt.order_id
	GROUP BY rt.order_id, diff_time
	HAVING rt.pickup_time - ct.order_time is not null
	ORDER BY pizza_count)
SELECT pizza_count, to_char(AVG(diff_time),'MI:SS') AS avg_prep_time
FROM cte
GROUP BY pizza_count;
```

**Explanation:**
- Create a `cte` and calculate the `diff_time` between the `rt.pickup_time` and `ct.order_time`.
- **COUNT** the number of pizzas ordered in each `rt.order_id`
- This `diff_time` is going to be consistent in each `order_id` since no matter the number of pizzas ordered, they are all going to be picked up at the same time.
- Use the **GROUP BY** clause with `rt.order_id` and `diff_time`.
- Using this `cte` calculate the **AVG** time per group of `pizza_count`.
- Using the **TO_CHAR** function, change the formatting of `avg_prep_time` to get a precision of two seconds.

**Results and Analysis:**
|avg_prep_time|avg_prep_time|
|---|---|
|1|12:21|
|2|18:22|
|3|29:17|

- In average, preparing 1 pizza takes aroung 12 minutes and 21 seconds.
- Preparing 2 pizzas takes around 18 minutes and 22 seconds. This means 9 minutes and 11 seconds per pizza.
- Preparing 3 pizzas takes around 29 minutes and 17 seconds. Meaning 9 minutes and 45 seconds per pizza.

- Finally, there is a relationship between the number of pizzas and the time to prepare them. It takes aroung 9 to 10 minutes extra for each pizza added to the order. We could say that making 4 pizzas would take ~40 minutes to prepare.

**4. What was the average distance travelled for each customer?**
```sql
SELECT customer_id, round(AVG(distance)::numeric,2) AS avg_distance
FROM runner_orders_temp AS rt
LEFT JOIN customer_orders_temp AS ct ON ct.order_id = rt.order_id
GROUP BY customer_id
HAVING AVG(distance) IS NOT NULL
ORDER BY customer_id;
```

**Explanation:**

- Using the aggregate function **AVG** calculate the average of the distance for each group of `customer_id`. **ROUND** the result to 2 decimals.
- **JOIN** the tables `runner_orders_temp` and `customer_orders_temp`.
- Filter the resulting table to get only those results of `AVG(distance)` that are not **NULL**.

**Results and Analysis:**

|customer_id|avg_distance|
|---|---|
|101|20.00|
|102|16.73|
|103|23.40|
|104|10.00|
|105|25.00|

- The furthest that runners have to travel is in average 25km to get to customer 105.
- The customer closest for runners is 104, with an average distance of 10km.

**5. What was the difference between the longest and shortest delivery times for all orders?**

```sql
SELECT MAX(duration) AS longest_delivery, 
MIN(duration) AS shortest_delivery ,
MAX(duration) - MIN(duration) AS diff_duration
FROM runner_orders_temp
WHERE duration IS NOT NULL;
```

**Explanation:**

- Use the **MAX** and **MIN** functions to get the longest and shortest delivery duration times.
- Calculate the difference between the `longest_delivery` and `shortest_delivery`.
- Filter the `duration` to those values that are not **NULL**.

**Results and Analysis:**
|longest_delivery|shortest_delivery|diff_duration|
|---|---|---|
|40|10|30|

- The longest delivery made took 40 minutes.
- The shortest delivery made took 10 minutes.
- The difference between the longest and shortest delivery is 30 minutes.

**6. What was the average speed for each runner for each delivery and do you notice any trend for these values?**

- **Speed for each runner for each delivery**
```sql
SELECT runner_id, distance, duration,
ROUND(((distance/duration)*60)::numeric,2) AS speed_km_hr
FROM runner_orders_temp 
WHERE distance IS NOT NULL
ORDER BY runner_id, distance;
```

- **Average speed for each runner**
```sql
SELECT runner_id, ROUND(AVG(speed_km_hr),2) AS avg_speed 
FROM
	(SELECT runner_id, distance, duration,
	ROUND(((distance/duration)*60)::numeric,2) AS speed_km_hr
	FROM runner_orders_temp 
	WHERE distance IS NOT NULL
	ORDER BY runner_id, distance) AS x
GROUP BY runner_id;
```
**Explanation:**

- Calculate the speed(km/hr) for each delivery of the runners.
- **ROUND** the results to only 2 decimals for easier analysis.
- **ORDER BY** the `runner_id` and the `distance`.
- To calculate the `avg_speed` per `runner_id`. Use the past code as a subquery and use the **AVG** aggregate function on `speed_km_hr`.
- **GROUP BY** `runner_id`.

**Results and Analysis:**

- **Speed for each runner for each delivery**

|runner_id|distance|duration|speed_km_hr|
|---|---|---|---|
|1|10|10|60.00|
|1|13.4|20|40.20|
|1|20|32|37.50|
|1|20|27|44.44|
|2|23.4|15|93.60|
|2|23.4|40|35.10|
|2|25|25|60.00|
|3|10|15|40.00|

- **Average speed for each runner**

|runner_id|avg_speed|
|---|---|
|1|45.54|
|2|62.90|
|3|40.00|

- Runner 2 is the fastest of the runners. But this does not mean runner 2 is the most efficient, he/she might be speeding in some of his deliveries. This needs to be address before an accident happens.
- Runner 2 has the highest and slowest speed for deliveries. Both made in the same delivery distance. Consistency is important for a delivery service and runner 2 needs improvement. 
- Runner 3 is the slowest of all runners, but this conclusion is based only on one delivery. We might have to wait to see how he/she performs in the future.
- Runner 1 is consistent in his delivery speed, being 75% of the time between 37 and 44 km per hour.

**7. What is the successful delivery percentage for each runner?**

```sql
WITH cte AS 
	(SELECT runner_id,
	SUM(CASE WHEN cancellation = '' THEN 1 ELSE 0 END ) AS successful_deliveries,
	COUNT(order_id) AS total_orders
	FROM runner_orders_temp
	GROUP BY runner_id
	ORDER BY runner_id)
SELECT runner_id,
ROUND((successful_deliveries::numeric/total_orders::numeric),2)*100 AS success_percentage
FROM cte;
```

**Explanation:**
- Create a **CTE** in which use the aggregate function **SUM** to add 1 each time a runner successfully deliverd an order. This means that the `cancellation` column is ''.
- **COUNT** the total number of `order_id` for each `runner_id`.
- **GROUP BY** the `runner_id`
- Use the **CTE** to calculate the `success_percentage`. **CAST** both `successful_deliveries` and `total_orders` to **NUMERIC** type in order to perform the division.

**Results and Analysis:**
|runner_id|sucess_percentage|
|---|---|
|1|100.00|
|2|75.00|
|3|50.00|

- Runner 1 has a 100% sucessful delivery percentage.
- Runner 2 has a 75% sucessful delivery percentage.
- Runner 3 has a 50% sucessful delivery percentage.
---
### Ingredient Optimisation

**1. What are the standard ingredients for each pizza?**

```sql
WITH cte AS 
	(SELECT
	pizza_id,
	REGEXP_SPLIT_TO_TABLE(toppings, '[,\s]+')::INTEGER AS topping_id
	FROM pizza_recipes),
	cte2 AS 
	(SELECT cte.pizza_id, pn.pizza_name, cte.topping_id, topping_name
	 FROM cte
	 JOIN pizza_toppings pt ON cte.topping_id = pt.topping_id
	 JOIN pizza_names pn ON pn.pizza_id = cte.pizza_id)
SELECT cte2.pizza_id, cte2.pizza_name,
STRING_AGG(cte2.topping_name,', ') AS toppings
FROM cte2
GROUP BY cte2.pizza_id, cte2.pizza_name
ORDER BY cte2.pizza_id;
```
**Explanation:**
- Create 2 **CTE's**.
- In the first **CTE** split the list of toppings from `pizza_recipes` into a table using **REGEXP_SPLIT_TO_TABLE**.
- In the second **CTE** **JOIN** the tables `pizza_toppings` and `pizza_names` with the `cte`. This to get the `pizza_name` and `topping_name`.
- Finally, use the function **STRING_AGG** to concatenate all the `topping_name` for each `pizza_name`.

**Results and Analysis:**

|pizza_id|pizza_name|toppings|
|---|---|---|
|1|Meatlovers|Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami|
|2|Vegetarian|Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce|

**2. What was the most commonly added extra?**

```sql
WITH cte AS 
	(SELECT
	order_id,
	REGEXP_SPLIT_TO_TABLE(extras, '[,\s]+')::INTEGER AS extras_id
	FROM customer_orders_temp
	WHERE extras <> '')
SELECT topping_name,
COUNT(extras_id) AS topping_count
FROM cte
JOIN pizza_toppings pt ON pt.topping_id = cte.extras_id
GROUP BY topping_name
ORDER BY topping_count DESC;
```
**Explanation:**
- Create a **CTE** and split the `extras` column into a table of individual extras added.
- Using the **CTE**, **COUNT** the `extras_id` for each `topping_name`.
- **ORDER BY** the `topping_count` in descending order.

**Results and Analysis:**

|topping_name|topping_count|
|---|---|
|Bacon|4|
|Chicken|1|
|Cheese|1|

- Bacon is the most commonly added extra with 4 requests.

**3. What was the most common exclusion?**

```sql
WITH cte AS 
	(SELECT
	order_id,
	REGEXP_SPLIT_TO_TABLE(exclusions, '[,\s]+')::INTEGER AS exclusions_id
	FROM customer_orders_temp
	WHERE exclusions <> '')
SELECT topping_name,
COUNT(exclusions_id) AS topping_count
FROM cte
JOIN pizza_toppings pt ON pt.topping_id = cte.exclusions_id
GROUP BY topping_name
ORDER BY topping_count DESC;
```
**Explanation:**
- Use the same code for the last question but use the `exclusions` column instead.

**Results and Analysis:**
|topping_name|topping_count|
|---|---|
|Cheese|4|
|Mushrooms|1|
|BBQ Sauce|1|

- The most common exclusion is Cheese with 4 requests.

**4. Generate an order item for each record in the customers_orders table in the format of one of the following:**
- Meat Lovers
- Meat Lovers - Exclude Beef
- Meat Lovers - Extra Bacon
- Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers

**5. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients**

- For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"

**6. What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?**

---
### Pricing and Ratings

**1. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?**

```sql
SELECT pizza_name,
CASE
	WHEN pizza_name = 'Meatlovers' THEN COUNT(sub_x.pizza_id)*12
	WHEN pizza_name = 'Vegetarian' THEN COUNT(sub_x.pizza_id)*10
	END AS total_money
FROM 
	(SELECT rt.order_id, ct.pizza_id, pizza_name
	FROM runner_orders_temp rt
	JOIN customer_orders_temp ct ON ct.order_id = rt.order_id
	JOIN pizza_names pn ON pn.pizza_id = ct.pizza_id
	WHERE cancellation = ''
	ORDER BY rt.order_id) AS sub_x
GROUP BY pizza_name;
```
**Explanation:**

- We have to filter the `order_id` by only those that were not cancelled. This is done in `sub_x`.
- **JOIN** the tables `customer_orders_temp`, `pizza_names` and `runner_orders_temp` in order to get the right data.
- In the main query:
  
	- Use a **CASE** statement to multiply the **COUNT** of `pizza_id` for the corresponding price.
	- Use the **GROUP BY** clause with the `pizza_name` column to filter the results per pizza type.

**Results and Analysis:**

|pizza_name|total_money|
|---|---|
|Meatlovers|108|
|Vegetarian|30|

- Meatlovers pizza has made Pizza Runner a total of $108.
- Vegatarian pizza has made Pizza Runner a total of $30.
- In total, Pizza Runner has made $138 so far from their pizzas.

**2. What if there was an additional $1 charge for any pizza extras?**
- Add cheese is $1 extra

```sql
WITH cte_extras AS (
	SELECT pizza_name,
	SUM(CASE 
		WHEN topping_name ='Cheese' THEN 2
		WHEN topping_name IS NULL THEN 0
		ELSE 1
		END ) AS extra_charge
	FROM
		(SELECT order_id,pizza_id,
		CASE WHEN extras_id='' THEN 0 ELSE extras_id::integer END AS extras_id
		FROM
			(SELECT
			order_id, pizza_id,
			REGEXP_SPLIT_TO_TABLE(extras, '[,\s]+') AS extras_id
			FROM customer_orders_temp) AS sub_x
		WHERE order_id IN 
			(SELECT order_id FROM runner_orders_temp WHERE cancellation = '')) AS sub_y
	LEFT JOIN pizza_toppings pt ON pt.topping_id = sub_y.extras_id
	LEFT JOIN pizza_names pn on pn.pizza_id = sub_y.pizza_id
	GROUP BY pizza_name
),
cte_pizzas as (
	SELECT pizza_name,
	CASE
		WHEN pizza_name = 'Meatlovers' THEN COUNT(sub_x.pizza_id)*12
		WHEN pizza_name = 'Vegetarian' THEN COUNT(sub_x.pizza_id)*10
		END AS total_money
	FROM 
		(SELECT rt.order_id, ct.pizza_id, pizza_name
		FROM runner_orders_temp rt
		JOIN customer_orders_temp ct ON ct.order_id = rt.order_id
		JOIN pizza_names pn ON pn.pizza_id = ct.pizza_id
		WHERE cancellation = ''
		ORDER BY rt.order_id) AS sub_x
	GROUP BY pizza_name
)
SELECT cp.pizza_name,
SUM(total_money + extra_charge)
FROM cte_pizzas cp
JOIN cte_extras ce ON ce.pizza_name = cp.pizza_name
GROUP BY cp.pizza_name
```
|pizza_name|total|
|---|---|
|Meatlovers|112|
|Vegetarian|31|

- Adding the extra charges, Meatlovers pizza has made $112 and Vegetarian pizza has made $31.

**3. The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.**

**4. Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?**
- customer_id
- order_id
- runner_id
- rating
- order_time
- pickup_time
- Time between order and pickup
- Delivery duration
- Average speed
-Total number of pizzas

**5. If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?**

---
