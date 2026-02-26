# Introduction
This personal project is to apply SQL (Structured Query Language) and PowerBI to perform data analysis on a fictional superstore sales dataset.
The source of the dataset is from Tableau: https://public.tableau.com/app/learn/sample-data

## Importing Dataset
### MySQL
The superstore sales dataset from Tableau is in `.xlsx` extension. Create a copy in `.csv` extension such that MySQL Table Data Import Wizard can recognize the file.  
The dataset consists of 10194 rows with no missing data, however it is advised to drop the `Product Name` column as it contains some unicode symbols that are not recognized by MySQL which will affect the import process.  
In MySQL, create a schema called `project` and use the wizard to import the dataset as `superstore`.  
Note that the `Postal Code` column should be `text` not `int` because Canada's postal code is `varchar`, or you can drop this column as it will not be used in the analysis. 

```sql
CREATE DATABASE IF NOT EXISTS project;
```

Check whether the dataset is imported correctly using the query

```sql
SELECT COUNT(*)
FROM superstore;
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

### PowerBI
Microsoft PowerBI can import `.xlsx` directly and name the dataset as `superstore`. Make sure all the 10194 rows are imported in the Table view.

## Analysis
### Ship Mode
There are 4 shipping modes in the dataset, which are Standard Class, Second Class, First Class, and Same Day. By using the query

```sql
SELECT Ship_Mode, COUNT(*) AS Count
FROM superstore
GROUP BY Ship_Mode
ORDER BY Count DESC;
```

| Ship_Mode      | Count |
|----------------|-------|
| Standard Class | 6120  |
| Second Class   | 1979  |
| First Class    | 1548  |
| Same Day       | 547   |

In the data, all Standard Class start shipping at least 3 days after the order date and the largest delay is one week.  
The earliest shipping date for Second Class is 2 days after the order date and the latest is after 5 days.  
First Class can ship as early as on the next day of the order date and as late as 3 days after the order date.  
As the name suggest, Same Day ships within the order date with some delay to the next day.



