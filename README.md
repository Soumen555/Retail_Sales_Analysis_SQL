# Retail Sales Analysis SQL Project

## Project Overview

**Project Title**: Retail Sales Analysis   

This project is designed to demonstrate SQL skills and techniques typically used by data analysts to explore, clean, and analyze retail sales data. The project involves setting up a retail sales database, performing exploratory data analysis (EDA), and answering specific business questions through SQL queries. This project is ideal for those who are starting their journey in data analysis and want to build a solid foundation in SQL.

## Objectives

1. **Set up a retail sales database**: Create and populate a retail sales database with the provided sales data.
2. **Data Cleaning**: Identify and remove any records with missing or null values.
3. **Exploratory Data Analysis (EDA)**: Perform exploratory data analysis to understand the dataset.
4. **Business Analysis**: Use SQL to answer specific business questions and derive insights from the sales data.

## Project Structure

### 1. Database Setup

- **Database Creation**: The project starts by creating a database named `sales`.
- **Table Creation**: A table named `retail_sales` is created to store the sales data. The table structure includes columns for transaction ID, sale date, sale time, customer ID, gender, age, product category, quantity sold, price per unit, cost of goods sold (COGS), and total sale amount.

```sql
CREATE DATABASE sales;
use sales;

CREATE TABLE retail_sales (
    transactions_id INT PRIMARY KEY,
    sale_date DATE,
    sale_time TIME,
    customer_id INT,
    gender VARCHAR(15),
    age INT,
    category VARCHAR(20),
    quantiy INT,
    price_per_unit FLOAT,
    cogs FLOAT,
    total_sale FLOAT
);
        
select * from retail_sales;
alter table retail_sales change column quantiy quantity int;
```

### 2. Data Exploration & Cleaning

- **Record Count**: Determine the total number of records in the dataset.
- **Customer Count**: Find out how many unique customers are in the dataset.
- **Category Count**: Identify all unique product categories in the dataset.
- **Null Value Check**: Check for any null values in the dataset and delete records with missing data.

```sql
SET SQL_SAFE_UPDATES = 0;

DELETE FROM retail_sales 
WHERE
    sale_date IS NULL OR sale_time IS NULL
    OR customer_id IS NULL
    OR gender IS NULL
    OR age IS NULL
    OR category IS NULL
    OR quantity IS NULL
    OR price_per_unit IS NULL
    OR cogs IS NULL;

SET SQL_SAFE_UPDATES = 1;
```

### 3. Data Analysis & Findings

The following SQL queries were developed to answer specific business questions:

1. **Write a SQL query to retrieve all columns for sales made on '2022-11-05**:
```sql
SELECT *
FROM retail_sales
WHERE sale_date = '2022-11-05';
```

2. **Write a SQL query to retrieve all transactions where the category is 'Clothing' and the quantity sold is more than 3 in the month of Nov-2022**:
```sql
SELECT 
    *
FROM
    retail_sales
WHERE
    category = 'Clothing' AND quantity > 3
        AND sale_date BETWEEN '2022-11-01' AND '2022-11-30';
```

3. **Write a SQL query to calculate the total sales (total_sale) for each category.**:
```sql
SELECT 
    category,
    SUM(total_sale) as net_sale,
    COUNT(*) as total_orders
FROM retail_sales
GROUP BY 1
```

4. **Write a SQL query to find the average age of customers who purchased items from the 'Beauty' category.**:
```sql
SELECT 
    category, ROUND(AVG(age)) AS Average_Age
FROM
    retail_sales
WHERE
    category = 'Beauty'
GROUP BY category;
```

5. **Write a SQL query to find all transactions where the total_sale is greater than 1000.**:
```sql
SELECT 
    transactions_id, category, total_sale
FROM
    retail_sales
WHERE
    total_sale > 1000
ORDER BY total_sale DESC;
```

6. **Write a SQL query to find the total number of transactions (transaction_id) made by each gender in each category.**:
```sql
SELECT 
    gender,
    category,
    COUNT(transactions_id) AS Total_Transaction
FROM
    retail_sales
GROUP BY gender , category
ORDER BY category;
```

