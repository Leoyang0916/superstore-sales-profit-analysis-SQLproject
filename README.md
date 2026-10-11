# superstore-sales-profit-analysis-SQLproject
# Author: Yen-Ting Yang
# SQL Tools: MySQL and MySQL Workbench
# Visualization Tool: Power BI (Planning)

This project uses Sample Superstore retail data to analyze the distribution of sales and profit, identify products and markets that need priority checking, and explore the relationship between discounts and profitability. By organizing data with SQL, performing grouped comparisons, and visualizing results in Power BI, the query results are turned into actionable business recommendations ready for discussion and validation. 

# 1. Business Problem
This project simulates a business analysis scenario for a retail company and is not an actual consulting project commissioned by a company. The assumption is that the retail manager wants to know which products and markets drive sales growth and which product combinations drag down profit. Looking at sales alone might overlook high-sales but low-profit subcategories and cannot identify whether issues are concentrated in specific regions or discount ranges. Therefore, this analysis evaluates sales, profit, and profit margin simultaneously, gradually diving from overall performance into product, region, and discount dimensions to help prioritize follow-up investigations and policy trials.

# 2. Approach
First, the original imported data is retained in MySQL, then checked for completeness, field formats, missing values, and duplicates. Next, suitable analysis tables are created, converting text dates and amounts into the correct types and standardizing column names. After data prep is complete, SQL is used to compare profit performance across overall, time, product, and region levels and explore differences across discount ranges. Later, Power BI will be used to present the results, with this README documenting key SQL, results, and business interpretations.

This project uses Sample Superstore retail data to analyze the distribution of sales and profit, identify products and markets that need priority checking, and explore the relationship between discounts and profitability. By organizing data with SQL, performing grouped comparisons, and visualizing results in Power BI, the query results are turned into actionable business recommendations ready for discussion and validation.

# 3. Analyzing the Issues - Key Questions
1. What are the total sales, profit, profit margin, and number of orders?
2. How do sales and profit change month by month and year by year?
3. Which product categories have high sales but low profit margins?
4. Which product subcategories are overall unprofitable?
5. Are losses concentrated in specific regions or states?
6. How do profit margins and loss details differ across different discount ranges?

# 4. Dataset

