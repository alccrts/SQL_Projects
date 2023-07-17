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

<details>
<summary>View SQL Query</summary>
<br>
  
```sql
  test
```
</details>
