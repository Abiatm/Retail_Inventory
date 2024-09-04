
# Retail_Inventory_Report
## Table of contents 
- [Project Overview](#project-overview)
- [Problem Statement](#problem-statement)
- [Data Source](#data-source)
- [Tools Used](#tools-used)
- [Data filtering_OR_Data Analysis](#Data_filtering_OR-Data_analysis)
- [Query](#Query)
- [Results](#results)
- [Image](#Image)
- [Story_telling](#Story_telling)
- [Recommendations](#recommendations)
### Project Overview
A retail store with arrays of products grouped by their category ranging from electronics to Accessories with many suppliers and warehouse availability at many locations.This store entertains different kinds of customers and due to the need for easy accesibility of their products and documentation of their sales and other neccessary information this database creation was made to ensure that the data are stored in one database for easy analysis.

### Problem Statement
- Fetch products with the same price.
- Find the second highest priced product and its category.
- Get the maximum price per category and the product name.
- Supplier-wise count of products sorted by count in descending order.
- Fetch only the first word from the ProductName and append the price.
- Fetch products with odd prices.
- Create a view to fetch products with a price greater than $500.
- Create a procedure to update product prices by 15% where the category is 'Electronics' and the supplier is not 'SupplierA'.
- Create a stored procedure to fetch product details along with their category, supplier, and warehouse location, including error handling.
### Tools Used
- SSMS
- MYSQL
### Data filtering_OR_Data Analysis
- In ordeer to fetch products with the same price,i have to filter the retail product data with an 'IN" command together with other commands like the (GROUP BY and HAVING clause) to select same price which inthe data given happen to be #1500 as the only one with same price
- To find the second highest priced product and its category,the highest priced product was jumped using the "OFFSET" command and The second highest was selected using the " FETCH NEXT 1 ROW ONLY"to take the highest next_row which is the second_max
- Get the maximum price per category and the product name, to obtain this 'DENSE_RANK" was used to obtain the ranking and after which the result was ordered by the price ,partition by CategoryID and the rankings with "1's"happen to be the maximun for each category and was selected
- Supplier-wise count of products sorted by count in descending order,here the product and the supplier tablewas joined and grouped by SupplierName 
- Fetch only the first word from the ProductName and append the price,here  the 'CASE" command was used to obtain the required result due to differences in the structure of the ProductName
- To fetch products with odd prices,modulus division was used.
- Create a view to fetch products with a price greater than $500,the create view command was used to obtain stored of data on seperate table
- Create a procedure to update product prices by 15% where the category is 'Electronics' and the supplier is not 'SupplierA',here create view and update was the major command used to obtain the result needed.
- Create a stored procedure to fetch product details along with their category, supplier, and warehouse location, including error handling,here create view and error handling commands was used to capture abnormal inputs.
### Query
```

(1) products with same price
SELECT * FROM  [RetailInventor].[dbo].[Products]
WHERE Price IN  --(#1500 -only same price and to be gotten from querry below)
(SELECT Price FROM [RetailInventor].[dbo].[Products]
GROUP BY Price
HAVING COUNT(Price)>1)


 (2) -Second highest price product and it's category
 
 SELECT [ProductName],[CategoryID],[Price] FROM [RetailInventor].[dbo].[Products]
 ORDER BY Price DESC
 OFFSET 2 ROW     --jumps the two highest rows
 FETCH NEXT 1 ROW ONLY; --takes the highest next_row which is the second_max

 (3) Get the maximum price per category and the product name
 WITH DENSE_RANKING AS(
 SELECT *,DENSE_RANK() OVER(PARTITION BY CategoryID ORDER BY Price DESC ) RANKINGS FROM [RetailInventor].[dbo].[Products])
 SELECT [ProductName],[CategoryID],[Price] Max_price_per_category FROM DENSE_RANKING 
WHERE RANKINGS=1
(4)Supplier-wise count of products sorted by count in descending order.
SELECT COUNT(DISTINCT P.[ProductName])  Count_P,S.SupplierName
  FROM [RetailInventor].[dbo].[Products] P
  LEFT JOIN [RetailInventor].[dbo].[Supplier] S
ON P.SupplierID=S.SupplierID
GROUP BY S.SupplierName


(5) FETCH ONLY THE FIRST WORD FROM THE PRODUCTNAME AND APPEND THE PRICE
SELECT 
CASE 
   -- The '% %' the shift btw the % shows conditions like first and last word then use of replace with parsename setat 2 will extract the first ones
   WHEN ProductName LIKE '% %' THEN PARSENAME(REPLACE(ProductName,' ','.'),2) + '_' +  CAST(Price as NVARCHAR)
   
   ELSE PARSENAME(ProductName,1) + '_' +    CAST(Price as NVARCHAR)  
   END  ProdName_Price
   FROM [RetailInventor].[dbo].[Products]

   
(6)-- FETCH PRODUCTS WITH ODD PRICES
SELECT * FROM [RetailInventor].[dbo].[Products] 
  WHERE Price%2 !=0

  (7)--- CREATE A VIEW TO FETCH PRODUCTS WITH SAME PRICE GREATER THAN $500
  CREATE VIEW vw_sameSalaries AS
  SELECT ProductName,Price FROM [RetailInventor].[dbo].[Products] 
  WHERE Price >500 

  (8) create a procedure to update product price by 15% where the category is 'Electronics' and the suplier name is not 'SupplierA'
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO

CREATE PROCEDURE sp_PerIncreases15
AS
BEGIN
UPDATE  P
 SET P.Price=1.15 * P.Price FROM [RetailInventor].[dbo].[Products] P
LEFT JOIN [RetailInventor].[dbo].[Category] C
ON P.CategoryID=C.CategoryID
LEFT JOIN [RetailInventor].[dbo].[Supplier] S
ON S.SupplierID=P.SupplierID 
 
 WHERE C.CategoryName='Electronics' AND P.ProductName NOT IN ('SupplierA')



END
GO

(9) CREATE A STORE PROCEDURE TO FETCH PRODUCT DETAILS ALONG WITH THEIR CATEGORY<SUPPLIER,AND WAREHOUSE LOCATION INCLUDING ERROR HANDLING
CREATE PROCEDURE sp_PerIncrease15I
AS
BEGIN
BEGIN TRY
SELECT P.ProductID, P.ProductName,C.CategoryName,S.SupplierName,W.Locations
 FROM [RetailInventor].[dbo].[Products] P
LEFT JOIN [RetailInventor].[dbo].[Category] C
ON P.CategoryID=C.CategoryID
LEFT JOIN [RetailInventor].[dbo].[Supplier] S
ON S.SupplierID=P.SupplierID 
LEFT JOIN [RetailInventor].[dbo].[Warehouse] W
ON S.SupplierID=W.WarehouseID 
   END TRY
   BEGIN CATCH
   DECLARE @ERRORMESSAGE NVARCHAR(4000)
    
   SET @ERRORMESSAGE =ERROR_MESSAGE()
   
 --  RAISEERROR (@ERRORMESSAGE,16,1);
   END CATCH
   END
```
### Results

- Product Diversity and Pricing:
The store offers a variety of products across four categories: Electronics, Appliances, Kitchenware, and Accessories.
Prices range from affordable items like headphones to more expensive products like laptops and TVs.
The most expensive product is a TV priced at 1500, while headphones are the most affordable at 100.
- Supplier Relationships:
The store sources products from a network of 10 suppliers, each specializing in different product categories.
SupplierA supplies the most products, followed by SupplierB and SupplierC.
- Price Analysis:
There are products with identical prices,the second-highest priced product is a TV, followed by a washing machine.
Product Categories and Performance:
Electronics is the most popular category, followed by Appliances.
Kitchenware and Accessories have lower sales volumes.

### Image

![RETAIL_PORT](https://github.com/user-attachments/assets/65120631-2be0-4c96-a7d8-b424ff6cbd0d)

### Story_telling
Imagine a customer walking into the store, searching for a new laptop.With the inventory system we can quickly find the desired product of need and check the stock level and inform the customer about the availability of the laptop. As the customer prepares to make a purchase, the system automatically updates the inventory records, ensuring that the laptop is accurately accounted for.
Behind the scenes, the sales data is analysed to identify trends and optimize inventory levels.
### Recommendations
- Awareness programs should be done.
- more locations should be provided to enable easy accesibility
- price evaluation and supplier satisfaction or need


