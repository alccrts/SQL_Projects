## Data Cleaning Practice
In this project, I take a dataset from the Metropolitan Museum of Art in New York.  I import it to BigQuery to clean and analyse the data using SQL.  

![GettyImages-1180468555-1024x683](https://github.com/alccrts/SQL_Projects/assets/138128361/8aa2c1de-6967-4d97-8d6f-abec6afe86b6)

**Step 1: Import the data**

The dataset I will be using is published from The United States Of America, where [The Metropolitan Museum of Art provides select datasets](https://github.com/metmuseum/openaccess/) of information on more than 470,000 artworks in its Collection for unrestricted commercial and noncommercial use.  The dataset contains over 480,000 records and over 50 columns per record.  It includes variables such as the name of the object, the artist name, the object number, the dimensions of the object and more.  I downloaded the data as CSV file from Github and previewed it in Excel.  A snapshot of the data can be seen below.   

![image](https://github.com/alccrts/SQL_Projects/assets/138128361/d6746e89-2516-4e51-a106-0406b26c9c41)

I uploaded the file to BigQuerey to begin the cleaning process but due to the data being so messy, I received two error messages that prevented the file from being uploaded.  Below I have described to errors and the solutions that I found on Stack Overflow:

  * "Error: Missing close double quote (") character" - this was returned because the data contains newline characters, but the BigQuery default is to assume that CSV data does not contain newlines.  The solution is to check allow newlines in advanced options `Advanced Options -> Quoted New lines`
  * "Error: could not parse as INT64 for field XXX Unable to parse file" - this was returned because I had selected the option to automatically detected the schema of the dataset, but because the data is unclean, there are many columns that include inconsistent data types.  The solution was to allow for some errors so that the file could be uploaded `Advanced Options -> Number of Errors Allowed: 100`
