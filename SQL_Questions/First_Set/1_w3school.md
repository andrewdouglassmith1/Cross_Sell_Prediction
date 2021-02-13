# Part 1: W3Schools SQL Lab 

*Introductory level SQL*

--

This challenge uses the [W3Schools SQL playground](http://www.w3schools.com/sql/trysql.asp?filename=trysql_select_all). Please add solutions to this markdown file and submit.

1. Which customers are from the UK? 
   
   - Customer ID: 4, 11, 16, 19, 38, 53, 72
   
2. What is the name of the customer who has the most orders?
   
   - Customer 20 with 10 orders 
   
3. Which supplier has the highest average product price?

   - Supplier Name: Aux joyeux ecclésiastiques
   - SupplierID: 18
   - Average Price: 140.75

   ```sql
   WITH Products_Suppliers AS(
   	Select *
   	FROM [Suppliers]
   		JOIN [Products]
           	ON Products.SupplierID = Suppliers.SupplierID)
   SELECT SupplierName, AVG(Price) 
   FROM Products_Suppliers
   GROUP BY SupplierName
   ORDER BY AVG(Price) DESC;
   ```

4. How many different countries are all the customers from? (*Hint:* consider [DISTINCT](http://www.w3schools.com/sql/sql_distinct.asp).)

   ```sql
   SELECT COUNT(DISTINCT(Country)) 
   FROM [Customers];
   ```

   - 21 different countries

5. What category appears in the most orders?

   ```sql
   WITH OrderDetailsCategory AS 			
   	(SELECT *
       FROM[Products]
       	JOIN [OrderDetails]
           	ON OrderDetails.ProductID = Products.ProductID)
   Select CategoryID, COUNT(CategoryID)
   FROM OrderDetailsCategory
   GROUP BY CategoryID
   ORDER BY COUNT(CategoryID) DESC;
   ```

   - Category ID 4 (Dairy Products), appears the most times 

6. What was the total cost for each order?

   - See code below
   - Since no unit cost is listed I assumed "price" was synonymous with cost

   ```SQL
   WITH OrderDetailsPrices AS (
   	Select *,Price*Quantity AS OrderCost
       FROM [OrderDetails]
       	JOIN [Products]
           	ON OrderDetails.ProductID = Products.ProductID
       GROUP BY OrderID)
   Select * 
   FROM OrderDetailsPrices;
   ```

7. Which employee made the most sales (by total price)?

   ```sql
   WITH OrderDetailsPrices AS (
   	Select *,Price*Quantity AS OrderCost
       FROM [OrderDetails]
       	JOIN [Products]
           	ON OrderDetails.ProductID = Products.ProductID
       GROUP BY OrderID)
   Select EmployeeID, OrderCost
   FROM OrderDetailsPrices
   	JOIN [Orders]
       	ON OrderDetailsPrices.OrderID = Orders.OrderID
   GROUP BY EmployeeID
   ORDER BY OrderCost DESC;
   ```

   - Employee 2 had the highest number of sales with $1,170

8. Which employees have BS degrees? (*Hint:* look at the [LIKE](http://www.w3schools.com/sql/sql_like.asp) operator.)

   - Employees 3 and 5 are the employees with BS degrees

     ```sql
     SELECT * FROM [Employees]
     WHERE Notes LIKE '%BS%';
     ```

9. Which supplier of three or more products has the highest average product price? (*Hint:* look at the [HAVING](http://www.w3schools.com/sql/sql_having.asp) operator.)

   - SupplierID 4 (Tokyo Traders) with an average price of $46

   ```sql
   SELECT SupplierID, COUNT(SupplierID) AS TotalProducts, AVG(Price) AS AveragePrice
   FROM [Products]
   GROUP BY SupplierID
   HAVING COUNT(SupplierID) > 2
   ORDER BY AveragePrice DESC;
   ```

   
