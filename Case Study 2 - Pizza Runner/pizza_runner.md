# Case Study 2 - Pizza Runner
All information for this case study can be found here - [Pizza Runner Case Study](https://8weeksqlchallenge.com/case-study-2/)


In this case study, I'll be leveraging a local PostgreSQL database to execute queries and derive insights from the following datasets:
![alt text](image.png)

This case study has several questions which will be divided into the following sections:

1. **Pizza Metrics**
2. **Runner and Customer Experience**
3. **Ingredient Optimization**
4. **Pricing and Ratings**
5. **Bonus Data Manipulation Language Question**

However, before we dive into the data, we have to clean up the `customer_orders` and `runner_orders` tables.

---

### Data Cleaning: Customer Orders Table

There are two issues with the `customer_order` table

1. `exclusions` column shows missing values as an empty string or the string 'null'.
2. `extras` column also shows missing values as an empty string or the string 'null'.

| order\_id | customer\_id | pizza\_id | exclusions | extras | order\_time |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | 101 | 1 |  |  | 2020-01-01 18:05:02.000000 |
| 2 | 101 | 1 |  |  | 2020-01-01 19:00:52.000000 |
| 3 | 102 | 1 |  |  | 2020-01-02 23:51:23.000000 |
| 3 | 102 | 2 |  | null | 2020-01-02 23:51:23.000000 |
| 4 | 103 | 1 | 4 |  | 2020-01-04 13:23:46.000000 |
| 4 | 103 | 1 | 4 |  | 2020-01-04 13:23:46.000000 |
| 4 | 103 | 2 | 4 |  | 2020-01-04 13:23:46.000000 |
| 5 | 104 | 1 | null | 1 | 2020-01-08 21:00:29.000000 |
| 6 | 101 | 2 | null | null | 2020-01-08 21:03:13.000000 |
| 7 | 105 | 2 | null | 1 | 2020-01-08 21:20:29.000000 |
| 8 | 102 | 1 | null | null | 2020-01-09 23:54:33.000000 |
| 9 | 103 | 1 | 4 | 1, 5 | 2020-01-10 11:22:59.000000 |
| 10 | 104 | 1 | null | null | 2020-01-11 18:34:49.000000 |
| 10 | 104 | 1 | 2, 6 | 1, 4 | 2020-01-11 18:34:49.000000 |


To clean the data, we can simply use a CASE statement to find and replace empty strings and 'null' strings with NULL. 

- Create view of cleaned table

> ```sql
> CREATE VIEW customer_order AS
> SELECT order_id,
>        customer_id,
>        pizza_id,
>        CASE
>            WHEN exclusions = 'null' OR exclusions = '' THEN NULL
>            ELSE exclusions
>        END AS exclusions,
>        CASE
>            WHEN extras = 'null' OR extras = '' OR extras IS NULL THEN NULL
>            ELSE extras
>        END AS extras,
>         order_time
> FROM customer_orders;
> 
> ```
| order\_id | customer\_id | pizza\_id | exclusions | extras | order\_time |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | 101 | 1 | null | null | 2020-01-01 18:05:02.000000 |
| 2 | 101 | 1 | null | null | 2020-01-01 19:00:52.000000 |
| 3 | 102 | 1 | null | null | 2020-01-02 23:51:23.000000 |
| 3 | 102 | 2 | null | null | 2020-01-02 23:51:23.000000 |
| 4 | 103 | 1 | 4 | null | 2020-01-04 13:23:46.000000 |
| 4 | 103 | 1 | 4 | null | 2020-01-04 13:23:46.000000 |
| 4 | 103 | 2 | 4 | null | 2020-01-04 13:23:46.000000 |
| 5 | 104 | 1 | null | 1 | 2020-01-08 21:00:29.000000 |
| 6 | 101 | 2 | null | null | 2020-01-08 21:03:13.000000 |
| 7 | 105 | 2 | null | 1 | 2020-01-08 21:20:29.000000 |
| 8 | 102 | 1 | null | null | 2020-01-09 23:54:33.000000 |
| 9 | 103 | 1 | 4 | 1, 5 | 2020-01-10 11:22:59.000000 |
| 10 | 104 | 1 | null | null | 2020-01-11 18:34:49.000000 |
| 10 | 104 | 1 | 2, 6 | 1, 4 | 2020-01-11 18:34:49.000000 |

---

### Data Cleaning: Runner Orders Table

The `runner_orders` table has the following issues:

- `pickup_time` column is a string data type.
- `distance` column is a string data type, some values are 'null' strings, some values are labeled with units and some without units.
- `duration` column is a string data type, some values are 'null' strings, some values are labeled with units and some without units.
- `cancellation` column contains empty strings and 'null' strings.

| order\_id | runner\_id | pickup\_time | distance | duration | cancellation |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | 1 | 2020-01-01 18:15:34 | 20km | 32 minutes |  |
| 2 | 1 | 2020-01-01 19:10:54 | 20km | 27 minutes |  |
| 3 | 1 | 2020-01-03 00:12:37 | 13.4km | 20 mins | null |
| 4 | 2 | 2020-01-04 13:53:03 | 23.4 | 40 | null |
| 5 | 3 | 2020-01-08 21:10:57 | 10 | 15 | null |
| 6 | 3 | null | null | null | Restaurant Cancellation |
| 7 | 2 | 2020-01-08 21:30:45 | 25km | 25mins | null |
| 8 | 2 | 2020-01-10 00:15:02 | 23.4 km | 15 minute | null |
| 9 | 2 | null | null | null | Customer Cancellation |
| 10 | 1 | 2020-01-11 18:50:20 | 10km | 10minutes | null |

These are the steps to clean the table:

- Change `pickup_time` column to timestamp type.
- In `distance` column, replace 'null' strings with NULL and replace unit labels with blank string.
- In `duration` column, replace 'null' strings with NULL and replace unit labels with blank string.
- In `cancellation` column, replace 'null' strings and blank strings with NULL
  
> ```sql
> CREATE VIEW runner_order AS
> SELECT order_id,
>        runner_id,
>        CASE
>            WHEN pickup_time LIKE 'null' THEN NULL
>            ELSE CAST(pickup_time AS TIMESTAMP WITHOUT TIME ZONE)
>        END AS pickup_time,
>        CASE
>            WHEN distance LIKE 'null' THEN NULL
>            WHEN distance LIKE '%km' THEN CAST(REPLACE(distance, 'km', '') AS NUMERIC)
>            ELSE CAST(distance AS NUMERIC)
>        END AS distance,
>        CASE
>            WHEN duration LIKE 'null' THEN NULL
>            WHEN duration LIKE '%minutes' THEN CAST(REPLACE(duration, 'minutes', '') AS NUMERIC)
>            WHEN duration LIKE '%mins' THEN CAST(REPLACE(duration, 'mins', '') AS NUMERIC)
>            WHEN duration LIKE '%minute' THEN CAST(REPLACE(duration, 'minute', '') AS NUMERIC)
>            ELSE CAST(duration AS NUMERIC)
>        END AS duration,
>        CASE
>            WHEN cancellation = 'null' OR cancellation = '' OR cancellation IS NULL THEN NULL
>            ELSE cancellation
>        END AS cancellation
> FROM runner_orders;
> 
> ```


| order\_id | runner\_id | pickup\_time | distance | duration | cancellation |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | 1 | 2020-01-01 18:15:34.000000 | 20 | 32 | null |
| 2 | 1 | 2020-01-01 19:10:54.000000 | 20 | 27 | null |
| 3 | 1 | 2020-01-03 00:12:37.000000 | 13.4 | 20 | null |
| 4 | 2 | 2020-01-04 13:53:03.000000 | 23.4 | 40 | null |
| 5 | 3 | 2020-01-08 21:10:57.000000 | 10 | 15 | null |
| 6 | 3 | null | null | null | Restaurant Cancellation |
| 7 | 2 | 2020-01-08 21:30:45.000000 | 25 | 25 | null |
| 8 | 2 | 2020-01-10 00:15:02.000000 | 23.4 | 15 | null |
| 9 | 2 | null | null | null | Customer Cancellation |
| 10 | 1 | 2020-01-11 18:50:20.000000 | 10 | 10 | null |

---

### A. Pizza Metrics

1. How many pizzas were ordered?
2. How many unique customer orders were made?
3. How many successful orders were delivered by each runner?
4. How many of each type of pizza was delivered?
5. How many Vegetarian and Meatlovers were ordered by each customer?
6. What was the maximum number of pizzas delivered in a single order?
7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
8. How many pizzas were delivered that had both exclusions and extras?
9. What was the total volume of pizzas ordered for each hour of the day?
10. What was the volume of orders for each day of the week?

### B. Runner and Customer Experience

1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)
2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
3. Is there any relationship between the number of pizzas and how long the order takes to prepare?
4. What was the average distance travelled for each customer?
5. What was the difference between the longest and shortest delivery times for all orders?
6. What was the average speed for each runner for each delivery and do you notice any trend for these values?
7. What is the successful delivery percentage for each runner?

### C. Ingredient Optimisation

1. What are the standard ingredients for each pizza?
2. What was the most commonly added extra?
3. What was the most common exclusion?
4. Generate an order item for each record in the customers_orders table in the format of one of the following:
    - `Meat Lovers`
    - `Meat Lovers - Exclude Beef`
    - `Meat Lovers - Extra Bacon`
    - `Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers`
5. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients
    - For example: `"Meat Lovers: 2xBacon, Beef, ... , Salami"`
6. What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?

### D. Pricing and Ratings

1. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?
2. What if there was an additional $1 charge for any pizza extras?
    - Add cheese is $1 extra
3. The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.
4. Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?
    - customer_id
    - order_id
    - runner_id
    - rating
    - order_time
    - pickup_time
    - Time between order and pickup
    - Delivery duration
    - Average speed
    - Total number of pizzas
5. If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?

### E. Bonus Questions

1. If Danny wants to expand his range of pizzas - how would this impact the existing data design? Write an INSERT statement to demonstrate what would happen if a new Supreme pizza with all the toppings was added to the Pizza Runner menu?

