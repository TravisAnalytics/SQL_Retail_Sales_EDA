# SQL Retail Sales Analysis Project

## Project Overview

**Project Title**: Retail Sales Analysis  
**Level**: Beginner  
**Database**: `retail_db`

This project is designed to demonstrate SQL skills and techniques typically used by data analysts to explore, clean, and analyze retail sales data. The project involves setting up a retail sales database, performing exploratory data analysis (EDA), and answering specific business questions through SQL queries. This project uses MySQL to perform data analysis. 

## Objectives

1. **Set up a retail sales database**: Create and populate a retail sales database with the provided sales data.
2. **Data Cleaning**: Identify and remove any records with missing or null values.
3. **Exploratory Data Analysis (EDA)**: Perform basic exploratory data analysis to understand the dataset.
4. **Business Analysis**: Use SQL to answer specific business questions and derive insights from the sales data.

## Project Structure

### 1. Database Setup

- **Database Creation**: The project starts by creating a database named `p1_retail_db`.
- **Table Creation**: A table named `retail_sales` is created to store the sales data. The table structure includes columns for transaction ID, sale date, sale time, customer ID, gender, age, product category, quantity sold, price per unit, cost of goods sold (COGS), and total sale amount.
- **Database Import**: Data is imported using LOCAL INFILE using data user specified path. 
- **Null Value Identification**: Columns from data are filtered to replace blank values in .CSV files as 'NULL'. 

```sql
CREATE DATABASE p1_retail_db;

CREATE TABLE retail_sales
(
    transactions_id INT PRIMARY KEY,
    sale_date DATE,	
    sale_time TIME,
    customer_id INT,	
    gender VARCHAR(10),
    age INT,
    category VARCHAR(35),
    quantity INT,
    price_per_unit FLOAT,	
    cogs FLOAT,
    total_sale FLOAT
);

LOAD DATA LOCAL INFILE '[User File Path Name]'
INTO TABLE retail_sales_eda
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\r\n'
IGNORE 1 LINES
(
 @transactions_id,
 @sale_date,
 @sale_time,
 @customer_id,
 @gender,
 @age,
 @category,
 @quantiy,
 @price_per_unit,
 @cogs,
 @total_sale
)
SET
  transactions_id = NULLIF(@transactions_id,''),
  sale_date       = NULLIF(@sale_date,''),
  sale_time       = NULLIF(@sale_time,''),
  customer_id     = @customer_id,
  gender          = @gender,
  age             = NULLIF(@age,''),
  category        = @category,
  quantiy         = NULLIF(@quantiy,''),
  price_per_unit  = NULLIF(@price_per_unit,''),
  cogs            = NULLIF(@cogs,''),
  total_sale      = NULLIF(@total_sale,'');
```

### 2. Data Exploration & Cleaning

- **Record Count**: Determine the total number of records in the dataset.
- **Customer Count**: Find out how many unique customers are in the dataset.
- **Category Count**: Identify all unique product categories in the dataset.
- **Data Spelling Check**: Check for any misspelled columns or data. 
- **Null Value Check**: Check for any null values in the dataset and delete records with missing data.

```sql
SELECT COUNT(*)
FROM retail_sales;

SELECT COUNT(DISTINCT customer_id)
FROM retail_sales;

SELECT DISTINCT category
FROM retail_sales;

ALTER TABLE retail_sales
RENAME COLUMN quantiy TO quantity;

SELECT *
FROM retail_sales
WHERE 
    sale_date IS NULL OR sale_time IS NULL OR customer_id IS NULL OR 
    gender IS NULL OR age IS NULL OR category IS NULL OR 
    quantity IS NULL OR price_per_unit IS NULL OR cogs IS NULL;

DELETE FROM retail_sales
WHERE 
    sale_date IS NULL OR sale_time IS NULL OR customer_id IS NULL OR 
    gender IS NULL OR age IS NULL OR category IS NULL OR 
    quantity IS NULL OR price_per_unit IS NULL OR cogs IS NULL;

```

### 3. Data Analysis & Findings

The following SQL queries were developed to answer specific business questions:

1. **Write a SQL query to retrieve all columns for sales made on '2022-11-05**:
```sql
SELECT *
FROM retail_sales
WHERE sale_date = '2022-11-05';
```

2. **Write a SQL query to retrieve all transactions where the category is 'Clothing' and the quantity sold is more than 4 in the month of Nov-2022**:
```sql
SELECT 
    category,
    SUM(quantity) AS total_quantity
FROM retail_sales
WHERE category = 'Clothing'
  AND DATE_FORMAT(sale_date, '%Y-%m') = '2022-11'
GROUP BY category
HAVING SUM(quantity) >= 4;

```

