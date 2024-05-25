

### What is the total amount each customer spent at the restaurant?

- **INNER JOIN** `sales` with `menu` to obtain price of each product.
- **GROUP BY** `customer_id` and use **SUM** to aggregate total price

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

- **GROUP BY** `customer_id`
- **COUNT** and **DISTINCT** to count unique dates

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

- Construct CTE with `product_name` and rank by `order_date`
    - **INNER JOIN** `sales` with `menu` to obtain `product_name`
    - Rank by `order_date` using window function **DENSE_RANK()** with **PARTITION BY** `customer_id` and **ORDER BY** `order_date`
- Use CTE in main query and filter table using **WHERE** to show only rank 1

> ```sql
> WITH temp_table AS (SELECT customer_id,
>                            order_date,
>                            sales.product_id,
>                            product_name,
>                            DENSE_RANK() OVER (PARTITION BY customer_id ORDER BY order_date) AS product_rank
>                     FROM sales
>                          INNER JOIN menu ON sales.product_id = menu.product_id)
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