7. **Write a SQL query to calculate the average sale for each month. Find out best selling month in each year**:
```sql
select s_year as Year,s_month as Month,Avg_sales from (
with sales_table as(
select year(sale_date) as s_year, monthname(sale_date) as s_month, round(avg(total_sale)) as Avg_sales
from retail_sales
group by s_year, s_month
order by s_year,s_month)
select *, rank()over(partition by s_year order by Avg_sales desc) as S_Rank
from sales_table) as t1
where S_Rank = 1;
```

8. **Write a SQL query to find the top 5 customers based on the highest total sales. **:
```sql
SELECT 
    customer_id, SUM(total_sale) AS Total_Sale
FROM
    retail_sales
GROUP BY customer_id
ORDER BY Total_Sale DESC
LIMIT 5;
```

9. **Write a SQL query to find the number of unique customers who purchased items from each category.**:
```sql
SELECT 
    category, COUNT(DISTINCT customer_id) AS no_of_customer
FROM
    retail_sales
GROUP BY category;
```

10. **Write a SQL query to create each shift and number of orders (Example Morning <=12, Afternoon Between 12 & 17, Evening >17) and find the total no of order shift wise**:
```sql
with shift_table as(
select *,
	case
    when sale_time < "12:00:00" then "Morning"
    when sale_time between "12:00:00" and "17:00:00" then "Afternoon"
    when sale_time > "17:00:00" then "Evening"
    end as Shift
 from retail_sales)
 select shift, count(transactions_id) as Total_Transaction
 from shift_table
 group by shift
 order by Total_Transaction desc;
```

11. **Question: Write a query to find the earliest purchase date for each customer (their cohort) and calculate how many total subsequent orders they placed in different calendar months after their initial cohort month.
Business Value: Measures customer loyalty, lifecycle length, and repeat purchasing behavior.**:
```sql
    with customer_cohorts as(
    SELECT 
        customer_id,
        MIN(sale_date) AS cohort_date,
        DATE_FORMAT(MIN(sale_date), '%Y-%m-01') AS cohort_month
    FROM
        retail_sales
    GROUP BY customer_id)
    select c.cohort_month as Cohort_Month,
    count(distinct c.customer_id) as Retained_Customer,
    count(r.transactions_id) as Total_Subsequence_Orders
    from customer_cohorts c join retail_sales r
    on c.customer_id = r.customer_id
    where r.sale_date > c.cohort_date
    group by c.cohort_month
    order by c.cohort_month;
```

12. **Question: Calculate the net profit (total_sale - cogs) and the exact net profit margin percentage for each product category. Filter the results to only show demographics (gender and age groups divided into brackets: Under 30, 30-50, Over 50) that generate an average profit margin higher than 40%.
Business Value: Helps marketing teams target advertising spend on high-yielding demographic pockets rather than broad categories.**:
```sql
SELECT 
    category,
    gender,
    CASE
        WHEN age < 30 THEN 'Under 30'
        WHEN age BETWEEN 30 AND 50 THEN '30-50'
        WHEN age > 50 THEN 'Over 50'
    END AS age_group,
    ROUND(SUM((total_sale - cogs)), 2) AS total_net_profit,
    ROUND((SUM(total_sale - cogs) / SUM(total_sale)) * 100,
            2) AS net_profit_mergin
FROM
    retail_sales
GROUP BY category , gender , age_group
HAVING net_profit_mergin > '40.00'
ORDER BY net_profit_mergin DESC
;
```

13. **Question: Create a time-series report showing the total revenue generated for each calendar month, alongside the previous month's revenue and the strict percentage growth rate compared to that previous month.
Business Value: Standard executive KPI used to track business health trends and seasonal demand shifts.**:
```sql
with mr as(
select date_format(sale_date,"%Y-%m") as YearMonth,
	sum(total_sale) as Current_Month_Revenue
from retail_sales
group by YearMonth)
select YearMonth,
	Current_Month_Revenue,
    lag(Current_Month_Revenue) over(order by YearMonth) as Previous_Year_Revenue,
    round(((Current_Month_Revenue - lag(Current_Month_Revenue) over(order by YearMonth))
    /lag(Current_Month_Revenue) over(order by YearMonth))*100 ,2) as Percentage_Growth
    from mr
    order by YearMonth;
```

