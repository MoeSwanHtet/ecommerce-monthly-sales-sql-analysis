# ecommerce-monthly-sales-sql-analysis

This project focuses on extracting monthly sales data and analyzing Month-on-Month (MoM) revenue trends for the year 2025. By evaluating historical data, this analysis helps the business team track growth trajectories and understand seasonal purchasing patterns.

### 💼 Business Problem
The management team needs a data-driven report to compare each month's total revenue directly with the previous month's performance. The objective is to identify which months experienced revenue growth or decline, without manually shifting rows in Excel.

### 📊 Data Source
The dataset used for this analysis is the **TheLook E-commerce** public dataset, hosted on **Google BigQuery**. 
* **Source:** [Google Cloud Public Datasets - TheLook E-commerce](https://console.cloud.google.com/marketplace/product/bigquery-public-data/thelook-ecommerce)
* **Dataset ID:** `bigquery-public-data.thelook_ecommerce`
* **Table Used:** `warehouse2016.order_items` (Contains transaction records, sale prices, and shipping statuses)

### 🛠️ SQL Technical Approach
To achieve a clean and optimized data extraction, the query utilizes advanced SQL techniques:
* **Common Table Expressions (CTEs):** Used `with sale_2025 as (...)` to aggregate and filter data first, breaking down a complex problem into readable steps.
* **Data Formatting:** Applied `FORMAT_TIMESTAMP` to standardize the timeline into a `YYYY-MM` format.
* **Window Functions (`LAG`):** Utilized the `LAG()` function ordered by the timeline (`ASC`) to dynamically pull the previous month's name and revenue into the current row for seamless comparison.

### 💻 The SQL Code
```sql
with sale_2025 as (select 
FORMAT_TIMESTAMP('%d-%m-%Y', created_at) AS sales_month,
sum (sale_price) as total_sale_price
from warehouse2016.order_items
where FORMAT_TIMESTAMP('%Y', created_at) = '2025'
and status = 'Shipped'
group by sales_month ),
previous_month as (
select 
sales_month as months,
total_sale_price,
LAG(sales_month, 1) OVER (ORDER BY sales_month ASC) AS previous_months,
LAG(total_sale_price, 1) OVER (ORDER BY sales_month ASC) AS previous_month_revenue
from sale_2025
)
select * from previous_month;
