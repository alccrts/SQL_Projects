## UK Arts Funding Analysis using SQL

![Arts Funding Dashboard 2](https://github.com/alccrts/Data_Visualizations/assets/138128361/4a1099a3-13d3-48b8-8e56-b8fb62359923)

In this project, I have taken data from Arts Council England to review their funding patterns across five years.  I have used Biq Query to explore the data and Excel to create a dashboard.

### Key Findings
* Funding increased by over 30% from 2019 to 2023.
* A 50% reduction in funding took place in the year following the pandemic.
* There is a trend towards granting fewer awards of a higher amount, rather than a large quantity of small value awards.  The number of awards granted has decreased by approximately 25%, but the value of the awards has increased by over 50% from 2019 to 2023.
* The London region consistently receives the highest proportion of funds, between 28% - 31%.
* The artistic disciplines that recieve the highest proportion of the awards over the 5 years are Theatre, Combined Arts and Visual Arts.


***


### Preparing the Data

The Arts Council England is UK's national development agency for creativity and culture.  The council distributes grants funded by The National Lottery to support individual practitioners, communities and cultural organisations with projects that focus on Combined Arts including festivals and carnivals, Dance, Libraries, Literature, Museums, Music, Theatre or Visual Arts.  This analysis will review The National Lottery Grants given during 2018 - 2023.  You can view the [Arts Council Website](https://www.artscouncil.org.uk/ProjectGrants) for more information.   

The data is published by the council - [you can find the data here](https://www.artscouncil.org.uk/research-and-data/our-data).   Preparing the data required little work, as it was already well processed by the council.  I could use Excel to make some small changes.  My main tasks were:
* combine the annual data of five years into a single CSV file 
* removed currency formatting
* changed all text to lower case
* complete missing location data by researching the awarded projects online
* upload to BigQuery (preview below)

![image](https://github.com/alccrts/SQL_Projects/assets/138128361/039e7763-8ecf-43af-b698-9d8df64166c5)


***


### Exploring the Data

Below is a sample of the questions I devised in order to explore the data.  

**What was the total value of the grants distributed each year and across all five years?** 

![image](https://github.com/alccrts/SQL_Projects/assets/138128361/8019695c-c754-44c4-bf1b-03bd44cfcf64)
<details>
<summary>View SQL Query</summary>
<br>
  
```sql
SELECT 

(
  
SELECT sum(award_amount) 
FROM `acedata.acedata.acedata`
WHERE year = 2019) AS total_2019 ,  

(

SELECT sum(award_amount) 
FROM `acedata.acedata.acedata`
WHERE year = 2020) AS total_2020, 

(

SELECT sum(award_amount) 
FROM `acedata.acedata.acedata`
WHERE year = 2021) AS total_2021, 

(

SELECT sum(award_amount) 
FROM `acedata.acedata.acedata`
WHERE year = 2022) AS total_2022, 

(

SELECT sum(award_amount) 
FROM `acedata.acedata.acedata`
WHERE year = 2023) AS total_2023, 

(
SELECT sum(award_amount) 
FROM `acedata.acedata.acedata`) AS total_all_years

```
</details>


**What is the average amount of funding each location received over the last five years?**

![image](https://github.com/alccrts/SQL_Projects/assets/138128361/6592e0d1-3ec1-4a6e-b9e8-fe9e1872672c)
<details>
<summary>View SQL Query</summary>
<br>
  
```sql
WITH `acedata.acedata.sumaward` as (
  
  SELECT 
    local_authority, 
    SUM(award_amount) as t
FROM `acedata.acedata.acedata` 
GROUP BY local_authority
ORDER BY t DESC)

SELECT 
  ROUND(AVG(t),0) AS average_funding FROM `acedata.acedata.sumaward`
```
</details>

**Which are the top three locations that receive the most funding each year?**

![image](https://github.com/alccrts/SQL_Projects/assets/138128361/d6dbaa31-e0f8-4b8a-903b-21d4a56bb7a9)
<details>
<summary>View SQL Query</summary>
<br>
  
```sql
WITH  `acedata.acedata.top_locations19` AS(

SELECT 
  local_authority AS local_authority_2019,
  DENSE_RANK() OVER (PARTITION BY year ORDER BY sum(award_amount) DESC ) AS RANK
FROM `acedata.acedata.acedata`
WHERE YEAR = 2019
GROUP BY local_authority, year), 

 `acedata.acedata.top_locations20` AS(

SELECT 
  local_authority AS local_authority_2020,
  DENSE_RANK() OVER (PARTITION BY year ORDER BY sum(award_amount) DESC ) AS RANK
FROM `acedata.acedata.acedata`
WHERE YEAR = 2020
GROUP BY local_authority, year), 

 `acedata.acedata.top_locations21` AS(

SELECT 
  local_authority AS local_authority_2021,
  DENSE_RANK() OVER (PARTITION BY year ORDER BY sum(award_amount) DESC ) AS RANK
FROM `acedata.acedata.acedata`
WHERE YEAR = 2021
GROUP BY local_authority, year), 

 `acedata.acedata.top_locations22` AS(

SELECT 
  local_authority AS local_authority_2022,
  DENSE_RANK() OVER (PARTITION BY year ORDER BY sum(award_amount) DESC ) AS RANK
FROM `acedata.acedata.acedata`
WHERE YEAR = 2022
GROUP BY local_authority, year), 

 `acedata.acedata.top_locations23` AS(

SELECT 
  local_authority AS local_authority_2023,
  DENSE_RANK() OVER (PARTITION BY year ORDER BY sum(award_amount) DESC ) AS RANK
FROM `acedata.acedata.acedata`
WHERE YEAR = 2023
GROUP BY local_authority, year)


SELECT a.local_authority_2019, b.local_authority_2020, c.local_authority_2021, d.local_authority_2022, e.local_authority_2023 from `acedata.acedata.top_locations19` AS a JOIN `acedata.acedata.top_locations20` AS b ON (a.rank=b.rank) JOIN `acedata.acedata.top_locations21` as c ON (c.rank=b.rank) JOIN `acedata.acedata.top_locations22` as d ON (c.rank=d.rank) JOIN `acedata.acedata.top_locations23` as e ON (e.rank=d.rank)
WHERE a.rank < 4
```
</details>


**Which are the top three disciplines that receive the most funding each year?**

![image](https://github.com/alccrts/SQL_Projects/assets/138128361/50035312-8b1f-40a0-a17c-7c157695c83e)
<details>
<summary>View SQL Query</summary>
<br>
  
```sql
WITH  `acedata.acedata.top_disciplines19` AS(

SELECT 
  main_discipline AS top_dis_2019,
  DENSE_RANK() OVER (PARTITION BY year ORDER BY sum(award_amount) DESC ) AS RANK
FROM `acedata.acedata.acedata`
WHERE YEAR = 2019
GROUP BY main_discipline, year), 

 `acedata.acedata.top_disciplines20` AS(

SELECT 
  main_discipline AS top_dis_2020,
  DENSE_RANK() OVER (PARTITION BY year ORDER BY sum(award_amount) DESC ) AS RANK
FROM `acedata.acedata.acedata`
WHERE YEAR = 2020
GROUP BY main_discipline, year), 

 `acedata.acedata.top_disciplines21` AS(

SELECT 
  main_discipline AS top_dis_2021,
  DENSE_RANK() OVER (PARTITION BY year ORDER BY sum(award_amount) DESC ) AS RANK
FROM `acedata.acedata.acedata`
WHERE YEAR = 2021
GROUP BY main_discipline, year), 

 `acedata.acedata.top_disciplines22` AS(

SELECT 
  main_discipline AS top_dis_2022,
  DENSE_RANK() OVER (PARTITION BY year ORDER BY sum(award_amount) DESC ) AS RANK
FROM `acedata.acedata.acedata`
WHERE YEAR = 2022
GROUP BY main_discipline, year), 

 `acedata.acedata.top_disciplines23` AS(

SELECT 
  main_discipline AS top_dis_2023,
  DENSE_RANK() OVER (PARTITION BY year ORDER BY sum(award_amount) DESC ) AS RANK
FROM `acedata.acedata.acedata`
WHERE YEAR = 2023
GROUP BY main_discipline, year)


SELECT a.top_dis_2019, b.top_dis_2020, c.top_dis_2021, d.top_dis_2022, e.top_dis_2023 from `acedata.acedata.top_disciplines19` AS a JOIN `acedata.acedata.top_disciplines20` AS b ON (a.rank=b.rank) JOIN `acedata.acedata.top_disciplines21` as c ON (c.rank=b.rank) JOIN `acedata.acedata.top_disciplines22` as d ON (c.rank=d.rank) JOIN `acedata.acedata.top_disciplines23` as e ON (e.rank=d.rank)
WHERE a.rank < 4
```
</details>


**Which are the activities that received the most funding during the time period?**

![image](https://github.com/alccrts/SQL_Projects/assets/138128361/92e8db2d-fce6-4dab-ad55-10f5d631e5c4)
<details>
<summary>View SQL Query</summary>
<br>
  
```sql
SELECT activity_name, main_discipline, award_amount, year
FROM `acedata.acedata.acedata`
ORDER BY award_amount DESC
LIMIT 5
```
</details>


**Which area of Enland receives the least funding?**

![image](https://github.com/alccrts/SQL_Projects/assets/138128361/432d15ef-dc9f-4094-9157-59fd5eb88a09)
<details>
<summary>View SQL Query</summary>
<br>
  
```sql
WITH  `acedata.acedate.area19` AS(

SELECT 
  ace_area AS area2019,
  DENSE_RANK() OVER (PARTITION BY year ORDER BY sum(award_amount) ASC ) AS RANK
FROM `acedata.acedata.acedata`
WHERE YEAR = 2019
GROUP BY  ace_area, year), 

 `acedata.acedata.area20` AS(

SELECT 
  ace_area AS area2022,
  DENSE_RANK() OVER (PARTITION BY year ORDER BY sum(award_amount) ASC ) AS RANK
FROM `acedata.acedata.acedata`
WHERE YEAR = 2020
GROUP BY  ace_area, year), 

 `acedata.acedata.area21` AS(

SELECT 
  ace_area AS area2021,
  DENSE_RANK() OVER (PARTITION BY year ORDER BY sum(award_amount) ASC ) AS RANK
FROM `acedata.acedata.acedata`
WHERE YEAR = 2021
GROUP BY  ace_area, year), 

 `acedata.acedata.area22` AS(

SELECT 
  ace_area AS area2022,
  DENSE_RANK() OVER (PARTITION BY year ORDER BY sum(award_amount) ASC ) AS RANK
FROM `acedata.acedata.acedata`
WHERE YEAR = 2022
GROUP BY  ace_area, year), 

 `acedata.acedata.area23` AS(

SELECT 
  ace_area AS area23,
  DENSE_RANK() OVER (PARTITION BY year ORDER BY sum(award_amount) ASC ) AS RANK
FROM `acedata.acedata.acedata`
WHERE YEAR = 2023
GROUP BY  ace_area, year)


SELECT a.area2019, b.area2022, c.area2021, d.area2022, e.area23 from `acedata.acedate.area19` AS a JOIN `acedata.acedata.area20` AS b ON (a.rank=b.rank) JOIN `acedata.acedata.area21` as c ON (c.rank=b.rank) JOIN `acedata.acedata.area22` as d ON (c.rank=d.rank) JOIN `acedata.acedata.area23` as e ON (e.rank=d.rank)
WHERE a.rank < 2
```
</details>


***


### Creating a Dashboard. 

After exploring the data using SQL, I decided a dashboard would be a great way to present the data.  The final dashboard features various slicers which allows users to explore the data based on region, year and artistic discipline.  [You can view a preview of the dashboard here.](https://github.com/alccrts/Data_Visualizations/assets/138128361/4a1099a3-13d3-48b8-8e56-b8fb62359923)  I used the dashboard and SQL queries to put together some key findings at the top of this page.


