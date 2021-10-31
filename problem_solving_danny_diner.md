# Problem Solving 

In this section the questions were solved wiht the use of the base tables generated on the previous section.

Base Tables that were used to solve the questions:

**sales and menu tables combined**

This base table was used for questions 1, 3, 4, 5 and 9.
```sql
DROP TABLE IF EXISTS menu_sales;
CREATE TEMP TABLE menu_sales AS
SELECT 
  sales.customer_id,
  sales.order_date,
  sales.product_id,
  menu.product_name,
  menu.price
FROM dannys_diner.sales
INNER JOIN dannys_diner.menu 
  ON sales.product_id = menu.product_id;
```
**sales, menu and members tables combined**

This base table was used for questions 6, 7, 8 and 10.
```sql
DROP TABLE IF EXISTS menu_sales_members;
CREATE TEMP TABLE menu_sales_members AS
SELECT 
  sales.customer_id,
  sales.order_date,
  sales.product_id,
  menu.product_name,
  menu.price,
  members.join_date
FROM dannys_diner.sales
INNER JOIN dannys_diner.menu 
  ON sales.product_id = menu.product_id
INNER JOIN dannys_diner.members
  ON sales.customer_id = members.customer_id;
```

**sales, menu and members tables combined**

This base table was used for the bonus questions 11 and 12.

```sql
DROP TABLE IF EXISTS menu_sales_members;
CREATE TEMP TABLE menu_sales_members AS
SELECT 
  sales.customer_id,
  sales.order_date,
  sales.product_id,
  menu.product_name,
  menu.price,
  members.join_date
FROM dannys_diner.sales
INNER JOIN dannys_diner.menu 
  ON sales.product_id = menu.product_id
LEFT JOIN dannys_diner.members
  ON sales.customer_id = members.customer_id;
``` 

**1.- What is the total amount each customer spent at the restaurant?**

Using a ``GROUP BY`` for the ``customer_id`` and a ``SUM`` of the prices of the products that each customer had helped to generate how much money every customer spent at the diner. 

```sql
SELECT 
  customer_id,
  SUM(price) AS money_spent
FROM menu_sales
GROUP BY customer_id
ORDER BY money_spent DESC;
```

| customer_id | money_spent |
|-------------|-------------|
| A           | 76          |
| B           | 74          |
| C           | 36          |

**Findings:**

Customer A spent 76$ in total, Customer B spent 74$ in total and Customer C spent only 36$ in total.

**2.- How many days has each customer visited the restaurant?**

By doing a ``GROUP BY`` on ``customer_id`` and counting every distinct ``order_date`` from every ``customer_id``, generated the output for each customer's visit.

```sql
SELECT 
  customer_id,
  COUNT(DISTINCT order_date) AS visits
FROM dannys_diner.sales
GROUP BY customer_id
ORDER BY visits DESC;
```

| customer_id | visits |
|-------------|--------|
| B           | 6      |
| A           | 4      |
| C           | 2      |

**Findings:**

Customer B visited the store 6 times, customer A 4 times and customer C only 2 times.

**3.- What was the first item from the menu purchased by each customer?**

Using window function ``RANK`` to do a ranking of the products that each customer had, based on the ordering dates, where the products that were bought on the first order date would be rank number 1. The window function was created inside a CTE so that afterwards only the number 1 rank for every customer could be filtered and as a result giving the first product or products each customer had for the first time.

```sql
WITH rank_orders AS(
  SELECT 
    customer_id,
    product_name,
    order_date,
    RANK() 
    OVER
    (
      PARTITION BY 
        customer_id
      ORDER BY 
        order_date ASC
    ) AS ranking
  FROM menu_sales
)

SELECT DISTINCT
  customer_id,
  product_name,
  order_date
FROM rank_orders
WHERE ranking = 1;
```

| customer_id | product_name | order_date |
|-------------|--------------|------------|
| A           | curry        | 2021-01-01 |
| A           | sushi        | 2021-01-01 |
| B           | curry        | 2021-01-01 |
| C           | ramen        | 2021-01-01 |

**Findings:**

Customer A had curry and sushi for the first time, customer B had curry and customer C had ramen.

**4.- What is the most purchased item on the menu and how many times was it purchased by all customers?**

Doing a ``GROUP BY`` on ``product_name`` and counting how many times a product was bought helped to generate the most sold products where the first row indicates the most sold product.

```sql
SELECT
  product_name,
  COUNT(*) AS times_consumed
FROM menu_sales
GROUP BY product_name
ORDER BY times_consumed DESC; 
```

| product_name | times_consumed |
--------------|----------------|
| ramen        | 8              |
| curry        | 4              |
| sushi        | 3              |

 **Findings:**

The most consumed product was ramen as it was purchased 8 times.

**5.- Which item was the most popular for each customer?**

Two CTEs were generated, one CTE had the counts of each product that each customer consumed and the other CTE had the ranks of the customer product counts from the first CTE. After that a query that filtered the ranking 1 from the second CTE was created and as a result generated the most popular dish/dishes of each customer. 

