# 📊 E-Commerce Sales & Location Performance Dashboard (2025)
An end-to-end data analytics project showcasing how raw e-commerce data is extracted from a cloud data warehouse, transformed using advanced SQL logic, and turned into actionable executive-level insights via Power BI.

## 🎯 Project Overview
This project empowers business stakeholders to monitor regional performance, analyze seasonal sales trends, and track monthly revenue metrics. By combining structured relational data with dynamic visualizations, leadership can quickly identify peak sales months and top-performing market locations to optimize marketing spend and supply chain efficiency.

## 🛠️ Tech Stack & Tools
* **Data Warehouse:** Google BigQuery
* **Data Transformation:** SQL (CTEs, Window Functions, SAFE_CAST, Aggregations)
* **BI & Visualization:** Power BI Desktop

## 🖼️ Dashboard Preview
> *Replace the placeholder link below with your actual dashboard screenshot image from your repository.*
![E-Commerce Sales Dashboard](YOUR_IMAGE_LINK_HERE)

---

## ⚙️ Data Pipeline & Transformation Process

### Phase 1: Advanced SQL Engineering (BigQuery)
To maintain a high-performance and lightweight Power BI data model, data was pre-aggregated directly in Google BigQuery. The SQL pipeline uses multi-layered **Common Table Expressions (CTEs)** and **Window Functions** to solve critical business questions:

1. **`total_sale` CTE:** Joins `order_items`, `orders`, and `users` tables, filters records for the year 2025, and aggregates total monthly sales and distinct order counts per country.
2. **`Rank_location` CTE:** Implements a `DENSE_RANK() OVER (PARTITION BY sales_month ORDER BY Total_sales_price DESC)` window function to dynamically rank countries based on revenue contribution each month.
---

## 💻 How to Run the SQL Script
The optimized database query code used to drive this entire report can be found below:

```sql
WITH total_sale AS (
    SELECT
        usr.country,
        SUM(SAFE_CAST(od.sale_price AS FLOAT64)) AS Total_sales_price,
        FORMAT_TIMESTAMP('%m-%Y', od.created_at) AS sales_month,
        COUNT(DISTINCT o.order_id) AS country_orders
    FROM
        warehouse2016.order_items AS od
    INNER JOIN warehouse2016.orders AS o ON od.order_id = o.order_id
    INNER JOIN warehouse2016.users AS usr ON usr.id = o.user_id
    WHERE
        FORMAT_TIMESTAMP('%Y', od.created_at) = '2025'
    GROUP BY
        usr.country,
        FORMAT_TIMESTAMP('%m-%Y', od.created_at)
),
Rank_location AS (
    SELECT
        ts.country,
        ts.sales_month,
        ts.Total_sales_price,
        ts.country_orders,
        DENSE_RANK() OVER (
            PARTITION BY ts.sales_month 
            ORDER BY ts.Total_sales_price DESC
        ) AS location_rank
    FROM
        total_sale AS ts
)
SELECT
    fin.country,
    fin.sales_month,
    SUM(fin.Total_sales_price) AS total_monthly_revenue,
    SUM(fin.country_orders) AS total_country_order,
    fin.location_rank
FROM
    Rank_location AS fin
GROUP BY
    fin.sales_month,
    fin.country,
    fin.location_rank
ORDER BY 
    fin.sales_month, 
    fin.location_rank;
