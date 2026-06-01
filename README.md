# AdventureWorks2025_Sales-Performance-by-Category-and-Territory_TSQL-PowerBI
Identify Product Categories that drive growth and Territories that are lagging behind in profit margins. Explain why this may be happening.



## Data Preparation: Microsoft SQL Server (T-SQL)
Create a VIEW (**Product Sales**) table, by joinning DIMENSION tables *Production.Product, Production.ProductCategory, Production.ProductSubcategory* and *Sales.SalesOrderHeader* with FACT table *Sales.SalesOrderDetail*. This is for performance optimisation, so that only the data required are imported into Power BI. The VIEW table is created in a way so that a column with calculated Profit is inculded in that table. Import into Power BI. Pull *Sales.SalesTerritory* table separately into Power BI (renamed to **Sales Territory**). 



## Modeling & Visualisation: Power BI
Create a Date table with columns for Date, Year and Month, as well as calculated **Fiscal Year** and  **Fiscal Quarter** (June to May for US fiscal year). Filter-out dates outside 2023-2025 fiscal years. Rename columns for clarity. Establish connections of **Product Sales** table with **Sales Territory** and **Date** table. Calculate **Average Monthly Profit** and **Average Monthly Revenue** (important for product comparison where individual products have no sales for certain months/years). Create DAX MEASURES of **Total Profit**, **Total Revenue** and **Profit Margin (%)**. Display information of Total or Monthly Revenue, Profit, Items Sold and Profit Margin in the form of a multi-row card (with conditional formatting). Display also Profit/Revenue, Items Sold and Average Product Cost per Category/Territory/time-period in the form of bar, bar-and-line and/or scatter charts. Produce two pages: one that can be filtered by Category and one that can be filtered by Territory.



<img width="824" height="461" alt="image" src="https://github.com/user-attachments/assets/91c539f0-6473-4668-ba67-8b029aab7cb5" />

<img width="1337" height="749" alt="image" src="https://github.com/user-attachments/assets/9c4ad553-d221-4e83-99f1-0baae01e16c4" />

<img width="1316" height="742" alt="image" src="https://github.com/user-attachments/assets/16c554c4-d531-4b3f-87d0-6345fa4d8cf9" />



## Key Findings: Power BI
  1. While Bikes are the ones that generate the highest revenue and profit, Asccessories are the one that have the highest Profit Margin, with significantly increased sales over the last recorded year.
  2. Australia is the territory displaying the highest profit, largely due to  high sales of Bikes.
  3. Underperforming territories, like US Northeast, Southeast and Central, tend to sell large amounts (>3,000 units) of Road Bikes at a loss.



<img width="1334" height="752" alt="image" src="https://github.com/user-attachments/assets/14662111-a842-4fdc-b8a5-3f6bfd6d2146" />

<img width="1314" height="738" alt="image" src="https://github.com/user-attachments/assets/0f895460-5949-4c7f-8e31-a742d8ee323b" />

<img width="1316" height="739" alt="image" src="https://github.com/user-attachments/assets/f8ee314c-af75-4dd0-8da1-5e91efbbfb57" />

## T-SQL Script & DAX formulas

<details>
<summary><b>Click to expand: Optimised T-SQL View Script</summary> 
  
```sql
CREATE VIEW Product_Sales AS
SELECT
Production.Product.Name AS 'Product Name',
Production.ProductCategory.Name AS 'Category',
Production.ProductSubcategory.Name AS 'Subcategory',
ROUND(Production.Product.StandardCost,2) AS 'Cost',
Sales.SalesOrderDetail.OrderQty AS 'Quantity',
ROUND(Sales.SalesOrderDetail.LineTotal,2) AS 'Product Revenue',
ROUND(Sales.SalesOrderDetail.LineTotal - Production.Product.StandardCost * Sales.SalesOrderDetail.OrderQty, 2) AS 'Product Profit',
CAST(Sales.SalesOrderHeader.OrderDate AS DATE) AS 'Order Date',
Sales.SalesOrderHeader.TerritoryID
FROM Sales.SalesOrderDetail
INNER JOIN Production.Product ON Sales.SalesOrderDetail.ProductID = Production.Product.ProductID
INNER JOIN Production.ProductSubcategory ON Production.Product.ProductSubcategoryID = Production.ProductSubcategory.ProductSubcategoryID
INNER JOIN Production.ProductCategory ON Production.ProductCategory.ProductCategoryID = Production.ProductSubcategory.ProductCategoryID
INNER JOIN Sales.SalesOrderHeader ON Sales.SalesOrderHeader.SalesOrderID = Sales.SalesOrderDetail.SalesOrderID
```

</details>

<details>
<summary><b>Click to expand: DAX formulas</summary> 

```dax
Fiscal Year = IF('Date'[Month]>=6, "FY" & 'Date'[Year] +1, "FY" & 'Date'[Year])

Fiscal Quarter = SWITCH('Date'[Month], 6, "FQ1", 7, "FQ1", 8, "FQ1", 9, "FQ2", 10, "FQ2", 11, "FQ2", 12, "FQ3", 1, "FQ3", 2, "FQ3", 3, "FQ4", 4, "FQ4", 5, "FQ4")

Total Profit = SUM('Product Sales'[Product Profit])

Total Revenue = SUM('Product Sales'[Product Revenue])

Average Monthly Profit = AVERAGEX(SUMMARIZE('Product Sales', 'Date’[Year], 'Date’[Month]), [Total Profit])

Average Monthly Revenue = AVERAGEX(SUMMARIZE('Product Sales', 'Date’[Year], 'Date’[Month]), [Total Revenue])
```

</details>
