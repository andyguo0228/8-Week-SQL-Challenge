
# Case Study 1 - Danny's Dinner

[https://8weeksqlchallenge.com/case-study-1/](https://8weeksqlchallenge.com/case-study-1/)

## Problem Statement

Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.

He plans on using these insights to help him decide whether he should expand the existing customer loyalty program - additionally he needs help to generate some basic datasets so his team can easily inspect the data without needing to use SQL.

Danny has provided you with a sample of his overall customer data due to privacy issues - but he hopes that these examples are enough for you to write fully functioning SQL queries to help him answer his questions!

Danny has shared with you 3 key datasets for this case study:

`sales`
`menu`
`members`

<iframe width="560" height="315" src='https://dbdiagram.io/e/66537a13f84ecd1d222d7f71/66537a21f84ecd1d222d7ffc'> </iframe>

## Case Study Questions

Each of the following case study questions can be answered using a single SQL statement:

1. What is the total amount each customer spent at the restaurant?
2. How many days has each customer visited the restaurant?
3. What was the first item from the menu purchased by each customer?
4. What is the most purchased item on the menu and how many times was it purchased by all customers?
5. Which item was the most popular for each customer?
6. Which item was purchased first by the customer after they became a member?
7. Which item was purchased just before the customer became a member?
8. What is the total items and amount spent for each member before they became a member?
9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

Bonus Questions Join All The Things The following questions are related creating basic data tables that Danny and his team can use to quickly derive insights without needing to join the underlying tables using SQL.

Recreate the following table output using the available data:

| **customer\_id** | **order\_date** | **product\_name** | **price** | **member** |
| --- | --- | --- | --- | --- |
| A | 2021-01-01 | curry | 15 | N |
| A | 2021-01-01 | sushi | 10 | N |
| A | 2021-01-07 | curry | 15 | Y |
| A | 2021-01-10 | ramen | 12 | Y |
| A | 2021-01-11 | ramen | 12 | Y |
| A | 2021-01-11 | ramen | 12 | Y |
| B | 2021-01-01 | curry | 15 | N |
| B | 2021-01-02 | curry | 15 | N |
| B | 2021-01-04 | sushi | 10 | N |
| B | 2021-01-11 | sushi | 10 | Y |
| B | 2021-01-16 | ramen | 12 | Y |
| B | 2021-02-01 | ramen | 12 | Y |
| C | 2021-01-01 | ramen | 12 | N |
| C | 2021-01-01 | ramen | 12 | N |
| C | 2021-01-07 | ramen | 12 | N |

Rank All The Things Danny also requires further information about the ranking of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ranking values for the records when customers are not yet part of the loyalty program.

| **customer\_id** | **order\_date** | **product\_name** | **price** | **member** | **ranking** |
| --- | --- | --- | --- | --- | --- |
| A | 2021-01-01 | curry | 15 | N | null |
| A | 2021-01-01 | sushi | 10 | N | null |
| A | 2021-01-07 | curry | 15 | Y | 1 |
| A | 2021-01-10 | ramen | 12 | Y | 2 |
| A | 2021-01-11 | ramen | 12 | Y | 3 |
| A | 2021-01-11 | ramen | 12 | Y | 3 |
| B | 2021-01-01 | curry | 15 | N | null |
| B | 2021-01-02 | curry | 15 | N | null |
| B | 2021-01-04 | sushi | 10 | N | null |
| B | 2021-01-11 | sushi | 10 | Y | 1 |
| B | 2021-01-16 | ramen | 12 | Y | 2 |
| B | 2021-02-01 | ramen | 12 | Y | 3 |
| C | 2021-01-01 | ramen | 12 | N | null |
| C | 2021-01-01 | ramen | 12 | N | null |
| C | 2021-01-07 | ramen | 12 | N | null |

---

### What is the total amount each customer spent at the restaurant?

- **SELECT**: `sales.customer_id, SUM(menu.price) AS total_price`
  - Selects the customer ID and calculates the total price of products, aliasing it as `total_price`.

- **INNER JOIN**: `menu ON sales.product_id = menu.product_id`
  - Joins the `sales` table with the `menu` table to get the price of each product based on `product_id`.

- **GROUP BY**: `sales.customer_id`
  - Groups the results by `sales.customer_id` to aggregate the sum of prices per customer.

> ```sql
> SELECT sales.customer_id,
>     SUM(menu.price) AS total_price
> FROM sales 
> INNER JOIN menu
>     ON sales.product_id = menu.product_id
> GROUP BY sales.customer_id;
> ```

| customer\_id | total\_price |
| :--- | :--- |
| B | 74 |
| C | 36 |
| A | 76 |

---

### How many days has each customer visited the restaurant?

- **SELECT**: `customer_id, COUNT(DISTINCT(order_date)) AS total_days_visited`
  - Selects the customer ID and counts the distinct order dates, aliasing it as `total_days_visited`.
  
- **GROUP BY**: `customer_id`
  - Groups the results by `customer_id` to aggregate the count of distinct order dates per customer.

> ```sql
> SELECT customer_id, 
>     COUNT(DISTINCT(order_date)) AS total_days_visited
> FROM sales
> GROUP BY customer_id;
> ```

| customer\_id | total\_days\_visited |
| :--- | :--- |
| A | 4 |
| B | 6 |
| C | 2 |

---

### What was the first item from the menu purchased by each customer?



- **CTE**: `WITH temp_table AS (...)`
  - Creates a temporary result set that can be referenced in the main query.
  
- **DENSE_RANK() Window Function**: `DENSE_RANK() OVER (PARTITION BY customer_id ORDER BY order_date) AS product_rank`
  - Assigns a rank to each product per customer based on the order date.

- **Filtering the First Purchase**: `WHERE product_rank = 1`
  - Ensures only the first-ranked product (i.e., the first product ordered by each customer) is selected.

> ```sql
> WITH temp_table AS (
>     SELECT customer_id,
>            order_date,
>            sales.product_id,
>            product_name,
>            DENSE_RANK() OVER (PARTITION BY customer_id ORDER BY order_date) AS product_rank
>     FROM sales
>          INNER JOIN menu ON sales.product_id = menu.product_id
> )
> SELECT customer_id,
>        order_date,
>        product_name
> FROM temp_table
> WHERE product_rank = 1;
> 
> ```

| customer\_id | order\_date | product\_name |
| :--- | :--- | :--- |
| A | 2021-01-01 | curry |
| A | 2021-01-01 | sushi |
| B | 2021-01-01 | curry |
| C | 2021-01-01 | ramen |
| C | 2021-01-01 | ramen |

---

### What is the most purchased item on the menu and how many times was it purchased by all customers?

- **CTE**: `WITH temp_table AS (...)`
  - Creates a temporary result set that includes customer ID, order date, product ID, product name, and price.

- **INNER JOIN**: `INNER JOIN menu ON sales.product_id = menu.product_id`
  - Joins the `sales` table with the `menu` table to get the product details based on `product_id`.

- **Selecting Product Count**: `SELECT product_name, COUNT(product_id) AS product_count`
  - Selects the product name and counts the occurrences of each product, aliasing it as `product_count`.

- **Grouping and Ordering**: `GROUP BY product_name ORDER BY product_count DESC LIMIT 1`
  - Groups the results by `product_name`, orders them by `product_count` in descending order, and limits the result to the top product.

> ```sql
> WITH temp_table AS (
>     SELECT customer_id,
>            order_date,
>            sales.product_id,
>            product_name,
>            price
>     FROM sales
>          INNER JOIN menu ON sales.product_id = menu.product_id
> )
> SELECT product_name,
>        COUNT(product_id) AS product_count
> FROM temp_table
> GROUP BY product_name
> ORDER BY product_count DESC
> LIMIT 1;
> 
> ```

| product\_name | product\_count |
| :--- | :--- |
| ramen | 8 |

---

### Which item was the most popular for each customer?

- **CTE**: `WITH most_pop AS (...)`
  - Creates a temporary result set that includes customer ID, product name, product count, and product rank.

- **DENSE_RANK() Window Function**: `DENSE_RANK() OVER (PARTITION BY sales.customer_id ORDER BY COUNT(sales.customer_id) DESC) AS product_rank`
  - Assigns a rank to each product per customer based on the count of products, ordered in descending order.

- **INNER JOIN**: `INNER JOIN menu ON sales.product_id = menu.product_id`
  - Joins the `sales` table with the `menu` table to get product details based on `product_id`.

- **GROUP BY**: `customer_id, menu.product_name`
  - Groups the results by `customer_id` and `product_name` to aggregate the count of each product.

- **Selecting Top Product**: `SELECT customer_id, product_name, product_count, product_rank`
  - Selects the customer ID, product name, product count, and product rank from the CTE.

- **Filtering the Top Product**: `WHERE product_rank = 1`
  - Ensures only the top-ranked product (i.e., the most frequently purchased product by each customer) is selected.

> ```sql
> WITH most_pop AS (
>     SELECT customer_id,
>            menu.product_name,
>            COUNT(menu.product_name) AS product_count,
>            DENSE_RANK() OVER (PARTITION BY sales.customer_id ORDER BY COUNT(sales.customer_id) DESC) AS product_rank
>     FROM sales
>          INNER JOIN menu ON sales.product_id = menu.product_id
>     GROUP BY customer_id, menu.product_name
> )
> SELECT customer_id,
>        product_name,
>        product_count,
>        product_rank
> FROM most_pop
> WHERE product_rank = 1;
> 
> ```

| customer\_id | product\_name | product\_count | product\_rank |
| :--- | :--- | :--- | :--- |
| A | ramen | 3 | 1 |
| B | sushi | 2 | 1 |
| B | curry | 2 | 1 |
| B | ramen | 2 | 1 |
| C | ramen | 3 | 1 |

### Which item was purchased first by the customer after they became a member?

- **CTE**: `WITH first_item AS (...)`
  - Creates a temporary result set that includes customer ID, order date, product name, and rank.

- **DENSE_RANK() Window Function**: `DENSE_RANK() OVER (PARTITION BY sales.customer_id ORDER BY sales.order_date ASC) AS rank`
  - Assigns a rank to each product per customer based on the order date, in ascending order.

- **INNER JOIN**: 
  - `INNER JOIN members ON sales.customer_id = members.customer_id AND sales.order_date > members.join_date`
    - Joins `sales` with `members` on `customer_id` and filters for orders placed after the member's join date.
  - `INNER JOIN menu ON sales.product_id = menu.product_id`
    - Joins `sales` with `menu` to get product details based on `product_id`.

- **Selecting First Purchase**: `SELECT customer_id, order_date, product_name`
  - Selects the customer ID, order date, and product name from the CTE.

- **Filtering the First Purchase**: `WHERE rank = 1`
  - Ensures only the first-ranked product (i.e., the first product ordered by each customer after joining) is selected.

> ```sql
> WITH first_item AS (
>     SELECT sales.customer_id,
>            sales.order_date,
>            menu.product_name,
>            DENSE_RANK() OVER (PARTITION BY sales.customer_id ORDER BY sales.order_date ASC) AS rank
>     FROM sales
>          INNER JOIN members ON sales.customer_id = members.customer_id
>         AND sales.order_date > members.join_date
>          INNER JOIN menu ON sales.product_id = menu.product_id
> )
> SELECT customer_id,
>        order_date,
>        product_name
> FROM first_item
> WHERE rank = 1;
> 
> ```

| customer\_id | order\_date | product\_name |
| :--- | :--- | :--- |
| A | 2021-01-10 | ramen |
| B | 2021-01-11 | sushi |

---

### Which item was purchased just before the customer became a member?

- **CTE**: `WITH first_item AS (...)`
  - Creates a temporary result set that includes customer ID, order date, product name, and rank.

- **DENSE_RANK() Window Function**: `DENSE_RANK() OVER (PARTITION BY sales.customer_id ORDER BY sales.order_date DESC) AS rank`
  - Assigns a rank to each product per customer based on the order date, in descending order.

- **INNER JOIN**: 
  - `INNER JOIN members ON sales.customer_id = members.customer_id AND sales.order_date < members.join_date`
    - Joins `sales` with `members` on `customer_id` and filters for orders placed before the member's join date.
  - `INNER JOIN menu ON sales.product_id = menu.product_id`
    - Joins `sales` with `menu` to get product details based on `product_id`.

- **Selecting First Purchase**: `SELECT customer_id, order_date, product_name`
  - Selects the customer ID, order date, and product name from the CTE.

- **Filtering the First Purchase**: `WHERE rank = 1`
  - Ensures only the first-ranked product (i.e., the most recent product ordered by each customer before joining) is selected.

> ```sql
> WITH first_item AS (
>     SELECT sales.customer_id,
>            sales.order_date,
>            menu.product_name,
>            DENSE_RANK() OVER (PARTITION BY sales.customer_id ORDER BY sales.order_date DESC) AS rank
>     FROM sales
>          INNER JOIN members ON sales.customer_id = members.customer_id
>         AND sales.order_date < members.join_date
>          INNER JOIN menu ON sales.product_id = menu.product_id
> )
> SELECT customer_id,
>        order_date,
>        product_name
> FROM first_item
> WHERE rank = 1;
> 
> ```

| customer\_id | order\_date | product\_name |
| :--- | :--- | :--- |
| A | 2021-01-01 | sushi |
| A | 2021-01-01 | curry |
| B | 2021-01-04 | sushi |

---

### What is the total items and amount spent for each member before they became a member?

- **CTE**: `WITH temp_table AS (...)`
  - Creates a temporary result set that includes customer ID, order date, product ID, and price.

- **INNER JOIN**: 
  - `INNER JOIN members ON sales.customer_id = members.customer_id AND sales.order_date < members.join_date`
    - Joins `sales` with `members` on `customer_id` and filters for orders placed before the member's join date.
  - `INNER JOIN menu ON sales.product_id = menu.product_id`
    - Joins `sales` with `menu` to get product details based on `product_id`.

- **Selecting Aggregated Data**: `SELECT customer_id, COUNT(product_id) AS total_items, SUM(price) AS total_price`
  - Selects the customer ID, counts the total number of items purchased, and calculates the total price of the items.

- **GROUP BY**: `GROUP BY customer_id`
  - Groups the results by `customer_id` to aggregate the count and sum of items per customer.

> ```sql
> WITH temp_table AS (
>     SELECT sales.customer_id,
>            sales.order_date,
>            menu.product_id,
>            menu.price
>     FROM sales
>          INNER JOIN members ON sales.customer_id = members.customer_id
>         AND sales.order_date < members.join_date
>          INNER JOIN menu ON sales.product_id = menu.product_id
> )
> SELECT customer_id,
>        COUNT(product_id) AS total_items,
>        SUM(price) AS total_price
> FROM temp_table
> GROUP BY customer_id;
> 
> ```

| customer\_id | total\_items | total\_price |
| :--- | :--- | :--- |
| B | 3 | 40 |
| A | 2 | 25 |

---

### If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

- **CTE**: `WITH temp_table AS (...)`
  - Creates a temporary result set that includes customer ID and points calculated based on product ID.

- **CASE Statement**: `CASE WHEN menu.product_id = 1 THEN price * 20 ELSE price * 10 END AS points`
  - Calculates points based on whether the product ID is 1 or not, assigning 20 times the price for product ID 1 and 10 times the price for other products.

- **INNER JOIN**: `INNER JOIN menu ON sales.product_id = menu.product_id`
  - Joins `sales` with `menu` to get product details based on `product_id`.

- **Aggregation**: `SELECT customer_id, SUM(points)`
  - Selects the customer ID and sums the points for each customer.

- **GROUP BY**: `GROUP BY customer_id`
  - Groups the results by `customer_id` to aggregate the sum of points for each customer.

> ```sql
> WITH temp_table AS (
>     SELECT customer_id,
>            CASE
>                WHEN menu.product_id = 1 THEN price * 20
>                ELSE price * 10
>                END AS points
>     FROM sales
>          INNER JOIN menu ON sales.product_id = menu.product_id
> )
> SELECT customer_id,
>        SUM(points)
> FROM temp_table
> GROUP BY customer_id;
> 
> ```

| customer\_id | sum |
| :--- | :--- |
| B | 940 |
| C | 360 |
| A | 860 |

---

### In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

- **SELECT**: `SELECT sales.customer_id, SUM(CASE ... END) AS total_points`
  - Selects the customer ID and sums the points for each customer, aliasing it as `total_points`.

- **CASE Statement**: `CASE WHEN sales.product_id = 1 THEN menu.price * 20 ELSE menu.price * 10 END`
  - Calculates points by assigning 20 times the price for product ID 1 and 10 times the price for other products.

- **INNER JOIN**:
  - `INNER JOIN members ON sales.customer_id = members.customer_id`
    - Joins `sales` with `members` on `customer_id`.
  - `INNER JOIN menu ON sales.product_id = menu.product_id`
    - Joins `sales` with `menu` to get product details based on `product_id`.

- **WHERE**:
  - `WHERE order_date >= members.join_date`
    - Filters results to include only orders placed on or after the member's join date.
  - `AND EXTRACT(MONTH FROM order_date) = 1`
    - Filters results to include only orders placed in January.

- **GROUP BY**: `GROUP BY sales.customer_id`
  - Groups the results by `customer_id` to aggregate the total points for each customer.

> ```sql
> SELECT sales.customer_id,
>        SUM(CASE
>                WHEN sales.product_id = 1 THEN menu.price * 20
>                ELSE menu.price * 10
>            END) AS points
> FROM sales
>      INNER JOIN members ON sales.customer_id = members.customer_id
>      INNER JOIN menu ON sales.product_id = menu.product_id
> WHERE order_date >= members.join_date
>   AND EXTRACT(MONTH FROM order_date) = 1
> GROUP BY sales.customer_id;
> 
> ```

| customer\_id | points |
| :--- | :--- |
| B | 320 |
| A | 510 |

---

### Recreate the following table output using the available data:

| **customer\_id** | **order\_date** | **product\_name** | **price** | **member** |
| --- | --- | --- | --- | --- |
| A | 2021-01-01 | curry | 15 | N |
| A | 2021-01-01 | sushi | 10 | N |
| A | 2021-01-07 | curry | 15 | Y |
| A | 2021-01-10 | ramen | 12 | Y |
| A | 2021-01-11 | ramen | 12 | Y |
| A | 2021-01-11 | ramen | 12 | Y |
| B | 2021-01-01 | curry | 15 | N |
| B | 2021-01-02 | curry | 15 | N |
| B | 2021-01-04 | sushi | 10 | N |
| B | 2021-01-11 | sushi | 10 | Y |
| B | 2021-01-16 | ramen | 12 | Y |
| B | 2021-02-01 | ramen | 12 | Y |
| C | 2021-01-01 | ramen | 12 | N |
| C | 2021-01-01 | ramen | 12 | N |
| C | 2021-01-07 | ramen | 12 | N |


> ```sql
> SELECT sales.customer_id,
>        sales.order_date,
>        menu.product_name,
>        menu.price,
>        CASE
>            WHEN order_date >= members.join_date THEN 'Y'
>            ELSE 'N'
>            END AS member
> FROM sales
>      INNER JOIN members ON sales.customer_id = members.customer_id
>      INNER JOIN menu ON sales.product_id = menu.product_id;
> 
> ```

---

### Rank All The Things Danny also requires further information about the ranking of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ranking values for the records when customers are not yet part of the loyalty program.

| **customer\_id** | **order\_date** | **product\_name** | **price** | **member** | **ranking** |
| --- | --- | --- | --- | --- | --- |
| A | 2021-01-01 | curry | 15 | N | null |
| A | 2021-01-01 | sushi | 10 | N | null |
| A | 2021-01-07 | curry | 15 | Y | 1 |
| A | 2021-01-10 | ramen | 12 | Y | 2 |
| A | 2021-01-11 | ramen | 12 | Y | 3 |
| A | 2021-01-11 | ramen | 12 | Y | 3 |
| B | 2021-01-01 | curry | 15 | N | null |
| B | 2021-01-02 | curry | 15 | N | null |
| B | 2021-01-04 | sushi | 10 | N | null |
| B | 2021-01-11 | sushi | 10 | Y | 1 |
| B | 2021-01-16 | ramen | 12 | Y | 2 |
| B | 2021-02-01 | ramen | 12 | Y | 3 |
| C | 2021-01-01 | ramen | 12 | N | null |
| C | 2021-01-01 | ramen | 12 | N | null |
| C | 2021-01-07 | ramen | 12 | N | null |


WITH temp_table AS (
    SELECT sales.customer_id,
           sales.order_date,
           menu.product_name,
           menu.price,
           CASE
               WHEN order_date >= members.join_date THEN 'Y'
               ELSE 'N'
               END AS member
    FROM sales
         INNER JOIN members ON sales.customer_id = members.customer_id
         LEFT JOIN menu ON sales.product_id = menu.product_id
),
rank_table AS (
    SELECT temp_table.customer_id,
           temp_table.order_date,
           DENSE_RANK() OVER(PARTITION BY temp_table.customer_id ORDER BY temp_table.order_date) AS ranking
    FROM temp_table
    WHERE temp_table.member = 'Y'
)
SELECT temp_table.customer_id,
       temp_table.order_date,
       product_name,
       price,
       member,
       ranking
FROM temp_table
LEFT JOIN rank_table ON temp_table.order_date = rank_table.order_date;




WITH temp_table AS (
    SELECT
        s.customer_id,
        s.order_date,
        m.product_name,
        m.price,
        CASE
            WHEN s.order_date >= mem.join_date THEN 'Y'
            ELSE 'N'
        END AS member
    FROM
        sales s
        INNER JOIN menu m ON s.product_id = m.product_id
        LEFT JOIN members mem ON s.customer_id = mem.customer_id
),
ranked_table AS (
    SELECT
        customer_id,
        order_date,
        product_name,
        price,
        member,
        DENSE_RANK() OVER (PARTITION BY customer_id ORDER BY order_date) AS ranking
    FROM
        temp_table
    WHERE
        member = 'Y'
)
SELECT
    customer_id,
    order_date,
    product_name,
    price,
    member,
    CASE
        WHEN member = 'Y' THEN ranking
        ELSE NULL
    END AS ranking
FROM
    temp_table
LEFT JOIN
    ranked_table USING (customer_id, order_date, product_name, price, member)
ORDER BY
    customer_id, order_date;

SELECT *
FROM sales;