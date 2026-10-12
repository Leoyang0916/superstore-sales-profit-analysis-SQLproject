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

sql
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
```

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

### Q2. How do sales, profit, and discount rates trend by year and quarter?
* **Objective**: Track high-level historical performance and seasonal trend fluctuations.
```sql
SELECT 
    YEAR(order_date) AS order_year,
    QUARTER(order_date) AS order_quarter,
    COUNT(DISTINCT order_id) AS total_orders,
    ROUND(SUM(sales), 2) AS total_sales,
    ROUND(SUM(profit), 2) AS total_profit,
    ROUND(AVG(discount) * 100, 2) AS avg_discount_pct
FROM stg_superstore
WHERE order_date IS NOT NULL
GROUP BY YEAR(order_date), QUARTER(order_date)
ORDER BY order_year ASC, order_quarter ASC;
```
Result:
| order_year | order_quarter | total_orders | total_sales | total_profit | avg_discount_pct |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 2014 | 1 | 131 | 74447.86 | 3811.20 | 15.74% |
| 2014 | 2 | 201 | 86538.77 | 11204.16 | 14.55% |
| 2014 | 3 | 267 | 143633.18 | 12804.73 | 15.51% |
| 2014 | 4 | 370 | 179627.75 | 21723.97 | 16.76% |
| 2015 | 1 | 144 | 68851.74 | 9264.94 | 14.60% |
| 2015 | 2 | 214 | 89124.28 | 12190.92 | 17.16% |
| 2015 | 3 | 274 | 130259.51 | 16853.70 | 15.21% |
| 2015 | 4 | 406 | 182296.93 | 23309.13 | 15.25% |
| 2016 | 1 | 179 | 93237.20 | 11441.39 | 15.20% |
| 2016 | 2 | 294 | 136082.33 | 16390.30 | 16.09% |
| 2016 | 3 | 378 | 143787.43 | 15823.63 | 16.08% |
| 2016 | 4 | 464 | 236098.90 | 38139.95 | 14.69% |
| 2017 | 1 | 240 | 123144.84 | 23506.21 | 15.12% |
| 2017 | 2 | 367 | 133764.33 | 15499.42 | 17.17% |
| 2017 | 3 | 448 | 196251.94 | 26985.26 | 14.46% |
| 2017 | 4 | 632 | 280054.08 | 27448.88 | 15.88% |

### Q3. What are the Top 10 best-selling products by revenue, and are they all profitable?
* **Objective**: Evaluate whether top revenue-generating items contribute positively to bottom-line profits or suffer from margin erosion.
```sql
SELECT 
    product_id,
    product_name,
    category,
    sub_category,
    SUM(quantity) AS total_quantity_sold,
    ROUND(SUM(sales), 2) AS total_sales,
    ROUND(SUM(profit), 2) AS total_profit
FROM stg_superstore
GROUP BY product_id, product_name, category, sub_category
ORDER BY total_sales DESC
LIMIT 10;
```
Result:
| product_id | product_name | category | sub_category | total_quantity_sold | total_sales | total_profit |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| TEC-CO-10004722 | Canon imageCLASS 2200 Advanced Copier | Technology | Copiers | 20 | 61599.83 | 25199.94 |
| OFF-BI-10003527 | Fellowes PB500 Electric Punch Plastic Comb Binding System | Office Supplies | Binders | 31 | 27453.38 | 7753.06 |
| TEC-MA-10002412 | Cisco TelePresence System EX90 Videoconferencing System | Technology | Machines | 6 | 22638.48 | -1811.08 |
| FUR-CH-10002024 | HON 5400 Series Task Chairs for Big and Tall | Furniture | Chairs | 39 | 21870.57 | 0.00 |
| OFF-BI-10001359 | GBC DocuBind TL300 Electric Binding System | Office Supplies | Binders | 37 | 19823.48 | 2233.50 |
| OFF-BI-10000545 | GBC Ibimaster 500 Manual ProClick Binding System | Office Supplies | Binders | 48 | 19024.50 | 760.98 |
| TEC-CO-10001449 | Hewlett Packard LaserJet 3310 Copier | Technology | Copiers | 38 | 18839.68 | 6983.89 |
| TEC-MA-10001127 | HP Designjet T520 Inkjet Large Format Printer | Technology | Machines | 12 | 18374.90 | 4094.98 |
| OFF-BI-10004995 | GBC DocuBind P400 Electric Binding System | Office Supplies | Binders | 27 | 17965.07 | -1878.17 |
| OFF-SU-10000151 | High Speed Automatic Electric Letter Opener | Office Supplies | Supplies | 11 | 17030.31 | -262.00 |

### Q4. Which shipping modes suffer from high shipping latency and delays?
* **Objective**: Evaluate average shipping lead time across delivery modes and identify delayed shipment rates (orders with lead time > 5 days).

```sql
SELECT 
    ship_mode,
    COUNT(DISTINCT order_id) AS total_orders,
    ROUND(AVG(DATEDIFF(ship_date, order_date)), 2) AS avg_shipping_days,
    COUNT(DISTINCT CASE WHEN DATEDIFF(ship_date, order_date) > 5 THEN order_id END) AS delayed_orders,
    ROUND(
        COUNT(DISTINCT CASE WHEN DATEDIFF(ship_date, order_date) > 5 THEN order_id END) * 100.0 / COUNT(DISTINCT order_id), 
        2
    ) AS delayed_rate_pct
