# Supermarket Project Overview

The data used in this project comes from user Lovish Bansal on Kaggle. It represents three months of historical sales from a fictitious supermarket company which has three different branches. My goal for this analysis was to uncover insights about total sales, gross income, payment methods, and customer satifaction ratings.

## Preparing the Data

My first step of any analysis is getting a look at the data to better understand what I'm working with.

```
-- Get descriptive information about the table
DESCRIBE sales;

-- Are there any NULL or blank values in the dataset?
-- If anyone knows a better way of checking the whole dataset for NULL values, please let me know!
SELECT *
FROM sales
WHERE invoice_id IS NULL OR invoice_id = ''
OR branch IS NULL OR branch = ''
OR city IS NULL OR city = ''
OR customer_type IS NULL OR customer_type = ''
OR gender IS NULL OR gender = ''
OR product_line IS NULL OR product_line = ''
OR unit_price IS NULL OR unit_price = ''
OR quantity IS NULL OR quantity = ''
OR `tax_5%` IS NULL OR `tax_5%` = ''
OR total IS NULL OR total = ''
OR date IS NULL OR date = ''
OR time IS NULL OR time = ''
OR payment IS NULL OR payment = ''
OR cogs IS NULL OR cogs = ''
OR gross_margin_percentage IS NULL OR gross_margin_percentage = ''
OR gross_income IS NULL OR gross_income = ''
OR rating IS NULL OR rating = '';
```

When importing the data into MySQL Workbench the date column was a string datatype. I converted the date column to a date datatype.

```
-- Convert the date column from string datatype to date datatype.
ALTER TABLE sales ADD COLUMN new_date DATE; -- Add a new column with the date datatype
UPDATE sales SET new_date = STR_TO_DATE(`date`, '%c/%e/%Y'); -- Update the new column with the converted values from the old column
ALTER TABLE sales DROP COLUMN `date`; -- Drop the old string column
ALTER TABLE sales CHANGE COLUMN new_date `date` DATE; -- Rename the new column to the original column name
```

I also created some new columns that extracted the day name, month name, and the time of day from the new date column.

```
-- Create a new column for the day of the week.
ALTER TABLE sales ADD COLUMN weekday TEXT; 
UPDATE sales SET weekday = DAYNAME(date); 

-- Create a new column for the month.
ALTER TABLE sales ADD COLUMN month TEXT;
UPDATE sales SET month = MONTHNAME(date);

-- Create a new column for the time of day.
-- This store is open from 10am to 11pm so I will create four even time segments.
ALTER TABLE sales ADD COLUMN time_of_day TEXT;
UPDATE sales SET time_of_day = 
CASE WHEN time BETWEEN '10:00' AND '13:15' THEN 'Morning'
WHEN time BETWEEN '13:16' AND '16:30' THEN 'Afternoon'
WHEN time BETWEEN '16:31' AND '19:45' THEN 'Evening'
WHEN time BETWEEN '19:46' AND '22:00' THEN 'Closing'
ELSE 'Something went wrong' END;
```

## Querying the Data

I typically like to make comments about what question I'm trying to answer so that it is at the top of mind while writing queries. I also tend to start with broad questions and drill down into more detail (i.e. branch -> month -> week -> weekday -> time of day) while progressing through queries.

```
-- What is the total sales and total gross income for each branch?
SELECT
branch,
ROUND(SUM(total),2) AS total_sales,
ROUND(SUM(gross_income),2) AS total_gross_income
FROM sales
GROUP BY branch;
```

[Results as Image](C:\Users\01bay.PROCRASTINATION\Documents\GitHub\Supermarket_Project\QueryResults\Image\Q1_Total_Sales_Total_Gross_Income_by_Branch.png) | [Results as CSV](C:\Users\01bay.PROCRASTINATION\Documents\GitHub\Supermarket_Project\QueryResults\CSV\Q1_Total_Sales_Total_Gross_Income_by_Branch.csv)

```
-- What is the total sales and total gross income for each branch, broken down by gender?
SELECT
branch,
gender,
ROUND(SUM(total),2) AS total_sales,
ROUND(SUM(gross_income),2) AS total_gross_income
FROM sales
GROUP BY branch, gender
ORDER BY branch, gender;
```

[Results as Image](QueryResults\Image\Q2_Total_Sales_Total_Gross_Income_by_Branch_and_Gender.png) | [Results as CSV](QueryResults\CSV\Q2_Total_Sales_Total_Gross_Income_by_Branch_and_Gender.csv)

```
-- What is the total sales and total gross income for each branch, broken down by month?
SELECT
branch,
month,
ROUND(SUM(total),2) AS total_sales,
ROUND(SUM(gross_income),2) AS total_gross_income
FROM sales
GROUP BY branch, month
ORDER BY branch, FIELD(month, 'January', 'February', 'March');
```

[Results as Image](QueryResults\Image\Q3_Total_Sales_Total_Gross_Income_by_Branch_and_Month.png) | [Results as CSV](QueryResults\CSV\Q3_Total_Sales_Total_Gross_Income_by_Branch_and_Month.csv)

