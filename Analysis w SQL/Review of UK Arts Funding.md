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

What was the top three disciplines?

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