FROM stg_superstore
GROUP BY ship_mode
ORDER BY avg_shipping_days DESC;
```
Result:
| ship_mode | total_orders | avg_shipping_days | delayed_orders | delayed_rate_pct |
| :--- | :--- | :--- | :--- | :--- |
| Standard Class | 2994 | 5.01 | 904 | 30.19% |
| Second Class | 964 | 3.24 | 0 | 0.00% |
| First Class | 787 | 2.18 | 0 | 0.00% |
| Same Day | 264 | 0.04 | 0 | 0.00% |

### Q5. Which product sub-categories are overall unprofitable?
* **Objective**: Audit product sub-categories operating at a net cumulative loss to identify primary drivers of negative margins.
```sql
SELECT 
    category,
    sub_category,
    ROUND(SUM(sales), 2) AS total_sales,
    ROUND(SUM(profit), 2) AS total_profit,
    ROUND(AVG(discount) * 100, 2) AS avg_discount_pct,
    ROUND((SUM(profit) / SUM(sales)) * 100, 2) AS profit_margin_pct
FROM stg_superstore
GROUP BY category, sub_category
HAVING total_profit < 0
ORDER BY total_profit ASC;
```
Result:
| category | sub_category | total_sales | total_profit | avg_discount_pct | profit_margin_pct |
| :--- | :--- | :--- | :--- | :--- | :--- |
| Furniture | Tables | 206965.68 | -17725.59 | 26.13% | -8.56% |
| Furniture | Bookcases | 114880.05 | -3472.56 | 21.11% | -3.02% |
| Office Supplies | Supplies | 46673.52 | -1188.99 | 7.68% | -2.55% |

### Q6. At what discount threshold do orders become deeply unprofitable?
* **Objective**: Group discount rates into tiers to uncover the critical tipping point where promotional discounts lead to widespread order losses.
```sql
SELECT 
    CASE 
        WHEN discount = 0 THEN '0% (No Discount)'
        WHEN discount > 0 AND discount <= 0.10 THEN '0% - 10%'
        WHEN discount > 0.10 AND discount <= 0.20 THEN '10% - 20%'
        WHEN discount > 0.20 AND discount <= 0.30 THEN '20% - 30%'
        ELSE '30%+'
    END AS discount_tier,
    COUNT(DISTINCT order_id) AS total_orders,
    ROUND(SUM(sales), 2) AS total_sales,
    ROUND(SUM(profit), 2) AS total_profit,
    COUNT(DISTINCT CASE WHEN profit < 0 THEN order_id END) AS loss_orders,
    ROUND(COUNT(DISTINCT CASE WHEN profit < 0 THEN order_id END) * 100.0 / COUNT(DISTINCT order_id), 2) AS loss_order_rate_pct
FROM stg_superstore
GROUP BY discount_tier
ORDER BY MIN(discount) ASC;
```
Result:
| discount_tier | total_orders | total_sales | total_profit | loss_orders | loss_order_rate_pct |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 0% (No Discount) | 2644 | 1087908.47 | 320987.88 | 0 | 0.00% |
| 0% - 10% | 89 | 54369.30 | 9029.21 | 4 | 4.49% |
| 10% - 20% | 2436 | 792152.87 | 91757.14 | 464 | 19.05% |
| 20% - 30% | 211 | 103226.76 | -10369.34 | 195 | 92.42% |
| 30%+ | 888 | 259543.67 | -125007.10 | 872 | 98.20% |

### Q7. What are the Top 3 selling product sub-categories in each region?
* **Objective**: Rank product sub-categories by sales revenue within each geographic region using SQL Window Functions (`DENSE_RANK`).
```sql
WITH regional_sales AS (
    SELECT 
        region,
        sub_category,
        ROUND(SUM(sales), 2) AS total_sales,
        DENSE_RANK() OVER (
            PARTITION BY region 
            ORDER BY SUM(sales) DESC
        ) AS sales_rank
    FROM stg_superstore
    GROUP BY region, sub_category
)
SELECT 
    region,
    sales_rank,
    sub_category,
    total_sales
