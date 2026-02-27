# Introduction
This personal project is to apply SQL (Structured Query Language) and Power BI to perform data analysis on a fictional superstore sales dataset.
The source of the dataset is from Tableau: https://public.tableau.com/app/learn/sample-data

## Importing Dataset
### MySQL
The superstore sales dataset from Tableau is in `.xlsx` format. Create a copy in `.csv` format so that MySQL Table Data Import Wizard can recognize it.  

The dataset consists of 10194 rows with no missing data. However, it is recommended to drop the `Product Name` column as it contains Unicode characters that are not recognized by MySQL, which may affect the import process.  

In MySQL, create a schema named `project` and use the wizard to import the dataset as `superstore`.  

Note that the `Postal Code` column should be defined as `text` rather than `int` because Canadian postal code are alphanumeric. Otherwise, this column can be dropped as it will not be used in the analysis. 

```sql
CREATE DATABASE IF NOT EXISTS project;
```

Check whether the dataset has been imported correctly using the following query:

```sql
SELECT COUNT(*)
FROM superstore;
```

Then, rename several columns to align with SQL naming conventions:

```sql
ALTER TABLE superstore
RENAME COLUMN `Ship Mode` TO Ship_Mode,
RENAME COLUMN `Customer Name` TO Customer_Name,
RENAME COLUMN `Country/Region` TO Country,
RENAME COLUMN `State/Province` TO State,
RENAME COLUMN `Sub-Category` TO Sub_Category;
```

### Power BI
Microsoft Power BI can import `.xlsx` files directly. Import and name the dataset as `superstore`. Make sure all the 10194 rows are successfully loaded by checking the Table view.

## Analysis
### Ship Mode
There are 4 shipping modes in the dataset: Standard Class, Second Class, First Class, and Same Day. The following query is used to obtain the count for each shipping mode:

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

From the data, all Standard Class orders begin shipping at least 3 days after the order date, with the longest delay being one week.  
For Second Class, the earliest shipping date is 2 days after the order date, while the latest is after 5 days.  
First Class orders can ship as early as at the next day after the order date and as late as 3 days afterwards.  
As the name suggests, Same Day orders ship on the order date, with some cases delay to the next day.

### Segment
There are 3 segments in the dataset: Consumer, Corporate, and Home Office. A similar query can be used to obtain the count for each segment:

```sql
SELECT Segment, COUNT(*) AS Count
FROM superstore
GROUP BY Segment
ORDER BY Count DESC;
```

| Segment     | Count |
|-------------|-------|
| Consumer    | 5281  |
| Corporate   | 3090  |
| Home Office | 1823  |

The majority of orders come from the Consumer segment, followed by Corporate and Home Office.

### Country and State

