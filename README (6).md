
# 🍜 Case Study #1: Danny's Diner

# Solutions



### 1. What is the total amount each customer spent at the restaurant?



``` sql
SELECT s.customer_id, SUM(price) AS total_sales
FROM dbo.sales AS s
JOIN dbo.menu AS m
   ON s.product_id = m.product_id
GROUP BY customer_id; 

```

### Steps:
- Use **SUM** and **GROUP** BY to find out total_sales contributed by each customer.

- Use **JOIN** to merge sales and menu tables as customer_id and price are from both tables.

| customer_id            | total_sales                                                            |
| ----------------- | ------------------------------------------------------------------ |
| A | 76 |
| B | 74 |
| C | 36 |

- Customer A spent $76.

- Customer B spent $74.

- Customer C spent $36

### 2. How many days has each customer visited the restaurant?

``` sql
SELECT customer_id, COUNT(DISTINCT(order_date)) AS visit_count
FROM dbo.sales
GROUP BY customer_id;

```
### Steps:
- Use **DISTINCT** and wrap with **COUNT** to find out the visit_count for each customer.

- If we do not use **DISTINCT** on order_date, the number of days may be repeated. For example, if Customer A visited the restaurant twice on '2021–01–07', then number of days is counted as 2 days instead of 1 day.

| customer_id            | visit_count                                                           |
| ----------------- | ------------------------------------------------------------------ |
| A | 4 |
| B | 6 |
| C | 2 |

- Customer A visited 4 times.

- Customer B visited 6 times.

- Customer C visited 2 times

### 3. What was the first item from the menu purchased by each customer?

``` sql
WITH ordered_sales_cte AS
(
   SELECT customer_id, order_date, product_name,
      DENSE_RANK() OVER(PARTITION BY s.customer_id
      ORDER BY s.order_date) AS rank
   FROM dbo.sales AS s
   JOIN dbo.menu AS m
      ON s.product_id = m.product_id
)

SELECT customer_id, product_name
FROM ordered_sales_cte
WHERE rank = 1
GROUP BY customer_id, product_name;

```
### Steps:
- Create a temp table order_sales_cte and use **Windows** function with **DENSE_RANK** to create a new column rank based on order_date

- Instead of **ROW_NUMBER** or **RANK**, use **DENSE_RANK** as order_date is not time-stamped hence, there is no sequence as to which item is ordered first if 2 or more items are ordered on the same day.

- Subsequently, **GROUP** BY all columns to show rank = 1 only.

| customer_id            | product_name                                                          |
| ----------------- | ------------------------------------------------------------------ |
| A | curry |
| A | sushi |
| B | curry |
| C | ramen |

- Customer A's first orders are curry and sushi.

- Customer B's first order is curry.

- Customer C's first order is ramen.

### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

``` sql
SELECT TOP 1 (COUNT(s.product_id)) AS most_purchased, product_name
FROM dbo.sales AS s
JOIN dbo.menu AS m
   ON s.product_id = m.product_id
GROUP BY s.product_id, product_name
ORDER BY most_purchased DESC;

```
### Steps:
- **COUNT** number of product_id and **ORDER BY** most_purchased by descending order.

- Then, use TOP 1 to filter highest number of purchased item.


| most_purchased           | product_name                                                         |
| ----------------- | ------------------------------------------------------------------ |
| 8 | ramen |


- Most purchased item on the menu is ramen which is 8 times.

### 5. Which item was the most popular for each customer?

``` sql
WITH fav_item_cte AS
(
   SELECT s.customer_id, m.product_name, COUNT(m.product_id) AS order_count,
      DENSE_RANK() OVER(PARTITION BY s.customer_id
      ORDER BY COUNT(s.customer_id) DESC) AS rank
   FROM dbo.menu AS m
   JOIN dbo.sales AS s
      ON m.product_id = s.product_id
   GROUP BY s.customer_id, m.product_name
)

SELECT customer_id, product_name, order_count
FROM fav_item_cte 
WHERE rank = 1;

```
### Steps:
- Create a fav_item_cte and use **DENSE_RANK** to rank the order_count for each product by descending order for each customer

- Generate results where product rank = 1 only as the most popular product for each customer.


| customer_id| product_name|order_count                                                           |
| ----------------- | ------------------ |----------------------|
| A | ramen |3|
| B | sushi |2|
| B | curry |2|
| B | ramen |2|
| C | ramen |3|

- Customer A and C's favourite item is ramen.

- Customer B enjoys all items on the menu. He/she is a true foodie, sounds like me!

### 6. What is the total items and amount spent for each member before they became a member?

``` sql
SELECT s.customer_id, COUNT(DISTINCT s.product_id) AS unique_menu_item, 
   SUM(mm.price) AS total_sales
FROM sales AS s
JOIN members AS m
   ON s.customer_id = m.customer_id
JOIN menu AS mm
   ON s.product_id = mm.product_id
WHERE s.order_date < m.join_date
GROUP BY s.customer_id;

```
### Steps:
- Filter order_date before join_date and perform a **COUNT DISTINCT** on product_id and **SUM** the total spent before becoming member.


| customer_id            | unique_menu_item |total_sales
| ----------------- | --------------- |-----------------|
| A | 2 | 25
| B | 2 | 40


- Customer A spent $ 25 on 2 items.

- Customer B spent $40 on 2 items.

