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

Low stock products:
```sql
SELECT p.productCode, ROUND(SUM(quantityOrdered) *1.0 /(SELECT  quantityInStock FROM products p WHERE od.productCode=p.productCode),2) AS low_stock,p.productName,p.productLine
  FROM orderdetails od
 INNER JOIN products p
    ON od.productCode=p.productCode
 GROUP BY p.productCode
 ORDER BY low_stock DESC
 LIMIT 10;
```
Product performance: 
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

Q2: How should we tailor marketing and communication strategies to customer behaviors?
To answer this question, we need to figure out the customers who have spent the most and the customers who spent the least. This is categorized into 2 groups of people being: VIP customers(most engaged) and less engaged customers. By decifering this information, some examples of enticing customers can be by coordinating events to enhance loyalty among VIPs and initiate a campaign targeting less engaged individuals.

Lets start by calculating the revenue generated by the customers:
Customer Revenue:
```sql
SELECT o.customerNumber, SUM(quantityOrdered *(priceEach-buyPrice)) AS profit_per_customer
  FROM orders o
 INNER JOIN orderdetails od
    ON o.orderNumber = od.orderNumber
 INNER JOIN products p
    ON od.productCode = p.productCode
 GROUP BY customerNumber;
```
To find the most engaged and least engaged customers, we would need to tweek the previous query.

Less Engaged Customers:
```sql
WITH 
less_engaged_customers AS (
SELECT o.customerNumber, SUM(quantityOrdered * (priceEach - buyPrice)) AS revenue
  FROM products p
  JOIN orderdetails od
    ON p.productCode = od.productCode
  JOIN orders o
    ON o.orderNumber = od.orderNumber
 GROUP BY o.customerNumber
 ) 
 
SELECT contactLastName, contactFirstName,city,country, le.revenue
  FROM customers c
 INNER JOIN less_engaged_customers le
    ON c.customerNumber=le.customerNumber
 ORDER BY le.revenue 
 LIMIT 5;
 
VIP(most engaged) customers:
```sql
WITH 
VIP_customers AS (
SELECT o.customerNumber, SUM(quantityOrdered * (priceEach - buyPrice)) AS revenue
  FROM products p
  JOIN orderdetails od
    ON p.productCode = od.productCode
  JOIN orders o
    ON o.orderNumber = od.orderNumber
 GROUP BY o.customerNumber
 ) 
 
SELECT contactLastName, contactFirstName,city,country, vip.revenue
  FROM customers c
 INNER JOIN VIP_customers vip
    ON c.customerNumber=vip.customerNumber
 ORDER BY vip.revenue DESC
 LIMIT 5;

Q3: How much can we spend on acquiring new customers?
When looking at the expenses we are able to spend on new customers, we need to find the average revenue generated by the overall customer base. This number would give us the rough estimate to the amount of expenses needed to be issued for customer acquisition.

Customer Acquisition Allocation:

```sql
WITH 
money_on_marketing AS (
SELECT o.customerNumber, SUM(quantityOrdered * (priceEach - buyPrice)) AS revenue
  FROM products p
  JOIN orderdetails od
    ON p.productCode = od.productCode
  JOIN orders o
    ON o.orderNumber = od.orderNumber
 GROUP BY o.customerNumber
 ) 
 
 SELECT AVG(mm.revenue) AS Average_revenue_per_customer
   FROM customers c
  INNER JOIN money_on_marketing mm
     ON c.customerNumber=mm.customerNumber
 ```

### Conclusion
Through the data mining previously explained before, we are able to decifer 3 major aspects of this car business, which are:

Q1: Which products should we order more of or less of?

Restock Priority: 
| Product Name  | Type of Car | 
| ------------- | ------------- |
| 1952 Alpine Renault 1300 | Classic Cars |
| 2003 Harley-Davidson Eagle Drag Bike | Motorcyles |
| 1968 Ford Mustang | Classic Cars |
| 2001 Ferrari Enzo | Classic Cars |
| 2002 Suzuki XREO | Motorcycles |
| 1969 Ford Falcon | Classic Cars |
| 1980s Black Hawk Helicopter |	Planes |
| 1917 Grand Touring Sedan | Vintage Cars |
| 1998 Chrysler Plymouth Prowler | Classic Cars |
| 1992 Ferrari 360 Spider red |	Classic Cars |

In this case, Classic Cars should be a priority for restock. 

Q2: How should we tailor marketing and communication strategies to customer behaviors?

By knowing this information, we can tailor our marketing strategies by coordinating events to enhance loyalty among VIPs and initiating a campaign targeting less engaged individuals.
Most Engaged Customers: 

| LastName | FirstName | City | Country   | Profit    |
|-----------------|------------------|------------|-----------|-----------|
| Freyre          | Diego            | Madrid     | Spain     | $326,519.66 |
| Nelson          | Susan            | San Rafael | USA       | $236,769.39 |
| Young           | Jeff             | NYC        | USA       | $72,370.09  |
| Ferguson        | Peter            | Melbourne  | Australia | $70,311.07  |
| Labrune         | Janine           | Nantes     | France    | $60,875.30  |

Least Engaged Customers:

| Last Name | First Name | City | Country | Profit   |
|-----------------|------------------|------------|---------|----------|
| Young           | Mary             | Glendale   | USA     | $2,610.87  |
| Taylor          | Leslie           | Brickhaven | USA     | $6,586.02  |
| Ricotti         | Franco           | Milan      | Italy   | $9,532.93  |
| Schmitt         | Carine           | Nantes     | France  | $10,063.80 |
| Smith           | Thomas           | London     | UK      | $10,868.04 |

Q3: How much can we spend on acquiring new customers?
After calculating our budget for acquiring new customers, our average revenue per customer is

| Average Revenue Per Customer |
|:------------:|
| $39,039.59 |
