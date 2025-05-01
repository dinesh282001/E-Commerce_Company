# E-Commerce_Company | SQL
*** SQL
# 1. Analyze all the tables by describing their contents.
DESCRIBE Customers;
DESCRIBE Products;
DESCRIBE Orders;
DESCRIBE OrderDetails;

# Renaming columns:
ALTER TABLE Customers
RENAME COLUMN ï»¿customer_id to customer_id;

ALTER TABLE Products
RENAME COLUMN ï»¿product_id TO product_id;

ALTER TABLE Orders
RENAME COLUMN ï»¿order_id to order_id;

ALTER TABLE OrderDetails
RENAME COLUMN ï»¿order_id to order_id;

# 2. Identify the top 3 cities with the highest number of customers.
#(To determine key markets for targeted marketing and logistic optimization)
SELECT location, COUNT(*) AS number_of_customers
FROM Customers
GROUP BY location
ORDER BY number_of_customers  DESC LIMIT 3;

# 3. Determine the distribution of customers by the number of orders placed.
#(will help in segmenting customers for tailored marketing strategies like "One-time buyers", "Occasional shoppers", "Regular customers")
SELECT OrderCount AS NumberOfOrders, COUNT(customer_id) AS CustomerCount
FROM
	(
	 SELECT customer_id, COUNT(order_id) AS OrderCount
	 FROM Orders
	 GROUP BY customer_id
	) AS CustomerOrderCount
GROUP BY OrderCount
ORDER BY OrderCount ASC;

# 4. Identify products where the average purchase quantity per order is 2 but with a high total revenue.
#(suggests premium product trends)
SELECT product_id AS Product_Id, AVG(quantity) AS AvgQuantity, SUM(price_per_unit * quantity) AS TotalRevenue
FROM OrderDetails
GROUP BY Product_Id
HAVING AVG(quantity) = 2 ORDER BY TotalRevenue DESC;

# 5. For each product category, calculate the unique number of customers purchasing from it.
#(Will help understand which categories have wider appeal across the customer base)
SELECT category, COUNT(DISTINCT(customer_id)) AS unique_customers
FROM Products
JOIN OrderDetails ON OrderDetails.product_id = Products.product_id
JOIN orders ON OrderDetails.order_id = orders.order_id
GROUP BY category
ORDER BY unique_customers DESC;

# 6. Analyze the month-on-month percentage change in total sales to identify growth trends.
SELECT Month, TotalSales, ROUND((TotalSales-LAG(TotalSales) OVER (ORDER BY Month))*100/LAG(TotalSales) OVER (ORDER BY Month),2) AS PercentChange
FROM 
(SELECT DATE_FORMAT(order_date,"%Y-%m") AS Month, SUM(total_amount) AS TotalSales
FROM Orders
GROUP BY Month) AS MonthlySales;

# 7. Examine how the average order value changes month-on-month.
#(Can guide pricing and promotional strategies to enhance order value)
WITH orderchange AS 
(
SELECT DATE_FORMAT(order_date, "%Y-%m") AS Month, 
AVG(total_amount) AS AvgOrderValue
FROM Orders
GROUP BY Month
)
SELECT Month, AvgOrderValue, ROUND(AvgOrderValue - LAG(AvgOrderValue) OVER (ORDER BY Month),2) AS ChangeInValue
FROM orderchange
ORDER BY ChangeInValue DESC;

# 8. Based on sales data, identify products with the fastest turnover rates.
#(suggesting high demand and the need for frequent restocking)
SELECT product_id, COUNT(order_id) AS SalesFrequency
FROM OrderDetails
GROUP BY product_id
ORDER BY SalesFrequency DESC LIMIT 5;

# 9. List products purchased by less than 40% of the customer base.
#(indicates potential mismatches between inventory and customer interest)
SELECT Products.product_id AS Product_id, name AS Name, COUNT(DISTINCT(customer_id)) AS UniqueCustomerCount
FROM Products
JOIN OrderDetails ON OrderDetails.product_id = Products.product_id
JOIN Orders ON Orders.order_id = OrderDetails.order_id
GROUP BY Products.Product_id, Name
HAVING  COUNT(DISTINCT(Orders.customer_id)) < 0.4* (SELECT COUNT(*) FROM customers);

# 10. Evaluate the month-on-month growth rate in the customer base.
#(To understand the effectiveness of marketing campaigns and market expansion efforts)
WITH MONTHLYNEWCUSTOMERS AS
(
  SELECT DATE_FORMAT(MIN(order_date), "%Y-%m") AS FirstPurchaseMonth, COUNT(DISTINCT(customer_id)) AS NewCustomers
  FROM Orders
  GROUP BY customer_id
)
SELECT FirstPurchaseMonth, SUM(NewCustomers) AS TotalNewCustomers
FROM MONTHLYNEWCUSTOMERS
GROUP BY FirstPurchaseMonth
ORDER BY FirstPurchaseMonth;

# 11. Identify the months with the highest sales volume.
#(Aiding in planning for stock levels, marketing efforts, and staffing in anticipation of peak demand periods)
SELECT DATE_FORMAT(order_date, "%Y-%m") AS Month, SUM(total_amount) AS TotalSales
FROM Orders
GROUP BY Month
ORDER BY TotalSales DESC LIMIT 3;

***