```sql
WITH customer_product_count AS (
  SELECT 
    customer_id,
    product_name,
    COUNT(*) AS times_consumed
  FROM menu_sales
  GROUP BY 
    customer_id, 
    product_name
),

rank_customer_fav AS (
  SELECT 
  customer_id,
  product_name,
  times_consumed,
  RANK()
  OVER
  (
    PARTITION BY customer_id
    ORDER BY times_consumed DESC
  ) AS ranking
  FROM customer_product_count
)

SELECT 
  customer_id,
  product_name,
  times_consumed
FROM rank_customer_fav
WHERE ranking = 1
ORDER BY 
  customer_id ASC,
  times_consumed DESC;
```  

| customer_id | product_name | times_consumed |
|-------------|--------------|----------------|
| A           | ramen        | 3              |
| B           | curry        | 2              |
| B           | ramen        | 2              |
| B           | sushi        | 2              |
| C           | ramen        | 3              |

**Findings:**

The favorite dish of customer A & customer C is ramen while the favorite dishes for customer B are curry, ramen and sushi.

**6.- Which item was purchased first by the customer after they became a member?**

First, all the records equal or after the member joining date of each customer were filtered after that the window function ``RANK`` was used to do a ranking of the products that each customer had, based on the ordering dates, where the product with the earliest date would be ranked number 1.

The window function was created inside a CTE so that afterwards only the number 1 rank for every customer could be filtered and as a result giving the product or products each customer had after becoming a member.

```sql
WITH product_after_member AS (
  SELECT 
    customer_id,
    product_name,
    RANK()OVER(
      PARTITION BY customer_id
      ORDER BY order_date
    ) AS ranking,
    order_date
  FROM menu_sales_members
  WHERE order_date >= join_date 
)

SELECT 
  customer_id,
  product_name,
  order_date
FROM product_after_member
WHERE ranking = 1
ORDER BY 
  customer_id ASC;
```

| customer_id | product_name | order_date |
|-------------|--------------|------------|
| A           | curry        | 2021-01-07 |
| B           | sushi        | 2021-01-11 |

**Findings:**

Customer A bought curry and customer B had sushi after being a member.

**7.- Which item was purchased just before the customer became a member?**

First, all the records before the member joining date of each customer were filtered after that the window function ``RANK`` was used to do a ranking of the products that each customer had, based on the ordering dates, where the product with the latest date would be ranked number 1.

The window function was created inside a CTE so that afterwards only the number 1 rank for every customer could be filtered and as a result giving the product or products each customer had before becoming a member.

```sql
WITH product_before_member AS (
  SELECT 
    customer_id,
    product_name,
    RANK()OVER(
      PARTITION BY customer_id
      ORDER BY order_date DESC
    ) AS ranking,
    order_date
  FROM menu_sales_members
  WHERE order_date < join_date 
)

SELECT 
  customer_id,
  product_name,
  order_date
FROM product_before_member
WHERE ranking = 1
ORDER BY 
  customer_id ASC;
```

| customer_id | product_name | order_date |
|-------------|--------------|------------|
| A           | sushi        | 2021-01-01 |
| A           | curry        | 2021-01-01 |
| B           | curry        | 2021-01-01 |

**Findings:**

Customer A bought sushi and curry while customer B had curry before being a member.

**8.- What is the total items and amount spent for each member before they became a member?**

The records before the member joining date for each customer were filtered. After, the ``customer_id`` was grouped so that the number of unique products and the amount spent could be calculated for each customer.

```sql
SELECT 
  customer_id,
  COUNT(DISTINCT product_name) AS unique_product,
  SUM(price) AS money_spent
FROM menu_sales_members
WHERE order_date < join_date 
GROUP BY customer_id
ORDER BY customer_id ASC;
```

| customer_id | unique_product | money_spent |
|-------------|----------------|-------------|
| A           | 2              | 25          |
| B           | 2              | 40          |

**Findings:**

Before becoming a member customer A spent 25$ on 2 unique products while customer B spent 40$ on 2 unique products.

**9.- If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**

Inside a CTE, all the points for each product that each customer had was calculated. To calculate the points each product's price was multiplied by 10 and times 20 if the product was sushi.

After the CTE was created, the points were summed up for every customer.

```sql
WITH customer_points AS (
  SELECT 
    customer_id,
    product_name,
    price,
    CASE 
      WHEN product_name = 'sushi' THEN price * 20
      ELSE price * 10
  END AS points  
  FROM menu_sales
)

SELECT 
  customer_id,
  SUM(points) AS total_points
FROM customer_points
GROUP BY customer_id
ORDER BY customer_id;
```

| customer_id | total_points |
|-------------|--------------|
| A           | 860          |
| B           | 940          |
| C           | 360          |

**Findings:**

Customer A has 860 points, customer B has 940 points and customer C has 360 points.

**10.- In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**

Inside a CTE, all the points for each product that each customer had in january was calculated. To calculate the points each product's price was multiplied by 10 and times 20 if the product was sushi. Additionally, if the customer became a member then the product's price would be multiplied by 20 for a week. 

