# SQL-Data-Analysis-Project
### a) Change Over Time Analysis
#### Purpose:
   ###### - To track trends, growth, and changes in key metrics over time.
   ###### - For time-series analysis and identifying seasonality.
   ######  - To measure growth or decline over specific periods.
#### SQL Functions Used:
   ###### - Date Functions: DATEPART(), DATETRUNC(), FORMAT()
   ###### - Aggregate Functions: SUM(), COUNT(), AVG()
#### Question: 
Analyse sales performance over time 
```bash
SELECT
    DATETRUNC(month, order_date) AS order_date,
    SUM(sales_amount) AS total_sales,
    COUNT(DISTINCT customer_key) AS total_customers,
    SUM(quantity) AS total_quantity
FROM gold.fact_sales
WHERE order_date IS NOT NULL
GROUP BY DATETRUNC(month, order_date)
ORDER BY DATETRUNC(month, order_date);
```
![image](https://github.com/user-attachments/assets/dee3fea2-79aa-46f3-8bbf-f7955acc75f1)

### b) Cumulative Analysis
#### Purpose:
   ###### - To calculate running totals or moving averages for key metrics.
   ###### - To track performance over time cumulatively.
   ###### - Useful for growth analysis or identifying long-term trends.
#### SQL Functions Used:
   ###### -- Window Functions: SUM() OVER(), AVG() OVER()
#### Question: 
Calculate the total sales per month and the running total of sales over time 
```bash
SELECT
	order_date,
	total_sales,
	SUM(total_sales) OVER (ORDER BY order_date) AS running_total_sales,
	AVG(avg_price) OVER (ORDER BY order_date) AS moving_average_price
FROM
(
    SELECT 
        DATETRUNC(month, order_date) AS order_date,
        SUM(sales_amount) AS total_sales,
        AVG(price) AS avg_price
    FROM gold.fact_sales
    WHERE order_date IS NOT NULL
    GROUP BY DATETRUNC(month, order_date)
) t
```
![image](https://github.com/user-attachments/assets/b3cba570-5180-4b8a-806c-fc398fd23c1a)

### c) Performance Analysis (Year-over-Year, Month-over-Month)
#### Purpose:
  ###### - To measure the performance of products, customers, or regions over time.
  ###### - For benchmarking and identifying high-performing entities.
  ###### - To track yearly trends and growth.

#### SQL Functions Used:
  ###### - LAG(): Accesses data from previous rows.
  ###### - AVG() OVER(): Computes average values within partitions.
  ###### - CASE: Defines conditional logic for trend analysis.

#### Question:
Analyze the yearly performance of products by comparing their sales to both the average sales performance of the product and the previous year's sales 
```bash 
WITH yearly_product_sales AS (
    SELECT
        YEAR(f.order_date) AS order_year,
        p.product_name,
        SUM(f.sales_amount) AS current_sales
    FROM gold.fact_sales f
    LEFT JOIN gold.dim_products p
        ON f.product_key = p.product_key
    WHERE f.order_date IS NOT NULL
    GROUP BY 
        YEAR(f.order_date),
        p.product_name
)
SELECT
    order_year,
    product_name,
    current_sales,
    AVG(current_sales) OVER (PARTITION BY product_name) AS avg_sales,
    current_sales - AVG(current_sales) OVER (PARTITION BY product_name) AS diff_avg,
    CASE 
        WHEN current_sales - AVG(current_sales) OVER (PARTITION BY product_name) > 0 THEN 'Above Avg'
        WHEN current_sales - AVG(current_sales) OVER (PARTITION BY product_name) < 0 THEN 'Below Avg'
        ELSE 'Avg'
    END AS avg_change,
    -- Year-over-Year Analysis
    LAG(current_sales) OVER (PARTITION BY product_name ORDER BY order_year) AS py_sales,
    current_sales - LAG(current_sales) OVER (PARTITION BY product_name ORDER BY order_year) AS diff_py,
    CASE 
        WHEN current_sales - LAG(current_sales) OVER (PARTITION BY product_name ORDER BY order_year) > 0 THEN 'Increase'
        WHEN current_sales - LAG(current_sales) OVER (PARTITION BY product_name ORDER BY order_year) < 0 THEN 'Decrease'
        ELSE 'No Change'
    END AS py_change
FROM yearly_product_sales
ORDER BY product_name, order_year;
```
![image](https://github.com/user-attachments/assets/b84ab631-7b09-4f69-9401-b3b29444ba79)

### d) Part-to-Whole Analysis
#### Purpose:
######    - To compare performance or metrics across dimensions or time periods.
######   - To evaluate differences between categories.
 ######    - Useful for A/B testing or regional comparisons.

#### SQL Functions Used:
 ######    - SUM(), AVG(): Aggregates values for comparison.
  ######   - Window Functions: SUM() OVER() for total calculations.

#### Question:
Which categories contribute the most to overall sales?
```bash
WITH category_sales AS (
    SELECT
        p.category,
        SUM(f.sales_amount) AS total_sales
    FROM gold.fact_sales f
    LEFT JOIN gold.dim_products p
        ON p.product_key = f.product_key
    GROUP BY p.category
)
SELECT
    category,
    total_sales,
    SUM(total_sales) OVER () AS overall_sales,
    ROUND((CAST(total_sales AS FLOAT) / SUM(total_sales) OVER ()) * 100, 2) AS percentage_of_total
FROM category_sales
ORDER BY total_sales DESC;
```
![image](https://github.com/user-attachments/assets/4e2a390c-da2e-40c3-a284-a296ae7a89d0)

### b) Cumulative Analysis
#### Purpose:
   ###### - To calculate running totals or moving averages for key metrics.
   ###### - To track performance over time cumulatively.
   ###### - Useful for growth analysis or identifying long-term trends.
#### SQL Functions Used:
   ###### -- Window Functions: SUM() OVER(), AVG() OVER()
#### Question: 
Calculate the total sales per month and the running total of sales over time 
```bash
