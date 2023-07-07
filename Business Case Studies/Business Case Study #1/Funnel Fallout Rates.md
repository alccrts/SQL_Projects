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
***
**4. What is the number of events for each event type?**
***
**5. What is the percentage of visits which have a purchase event?**
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

