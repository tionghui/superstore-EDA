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
There are 2 countries in the dataset: United States and Canada. The counts of countries and their respective states are obtained using the following queries:

```sql
SELECT Country, COUNT(*) AS Count
FROM superstore
GROUP BY Country;

SELECT Country, State, COUNT(*) AS Count
FROM superstore
GROUP BY Country, State
ORDER BY Country, Count;
```

![Map](assets/superstore_map.png)

The map visualization in Power BI shows that California has the highest number of orders (2001 orders), followed by New York (1128 orders).

### Category and Sub-Category
There are 3 categories in the dataset: Furniture, Office Supplies, and Technology. Each category is further divided into sub-categories. For example, Technology includes Copiers, Machines, Accessories, and Phones. The following query is used to obtain the count of each Category and its sub-categories:

```sql
WITH category_count AS (
	SELECT Category, Sub_Category, COUNT(*) AS Count
	FROM superstore
	GROUP BY Category, Sub_Category
)
SELECT Category, Sub_Category, Count, SUM(Count) OVER (PARTITION BY Category) AS Category_Total
FROM category_count
ORDER BY Category, Count;
```

| Category        | Sub_Category | Count | Category_Total |
|-----------------|--------------|-------|----------------|
| Furniture       | Bookcases    | 232   | 2201           |
| Furniture       | Tables       | 326   | 2201           |
| Furniture       | Chairs       | 634   | 2201           |
| Furniture       | Furnishings  | 1009  | 2201           |
| Office Supplies | Supplies     | 192   | 6128           |
| Office Supplies | Fasteners    | 229   | 6128           |
| Office Supplies | Envelopes    | 256   | 6128           |
| Office Supplies | Labels       | 368   | 6128           |
| Office Supplies | Appliances   | 474   | 6128           |
| Office Supplies | Art          | 821   | 6128           |
| Office Supplies | Storage      | 856   | 6128           |
| Office Supplies | Paper        | 1384  | 6128           |
| Office Supplies | Binders      | 1548  | 6128           |
| Technology      | Copiers      | 70    | 1865           |
| Technology      | Machines     | 117   | 1865           |
| Technology      | Accessories  | 775   | 1865           |
| Technology      | Phones       | 903   | 1865           |

Office Supplies has the highest number of orders (6128 orders), followed by Furniture (2201 orders) and Technology (1865 orders). Among the sub-categories, Furnishings has the highest number of orders in Furniture (1009), Binders in Office Supplies (1548), and Phones in Technology (903).

### Customer with the Highest Number of Orders
Note that some customers may share the same name, so it is recommended to use their IDs to differentiate them. In SQL, the following query is used to find the number of orders made by each customer:

```sql
SELECT Customer_Name, COUNT(*) AS Orders
FROM superstore
GROUP BY `Customer ID`, Customer_Name
ORDER BY Orders DESC;
```

The result shows that `William Brown` has the highest number of orders (41). Next, the following query is used to show his orders by category and sub-category:

```sql
SELECT Customer_Name, Category, Sub_Category, COUNT(*) AS Count
FROM superstore
WHERE Customer_Name = "William Brown"
GROUP BY Category, Sub_Category
ORDER BY Category;
```

| Customer_Name | Category        | Sub_Category | Count |
|---------------|-----------------|--------------|-------|
| William Brown | Furniture       | Chairs       | 1     |
| William Brown | Furniture       | Furnishings  | 5     |
| William Brown | Furniture       | Tables       | 2     |
| William Brown | Office Supplies | Appliances   | 3     |
| William Brown | Office Supplies | Art          | 2     |
| William Brown | Office Supplies | Binders      | 12    |
| William Brown | Office Supplies | Envelopes    | 1     |
| William Brown | Office Supplies | Fasteners    | 2     |
| William Brown | Office Supplies | Paper        | 3     |
| William Brown | Office Supplies | Storage      | 2     |
| William Brown | Technology      | Accessories  | 3     |
| William Brown | Technology      | Machines     | 1     |
| William Brown | Technology      | Phones       | 4     |

In Power BI, to differentiate customers with the same name, a new column can be calculated using the following DAX code:

```DAX
Flag = IF(CALCULATE(DISTINCTCOUNT(superstore[Customer ID]), ALLEXCEPT(superstore, superstore[Customer Name])) > 1, 1, 0)
```

This `Flag` column can later be used as a legend to distinguish customer names that have only one ID from those shared by multiple IDs. In this dataset, the name `Harry Olson` is shared by 5 different customers.

### Sum of Profit
The sum of profit is investigated against customers, segments, states, and categories. The query used is similar, with one of example given below:

```sql
SELECT Customer_Name, SUM(Profit) AS Profits
FROM superstore
GROUP BY Customer_Name
ORDER BY Profits DESC;
```

| Customer_Name          | Profits             |
|------------------------|---------------------|
| Tamara Chand           | 8981.3239           |
| Raymond Buch           | 6976.095900000001   |
| Sanjit Chand           | 5757.411899999997   |
| Hunter Lopez           | 5622.4292000000005  |
| Adrian Barton          | 5444.8054999999995  |
...
| Henry Goldwyn          | -2797.9635          |
| Sharelle Roach         | -3333.9144          |
| Luke Foster            | -3583.977           |
| Grant Thornton         | -4108.6589          |
| Cindy Stewart          | -6626.3895          |

Although `William Brown` has the highest number of orders, he didn't generate the most profit to the superstore. In fact, the sum of profit contributed by `William Brown` is $692.39.  

From the result, `Tamara Chand` generated the most profit ($8981.3239) which only has 12 orders. 10 orders are Office Supplies and the remaining 2 are Technology, however, the Office Supplies orders only generate $397.09 and Technology orders generate the remaining $8584.2339.

On the other hand, `Cindy Stewart` generated the largest loss ($6626.3895) with 9 orders. 6 orders are Office Supplies and the others are Technology. Similarly, the Office Supplies orders cost $228.49 and the Technology orders cost $6397.8995.

Based on the two results, it seems that Technology orders can either bring large profit or huge loss. 