FROM regional_sales
WHERE sales_rank <= 3
ORDER BY region, sales_rank;
```
Result:
| region | sales_rank | sub_category | total_sales |
| :--- | :--- | :--- | :--- |
| Central | 1 | Chairs | 85230.68 |
| Central | 2 | Phones | 72403.26 |
| Central | 3 | Binders | 56923.26 |
| East | 1 | Phones | 100615.02 |
| East | 2 | Chairs | 96260.65 |
| East | 3 | Storage | 71612.58 |
| South | 1 | Phones | 58304.43 |
| South | 2 | Machines | 53890.98 |
| South | 3 | Chairs | 45176.44 |
| West | 1 | Chairs | 101781.36 |
| West | 2 | Phones | 98684.39 |
| West | 3 | Tables | 84754.60 |

### Q8. What is the customer repeat purchase rate within 90 days of their first order?
* **Objective**: Measure initial customer retention and short-term repeat purchasing behavior to evaluate early customer lifetime value.
```sql
WITH customer_orders AS (
    SELECT 
        customer_id,
        order_id,
        order_date,
        MIN(order_date) OVER (PARTITION BY customer_id) AS first_order_date
    FROM stg_superstore
),
repeat_analysis AS (
    SELECT 
        customer_id,
        MAX(CASE WHEN DATEDIFF(order_date, first_order_date) BETWEEN 1 AND 90 THEN 1 ELSE 0 END) AS is_repeat_90d
    FROM customer_orders
    GROUP BY customer_id
)
SELECT 
    COUNT(customer_id) AS total_customers,
    SUM(is_repeat_90d) AS repeat_customers_90d,
    ROUND(SUM(is_repeat_90d) * 100.0 / COUNT(customer_id), 2) AS repeat_rate_90d_pct
FROM repeat_analysis;
```
Result:
| total_customers | repeat_customers_90d | repeat_rate_90d_pct |
| :--- | :--- | :--- |
| 793 | 181 | 22.82% |

### Q9. How can we segment customers using an RFM (Recency, Frequency, Monetary) model?
* **Objective**: Segment customer base into distinct value groups using SQL `NTILE(4)` quartiles to inform targeted retention and loyalty marketing strategies.
```sql
WITH max_date AS (
    SELECT MAX(order_date) AS anchor_date FROM stg_superstore
),
rfm_raw AS (
    SELECT 
        s.customer_id,
        s.customer_name,
        DATEDIFF(m.anchor_date, MAX(s.order_date)) AS recency_days,
        COUNT(DISTINCT s.order_id) AS frequency_count,
        ROUND(SUM(s.sales), 2) AS monetary_value
    FROM stg_superstore s
    CROSS JOIN max_date m
    GROUP BY s.customer_id, s.customer_name, m.anchor_date
),
rfm_scores AS (
    SELECT 
        customer_id,
        customer_name,
        recency_days,
        frequency_count,
        monetary_value,
        NTILE(4) OVER (ORDER BY recency_days DESC) AS r_score,
        NTILE(4) OVER (ORDER BY frequency_count ASC) AS f_score,
        NTILE(4) OVER (ORDER BY monetary_value ASC) AS m_score
    FROM rfm_raw
),
rfm_segmented AS (
    SELECT 
        customer_id,
        customer_name,
        recency_days,
        frequency_count,
        monetary_value,
        r_score,
        f_score,
        m_score,
        CASE 
            WHEN r_score >= 3 AND m_score >= 3 THEN 'VIP Champions (高價值核心客戶)'
            WHEN r_score <= 2 AND m_score >= 3 THEN 'At-Risk High Value (高消費潛在流失)'
            WHEN r_score >= 3 AND m_score <= 2 THEN 'Recent Low Spend (新進/低消費客戶)'
            ELSE 'Hibernating (沉睡客戶)'
        END AS customer_segment
    FROM rfm_scores
)
SELECT 
    customer_segment,
    COUNT(customer_id) AS total_customers,
    ROUND(SUM(monetary_value), 2) AS segment_total_sales,
    ROUND(AVG(recency_days), 1) AS avg_recency_days,
    ROUND(AVG(frequency_count), 1) AS avg_frequency_orders,
    ROUND(AVG(monetary_value), 2) AS avg_monetary_spend
