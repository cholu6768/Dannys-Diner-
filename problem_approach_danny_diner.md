# Problem Approach

In this section the possible output for every question was shown. This was done to have an idea on what to focus before fully solving the questions.  


## Question 1 - What is the total amount each customer spent at the restaurant?


| customer_id | money_spent |
|-------------|-------------|
| A           | 35          |
| B           | 98          |
| C           | 44          |


## Question 2 - How many days has each customer visited the restaurant?


| customer_id | visits |
|-------------|--------|
| B           | 10      |
| A           | 2     |
| C           | 1      |

## Question 3 - What was the first item from the menu purchased by each customer?


| customer_id | product_name | order_date |
|-------------|--------------|------------|
| A           | sushi        | 2021-01-03 |
| B           | curry        | 2021-01-05 |
| C           | ramen        | 2021-01-02 |


## Question 4  - What is the most purchased item on the menu and how many times was it purchased by all customers?


 product_name | times_consumed |
--------------|----------------|
 curry        | 5              |



## Question 5 - Which item was the most popular for each customer?


| customer_id | product_name | times_consumed |
|-------------|--------------|----------------|
| A           | ramen        | 2              |
| B           | curry        | 4              |
| C           | ramen        | 1              |

## Question 6 - Which item was purchased first by the customer after they became a member?

| customer_id | product_name | order_date |
|-------------|--------------|------------|
| A           | ramen        | 2021-01-09 |
| B           | sushi        | 2021-01-13 |

## Question 7 - Which item was purchased just before the customer became a member?

| customer_id | product_name | order_date |
|-------------|--------------|------------|
| A           | sushi        | 2021-01-01 |
| B           | curry        | 2021-01-01 |

## Question 8 - What is the total items and amount spent for each member before they became a member?

| customer_id | unique_product | money_spent |
|-------------|----------------|-------------|
| A           | 3              | 25          |
| B           | 1              | 40          |

## Question 9 - If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?


| customer_id | total_points |
|-------------|--------------|
| A           | 860          |
| B           | 940          |
| C           | 360          |

## Question 10 - In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

| customer_id | total_points |
|-------------|--------------|
| A           | 1370         |
| B           | 820          |

## Question 11 - Recreate the following table output using the available data of each customer:

| customer_id | order_date | product_name | price | member |
|-------------|------------|--------------|-------|--------|
| A           | 01/01/2021 | curry        | 15    | N      |
| A           | 01/01/2021 | sushi        | 10    | N      |
| A           | 07/01/2021 | curry        | 15    | Y      |
| A           | 10/01/2021 | ramen        | 12    | Y      |

## Question 12 - Danny also requires further information about the ranking of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ranking values for the records when customers are not yet part of the loyalty program.

| customer_id | order_date | product_name | price | member | ranking |
|-------------|------------|--------------|-------|--------|---------|
| A           | 2021-01-01 | sushi        | 10    | N      | null    |
| A           | 2021-01-01 | curry        | 15    | N      | null    |
| A           | 2021-01-07 | curry        | 15    | Y      | 1       |
| A           | 2021-01-10 | ramen        | 12    | Y      | 2       |
| A           | 2021-01-11 | ramen        | 12    | Y      | 3       |
| A           | 2021-01-11 | ramen        | 12    | Y      | 3       |


## Findings 

The output of questions 1, 3, 4, 5 and 9 would need a base table from the join of the ``sales`` and ``menu`` tables. While the output for questions 6, 7, 8, 10, 11 and 12 would need another base table consisting of the join of all 3 tables (sales, members and menu).

The next step would be on generating the required base tables.

## Join Implementation

In this next section the process of joining the tables was shown and also how the base tables were generated. 

[![table joins](view-join-implementation.svg)](https://github.com/cholu6768/Dannys-Diner-/blob/main/join_implementation_danny_diner.md)
