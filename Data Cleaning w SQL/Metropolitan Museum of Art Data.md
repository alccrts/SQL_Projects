## Data Cleaning Practice
In this project, I have taken a dataset from the Metropolitan Museum of Art in New York.  I have imported it into BigQuery to clean and analyse using SQL.  I have then created a visual using Tableau.  

![GettyImages-1180468555-1024x683](https://github.com/alccrts/SQL_Projects/assets/138128361/8aa2c1de-6967-4d97-8d6f-abec6afe86b6)

## Step 1: Import the data

The dataset I will be using is published by The Metropolitan Museum of Art, which provides [select datasets](https://github.com/metmuseum/openaccess/) of information on more than 470,000 artworks in its collection.  The dataset is licensed for unrestricted commercial and noncommercial use and contains over 480,000 records and over 50 columns.  It includes variables such `object_name`, `artist_name`, `object_ID` and `dimensions`.  I downloaded the data as a CSV file from Github and previewed it in Excel.  A snapshot of the data can be seen below.   

![image](https://github.com/alccrts/SQL_Projects/assets/138128361/d6746e89-2516-4e51-a106-0406b26c9c41)

I uploaded the file to BigQuerey to begin the cleaning process.  Due to the data being so messy, I received two error messages that prevented the file from being uploaded:

  * <em>"Error: Missing close double quote (") character"</em> - this was returned because the data contains newline characters, but the BigQuery default is to assume that CSV data does not contain newlines.  The solution is to 'allow newlines' in advanced options `Advanced Options -> Quoted New lines`
  * <em>"Error: could not parse as INT64 for field XXX Unable to parse file"</em> - this was returned because I had selected the option for BiqQuery to automatically detected the data types/schema, but because the data is unclean, there are many columns with inconsistent data types.  The solution was to allow for some errors so that the file could be uploaded `Advanced Options -> Number of Errors Allowed: 100`

Once these errors were corrected, the dataset successfully imported.  A snapshot of the schema can be viewed below. 

![image](https://github.com/alccrts/SQL_Projects/assets/138128361/8b177a1d-9bf5-4656-b696-c78b3e433383)

## Step 2: Clean the data.

My next step was to preview the data using the SQL query below.  I could then review the columns and consider where to begin the cleaning process.  It was then important to understand the purpose of my project and what I was cleaning the data for as this would have an impact of the data process.  I had decided to use the data to create a simple visualisation that summarises the contents of the museum and so I would bear this in mind as I clean and analyse the data. 

```sql
SELECT *
FROM `polar-fulcrum-392507.met_art_data.met_art_data` 
LIMIT 1000
```

**Checking for duplicates**

My first cleaning activity was to check for duplicates.  I could see that all the records were uniquley numbered with an `object_id`, so I decided to compare the **COUNT** of `object_id` with the distinct **COUNT** of `object_id`.  
![image](https://github.com/alccrts/SQL_Projects/assets/138128361/ff4636ac-2a6c-4741-ae9f-a2a99a05e5c1)

```sql

SELECT 
      COUNT (object_id) AS count_object_id,
      COUNT (DISTINCT object_id) AS distinct_count_object_id
FROM `polar-fulcrum-392507.met_art_data.met_art_data`

```
I was pleased to see the count results were the same, suggesting there were no dupicate records.  However, to verify this, I repeated this step with `object_number`, which revealed a discrepency between the count, meaning there could be duplicate rows.  

![image](https://github.com/alccrts/SQL_Projects/assets/138128361/cd698353-5ed9-4d67-a27b-888f4a7e10ec)

```sql
SELECT 
      COUNT (Object_Number) AS count_object_number,
      COUNT (DISTINCT Object_Number) AS distinct_count_object_number
FROM `polar-fulcrum-392507.met_art_data.met_art_data`
```

Next, I used an embedded **SELECT** statement to reveal the records which shared an `object_number`.  I could clearly see from these results that although they shared the same `object_number`, the other details of the objects were different and were therefore not duplicate records.  I considerd using the **UPDATE** option to edit these records so that they had unique object numbers, but given that I don't know how or why object numbers are assigned to objects, I decided it was best not to change them.  Besides, I was now satisifed that each record had a unique `object_id` and that there were no duplicate records.  

```sql

SELECT * 

FROM `polar-fulcrum-392507.met_art_data.met_art_data`

WHERE object_number IN (

SELECT 
      object_number
FROM `polar-fulcrum-392507.met_art_data.met_art_data`
GROUP BY (object_number)
HAVING COUNT (object_number) > 1

) 

ORDER BY object_number

```
![image](https://github.com/alccrts/SQL_Projects/assets/138128361/18dd7636-36a9-4421-8f69-ccbdecc809ad)


**Type Conversion**

My next cleaning activty was to check that the appropriate data type was identified for each column.  As the majority of the data is descriptive, most columns have been correctly identified by BigQuery as strings.  However, I did notice the following issues:

* `Object_Number` is listed as a string because the records containg periods and letters.  However, the column name suggests it's an interger field.  I therefore renamed the column to `object_code` using the query below to better reflect the contents of the column.  

```sql
ALTER TABLE `polar-fulcrum-392507.met_art_data.met_art_data`
RENAME COLUMN object_number TO object_code
```

* I also noticed that `AccessionYear` is recorded as an INTERGER type rather than the YEAR.  After searching online, I learnt BigQuery doesn't support the data type YEAR.  It only supports DATETIME, which is not suitable here. I therefore made no changes to this column.  

**Formatting Categories**

My next task was to check that all the categorical fields had consistent entries and that labelling errors had not created any inconsistencies.  I reviewed the different categories for `department` with the query below.  The results showed 19 unique departments with no labelling errors.  I also searched for NULL values for `department` - there weren't any. 

```sql 
SELECT DISTINCT department
FROM `polar-fulcrum-392507.met_art_data.met_art_data`; 

SELECT * 
FROM `polar-fulcrum-392507.met_art_data.met_art_data` 
WHERE department IS NULL 
```

![image](https://github.com/alccrts/SQL_Projects/assets/138128361/c5125e63-047a-41af-825c-28f740256db5)

I repeated this for the `object_name` column.  The results showed similar categories such as 'Platter' and 'Meat Platter' and 'Shoe Buckle' and 'Buckle'.  If possible, I would consults with the owner of the dataset to confirm whether or not the distinction between these categories were important.  For the sake of this project, I have decided they are not important to so I have changed combined the similar categories using the following query.

```sql

UPDATE `polar-fulcrum-392507.met_art_data.met_art_data`
SET object_name = 'Buckle'
WHERE object_name = 'Shoe buckle' ; 

UPDATE `polar-fulcrum-392507.met_art_data.met_art_data`
SET object_name = 'Platter'
WHERE object_name = 'Meat platter'
```

For tidiness, I converted all `object_name` to title case with the following query.

```sql
UPDATE `polar-fulcrum-392507.met_art_data.met_art_data`
SET object_name = INITCAP(object_name)
WHERE object_name IS NOT NULL
```

Cleaning the field `Country` was the most complex task throughout this cleaning process.  There were over 950 distinct values for Country.  There were several complications such as multiple variations of the same country, spelling mistakes, countries broken down into regions, city names instead of countries, multiple countries included in the same field, with different types of delimiters used.  It took several hours to clean the list.  Here is a summary of some of the SQL steps taken:

```sql
UPDATE `polar-fulcrum-392507.met_art_data.met_art_data` -- standaradise variations of United States to USA.  
SET Country = 'USA'
WHERE Country = 'United States' OR  Country = 'United States|United States' OR Country ='United States|United States|United States' OR Country = 'U.S.A.' OR Country ='United States of America' OR Country = 'United States ()' OR Country = 'United States()' OR Country = 'US'

UPDATE `polar-fulcrum-392507.met_art_data.met_art_data` -- standaradise variations of UK.
SET Country = 'UK'
WHERE Country = 'United Kingdom' OR  Country = 'United Kingdom '

UPDATE `polar-fulcrum-392507.met_art_data.met_art_data` -- standardise | delimiters to only commas
SET Country = REPLACE(Country, '|', ',')
WHERE Country LIKE '%|%' OR Country LIKE '%|'

UPDATE `polar-fulcrum-392507.met_art_data.met_art_data` -- standardise 'or' delimiter to only commmas
SET Country = REPLACE(Country, 'or', ',')
WHERE Country LIKE '%or%'

UPDATE `polar-fulcrum-392507.met_art_data.met_art_data` -- remove probably
SET Country = REPLACE(Country, 'probably', '')
WHERE Country IS NOT NULL

UPDATE `polar-fulcrum-392507.met_art_data.met_art_data` -- remove possibly
SET Country = REPLACE(Country, 'possibly', '')
WHERE Country IS NOT NULL

UPDATE `polar-fulcrum-392507.met_art_data.met_art_data` -- replace Ecaud with Ecuador
SET Country = REPLACE(Country, 'Ecuad', 'Ecuador')
WHERE Country IS NOT NULL

UPDATE `polar-fulcrum-392507.met_art_data.met_art_data` -- remove present-day
SET Country = REPLACE(Country, 'present-day', ' ')
WHERE Country IS NOT NULL

UPDATE `polar-fulcrum-392507.met_art_data.met_art_data` --  remove formerly
SET Country = REPLACE(Country, "f'merly", ' ')
WHERE Country IS NOT NULL

UPDATE `polar-fulcrum-392507.met_art_data.countries` -- remove country variations
SET Country1 = REPLACE(Country1, 'South France', "France")
WHERE Country1 IS NOT NULL;

CREATE TABLE `polar-fulcrum-392507.met_art_data.countries` -- once all delimiters were changed to a comma, i could then seperate multiple columns into multiple countries in a seperate table
AS 
select
split(Country, ',')[safe_offset(0)] as country1, 
split(Country, ',')[safe_offset(1)] as country2, 
split(Country, ',')[safe_offset(2)] as country3, 
split(Country, ',')[safe_offset(3)] as country4
FROM `polar-fulcrum-392507.met_art_data.met_art_data-2023-07-12T14_00_02_restore`

UPDATE `polar-fulcrum-392507.met_art_data.countries` -- Trim white space
SET Country1 = Trim(Country1)
WHERE Country1 IS NOT NULL;
```


**Dropping Columns** 

Many of the columns were not necessary to my analysis so I decided to drop them from the table.  These fields also contained no data so I was confident with the decision to drop them. 

```sql
ALTER TABLE `polar-fulcrum-392507.met_art_data.met_art_data`
DROP COLUMN Portfolio, 
DROP COLUMN Artist_RolE,
DROP COLUMN Constituent_ID, 
DROP COLUMN Artist_Display_Name, 
DROP COLUMN Artist_Display_Bio,
DROP COLUMN Artist_Suffix, 
DROP COLUMN Artist_Nationality,
DROP COLUMN Artist_Begin_Date,
DROP COLUMN Artist_End_Date,
DROP COLUMN Artist_Gender,
DROP COLUMN Artist_ULAN_URL,
DROP COLUMN Artist_Wikidata_URL,
DROP COLUMN Artist_Prefix,
DROP COLUMN Artist_Alpha_Sort,
DROP COLUMN Tags_Wikidata_URL, 
DROP COLUMN Tags_AAT_URL,
DROP COLUMN Tags, 
DROP COLUMN Metadata_Date, 
DROP COLUMN Rights_and_Reproduction,
DROP COLUMN Classification,
DROP COLUMN Region
```

There were several fields that could be relevant to my analysis but I noticed that most of the data was NULL.  I ran the below query to calculate the percentage of NULL values in these columns.  Given that so many were values were null (50 - 95%), I decided to drop these columns as well because they would not tell me much about the overall collection. 

```sql
SELECT 

ROUND(100*( SUM (CASE WHEN Period IS NULL then 1 else 0 end)/ COUNT(*)),0) AS period_nulls,

ROUND(100*( SUM (CASE WHEN Culture IS NULL then 1 else 0 end)/ COUNT(*)),0) AS culture_nulls,

ROUND(100*( SUM (CASE WHEN Dynasty IS NULL then 1 else 0 end)/ COUNT(*)),0) AS dynasty_nulls,

ROUND(100*( SUM (CASE WHEN Region IS NULL then 1 else 0 end)/ COUNT(*)),0) AS region_nulls,

FROM `polar-fulcrum-392507.met_art_data.met_art_data`
```

![image](https://github.com/alccrts/SQL_Projects/assets/138128361/d166473f-6459-4966-aa47-df484bf73c52)


```sql
ALTER TABLE `polar-fulcrum-392507.met_art_data.met_art_data`
DROP COLUMN Period, 
DROP COLUMN Culture,
DROP COLUMN Dynasty, 
DROP COLUMN Region;
```

**Looking for outliers** 

I decided to check for outliers in column `AccessionYear` by ordering the columns and showing the first and last 50 results.  The earliest accesstion year was 1870.  This was when the museum was founded and so would not be an outlier.  The latest accession was in 2023, which is also unlikely to be an outlier.  

```sql
SELECT * FROM `polar-fulcrum-392507.met_art_data.met_art_data`
WHERE AccessionYear IS NOT NULL
ORDER BY AccessionYear ASC
LIMIT 50 ;

SELECT * FROM `polar-fulcrum-392507.met_art_data.met_art_data`
ORDER BY AccessionYear DESC
LIMIT 50 
```

**Missing Data**

I have already dropped several columns with missing data because those columns were not relevant to my upcoming analysis.  However, there is still missing data in columns which are relevant to my analysis such as `AccessionYear`.  I have calculated the percentage of nulls in `AccessionYear` with the query below.  Less than 1% of the records didn't have an entry for `AccessionYear`, as this was such a small amount, I decided to delete these records. 3,837 results were deleted.

```sql
SELECT 

ROUND(100*( SUM (CASE WHEN AccessionYear IS NULL then 1 else 0 end)/ COUNT(*)),2) AS AccessionYear_nulls,

FROM `polar-fulcrum-392507.met_art_data.met_art_data`
```
```sql
DELETE FROM `polar-fulcrum-392507.met_art_data.met_art_data`
WHERE AccessionYear IS NULL
```






















What percentage of the works on in the public domain? 

