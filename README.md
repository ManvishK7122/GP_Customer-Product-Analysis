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

### Data Analysis Preparation
Before we get into the analysis of the data provided, my initial step that I took was to deduce how many fields and rows where in each table. Along with that, when looking at a database with multiple tables knowing where to join our tables with each other become vital to answering the questions above. Below we can see an example of how I created a table for the number of fields and rows there are in the database and combined them together using the UNION function. 

``` sql 
SELECT 'Customers' AS table_name, 13' AS number_of_columns, COUNT(*) AS number_of_rows
	FROM  customers
	
  UNION ALL
	
SELECT 'Employees' AS table_name, '8' AS number_of_columns, COUNT(*) AS number_of_rows
	FROM employees
```
Along with knowing our dataset, another major piece of information that is vital to answer our questions is to know the database schema for the scale model cars tables. Here is our schema within our database: ![db](https://github.com/ManvishK7122/GP_Customer-Product-Analysis/assets/151494568/ec45c029-78a8-4c2f-8608-b4a91f2459f3)

