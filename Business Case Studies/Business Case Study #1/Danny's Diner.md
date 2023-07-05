# 🍣 Case Study: Danny's Diner 

This case study comes from the [#8weeksqlchallenge](https://8weeksqlchallenge.com/case-study-1/) series.

## 📃 Contents
- [Business Problem](#business-problem)
- [Data](#data)
- [Questions & Analysis](#question-&-analysis)
- [Conculsion](#conclusion)

## Business Problem

Danny has recently opened a new restaurant. He hopes to deliver a personalised experience for his customers, so he needs help analysing the data he has collected during the first few months of operation.

## Data

Danny has provided three datasets: `sales`, `menu`, `members` in the `dannys_dataset` schema.  The data is somewhat limited because for privacy reasons, Danny provided only a sample of his overall customer data.  In addition, the data spans a limited date range due to Danny's business being new. I have analysed the data using PostgreSQL 13 on [DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138).  

![image](https://user-images.githubusercontent.com/81607668/127271130-dca9aedd-4ca9-4ed8-b6ec-1e1920dca4a8.png)

## Questions & Analysis

**1. What is the total amount each customer spent at the restaurant?**

To approach this question, I used the **JOIN** operator to combine two data sets on the shared key 'product_id'.  I then used the aggregate function **SUM** and **GROUP BY** to calculate the total amount spent by each customer, along with the **ORDER BY** function to sort the results from highest amount spend to lowest to easily see which customer spent the most.

| customer_id | total_sales |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |
````
SELECT
  sales.customer_id, SUM (menu.price) AS total_spent
FROM
  dannys_diner.menu JOIN dannys_diner.sales ON (sales.product_id=menu.product_id)
GROUP BY
  sales.customer_id
ORDER BY
  total_spent DESC
````

***

**2. How many days has each customer visited the restaurant?**

Here, I used **COUNT** with **DISTINCT** to total the number of days each customer placed an order.  This assumed that customers visited the store on this date and did not order online.  As before, I sorted the results using **ORDER BY** to easily see which customer made the most visits.  

It's interesting to see that Customer A spent more than Customer B, even though Customer B made more visits to the store.  Customer A is therefore a relatively high-spending customer. 😃

| customer_id | visit_count |
| ----------- | ----------- |
| B           | 6          |
| A           | 4          |
| C           | 2          |
````
SELECT
  sales.customer_id, COUNT (DISTINCT sales.order_date) AS total_days
FROM
  dannys_diner.sales 
GROUP BY
  sales.customer_id
ORDER BY
  total_days DESC
````

***

**3. What was the first item from the menu purchased by each customer?**

My first attempt at this answer used embedded **SELECT** statements to filter the **MIN** order date for each customer.  This returned the correct results, but I then realised it would be impossible to use embedded select statements if there were a larger number of customers.  I therefore researched an alternative answer using the **DENSE RANK()** function.  I have included both SQL queries below.  There were two results for Customer A, meaning he ordered two products at the same time for his first order.

| customer_id | product_name | 
| ----------- | ----------- |
| A           | curry        | 
| A           | sushi        | 
| B           | curry        | 
| C           | ramen        |

````
**1st Attempt**

SELECT
  customer_id, product_name
FROM
  dannys_diner.menu JOIN dannys_diner.sales ON (sales.product_id=menu.product_id)
WHERE
  order_date = 
      (SELECT
        MIN(order_date) 
      FROM
        dannys_diner.menu JOIN dannys_diner.sales ON (sales.product_id=menu.product_id)
      WHERE
        customer_id = 'A')

OR
 order_date = 
      (SELECT
        MIN(order_date) 
      FROM
        dannys_diner.menu JOIN dannys_diner.sales ON (sales.product_id=menu.product_id)
      WHERE
        customer_id = 'B')

OR
 order_date = 
      (SELECT
        MIN(order_date) 
      FROM
        dannys_diner.menu JOIN dannys_diner.sales ON (sales.product_id=menu.product_id)
      WHERE
        customer_id = 'C')
                                  
ORDER BY
  customer_id





***2nd Attempt***

WITH order_table AS (
  SELECT 
  sales.customer_id, 
  menu.product_name, 
  sales.order_date,
  DENSE_RANK () OVER (
    PARTITION BY sales.customer_id 
    ORDER BY sales.order_date ASC) AS order_rank
  FROM 
  	dannys_diner.menu JOIN dannys_diner.sales ON (sales.product_id=menu.product_id)
  						)
                 
SELECT 
	customer_id, product_name
FROM 
	order_table
WHERE order_rank = 1
GROUP BY customer_id, product_name;

````                          


***

**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**

This was a nice and easy query to build using **COUNT** to total the number of times a product was ordered.  Ramen was the product ordered the most! 

````
SELECT 
	menu.product_name, COUNT(product_name) AS order_quantity
FROM 
	dannys_diner.menu JOIN dannys_diner.sales ON (sales.product_id=menu.product_id)
GROUP BY 
	product_name 
ORDER BY 
	order_quantity DESC
LIMIT 1
````

***

**5. Which item was the most popular for each customer?**

As I discovered on question 3, this question was best answered by creating a CTE with the **DENSE RANK ()** function and then referencing that to find the most purchsed product among the users.  As we can see, Ramen was the most popular among all Customers.  However, Customer B equally enjoyed Sushi and Curry.  

| customer_id | product_name | order_count | rank
| ----------- | ---------- |------------  |------------  |
| A           | ramen        |  3   |  1   |
| B           | sushi        |  2   |  1   |
| B           | curry        |  2   |  1   |
| B           | ramen        |  2   |  1   |
| C           | ramen        |  3   |  1   |

````
WITH most_popular AS (
  SELECT 
    sales.customer_id, 
    menu.product_name, 
    COUNT(menu.product_id) AS order_count,
    DENSE_RANK() OVER (
      PARTITION BY sales.customer_id 
      ORDER BY COUNT(menu.product_id) DESC) AS rank
  FROM dannys_diner.menu
  INNER JOIN dannys_diner.sales
    ON menu.product_id = sales.product_id
  GROUP BY sales.customer_id, menu.product_name
)

SELECT 
  customer_id, 
  product_name, 
  order_count,
  rank
FROM most_popular 
WHERE rank = 1;
````

***

## 6. Which item was purchased first by the customer after they became a member?

The question asks what customers bought after they become a member, but they data we have .  It is possible a customer became a member on the same day they made an order.  Given that we don't have a more finite 'time' data, we cannot be sure whether the customer became a member first or placed their order first.  To answer this question, I have made the assumption that if a customer placed an order on the same day they become a member, then they became a member first before placing the order.  This seems the most likely scenario, given that being a member comes with promotions that might affect their order.  However, it would be sensible to confirm this with Danny.  

Based on the above assumption, I created a CTE table which includes only orders placed by a customer on or after the join date of that customer. I then used **DENSE_RANK()** to organise that data, followed by a **SELECT** statement to query that table to pull the first result or after the members join date.  

| customer_id | product_name |
| ----------- | ---------- |
| A           | curry        |
| B           | sushi        |

````
WITH members_orders AS (
SELECT 
  DENSE_RANK() OVER
  (PARTITION BY sales.customer_id ORDER BY sales.order_date ASC) AS rank, sales.customer_id, menu.product_name, sales.order_date
FROM
	dannys_diner.menu JOIN dannys_diner.sales ON (sales.product_id=menu.product_id) JOIN dannys_diner.members ON (sales.customer_id=members.customer_id)
WHERE order_date >= join_date 
  )
  
SELECT* FROM members_orders
WHERE rank = 1
````



## Conclusion 
