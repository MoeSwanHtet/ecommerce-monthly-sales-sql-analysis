# ecommerce-monthly-sales-sql-analysis

This project focuses on extracting monthly sales data and analyzing Month-on-Month (MoM) revenue trends for the year 2025. By evaluating historical data, this analysis helps the business team track growth trajectories and understand seasonal purchasing patterns.

### 💼 Business Problem
The management team needs a data-driven report to compare each month's total revenue directly with the previous month's performance. The objective is to identify which months experienced revenue growth or decline, without manually shifting rows in Excel.

### 🛠️ SQL Technical Approach
To achieve a clean and optimized data extraction, the query utilizes advanced SQL techniques:
* **Common Table Expressions (CTEs):** Used `with sale_2025 as (...)` to aggregate and filter data first, breaking down a complex problem into readable steps.
* **Data Formatting:** Applied `FORMAT_TIMESTAMP` to standardize the timeline into a `YYYY-MM` format.
* **Window Functions (`LAG`):** Utilized the `LAG()` function ordered by the timeline (`ASC`) to dynamically pull the previous month's name and revenue into the current row for seamless comparison.

### 💻 The SQL Code
```sql
WITH sale_2025 as (
    SELECT 
        FORMAT_TIMESTAMP('%Y-%m', created_at) AS sales_month, 
        SUM(sale_price) as total_sale_price 
    FROM warehouse2016.order_items 
    WHERE FORMAT_TIMESTAMP('%Y', created_at) = '2025' 
      AND status = 'Shipped' 
    GROUP BY sales_month 
), 
previous_month as ( 
    SELECT 
        sales_month as months, 
        total_sale_price, 
        LAG(sales_month, 1) OVER (ORDER BY sales_month ASC) AS previous_months, 
        LAG(total_sale_price, 1) OVER (ORDER BY sales_month ASC) AS previous_month_revenue 
    FROM sale_2025 
) 
SELECT * FROM previous_month;
