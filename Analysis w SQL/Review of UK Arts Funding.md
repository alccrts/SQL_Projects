## UK Arts Funding Analysis using SQL

In this project, I have taken data from Arts Council England to review their sfunding patterns across five years.  I have used Excel, BigQuery, Tableau and Pitch to present my analysis.  

![ArtsCouncil-2014070211374640](https://github.com/alccrts/SQL_Projects/assets/138128361/7bf3674b-c106-4c37-a7ca-c6f407c81554)

The Arts Council England is UK's national development agency for creativity and culture.  The council distributes public money from the UK Government as well as grants funded by The National Lottery to support individual practitioners, communities and cultural organisations with projects that focus on Combined Arts including festivals and carnivals, Dance, Libraries, Literature, Museums, Music, Theatre or Visual Arts.  This analysis will review The National Lottery Grants given during 2018 - 2023.  You can view the [Arts Council Website](https://www.artscouncil.org.uk/ProjectGrants) for more information.   

### Preparing the Data

The data is published by the council - [you can find the data here](https://www.artscouncil.org.uk/research-and-data/our-data).   Preparing the data required little work, as it was already well processed by the council.  I could use Excel to make some small changes as the quantity of data was not too large (around 18,000 records).  My main tasks were:
* combine the annual data of five years into a single CSV file 
* removed currency formatting
* changed all text to lower case
* upload to BigQuery (preview below)

![image](https://github.com/alccrts/SQL_Projects/assets/138128361/039e7763-8ecf-43af-b698-9d8df64166c5)

### Exploring the Data

Below is a sample of the questions I devised in order to explore the data.  The SQL queries I wrote to generate answers can be viewed by clicking the drop down arrows.  

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

**Which are top three locations that receive the most funding each year?**
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


What was the top three activities that received the  most funding?

which local authority received the least?

which region received the most funding? 

which month of year pays out the most funding?  

Which region had the most big budget awards over 100k. 

Which authority had the most small budget awards under 5k. 

is the majority of funding given as smaller or larger amounts? 

combine last years data > more or less or the same total funding?

which region saw the biggest change in funding 22 - 23?

which discipline saw change? 

did any recepients receive an award 5 years in a row?  