```
-- What is the total sales and total gross income for each branch, broken down by weekday?
SELECT
branch,
weekday,
ROUND(SUM(total),2) AS total_sales,
ROUND(SUM(gross_income),2) AS total_gross_income
FROM sales
GROUP BY branch, weekday
ORDER BY branch, FIELD(weekday, 'Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday');
```

[Results as Image](QueryResults\Image\Q4_Total_Sales_Total_Gross_Income_by_Branch_and_Weekday.png) | [Results as CSV](QueryResults\CSV\Q4_Total_Sales_Total_Gross_Income_by_Branch_and_Weekday.csv)

```
-- What is the total sales and total gross income for each branch, broken down by time of day?
SELECT
branch,
time_of_day,
ROUND(SUM(total),2) AS total_sales,
ROUND(SUM(gross_income),2) AS total_gross_income
FROM sales
GROUP BY branch, time_of_day
ORDER BY branch, FIELD(time_of_day, 'Morning', 'Afternoon', 'Evening', 'Closing');
```

[Results as Image](QueryResults\Image\Q5_Total_Sales_Total_Gross_Income_by_Branch_and_Time_of_Day.png) | [Results as CSV](QueryResults\CSV\Q5_Total_Sales_Total_Gross_Income_by_Branch_and_Time_of_Day.csv)

```
-- What is the total sales and total gross income from each product line?
SELECT
product_line,
ROUND(SUM(total),2) AS total_sales,
ROUND(SUM(gross_income),2) AS total_gross_income
FROM sales
GROUP BY product_line;
```

[Results as Image](QueryResults\Image\Q6_Total_Sales_Total_Gross_Income_by_Product_Line.png) | [Results as CSV](QueryResults\CSV\Q6_Total_Sales_Total_Gross_Income_by_Product_Line.csv)

```
-- What is the total sales and total gross income for each branch, broken down by product line?
SELECT
branch,
product_line,
ROUND(SUM(total),2) AS total_sales,
ROUND(SUM(gross_income),2) AS total_gross_income
FROM sales
GROUP BY branch, product_line
ORDER BY branch, product_line;
```

[Results as Image](QueryResults\Image\Q7_Total_Sales_Total_Gross_Income_by_Branch_and_Product_Line.png) | [Results as CSV](QueryResults\CSV\Q7_Total_Sales_Total_Gross_Income_by_Branch_and_Product_Line.csv)

```
-- Create a running total of sales and gross income for each branch by date.
WITH data AS (
SELECT
branch,
date,
ROUND(SUM(total),2) AS total_sales,
ROUND(SUM(gross_income),2) AS gross_income
FROM sales
GROUP BY branch, date)

SELECT
branch,
date,
gross_income,
SUM(total_sales) OVER (PARTITION BY branch order by date) AS cumulative_total_sales,
SUM(gross_income) OVER (PARTITION BY branch order by date) AS cumulative_gross_income
FROM data
ORDER BY branch, date;
```

[Results as Image](QueryResults\Image\Q8_Total_Sales_Total_Gross_Income_Running_Total_by_Branch_and_Date.png) | [Results as CSV](QueryResults\CSV\Q8_Total_Sales_Total_Gross_Income_Running_Total_by_Branch_and_Date.csv)

```
-- What is the average rating for each branch?
SELECT branch, ROUND(AVG(rating),1) AS avg_rating
FROM sales
GROUP BY branch;
```

[Results as Image](QueryResults\Image\Q9_Average_Rating_by_Branch.png) | [Results as CSV](QueryResults\CSV\Q9_Average_Rating_by_Branch.csv)

```
-- How many perfect ratings does each branch have?
SELECT branch, COUNT(*) AS total_perfect_ratings
FROM sales
WHERE rating = 10
GROUP BY branch;
```

[Results as Image](QueryResults\Image\Q10_Perfect_Ratings_by_Branch.png) | [Results as CSV](QueryResults\CSV\Q10_Perfect_Ratings_by_Branch.csv)

```
-- What is the average order value for each branch?
SELECT branch, ROUND(AVG(total),2) AS avg_order_value
FROM sales
GROUP BY branch;
```

[Results as Image](QueryResults\Image\Q11_AOV_by_Branch.png) | [Results as CSV](QueryResults\CSV\Q11_AOV_by_Branch.csv)

```
-- Which payment methods are the most popular?
SELECT
payment,
COUNT(*) AS num_transactions,
ROUND(SUM(total),2) AS total_sales
FROM sales
GROUP BY payment;
```

[Results as Image](QueryResults\Image\Q12_Top_Payment_Methods_by_Branch.png) | [Results as CSV](QueryResults\CSV\Q12_Top_Payment_Methods_by_Branch.csv)



The data used in this project comes from user Lovish Bansal on Kaggle. 
The link for this data is [here.](https://www.kaggle.com/datasets/lovishbansal123/sales-of-a-supermarket)
Use of this data is allowed under the Apache 2.0 License found [here.](https://www.apache.org/licenses/LICENSE-2.0)

