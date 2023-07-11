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

* I also noticed that `AccessionYear` is recorded as an INTERGER type rather than the YEAR data type.  However, after researching, I discovered BigQuery doesn't support the data type YEAR.  It only supports DATETIME, which is not suitable here. 

Change Object_NUmber to object code.  
Medium - seperate different mediums into medium 1, 2 , 3
check for nulls in all columns and decide what to do. 
change accession to YEAR, but big query doesn't support YEAR type, so leave it as INT.
Select 'distinct' department to make sure there are no duplicate departments 
Select 'distinct' object_name to see if there are similar / duplicate object names. 
Does object_name describe well the attribute
Object name = object title 
culture - null should go to 'culture unknown' 
change some of the categories of culture - e.g native american: xxx
check then drop null columns 
no medium available - what to do?
replace N/A with NULL 
standardise upper / lower case.
TRIM unnecessary white space
decide what to do with missing data > won't delete records with missing data because missing data wouldn't affect our analysis and also it's not worth the loss of info.  
sort the columns like accessionyear to look for outliers. 


What percentage of the works on in the public domain? 

