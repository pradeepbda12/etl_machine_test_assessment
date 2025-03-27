Final Documentation: ETL Process and Power BI Integration 
 
1. Introduction 
 
This document outlines the complete ETL (Extract, Transform, Load) process for loading and 
analyzing sales data in PostgreSQL, along with Power BI integration for data visualization. 
 
2. Database Configuration and Setup 
 
2.1 Configuration Details 
 
The PostgreSQL database is configured with the following credentials: 
 
2.2 Connecting to PostgreSQL 
 
A Python script using psycopg2 is used to connect to PostgreSQL: 
 
import psycopg2 
connection = psycopg2.connect( 
    dbname='postgres', 
    user='postgres', 
    password='********', 
    host='localhost', 
    port='5432' 
) 
 
3. Database Schema and Table Creation 
 
3.1 Normalized Tables 
 
The database consists of three main tables: 
 
Products Table 
 
CREATE TABLE IF NOT EXISTS products ( 
    product_id VARCHAR(10) PRIMARY KEY, 
    product_name TEXT NOT NULL, 
    category TEXT NOT NULL, 
    price DECIMAL(10,2) NOT NULL 
); 
 
Customers Table 
 
CREATE TABLE IF NOT EXISTS customers ( 
    customer_id VARCHAR(10) PRIMARY KEY, 
    region TEXT NOT NULL 
); 
 
Transactions Table 
 
CREATE TABLE IF NOT EXISTS transactions ( 
    transaction_id VARCHAR(10) PRIMARY KEY, 
    customer_id VARCHAR(10), 
    product_id VARCHAR(10), 
    quantity INT CHECK (quantity > 0), 
    total_value DECIMAL(10,2), 
    date DATE NOT NULL, 
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id) ON DELETE SET 
NULL, 
    FOREIGN KEY (product_id) REFERENCES products(product_id) ON DELETE CASCADE 
); 
 
Indexes for Performance Optimization 
 
CREATE INDEX IF NOT EXISTS idx_date ON transactions(date); 
CREATE INDEX IF NOT EXISTS idx_region ON customers(region); 
 
4. ETL Process 
 
4.1 Data Loading and Transformation 
 
Python’s pandas and sqlalchemy are used to load data efficiently into PostgreSQL using to_sql: 
 
import pandas as pd 
from sqlalchemy import create_engine 
 
df_products.to_sql('products', engine, if_exists='append', index=False, chunksize=1000) 
df_customers.to_sql('customers', engine, if_exists='append', index=False, chunksize=1000) 
df_transactions.to_sql('transactions', engine, if_exists='append', index=False, chunksize=1000) 
 
4.2 Data Quality Checks 
 
Handling missing customer_id: 
 
df['customer_id'] = df['customer_id'].fillna('Unknown') 
 
Removing negative quantity values: 
 
df.loc[df['quantity'] < 0, 'quantity'] = 0 

Checking for duplicate transaction_id: 
if df['transaction_id'].duplicated().any(): 
logging.error("Duplicate transaction_id entries found. Aborting load.") 

5. Query Execution and Optimization 
5.1 Key Business Queries
   
Total Sales Value by Region 
SELECT c.region, SUM(t.total_value) AS total_sales 
FROM transactions t 
JOIN customers c ON t.customer_id = c.customer_id 
GROUP BY c.region;

Top 5 Products by Total Sales Value 

SELECT p.product_name, SUM(t.total_value) AS total_sales 
FROM transactions t 
JOIN products p ON t.product_id = p.product_id 
GROUP BY p.product_name 
ORDER BY total_sales DESC 
LIMIT 5; 
Monthly Sales Trends 
SELECT DATE_TRUNC('month', t.date) AS sale_month, SUM(t.total_value) AS monthly_sales 
FROM transactions t 
GROUP BY sale_month 
ORDER BY sale_month; 

5.2 Query Optimization 
Using EXPLAIN ANALYZE to analyze query performance. 
Creating indexes (idx_date, idx_region) to speed up queries. 

7. Power BI Integration 
Connected Power BI to PostgreSQL: 
Created Dashboards: 
Sales Trends Dashboard: Line charts for monthly sales. 
Product Performance Dashboard: Bar charts for top-selling products. 
Regional Sales Breakdown: Map visualization for total sales by region. 
1. Total Sales 

Total Sales = SUM(sales[total_value]) 
Use Case: This measure calculates the total sales across all transactions. Useful for the bar 
chart showing sales by region. 

2. Average Sales Value per Transaction 
Average Sales = AVERAGE(sales[total_value]) 
Use Case: Displays the average sales value per transaction. Use this in the card visual.
 
4. Year-to-Date (YTD) Sales 
YTD Sales = TOTALYTD(SUM(sales[total_value]), sales[date]) 
Use Case: Useful for the line chart to track cumulative sales trends over time. 

5. Count of Significant Sales (Exceeding $1000)  
Significant Sales = COUNTROWS(FILTER(sales, sales[total_value] > 1000)) 
Use Case: Highlights transactions where the total value exceeds $1000. Use this in the card 
visual.

7. Total Sales for Electronics 
Electronics Sales = CALCULATE(SUM(sales[total_value]), sales[category] = "Electronics") 
Use Case: Focuses specifically on sales in the "Electronics" category. Useful for insights or 
filtering.

3. Build the Dashboard 
Create the following visuals: 
1. Bar Chart: Total Sales by Region 
Data: 
Axis: sales[region] 
Values: Total Sales 
Insight: Highlights which region contributes the most to overall sales. 
2. Line Chart: Monthly Sales Trend 
Data: 
Axis: sales[date] (set to Month/Year format). 
Values: YTD Sales 
Insight: Tracks cumulative sales trends over time and detects seasonality. 
3. Table: Top 5 Products by Total Sales Value 
Columns: sales[product_name], sales[category], Total Sales 
Filter: 
Apply a Top N filter on Total Sales, selecting the top 5 products. 
Insight: Shows the highest-performing products based on sales value. 
4. Card Visual: Key Metric 
Add a card and use either Average Sales or Total Sales. 
Insight: Provides a high-level overview of key performance indicators. 
5. Slicer: Filter by Category/Region/Date 
Add slicers to filter data by: 
sales[category] (e.g., Electronics, Fashion, etc.). 
sales[region] (e.g., North, South, etc.). 
sales[date] (selectable by year or month). 
4. Insights Description 
Create a brief description of each visual and measure. Here’s an example: 
Bar Chart: Shows total sales by region, indicating which region is performing best. 
Line Chart: Displays the monthly sales trend to identify seasonal trends or growth over time. 
Table: Lists the top 5 products by total sales value, highlighting the most successful products. 
Card Visual: Provides a high-level view of the average sales value or total sales, acting as a 
summary metric. 
Slicer: Allows users to filter and analyze data by category, region, or date.



7. Conclusion 
This document provides an end-to-end guide for ETL implementation using PostgreSQL and 
Python, along with Power BI integration for data visualization. By following this setup, 
businesses can efficiently analyze sales data and derive insights for decision-making. 
