## Join Implementation

In this section 3 base tables were generated. The base table of the join between the sales table and the menu table was generated first and afterwards the other two base tables were generated. 

These two last tables are the combination of all the 3 tables in the ERD but one base table had more rows than the other which was explained later on before the 3rd base table was generated.

The first base table (join of sales & menu) should include the following columns:
- customer_id
- order_date
- product_id
- product_name
- price

The second and third base tables should include these columns:
- customer_id
- order_date
- product_id
- product_name
- price
- join_date

The process for generating the tables was by answering the questions down below before a join of tables took place. The questions helped to choose between using an inner join or left join so that no data is lost when a join happens.

```
1. What is the purpose of joining these two tables?

a. What contextual hypotheses do we have about the data?

b. How can we validate these assumptions?

2. What is the distribution of foreign keys within each table?

3. How many overlapping and missing unique foreign key values are there between the two tables?
```

## sales and menu joint table

Only one join was necessary to generate the desired base table. 

| Join Journey Part | Start         | End           | Foreign Key  |
|-------------------|---------------|---------------|--------------|
| Part 1            | ``sales``        | ``menu``     | ``product_id`` |


**1. What is the purpose of joining these two tables?**

Match the ``product_id`` from the sales table on the ``product_id`` of the menu table to obtain the name and price of each product.

**a. What contextual hypotheses do we have about the data?**

**Hypothesis 1:**
> There will be multiple records per unique ``product_id`` in the ``sales`` table since one specific product might have multiple sales

```sql
WITH counts_base AS (
  SELECT
    product_id,
    COUNT(product_id) AS row_counts
  FROM dannys_diner.sales
  GROUP BY product_id
)
SELECT
  row_counts,
  COUNT(DISTINCT product_id) AS unique_product_id_values
FROM counts_base
GROUP BY row_counts
ORDER BY row_counts;
```

| row_counts | unique_product_id_values |
|------------|--------------------------|
| 3          | 1                        |
| 4          | 1                        |
| 8          | 1                        |

**Findings**

There were multiple records per ``product_id`` value in the ``sales`` table.

**Hypothesis 2:**

> In the menu table there should be a 1-to-1 relationship for the ``product_id`` since it would not make sense to have duplicate ``product_id`` values.

```sql
WITH counts_base AS (
  SELECT
    product_id,
    COUNT(product_id) AS row_counts
  FROM dannys_diner.menu
  GROUP BY product_id
)
SELECT
  row_counts,
  COUNT(DISTINCT product_id) AS unique_product_id_values
FROM counts_base
GROUP BY row_counts
ORDER BY row_counts;
```
| row_counts | unique_product_id_values |
|------------|--------------------------|
| 1          | 3                        |

**Findings**

There is a 1-to-1 relationship in the ``menu`` table.


**2. What is the distribution of foreign keys within each table?**

This was done with the validation of the hypotheses above.


**3. How many overlapping and missing unique foreign key values are there between the two tables?**

By using an anti join, the foreign keys which only exist in the sales table and not in the menu table are shown.

**Sales Table**
```sql
SELECT
  COUNT(DISTINCT product_id)
FROM dannys_diner.sales
WHERE NOT EXISTS (
  SELECT product_id
  FROM dannys_diner.menu
  WHERE sales.product_id = menu.product_id
);
```

| count |
|-------|
| 0     |

**Findings**

The ``product_id`` values from the ``sales`` table exist in the ``menu`` table.

**Menu Table**

The unique foreign keys that only exists in the menu table and not in the sales table were also checked.

```sql
SELECT
  COUNT(DISTINCT product_id)
FROM dannys_diner.menu
WHERE NOT EXISTS (
  SELECT product_id
  FROM dannys_diner.sales
  WHERE menu.product_id = sales.product_id  
);
```
| count |
|-------|
| 0     |

**Findings**

The ``product_id`` values from the ``menu`` table exist in the ``sales`` table.

