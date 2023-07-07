![image](https://qul.imgix.net/9d127111-d042-43db-982f-ca34c84e54c9/462298_sld.jpg)
# Case Study: Funnel Fallout Rates

This case study comes from the [#8weeksqlchallenge](https://8weeksqlchallenge.com/case-study-1/) series.  

## 📃 Contents
- [Business Problem](#business-problem)
- [Data](#data)
- [Questions & Analysis](#questions--analysis)
- [Conculsion](#conclusion)

## Business Problem

This case study is centered around a fictional online food store.  The founder of the company wants to analyse customer data to find creative solutions to calculate funnel fallout rates.  

## Data

The founder has provided five datasets: `users`, `events`, `campaign_identifier`, `page_identifier`, `event_identifier`.  Customers who visit the Clique Bait website are tagged via their `cookie_id`. The data is somewhat limited because for privacy reasons, Danny provided only a sample of his overall customer data.  In addition, the data spans a limited date range due to Danny's business being new. I have analysed the data using PostgreSQL 13 on [DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138).  

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

**6. Which item was purchased first by the customer after they became a member?**

The question asks what customers bought after they become a member, but they data we have .  It is possible a customer became a member on the same day they made an order.  Given that we don't have a more finite 'time' data, we cannot be sure whether the customer became a member first or placed their order first.  To answer this question, I have made the assumption that if a customer placed an order on the same day they become a member, then they became a member first before placing the order.  This seems the most likely scenario, given that being a member comes with promotions that might affect their order.  However, it would be sensible to confirm this with Danny.  

Based on the above assumption, I created a CTE table which includes only orders placed by a customer on or after the join date of that customer. I then used **DENSE_RANK()** to organise that data, followed by a **SELECT** statement to query that table to pull the first result or after the members join date.  

|  rank  | customer_id | product_name |  order_date  |
|------| ----------- | ---------- | ---------- |
|  1  | A           | curry        |  2021-01-07T00:00:00.000Z  | 
|  1  | B           | sushi        |  2021-01-11T00:00:00.000Z  |

````
WITH members_orders AS (
SELECT 
  DENSE_RANK() OVER
  (PARTITION BY sales.customer_id ORDER BY sales.order_date ASC) AS rank, sales.customer_id, menu.product_name, sales.order_date
FROM dannys_diner.menu
JOIN dannys_diner.sales ON (sales.product_id=menu.product_id)
JOIN dannys_diner.members ON (sales.customer_id=members.customer_id)
WHERE order_date >= join_date 
  )
  
SELECT* FROM members_orders
WHERE rank = 1
````

**7. Which item was purchased just before the customer became a member?**

As discussed in Question 6, to answer this question I have assumed we are searching for orders placed on dates before the customer became a member.  I have therefore employed the same query as above but have inverted the boolean operator to return only orders before the join date in the CTE table.  I also inverted the sorting from **ASC** to **DESC**.  Customer A ordered two products just before they became a member, Sushi and Curry, whereas Customer B purchased only Sushi.  

|  rank  | customer_id | product_name |  order_date  |
|------| ----------- | ---------- | ---------- |
|  1  | A           | sushi        |  2021-01-01T00:00:00.000Z  | 
|  1  | A           | curry        |  2021-01-01T00:00:00.000Z  |
|  1  | B           | sushi        |  2021-01-04T00:00:00.000Z  |

````
WITH members_orders AS (
SELECT 
  DENSE_RANK() OVER
  (PARTITION BY sales.customer_id ORDER BY sales.order_date DESC) AS rank, sales.customer_id, menu.product_name, sales.order_date
FROM dannys_diner.menu
JOIN dannys_diner.sales ON (sales.product_id=menu.product_id)
JOIN dannys_diner.members ON (sales.customer_id=members.customer_id)
WHERE order_date < join_date 
  )
  
SELECT* FROM members_orders
WHERE rank = 1
````

**8. What is the total items and amount spent for each member before they became a member?**

Here I used **SUM** to total the amount spent by each customer and **COUNT** to count the number of items.  These were results were joined with the members table using **JOIN** so that I could filter them to show only results before the join date of members.  I then repeated this query but filtered the results to include only results following the join date of members.  This way I could compare Customer spending before and after their membership to see if the membership was encouraging them to pay more.  In the results below we can see that Customer A began to spend more after he became a member, whereas Customer B spend a little bit less but the amount of itmes they bought did not increase.  We are therefore unable to say for certain if membership encourages customers to pay more.   

| customer_id | total_spent_before | total_items_before | total_spent_after | total_items_after |
| ----------- | ------------------ | ------------------ | ----------------- | ----------------- |
| A           | 25                 | 2                  | 51                | 4                 |
| B           | 40                 | 3                  | 34                | 3                 |

````
WITH after AS (

SELECT 
	sales.customer_id, SUM(menu.price) AS total_spent_after, 
	COUNT (sales.product_id) AS total_items_after
FROM 
	dannys_diner.menu
JOIN 
	dannys_diner.sales ON (sales.product_id=menu.product_id)
JOIN 
	dannys_diner.members ON (sales.customer_id=members.customer_id)
WHERE 
	order_date >= join_date 
GROUP BY 
	sales.customer_id
ORDER BY 
	sales.customer_id
), before AS (

SELECT 
	sales.customer_id, SUM(menu.price) AS total_spent_before, 
	COUNT (sales.product_id) AS total_items_before
FROM 
	dannys_diner.menu
JOIN 
	dannys_diner.sales ON (sales.product_id=menu.product_id)
JOIN 
	dannys_diner.members ON (sales.customer_id=members.customer_id)
WHERE 
	order_date < join_date 
GROUP BY 
	sales.customer_id
ORDER BY 
	sales.customer_id
)

SELECT before.customer_id, total_spent_before, total_items_before, total_spent_after, total_items_after
FROM before JOIN after ON (before.customer_id=after.customer_id)

````

**9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**

To calcuate the points each customer has, I used the **CASE** operator to multiply their total spending by the 1 point for each dollar.  By referring to the ``product_id``, I was able to multiply spending on Sushi x2.  

| customer_id | points |
| ----------- | ------ |
| A           | 860    |
| B           | 940    |
| C           | 360    |

SELECT 
	sales.customer_id, SUM(CASE (menu.product_id)
	WHEN 1 THEN 20*price
    ELSE 10*price
    END) AS points
FROM 
	dannys_diner.menu
JOIN 
	dannys_diner.sales ON (sales.product_id=menu.product_id)
GROUP BY
	sales.customer_id
ORDER BY
	customer_id;



**10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**
**Schema (PostgreSQL v13)**

The goal here was to identify all the orders that fall within the promotional period and then calculate the amount of points each customer received during that period.  I then calculated the amount of points the customers would have recevied following the promotional period, up until the end of January.  This invovled the creation of two CTE tables, which I joined together using the **UNION ALL** operator to create a table with all points.  I could then use **SUM** and **GROUP BY** to total the points for each customer.  

| customer_id | total_points  |
| ----------- | ---- |
| A           | 1020 |
| B           | 320  |

````
    WITH order_points AS (
    WITH promo_orders AS (
    SELECT 
      sales.customer_id, price*20 AS points 
    FROM 
      dannys_diner.menu
    JOIN 
      dannys_diner.sales ON (sales.product_id=menu.product_id)
    JOIN 
      dannys_diner.members ON (sales.customer_id=members.customer_id)
    WHERE 
      order_date <= (join_date+6) AND order_date >= join_date)
    
    , post_promo_orders AS (
    SELECT 
      sales.customer_id, price*10 AS points 
    FROM 
      dannys_diner.menu
    JOIN 
      dannys_diner.sales ON (sales.product_id=menu.product_id)
    JOIN 
      dannys_diner.members ON (sales.customer_id=members.customer_id)
    WHERE 
      order_date > (join_date+6) AND order_date < '2021-02-01'
    )
    
    SELECT * 
    FROM 
      post_promo_orders 
    UNION ALL 
    SELECT * 
    FROM promo_orders
    )
    
    
    SELECT
    	customer_id, SUM(points) AS total_points
    FROM 
    	order_points 
    GROUP BY 
    	customer_id;
````
***


## Conclusion 

After analysing the small data set from Danny’s Diner, we were able to identify the highest paying customers, their product preferences and spending habits.  

In general, customers began tasting Sushi and Curry, but ultimately Ramen was the most purchased item, with all customers switching to Ramen towards their later visits to the restaurant.  The Ramen dish is clearly popular and this knowledge could be used to create a better experience to attract more members.  For example, Danny could also offer double points for purchasing Ramen.

It was encouraging to see that Customers A and B continued their high spending behaviour after they had become members.  However, there was an absence of clear increase in spending following membership, which could suggest that the membership program is not doing enough to increase customer spending.  Danny could consider offering additional benefits if his goal is to increase customer spending. 
