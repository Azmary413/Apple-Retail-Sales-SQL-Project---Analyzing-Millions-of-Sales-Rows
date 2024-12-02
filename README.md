# Apple Retail Sales SQL Project Analyzing Millions of Sales Rows
![banner](https://github.com/Azmary413/Apple-Retail-Sales-SQL-Project---Analyzing-Millions-of-Sales-Rows/blob/main/background.jpg)

### Overview
This project involves analyzing millions of rows of sales data for Apple retail stores to derive actionable insights, optimize queries, and improve data performance using SQL. The dataset is designed to simulate a realistic sales environment, encompassing stores, products, categories, sales, and warranty claims.

### The project includes:

1. Data schema design and table creation.
2. Exploratory Data Analysis (EDA).
3. Query optimizations using indexing to enhance performance.
4. Solving key business problems through SQL queries.

## Key Features
- **Database Schema:** A well-structured relational database comprising five tables: stores, products, categories, sales, and warranty.
- **Performance Optimization:** Created indexes on frequently queried columns, improving query execution time significantly for large datasets.
- **Business Problem Analysis:** Tackles real-world business scenarios like sales trends, warranty claims analysis, and product performance.
- **Advanced Querying:** Includes window functions, subqueries, Common Table Expressions (CTEs), and aggregate functions.

## Database Schema
### Tables
1. `stores`: Contains information about store locations, including `store_id`, `store_name`, `city`, and `country`.
2. `category`: Represents product categories, identified by `category_id` and `category_name`.
3. `products`: Stores product details like `product_id`, `product_name`, `category_id`, `launch_date`, and `price`.
4. `sales`: Tracks sales transactions with details like `sale_id`, `sale_date`, `store_id`, `product_id`, and `quantity`.
5. `warranty`: Logs warranty claims with `claim_id`, c`laim_date`, `sale_id`, and `repair_status`.


## Performance Optimization
To handle millions of rows and ensure high performance, indexes were created on the following columns:

1. `sales(product_id)`: Improved query execution time for product-based sales analysis.
2. `sales(store_id)`: Enhanced performance when filtering sales by store.
3. `sales(sale_date)`: Accelerated date-based queries for identifying trends and seasonal patterns.
4. `sales(quantity)`: Boosted queries requiring analysis of sales volume thresholds.


## Example Performance Improvement

#### Filtering by Product ID
- Query: `SELECT * FROM sales WHERE product_id = 'P-40'; `

  - **Without Index:** Execution Time ~260 ms
  - **With Index:** Execution Time ~161 ms
 
#### Before Indexing:
  ![EXPLAIN Before Index](https://github.com/Azmary413/Apple-Retail-Sales-SQL-Project---Analyzing-Millions-of-Sales-Rows/blob/main/index%20before%20sales_quantity.jpg)

#### After Indexing:
 ![EXPLAIN Before Index](https://github.com/Azmary413/Apple-Retail-Sales-SQL-Project---Analyzing-Millions-of-Sales-Rows/blob/main/index%20after%20sales_quantity.jpg)

#### Filtering by Sale Date
- Query: `SELECT * FROM sales WHERE sale_date = '2020-04-18'; `

  - **Without Index:** Execution Time ~133 ms
  - **With Index:** Execution Time ~0.9 ms

#### Before Indexing:
 ![EXPLAIN Before Index](https://github.com/Azmary413/Apple-Retail-Sales-SQL-Project---Analyzing-Millions-of-Sales-Rows/blob/main/index%20befor%20sale_date.jpg)

 #### After Indexing:
 ![EXPLAIN Before Index](https://github.com/Azmary413/Apple-Retail-Sales-SQL-Project---Analyzing-Millions-of-Sales-Rows/blob/main/index%20after%20sale_date.jpg)
   
Indexes significantly reduced execution time, ensuring scalable performance for large datasets.

## Business Problems Solved
Here are some of the key questions and insights derived:

1. **Stores by Country:** Identify the number of stores in each country.
2. **Total Units Sold by Store:** Calculate the total sales volume per store.
3. **Seasonal Sales Trends:** Analyze sales patterns in December 2023.
4. **Warranty Claims Analysis:**
   - Percentage of rejected claims.
   - Claims filed within 180 days of product sale.
   - Products with the highest claims in the last two years.

5. **Sales Performance:**
   - Best-selling day for each store.
   - Year-over-year growth ratio for each store.
   - Monthly running totals over four years.
     
6. **Product Analysis:**
   - Average price per category.
   - Least-selling products by country and year.
   - Correlation between product price and warranty claims.



The project is split into three tiers of questions to test SQL skills of increasing complexity:

### Easy to Medium (10 Questions)
1. Find the number of stores in each country.
2. Calculate the total number of units sold by each store.
3. Identify how many sales occurred in December 2023.
4. Determine how many stores have never had a warranty claim filed.
5. Calculate the percentage of warranty claims marked as "Warranty Void".
6. Identify which store had the highest total units sold in the last year.
7. Count the number of unique products sold in the last year.
8. Find the average price of products in each category.
9. How many warranty claims were filed in 2020?
10. For each store, identify the best-selling day based on highest quantity sold.
### Medium to Hard (5 Questions)
11. Identify the least selling product in each country for each year based on total units sold.
12. Calculate how many warranty claims were filed within 180 days of a product sale.
13. Determine how many warranty claims were filed for products launched in the last two years.
14. List the months in the last three years where sales exceeded 5,000 units in the USA.
15. Identify the product category with the most warranty claims filed in the last two years.
### Complex (5 Questions)
16. Determine the percentage chance of receiving warranty claims after each purchase for each country.
17. Analyze the year-by-year growth ratio for each store.
18. Calculate the correlation between product price and warranty claims for products sold in the last five years, segmented by price range.
19. Identify the store with the highest percentage of "Paid Repaired" claims relative to total claims filed.
20. Write a query to calculate the monthly running total of sales for each store over the past four years and compare trends during this period.

## SQL Queries and Analysis
This project contains a variety of SQL queries structured around Exploratory Data Analysis (EDA) and Business Problem Solving. Some of the key SQL queries include:

### Exploratory Data Analysis

```sql

SELECT DISTINCT repair_status FROM warranty;
SELECT DISTINCT store_name FROM stores;
SELECT DISTINCT category_name FROM category;
```

Count of Total Sales:

```sql
SELECT COUNT(*) AS total_sales FROM sales;
```
Execution Plan Optimization: Using EXPLAIN ANALYZE to optimize query performance by creating necessary indexes:

```sql
CREATE INDEX sales_product_id ON sales(product_id);
EXPLAIN ANALYZE `SELECT * FROM sales WHERE product_id = 'P-40';`
```

## Business Problem Queries

**Identify the least selling product in each country for each year based on total units sold.**

```sql
WITH product_rank AS(
SELECT 
st.country, 
EXTRACT(YEAR FROM sale_date) AS sale_year, 
p.product_name, 
SUM(quantity) AS total_unit_sold,
RANK() OVER(PARTITION BY country, EXTRACT(YEAR FROM sale_date) ORDER BY SUM(quantity)) AS least_sold_product
FROM products AS p
JOIN sales AS s
ON p.product_id = s.product_id
JOIN stores AS st
ON st.store_id = s.store_id
GROUP BY st.country, EXTRACT(YEAR FROM sale_date), p.product_name
)
SELECT * FROM product_rank
WHERE least_sold_product = 1;

```
#### Result:


**Determine the percentage chance of receiving warranty claims after each purchase for each country.**

```sql
SELECT 
    country,
    total_sold,
    total_claim,
    ROUND((total_claim::numeric) / (total_sold::numeric) * 100, 2) AS percentage_of_risk
FROM (
    SELECT
        st.country,
        SUM(s.quantity) AS total_sold,
        COUNT(w.claim_id) AS total_claim
    FROM stores AS st
    JOIN sales AS s
        ON s.store_id = st.store_id
    JOIN warranty AS w
        ON s.sale_id = w.sale_id
    GROUP BY st.country
) AS subquery
ORDER BY percentage_of_risk DESC;
```

**Analyze the year-by-year growth ratio for each store.**

```sql
WITH yearly_sale AS (
    SELECT 
        st.store_id,
        st.store_name,
        EXTRACT(YEAR FROM s.sale_date) AS year_of_sale,
        SUM(s.quantity * p.price) AS total_sale
    FROM stores AS st
    JOIN sales AS s
        ON s.store_id = st.store_id
    JOIN products AS p
        ON p.product_id = s.product_id
    GROUP BY st.store_id, st.store_name, year_of_sale
    ORDER BY st.store_id, year_of_sale
),
growth_ratio AS (
    SELECT 
        store_name,
        year_of_sale,
        total_sale,
        LAG(total_sale) OVER (PARTITION BY store_name ORDER BY year_of_sale) AS prev_sale
    FROM yearly_sale
)
SELECT
    store_name,
    year_of_sale,
    prev_sale,
    total_sale AS curr_sale,
    ROUND(((total_sale - prev_sale)::numeric /prev_sale::numeric) * 100, 2) AS growth_ratio_YOY
FROM growth_ratio
WHERE prev_sale IS NOT NULL
ORDER BY store_name, year_of_sale;
```

**Calculate the correlation between product price and warranty claims for products sold in the last five years, segmented by price range.**

```sql
SELECT 
    CASE 
        WHEN p.price < 500 THEN 'Low Cost'
        WHEN p.price BETWEEN 500 AND 1000 THEN 'Moderate Cost'
        ELSE 'High Cost'
    END AS price_segment,
    c.category_name,
    COUNT(w.claim_id) AS total_claims,
    COUNT(s.sale_id) AS total_sales,
    ROUND((COUNT(w.claim_id)::NUMERIC / NULLIF(COUNT(s.sale_id), 0)) * 100, 2) AS claim_rate_percentage,
    ROUND(AVG(p.price)::NUMERIC, 2) AS avg_price_in_segment
FROM products AS p
JOIN sales AS s
    ON p.product_id = s.product_id
LEFT JOIN warranty AS w
    ON w.sale_id = s.sale_id
JOIN category AS c
    ON p.category_id = c.category_id
WHERE s.sale_date >= CURRENT_DATE - INTERVAL '5 years'
GROUP BY price_segment, c.category_name
ORDER BY claim_rate_percentage DESC;
```


**Write a query to calculate the monthly running total of sales for each store over the past four years and compare trends during this period.**

```sql
WITH monthly_sales AS (
    SELECT
        s.store_id,
        EXTRACT(YEAR FROM s.sale_date) AS year,
        EXTRACT(MONTH FROM s.sale_date) AS month,
        SUM(p.price * s.quantity) AS total_sales
    FROM sales AS s
    JOIN products AS p
        ON s.product_id = p.product_id
    WHERE s.sale_date >= CURRENT_DATE - INTERVAL '4 years'
    GROUP BY s.store_id, year, month
    ORDER BY s.store_id, year, month
)
SELECT
    store_id, 
    year, 
    month, 
    total_sales, 
    SUM(total_sales) OVER(PARTITION BY store_id ORDER BY year, month) AS running_total
FROM monthly_sales
ORDER BY store_id, year, month;
```


## Technologies Used
- Database: PostgreSQL
- SQL Concepts:
    - Window Functions
    - Common Table Expressions (CTEs)
    - Indexing
    - Subqueries and Joins
    - Aggregate Functions


## How to Use
1. Clone the repository to your local machine.
2. Execute the SQL script to create the database schema and populate tables.
3. Run the provided queries to explore insights and solve business problems.

## Future Improvements
- Automate indexing based on query logs.
- Implement partitioning for the sales table to further enhance performance.
- Integrate visualizations to represent trends and insights.


## Key Learnings
- **SQL Performance Optimization:** How to use indexing and EXPLAIN ANALYZE to speed up query execution.
- **Business Problem Solving with SQL:** Applying SQL to solve real-world business questions related to sales performance, warranty claims, and product analysis.
- **Data Insights:** Extracting actionable insights from large datasets to drive decisions in retail management.

## Contributions
Contributions are welcome! If you find any improvements or issues, feel free to fork this repository and submit a pull request.