FROM rfm_segmented
GROUP BY customer_segment
ORDER BY segment_total_sales DESC;
```
Result:
| customer_segment | total_customers | segment_total_sales | avg_recency_days | avg_frequency_orders | avg_monetary_spend |
| :--- | :--- | :--- | :--- | :--- | :--- |
| VIP Champions (高價值核心客戶) | 229 | 1048074.68 | 32.3 | 7.9 | 4576.75 |
| At-Risk High Value (高消費潛在流失) | 167 | 791005.42 | 214.0 | 7.1 | 4736.56 |
| Hibernating (沉睡客戶) | 230 | 251891.08 | 294.8 | 4.7 | 1095.18 |
| Recent Low Spend (新進/低消費客戶) | 167 | 206229.89 | 32.8 | 5.6 | 1234.91 |


## 7. Key Findings & Strategic Recommendations

### 1. Executive Revenue & Profit Drivers
* **Regional Dominance**: The **West Region** is the company's primary growth engine, with its Consumer segment leading all markets in both sales ($362,880.82) and net profit ($57,450.69).
* **Q4 Seasonality Surge**: Revenue demonstrates strong Q4 holiday seasonality, scaling consistently year-over-year to hit an all-time peak of **$280,054.08 sales** in 2017 Q4 (a >55% growth compared to 2014 Q4).
* **Hero Products**: The **Canon imageCLASS 2200 Copier** is the single highest profit driver, generating $61,599.83 in sales and $25,199.94 in net profit (~41% profit margin).

---

### 2. Profit Leakage & Discount Trap Audit
* **The 20% Discount Tipping Point**:
  * Discounts below **20%** maintain high profitability ($320.9k profit at 0% discount; $91.7k profit at 10%–20% discount).
  * Once discounts exceed **20%**, the loss-order rate skyrockets to **92.42%**.
  * At **30%+ discounts**, **98.20% of orders operate at a loss**, accumulating a massive cumulative deficit of **-$125,007.10**.
* **Unprofitable Sub-Categories**:
  * **Tables** (-$17,725.59 profit, -8.56% margin) and **Bookcases** (-$3,472.56 profit, -3.02% margin) are deeply unprofitable due to excessive average discounting (26.13% and 21.11% respectively).
* **Top Revenue vs. Profit Paradox**: 
  * 3 of the Top 10 revenue-generating products (e.g., *Cisco TelePresence EX90* at -$1,811.08) operate at net financial losses.
  * The *HON 5400 Series Task Chair* generated $21,870.57 in revenue but yielded **$0.00 net profit**.

---

### 3. Operational & Logistics Bottlenecks
* **Standard Class Latency**: While First Class, Second Class, and Same Day experience zero delays over 5 days, **Standard Class suffers a 30.19% delay rate** (904 out of 2,994 orders exceeding 5 shipping days).
* **Central Region Profit Margin Anomaly**: Despite generating substantial revenue ($252,031.43 in Consumer), the Central region yields disproportionately low profit ($8,564.05), driven by localized discounting and operational inefficiencies.

---

### 4. Customer Retention & RFM Behavioral Segmentation
* **Early Retention Baseline**: The 90-day repeat purchase rate stands at **22.82%** (181 out of 793 total customers), indicating a strong opportunity to improve initial onboarding and early engagement.
* **RFM Value Concentration**:
  * **VIP Champions (229 customers)**: Drive **$1,048,074.68** in revenue (~37.2% of total evaluated sales) with an average spend of $4,576.75 and an impressive average recency of 32.3 days.
  * **At-Risk High Value (167 customers)**: Represent **$791,005.42** in historical revenue with high average spending ($4,736.56), but have not purchased in an average of **214 days**.

---

### 5. Strategic Business Recommendations

1. **Implement Hard Discount Guardrails**:
   * Enforce a hard ceiling capping promotional discounts at **15–20%**. Any discount >20% must require manager approval to eliminate the -$125k loss trap.
2. **Restructure Loss-Making Product Lines**:
   * Reprice or renegotiate vendor costs for **Tables** and **Bookcases**. Audit bundle promotions for high-revenue loss products like the Cisco EX90 Videoconferencing System.
3. **Win Back At-Risk High Value Customers**:
   * Launch targeted email/phone outreach offering exclusive VIP incentives for the **167 At-Risk High Value customers** to reactivate $791k+ in potential recurring revenue.
4. **Optimize Standard Logistics Operations**:
   * Re-evaluate carrier fulfillment agreements for **Standard Class shipping** in peak seasons to reduce the 30.19% delivery delay rate.