3. **Write a SQL query to calculate the total sales (total_sale) and total orders for each category.**:
```sql
SELECT 
    category,
    SUM(total_sale) as total_sale,
    COUNT(*) as total_orders
FROM retail_sales
GROUP BY category;
```

4. **Write a SQL query to find the average age of customers who purchased items from the 'Beauty' category.**:
```sql
SELECT
	category,
    ROUND(AVG(age), 2) as avg_age
FROM retail_sales
WHERE category = 'Beauty';
```

5. **Write a SQL query to find all transactions where the total_sale is greater than 1000.**:
```sql
SELECT * 
FROM retail_sales
WHERE total_sale > 1000;
```

6. **Write a SQL query to find the total number of transactions (transaction_id) made by each gender in each category.**:
```sql
SELECT 
    category,
    gender,
    COUNT(*) AS transactions
FROM retail_sales
GROUP BY category, gender;
```

7. **Write a SQL query to calculate the average sale for each month. Find out best selling month in each year**:
```sql
SELECT
    sale_month,
    avg_sales,
    RANK() OVER (ORDER BY avg_sales DESC) AS sales_rank
FROM (
SELECT
        DATE_FORMAT(sale_date, '%Y-%m') AS sale_month,
        ROUND(AVG(total_sale), 2) AS avg_sales
    FROM retail_sales
    GROUP BY sale_month
) AS monthly_sales;
```

8. **Write a SQL query to find the top 5 customers based on the highest total sales **:
```sql
SELECT 
    customer_id,
    SUM(total_sale) as total_sales
FROM retail_sales
GROUP BY customer_id
ORDER BY total_sales DESC
LIMIT 5;
```

9. **Write a SQL query to find the number of unique customers who purchased items from each category.**:
```sql
SELECT
	category,
    COUNT(DISTINCT customer_id) unique_customers
FROM retail_sales
GROUP BY category;
```

10. **Write a SQL query to create each shift and number of orders (Example Morning <12, Afternoon Between 12 & 17, Evening >17)**:
```sql
WITH hourly_sales AS 
(
SELECT *,
	CASE 
		WHEN EXTRACT(HOUR FROM sale_time) < 12 THEN 'Morning'
		WHEN EXTRACT(HOUR FROM sale_time) BETWEEN 12 AND 17 THEN 'Afternoon'
        ELSE 'Evening' 
	END AS shift,
    CASE 
		WHEN EXTRACT(HOUR FROM sale_time) < 12 THEN 1
		WHEN EXTRACT(HOUR FROM sale_time) BETWEEN 12 AND 17 THEN 2
        ELSE 3
	END AS shift_order
FROM retail_sales
)
SELECT
	shift,
    shift_order,
	COUNT(*) total_orders
FROM hourly_sales
GROUP BY shift, shift_order
ORDER BY shift_order
```

## Findings & Reports

- **Customer Demographics**: The average age of a customer is 41 years old for both men and women.
- **High-Value Transactions**: Purchases over $1,000 are made by majorily customers age 37 or over.
- **Customer Insights**: Customers purchase more quantity of products in Electronics and Clothing. 
- **Sales Summary**: This dataset shows 2,000 total transactions and generating roughly over $900,000 in revenue. On average, each order was worth $456, indicating a moderate average order value. The most popular categories were both Electronics and Clothing with very similar sales, followed by Beauty which had a 3.6% lower average purchase-value.
- **Trend Analysis**: 2022 had slightly higher peaks and lower lows, however, trends for both years remain similar with 2023 having slightly lower on-average overall revenue. 

## Conclusion

The retail dataset highlights that customers average 41 years old, with high-value purchases over $1,000 largely made by those aged 37 and above. While Electronics and Clothing drive the highest sales volumes, Beauty lags slightly in purchase value. Overall, 2,000 transactions generated over $900,000 in revenue, with an average order value of $456.

However, year-to-date sales are trending below the previous year, suggesting potential challenges in sustaining growth. The concentration of spending among older customers underscores the importance of retaining this group while finding ways to engage younger buyers to stabilize future sales.

## Author - Travis the Analyst

This project is part of my portfolio, showcasing the SQL skills essential for data analyst roles. If you have any questions, feedback, or would like to collaborate, feel free to get in touch!

Thank you for your support, and I look forward to connecting with you!