Data source: [Kaggle Superstore Dataset](https://www.kaggle.com/datasets/vivek468/superstore-dataset-final?utm_source=chatgpt.com). The dataset size and date range below were calculated from the CSV used in this project.

| Item | Description |
| --- | --- |
| Original file | `Sample - Superstore.csv` |
| Number of records | 9,994 order line items |
| Number of original columns | 21 |
| Number of orders | 5,009 distinct orders |
| Order date range | 2014-01-03 to 2017-12-30 |
| Raw data table | `superstore_project.superstore_import` |

**Data granularity: Each row represents an individual order line item. A single order may contain multiple rows.** Order counts are calculated using distinct `Order ID` values; the number of rows should not be treated as the number of orders.

### Key Fields

| Field | Purpose |
| --- | --- |
| `Row ID` | Identifies each line item |
| `Order ID`, `Order Date` | Order counting and time-based analysis |
| `Customer ID`, `Segment` | Customer and customer segment comparisons |
| `Region`, `State`, `City` | Market and geographic comparisons |
| `Product ID`, `Category`, `Sub-Category` | Product and category analysis |
| `Sales` | Sales amount for each line item |
| `Quantity` | Quantity purchased for each line item |
| `Discount` | Discount rate; for example, 0.20 represents 20% |
| `Profit` | Profit reported in the dataset for each line item |

### KPI Definitions

| Metric | Calculation |
| --- | --- |
| Total Sales | `SUM(Sales)` |
| Total Profit | `SUM(Profit)` |
| Overall Profit Margin | `SUM(Profit) / SUM(Sales) × 100%` |
| Order Count | `COUNT(DISTINCT Order ID)` |
| Average Order Value | Total Sales / Order Count |
| Loss-Making Line Item Percentage | Number of line items with `Profit < 0` / Total number of line items × 100% |

Overall profit margin is calculated by dividing total profit by total sales, rather than averaging the profit margins of individual line items. The costs included in `Profit` must be determined from the dataset documentation. This project does not assume that `Profit` represents the company's net income.

# 5. Data Preparation & Cleaning
5. Data Preparation & Cleaning (SQL Staging)

Before diving into analysis, raw string data in `superstore_import` was cleaned and transformed into a structured staging table `stg_superstore`. Key operations included:
- **String Cleaning**: Removed currency symbols (`$`) and thousand separators (`,`) using `REPLACE()` to prevent `CAST` failures.
- **Date Parsing**: Converted text dates into standardized `DATE` format using `STR_TO_DATE()`.
- **Type Casting**: Converted financial metrics into `DECIMAL(10,2)` and quantities into `UNSIGNED INT`.

```sql
DROP TABLE IF EXISTS stg_superstore;

CREATE TABLE stg_superstore AS
SELECT 
    `Order ID` AS order_id,
    STR_TO_DATE(`Order Date`, '%m/%d/%Y') AS order_date,
    STR_TO_DATE(`Ship Date`, '%m/%d/%Y') AS ship_date,
    `Ship Mode` AS ship_mode,
    `Customer ID` AS customer_id,
    `Customer Name` AS customer_name,
    Segment AS segment,
    Region AS region,
    State AS state,
    City AS city,
    Category AS category,
    `Sub-Category` AS sub_category,
    `Product ID` AS product_id,
    `Product Name` AS product_name,
    CAST(REPLACE(REPLACE(Sales, '$', ''), ',', '') AS DECIMAL(10, 2)) AS sales,
    CAST(Quantity AS UNSIGNED) AS quantity,
    CAST(REPLACE(REPLACE(Discount, '$', ''), '%', '') AS DECIMAL(4, 2)) AS discount,
    CAST(REPLACE(REPLACE(Profit, '$', ''), ',', '') AS DECIMAL(10, 2)) AS profit
FROM superstore_import;

# 6. Data Analysis & Findings
---

### Q1. What are the sales, profit, and Average Order Value (AOV) performance across regions and customer segments?

* **Objective**: Evaluate revenue contribution and average basket size per order across different market segments.

```sql
SELECT 
    region,
    segment,
    COUNT(DISTINCT order_id) AS total_orders,
    SUM(quantity) AS total_items_sold,
    ROUND(SUM(sales), 2) AS total_sales,
    ROUND(SUM(profit), 2) AS total_profit,
    ROUND(SUM(sales) / COUNT(DISTINCT order_id), 2) AS avg_order_value
FROM stg_superstore
GROUP BY region, segment
ORDER BY region, total_sales DESC;

Result:

| region | segment | total_orders | total_items_sold | total_sales | total_profit | avg_order_value |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| Central | Consumer | 604 | 4519 | 252031.43 | 8564.05 | 417.27 |
| Central | Corporate | 348 | 2604 | 157995.79 | 18703.90 | 454.01 |
| Central | Home Office | 223 | 1657 | 91212.66 | 12438.50 | 409.03 |
| East | Consumer | 713 | 5347 | 350908.17 | 41190.96 | 492.16 |
| East | Corporate | 434 | 3358 | 200409.45 | 23622.60 | 461.77 |
| East | Home Office | 254 | 1913 | 127463.74 | 26709.28 | 501.83 |
| South | Consumer | 444 | 3178 | 195580.92 | 26913.63 | 440.50 |
| South | Corporate | 247 | 2016 | 121885.95 | 15215.40 | 493.47 |
| South | Home Office | 131 | 1015 | 74255.03 | 4620.68 | 566.83 |
| West | Consumer | 825 | 6477 | 362880.82 | 57450.69 | 439.86 |
| West | Corporate | 485 | 3630 | 225855.25 | 34437.55 | 465.68 |
| West | Home Office | 301 | 2159 | 136721.86 | 16530.55 | 454.23 |