```sql
WITH customer_points AS (
  SELECT 
    customer_id,
    product_name,
    price,
    CASE 
      WHEN order_date BETWEEN join_date AND (join_date + 6) THEN price * 20
      WHEN product_name = 'sushi' THEN price * 20
      ELSE price * 10
  END AS points  
  FROM menu_sales_members
  WHERE order_date <= '2021-01-31'
)

SELECT 
  customer_id,
  SUM(points) AS total_points
FROM customer_points
GROUP BY customer_id
ORDER BY customer_id;
```

| customer_id | total_points |
|-------------|--------------|
| A           | 1370         |
| B           | 820          |

**Findings:**

At the end of january, customer A would have 1370 points while customer B would have 820 points.

## Bonus Questions

**11.- Recreate the following table output using the available data of each customer:**

| customer_id | order_date | product_name | price | member |
|-------------|------------|--------------|-------|--------|
| A           | 01/01/2021 | curry        | 15    | N      |
| A           | 01/01/2021 | sushi        | 10    | N      |
| A           | 07/01/2021 | curry        | 15    | Y      |
| A           | 10/01/2021 | ramen        | 12    | Y      |

To check if a ``customer_id`` was a member or not, a ``CASE`` was applied where if the customer's ``order_date`` equalled or had a later date than the customer's ``join_date`` to be a member then a 'Y' was assigned, if the ``order_date `` was not the same or the ``order_date`` had a date before the customer's member ``join_date`` or if the customer was not in the member's table then a 'N' was assigned.

```sql
SELECT 
  customer_id,
  order_date,
  product_name,
  price,
  CASE 
    WHEN order_date >= join_date THEN 'Y'
    ELSE 'N'
  END AS member 
FROM menu_sales_members
ORDER BY 
  customer_id ASC,
  order_date ASC;
```

| customer_id | order_date | product_name | price | member |
|-------------|------------|--------------|-------|--------|
| A           | 2021-01-01 | sushi        | 10    | N      |
| A           | 2021-01-01 | curry        | 15    | N      |
| A           | 2021-01-07 | curry        | 15    | Y      |
| A           | 2021-01-10 | ramen        | 12    | Y      |
| A           | 2021-01-11 | ramen        | 12    | Y      |
| A           | 2021-01-11 | ramen        | 12    | Y      |
| B           | 2021-01-01 | curry        | 15    | N      |
| B           | 2021-01-02 | curry        | 15    | N      |
| B           | 2021-01-04 | sushi        | 10    | N      |
| B           | 2021-01-11 | sushi        | 10    | Y      |
| B           | 2021-01-16 | ramen        | 12    | Y      |
| B           | 2021-02-01 | ramen        | 12    | Y      |
| C           | 2021-01-01 | ramen        | 12    | N      |
| C           | 2021-01-01 | ramen        | 12    | N      |
| C           | 2021-01-07 | ramen        | 12    | N      |

**12.- Danny also requires further information about the ranking of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ranking values for the records when customers are not yet part of the loyalty program.**

The query from the previous question was put inside a CTE so that in another query the products that the customer had after becoming a member could be ranked and for the products that the customer had before being a member they would assigned with a null. 
 
```sql
WITH is_member AS(
  SELECT 
    customer_id,
    order_date,
    product_name,
    price,
    CASE 
      WHEN order_date >= join_date THEN 'Y'
      ELSE 'N'
    END AS member 
  FROM menu_sales_members
)

SELECT
  customer_id,
  order_date,
  product_name,
  price,
  member,
  CASE 
    WHEN member = 'N' THEN null
    ELSE RANK() OVER(
      PARTITION BY customer_id, member 
      ORDER BY order_date)
  END AS ranking
FROM is_member
```

| customer_id | order_date | product_name | price | member | ranking |
|-------------|------------|--------------|-------|--------|---------|
| A           | 2021-01-01 | sushi        | 10    | N      | null    |
| A           | 2021-01-01 | curry        | 15    | N      | null    |
| A           | 2021-01-07 | curry        | 15    | Y      | 1       |
| A           | 2021-01-10 | ramen        | 12    | Y      | 2       |
| A           | 2021-01-11 | ramen        | 12    | Y      | 3       |
| A           | 2021-01-11 | ramen        | 12    | Y      | 3       |
| B           | 2021-01-01 | curry        | 15    | N      | null    |
| B           | 2021-01-02 | curry        | 15    | N      | null    |
| B           | 2021-01-04 | sushi        | 10    | N      | null    |
| B           | 2021-01-11 | sushi        | 10    | Y      | 1       |
| B           | 2021-01-16 | ramen        | 12    | Y      | 2       |
| B           | 2021-02-01 | ramen        | 12    | Y      | 3       |
| C           | 2021-01-01 | ramen        | 12    | N      | null    |
| C           | 2021-01-01 | ramen        | 12    | N      | null    |
| C           | 2021-01-07 | ramen        | 12    | N      | null    |