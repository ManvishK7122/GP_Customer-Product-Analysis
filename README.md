# GP_Customer-Product-Analysis
## Introduction
This is a project done under the coursework of a MOOC (Massive Open Online Course) and was started in the middle of October 2023 and finished by the beginning of November 2023. This project involves extracting valuable insights from a SQL database containing sales records of scale model cars. The objective is to utilize this data to inform decision-making processes, providing a comprehensive understanding of sales trends and patterns within the scale model car market, and therefore also providing recommendations and strategies to increase customer interaction.

### Database Tables
Within this database there are 8 different tables that have information about sales, customers, employees, and order details. These tables include: 

- Customers: Comprehensive customer information
- Employees: A repository of all employee details
- Offices: Information pertaining to sales offices
- Orders: Records of customers' sales orders
- OrderDetails: Detailed sales order lines for each transaction
- Payments: Documentation of customers' payment records
- Products: A catalog featuring scale model cars
- ProductLines: A categorized list outlining different product lines

### Concepts:
- Data Analysis
- Data Mining
- Scalar Subqueries and CTE

### Questions Answered: 
In this project, there are 3 major questions that are going to be answered when looking into the data within the database: 

- Question 1: Which products should we order more of or less of?
- Question 2: How should we tailor marketing and communication strategies to customer behaviors?
- Question 3: How much can we spend on acquiring new customers?
- 
Within each question, the explaination for how I designated the KPIs for each problem will be included.  

### Prerequiste Steps for Data Analysis 
Before we get into the analysis of the data provided, my initial step that I took was to deduce how many fields and rows where in each table. Along with that, when looking at a database with multiple tables knowing where to join our tables with each other become vital to answering the questions above. Below we can see an example of how I created a table for the number of fields and rows there are in the database and combined them together using the UNION function. 

``` sql 
SELECT 'Customers' AS table_name, 13' AS number_of_columns, COUNT(*) AS number_of_rows
	FROM  customers
	
  UNION ALL
	
SELECT 'Employees' AS table_name, '8' AS number_of_columns, COUNT(*) AS number_of_rows
	FROM employees
```
Along with knowing our dataset, another major piece of information that is vital to answer our questions is to know the database schema for the scale model cars tables. Here is our schema within our database: ![db](https://github.com/ManvishK7122/GP_Customer-Product-Analysis/assets/151494568/ec45c029-78a8-4c2f-8608-b4a91f2459f3)

### Data Analysis

Q1: Which products should we order more of or less of?

When looking at this question refers to reports about inventory, specifically about 2 metrics(or KPIs): low stock and product performance.

- Low stock is derived as the ratio of the total quantity of each product ordered to the current quantity in stock. By deriving this, we are identifying the top ten products with the highest rates, which signify low stock levels.
- For product performance, this is derived by total sales per each unique product.
- Using these two metrics, our restocking priority would be products that have high product performance that are almost out of stock.

Here is the queries used to decifer this information: 

Finding our low stock products:
```sql
SELECT p.productCode, ROUND(SUM(quantityOrdered) *1.0 /(SELECT  quantityInStock FROM products p WHERE od.productCode=p.productCode),2) AS low_stock,p.productName,p.productLine
  FROM orderdetails od
 INNER JOIN products p
    ON od.productCode=p.productCode
 GROUP BY p.productCode
 ORDER BY low_stock DESC
 LIMIT 10;
```
Finding our product performance: 
```sql
SELECT p.productCode, ROUND(SUM(quantityOrdered* priceEach),2) AS product_performance,p.productName,p.productLine
  FROM orderdetails AS od
 INNER JOIN products p
    ON p.productCode=od.productCode
 GROUP BY p.productCode
 ORDER BY product_performance DESC
 LIMIT 10;
```
Product Restocking Priority:
```sql
WITH 
low_stock_products AS (
SELECT productCode, ROUND(SUM(quantityOrdered) *1.0 /(SELECT  quantityInStock FROM products p WHERE od.productCode=p.productCode),2) AS low_stock		 
  FROM orderdetails od
 GROUP BY productCode
 ORDER BY low_stock DESC
 LIMIT 10
)

SELECT productCode, SUM(quantityOrdered* priceEach) AS product_performance
  FROM orderdetails od
 WHERE productCode IN (SELECT productCode FROM low_stock_products)
 GROUP BY productCode 
 ORDER BY product_performance DESC
 LIMIT 10;
```