### Left Join or Inner Join?

The left and inner joins were implemented to validate that the raw counts are the same for both joins.

```sql
DROP TABLE IF EXISTS left_sales_join;
CREATE TEMP TABLE left_sales_join AS 
SELECT 
  sales.customer_id 
  sales.product_id,
  menu.product_name
FROM dannys_diner.sales
LEFT JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id;
  
DROP TABLE IF EXISTS inner_sales_join;
CREATE TEMP TABLE inner_sales_join AS 
SELECT 
  sales.customer_id, 
  sales.product_id,
  menu.product_name
FROM dannys_diner.sales
INNER JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id;
  
(
  SELECT
    'left join' AS join_type,
    COUNT(*) AS record_count,
    COUNT(DISTINCT product_id) AS unique_key_values 
  FROM left_sales_join
)  

UNION ALL 

(
  SELECT 
    'inner join' AS join_type,
    COUNT(*) AS record_count,
    COUNT(DISTINCT product_id) AS unique_key_values
  FROM inner_sales_join  
)
```

| join_type  | record_count | unique_key_values |
|------------|--------------|-------------------|
| left join  | 15           | 3                 |
| inner join | 15           | 3                 |

As seen in the table above, no matter which join is used the output will be the same.

## Output for the sales and menu joint table
The table joins to create the first base table  is shown below.

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
| customer_id | order_date | product_id | product_name | price |
|-------------|------------|------------|--------------|-------|
| B           | 2021-01-04 | 1          | sushi        | 10    |
| A           | 2021-01-01 | 1          | sushi        | 10    |
| B           | 2021-01-11 | 1          | sushi        | 10    |
| B           | 2021-01-01 | 2          | curry        | 15    |
| B           | 2021-01-02 | 2          | curry        | 15    |

## sales, menu and members joint table

To generate this base table, two joins were necessary.

| Join Journey Part | Start         | End           | Foreign Key  |
|-------------------|---------------|---------------|--------------|
| Part 1            | ``sales``        | ``menu``     | ``product_id`` |
| Part 2            | ``sales``     | ``members``          | ``customer_id``      |

Part 1 is the same process as the previous base table and since it was covered already above, it was not shown here again.

##  Join Journey Part 2

**1. What is the purpose of joining these two tables?**

Match the ``customer_id`` from the sales table on the ``customer_id`` of the member table to obtain the customers that are members.

**a. What contextual hypotheses do we have about the data?**

**Hypothesis 1:**
> There will be multiple records per unique ``customer_id`` in the ``sales`` table. 

```sql
WITH counts_base AS (
  SELECT
    customer_id,
    COUNT(customer_id) AS row_counts
  FROM dannys_diner.sales
  GROUP BY product_id
)
SELECT
  row_counts,
  COUNT(DISTINCT customer_id) AS unique_customer_id_values
FROM counts_base
GROUP BY row_counts
ORDER BY row_counts;
```

| row_counts | unique_product_id_values |
|------------|--------------------------|
| 3          | 1                        |
| 6          | 2                        |

**Findings**

There were multiple records per ``product_id`` value in the ``sales`` table.

**Hypothesis 2:**

> In the ``members`` table there should be a 1-to-1 relationship for the ``customer_id`` since it would not make sense to have duplicate ``customer_id`` values.

```sql
WITH counts_base AS (
  SELECT
    customer_id,
    COUNT(customer_id) AS row_counts
  FROM dannys_diner.members
  GROUP BY product_id
)
SELECT
  row_counts,
  COUNT(DISTINCT customer_id) AS unique_product_id_values
FROM counts_base
GROUP BY row_counts
ORDER BY row_counts;
```
| row_counts | unique_product_id_values |
|------------|--------------------------|
| 1          | 2                        |

**Findings**

There is a 1-to-1 relationship in the ``menu`` table.

**2. What is the distribution of foreign keys within each table?**

This was done with the validation of the hypotheses above.