14. **Question: Divide the 24-hour day into operational blocks (Morning: 06:00-11:59, Afternoon: 12:00-16:59, Evening: 17:00-21:59, Night: 22:00-05:59). For each category, determine which time block yields the highest average order value (AOV).
Business Value: Optimizes staffing schedules, store operational hours, and lightning-deal promotional timing.**:
```sql
with sales_n as(
select category,
case
when sale_time between "06:00:00" and "11:59:00" then "Morning"
when sale_time between "12:00:00" and "16:59:00" then "Afternoon"
when sale_time between "17:00:00" and "21:59:59" then "Evening"
else "Night"
end as Part_of_Day,
round(avg(total_sale),2) as Average_Sale,
row_number() over(partition by category order by avg(total_sale) desc) as rn
from retail_sales
group by category,Part_of_Day
order by Average_Sale desc)
select category, Part_of_Day, Average_Sale
from sales_n
where rn=1;
```

15. **Question: Write a query to find the top customers whose cumulative purchases account for the top 20% of the company's total historical sales revenue.
Business Value: Isolates VIP accounts ("whales") for loyalty retention programs. Note that in your sample database column, the spelling is quantiy instead of quantity—though this query targets total_sale to safely sidestep typo variations.**:
```sql
WITH CustomerSpending AS (
    SELECT 
        customer_id,
        SUM(total_sale) AS individual_spend,
        SUM(SUM(total_sale)) OVER () AS company_total_revenue
    FROM retail_sales
    GROUP BY customer_id
),
RunningTotals AS (
    SELECT 
        customer_id,
        individual_spend,
        SUM(individual_spend) OVER (ORDER BY individual_spend DESC) AS running_cumulative_spend,
        company_total_revenue
    FROM CustomerSpending
)
SELECT 
    customer_id,
    individual_spend,
    ROUND((running_cumulative_spend / company_total_revenue) * 100, 2) AS cumulative_revenue_contribution_pct
FROM RunningTotals
WHERE (running_cumulative_spend / company_total_revenue) <= 0.20
ORDER BY individual_spend DESC;
```

## Findings

- **Customer Demographics**: The dataset includes customers from various age groups, with sales distributed across different categories such as Clothing and Beauty.
- **High-Value Transactions**: Several transactions had a total sale amount greater than 1000, indicating premium purchases.
- **Sales Trends**: Monthly analysis shows variations in sales, helping identify peak seasons.
- **Customer Insights**: The analysis identifies the top-spending customers and the most popular product categories.

## Reports

- **Sales Summary**: A detailed report summarizing total sales, customer demographics, and category performance.
- **Trend Analysis**: Insights into sales trends across different months and shifts.
- **Customer Insights**: Reports on top customers and unique customer counts per category.

## Conclusion

This data analytics project leverages MySQL to extract actionable business intelligence from raw historical transaction records. By transforming transactional data into strategic insights, the project directly addresses operational inefficiencies, enhances marketing precision, and drives revenue expansion.
The analytical framework is engineered to accelerate business growth across four distinct domains:

1. Customer Retention & Lifecycle Maximization
   Cohort Tracking: Maps initial purchase dates to quantify long-term retention and repeat-purchase behavior.
   VIP Engagement: Isolates high-value customers contributing to the top 20% of historical revenue.
   Growth Impact: Enables marketing teams to deploy targeted VIP loyalty initiatives and reduce customer churn.
   
2. High-Yield Precision Marketing
   Demographic Profitability:Segregates profit margins across tailored age groups and gender categories.
   Targeted Campaigning: Identifies specific customer pockets generating over 40% net profit margins.
   Growth Impact: Optimizes advertising spend by eliminating broad, low-ROI campaigns in favor of high-yield segments.

3. Data-Driven Operational Optimization
   Time-Block Evaluation: Audits average order values across four specific, 24-hour business intervals.
   Resource Allocation: Informs precise store staffing, inventory replenishment, and time-sensitive promotions.
   Growth Impact: Minimizes overhead costs during dead hours while capturing peak-hour revenue potential.

4.Macro Financial Health & Forecasting
    MoM Growth Metrics: Tracks Month-over-Month revenue fluctuations and year-over-year seasonal peaks.
    Strategic Planning: Establishes key performance indicator (KPI) baselines for executive financial reviews.
    Growth Impact: Empowers executives to forecast cash flows, manage supply chains, and set realistic growth targets.


