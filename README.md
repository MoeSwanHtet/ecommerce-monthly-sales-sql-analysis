with sale_2025 as (select 
FORMAT_TIMESTAMP('%Y-%m', created_at) AS sales_month,
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

