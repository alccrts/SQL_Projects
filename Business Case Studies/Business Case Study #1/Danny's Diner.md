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

#### Answer:
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

#### Answer:
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

#### Answer:
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

## Conclusion 