**3. How many overlapping and missing unique foreign key values are there between the two tables?**

By using an anti join, the foreign keys which only exist in the ``sales`` table and not in the ``members`` table are shown.

**Sales Table**
```sql
SELECT
  COUNT(DISTINCT customer_id)
FROM dannys_diner.sales
WHERE NOT EXISTS (
  SELECT customer_id
  FROM dannys_diner.members
  WHERE sales.customer_id = members.customer_id
);
```

| count |
|-------|
| 1     |

**Findings**

There was one ``customer_id`` value that is not in the members table.

This value was checked.

```sql
SELECT DISTINCT
    customer_id
FROM dannys_diner.sales
WHERE NOT EXISTS (
  SELECT customer_id
  FROM dannys_diner.members
  WHERE sales.customer_id = members.customer_id
);
```
| customer_id |
|-------|
| C     |

``customer_id`` C was not a member

**Menu Table**

The unique foreign keys that only exists in the ``members`` table and not in the ``sales`` table were also checked.

```sql
SELECT
  COUNT(DISTINCT customer_id)
FROM dannys_diner.members
WHERE NOT EXISTS (
  SELECT customer_id
  FROM dannys_diner.sales
  WHERE members.customer_id = sales.customer_id  
);
```
| count |
|-------|
| 0     |

**Findings**

The ``customer_id`` values from the ``members`` table exist in the ``sales`` table.

### Left Join or Inner Join?

To generate the second base table an inner join had to be used because only the customers that are members would be included while to generate the third base table a left join was needed so that the customers that are not members would also be considered and could be used for the bonus questions. 

```sql
DROP TABLE IF EXISTS left_sales_join;
CREATE TEMP TABLE left_sales_join AS 
SELECT 
  sales.customer_id,
  members.join_date
FROM dannys_diner.sales
LEFT JOIN dannys_diner.members
  ON sales.customer_id = members.customer_id;
  
DROP TABLE IF EXISTS inner_sales_join;
CREATE TEMP TABLE inner_sales_join AS 
SELECT 
  sales.customer_id,
  members.join_date
FROM dannys_diner.sales
INNER JOIN dannys_diner.members
  ON sales.customer_id = members.customer_id;
  
(
  SELECT
    'left join' AS join_type,
    COUNT(*) AS record_count,
    COUNT(DISTINCT customer_id) AS unique_key_values 
  FROM left_sales_join
)  

UNION ALL 

(
  SELECT 
    'inner join' AS join_type,
    COUNT(*) AS record_count,
    COUNT(DISTINCT customer_id) AS unique_key_values
  FROM inner_sales_join  
)
```

| join_type  | record_count | unique_key_values |
|------------|--------------|-------------------|
| left join  | 15           | 3                 |
| inner join | 12           | 2                 |

## Output for the sales, menu and members joint table

**Second Base Table**
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
| customer_id | order_date | product_id | product_name | price | join_date  |
|-------------|------------|------------|--------------|-------|------------|
| A           | 2021-01-07 | 2          | curry        | 15    | 2021-01-07 |
| A           | 2021-01-11 | 3          | ramen        | 12    | 2021-01-07 |
| A           | 2021-01-11 | 3          | ramen        | 12    | 2021-01-07 |
| A           | 2021-01-10 | 3          | ramen        | 12    | 2021-01-0  |
| A           | 2021-01-01 | 1          | sushi        | 10    | 2021-01-07 |

**Third Base Table**
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

| customer_id | order_date | product_id | product_name | price | join_date |
|-------------|------------|------------|--------------|-------|-----------|
| C           | 2021-01-01 | 3          | ramen        | 12    | null      |
| C           | 2021-01-01 | 3          | ramen        | 12    | null      |
| C           | 2021-01-07 | 3          | ramen        | 12    | null      |

## Problem Solving 

In the final section all the questions including the bonus ones were answered.

[![table joins](view-problem-approach.svg)](github.com)