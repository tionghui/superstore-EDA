# Introduction
This personal project is to apply SQL (Structured Query Language) and PowerBI to perform data analysis on a fictional superstore sales dataset.
The source of the dataset is from Tableau: https://public.tableau.com/app/learn/sample-data

## Importing Dataset
The superstore sales dataset from Tableau is in `.xlsx` extension. Create a copy in `.csv` extension such that the MySQL Table Data Import Wizard can recognize the file.  
The dataset consists of 10194 rows with no missing data, however it is advised to drop the `Product Name` column as it contains some unicode symbols that are not recognized by MySQL which will affect the import process.  
In MySQL, create a schema in called `project` and use the wizard to import the dataset as `superstore`.  
Note that the `Postal Code` column should be `text` not `int` because Canada's postal code is `varchar`, or you can drop this column as it will not be used in the analysis. 

```sql
CREATE DATABASE IF NOT EXISTS project;
```

Then, rename some of the columns to match the standard of SQL.

```sql
ALTER TABLE superstore
RENAME COLUMN `Ship Mode` TO Ship_Mode,
RENAME COLUMN `Customer Name` TO Customer_Name,
RENAME COLUMN `Country/Region` TO Country,
RENAME COLUMN `State/Province` TO State,
RENAME COLUMN `Sub-Category` TO Sub_Category;
```

## Analysis
First, check whether the dataset is imported correctly using the query

```sql
SELECT COUNT(*)
FROM superstore;
```

