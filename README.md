# 📊 E-Commerce Sales & Location Performance Dashboard (2025)
An end-to-end data analytics project showcasing how raw e-commerce data is extracted from a cloud data warehouse, transformed using advanced SQL logic, and turned into actionable executive-level insights via Power BI.

## 🎯 Project Overview
This project empowers business stakeholders to monitor regional performance, analyze seasonal sales trends, and track monthly revenue metrics. By combining structured relational data with dynamic visualizations, leadership can quickly identify peak sales months and top-performing market locations to optimize marketing spend and supply chain efficiency.

## 🛠️ Tech Stack & Tools
* **Data Warehouse:** Google BigQuery
* **Data Transformation:** SQL (CTEs, Window Functions, Aggregations)
* **BI & Visualization:** Power BI Desktop

## 🖼️ Dashboard Preview
<img width="627" height="663" alt="image" src="https://github.com/user-attachments/assets/c93e7e94-fc63-439a-adf9-2e6723104315" />

## 📂 Project Structure
* `Monthly Sale.csv` - The raw dataset used for this analysis.
* `Monthly_Sales_and_location Report.pbix` - The core Power BI project file.

## ⚙️ Data Pipeline & Transformation Process

### Phase 1: Advanced SQL Engineering (BigQuery)
To maintain a high-performance and lightweight Power BI data model, data was pre-aggregated directly in Google BigQuery. The SQL pipeline uses multi-layered **Common Table Expressions (CTEs)** and **Window Functions** to solve critical business questions:

1. **`total_sale` CTE:** Joins `order_items`, `orders`, and `users` tables, filters records for the year 2025, and aggregates total monthly sales and distinct order counts per country.
2. **`Rank_location` CTE:** Implements a `DENSE_RANK() OVER (PARTITION BY sales_month ORDER BY Total_sales_price DESC)` window function to dynamically rank countries based on revenue contribution each month.
---

## 💻 How to Run the SQL Script
The optimized database query code used to drive this entire report can be found below:

```sql
with total_sale as (
select
	usr.country,
	sum(od.sale_price) as Total_sales_price,
	FORMAT_TIMESTAMP('%B', od.created_at) AS sales_month,
	COUNT(DISTINCT o.order_id) AS country_orders
from
	warehouse2016.order_items as od
inner join warehouse2016.orders as o 
on
	od.order_id = o.order_id
inner join warehouse2016.users as usr on
	usr.id = o.user_id
where
	FORMAT_TIMESTAMP('%Y',od.created_at) = '2025'
group by
	sales_month,
	country
	),
Rank_location as (
select
	ts.country,
	ts.sales_month ,
	ts.total_sales_price,
	ts.country_orders,
	DENSE_RANK() OVER (PARTITION BY ts.sales_month
ORDER BY
	ts.country DESC) AS location_rank
from
	total_sale as ts)
select
	fin.country,
	fin.sales_month,
	SUM(CAST(fin.Total_sales_price AS FLOAT64)) AS total_monthly_revenue,
	SUM(fin.country_orders) as total_country_order,
	fin.location_rank
from
	Rank_location as fin
group by
	fin.sales_month,
	fin.country,
	fin.location_rank


