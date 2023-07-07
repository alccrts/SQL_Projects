![image](https://github.com/alccrts/SQL_Projects/assets/138128361/022dfa4e-735a-4bf8-bb09-7b29ce9b400c)
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

The founder has provided five datasets: `users`, `events`, `campaign_identifier`, `page_identifier`, `event_identifier`.  To demonstrate the relationship between the tables, I have created the Entity Relationship Diagram below using the [DB Diagram tool](https://dbdiagram.io/home).  I have analysed the data using PostgreSQL 13 and you can view it at [DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138).

![image](https://github.com/alccrts/SQL_Projects/assets/138128361/5dca56a1-7afc-4df6-a786-271916719e8b)

## Questions & Analysis

## Part 1: Digital Analysis

**1. How many users are there?**

To find the total number of users, I ran a simple **COUNT (DISTINCT)** function
````
SELECT
	COUNT(DISTINCT user_id) AS total_users
FROM	clique_bait.users;
````

| total_users |
| ----------- |
| 500         |


***

**2. How many cookies does each user have on average?**

I created a cte table to first count the number of cookies per users.  I then averaged these results with a seperate **SELECT** statement.  I rounded the result to the nearest whole number because it isn't possible to have anything less than a whole cookie.  

````
    WITH user_count AS (
    
    	SELECT COUNT(cookie_id) AS cookie_count 
    	FROM clique_bait.users
    	GROUP BY (user_id))
       	
        
    SELECT ROUND(AVG(cookie_count),0) AS avg_cookies_per_user
    FROM user_count;
````

| avg_cookies_per_user |
| -------------------- |
| 4                    |


***

**3. What is the unique number of visits by all users per month?**

In order to group results by month, I used the **EXTRACT** operator to identify the month if each visit.  I then used a simple **COUNT** and **GROUP BY** operation to count the visit_ids per month.
    
````  
    SELECT EXTRACT (MONTH FROM event_time) AS month, COUNT (DISTINCT visit_id) AS unique_visit_ids
    FROM clique_bait.events
    GROUP BY (month)
    ORDER BY month;

````

| month | unique_visit_ids |
| ----- | ----- |
| 1     | 876   |
| 2     | 1488  |
| 3     | 916   |
| 4     | 248   |
| 5     | 36    |

***

**4. What is the number of events for each event type?**

This query used **COUNT** and **GROUP BY** to count the number of events for each event type. 

````
    SELECT event_type, COUNT (event_type) AS num_of_events
    FROM clique_bait.events
    GROUP BY event_type
    ORDER BY event_type;

````
| event_type | num_of_events |
| ---------- | ------------- |
| 1          | 20928         |
| 2          | 8451          |
| 3          | 1777          |
| 4          | 876           |
| 5          | 702           |


***
**5. What is the percentage of visits which have a purchase event?**

Despite being a simple percentage calculation, I found this question quite tricky.  Everything I tried returned the result of zero.  After some research, I discovered that this was because the character type of my **SELECT** statments were coming out as intergers, but the result of my division was a decimal number.  I therefore used the **CASE** operator to change the character type to float.  I was then able to multiple this result by 100 to get a percentage number. 

````
    SELECT 
    (CAST (COUNT(DISTINCT events.visit_id) AS FLOAT)
     /
    (SELECT CAST (COUNT(DISTINCT events.visit_id) AS FLOAT)
    FROM clique_bait.events) ) *100
    AS purchase_percentage
    FROM clique_bait.events JOIN clique_bait.event_identifier ON (event_identifier.event_type=events.event_type)
    WHERE event_identifier.event_name = 'Purchase';

````
| purchase_percentage |
| ------------------- |
| 49.85970819304153   |


***
**6. What is the percentage of visits which view the checkout page but do not have a purchase event?**
***
**7. What are the top 3 pages by number of views?**
***
**8. What is the number of views and cart adds for each product category?**
***
**9. What are the top 3 products by purchases?**
***




## Conclusion 

