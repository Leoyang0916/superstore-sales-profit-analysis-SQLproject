# superstore-sales-profit-analysis-SQLproject
# Author: Yen-Ting Yang
# SQL Tools: MySQL and MySQL Workbench
# Visualization Tool: Power BI

This project uses Sample Superstore retail data to analyze the distribution of sales and profit, identify products and markets that need priority checking, and explore the relationship between discounts and profitability. By organizing data with SQL, performing grouped comparisons, and visualizing results in Power BI, the query results are turned into actionable business recommendations ready for discussion and validation. 

# 1. Business Problem
This project simulates a business analysis scenario for a retail company and is not an actual consulting project commissioned by a company. The assumption is that the retail manager wants to know which products and markets drive sales growth and which product combinations drag down profit. Looking at sales alone might overlook high-sales but low-profit subcategories and cannot identify whether issues are concentrated in specific regions or discount ranges. Therefore, this analysis evaluates sales, profit, and profit margin simultaneously, gradually diving from overall performance into product, region, and discount dimensions to help prioritize follow-up investigations and policy trials.

# 2. Approach
First, the original imported data is retained in MySQL, then checked for completeness, field formats, missing values, and duplicates. Next, suitable analysis tables are created, converting text dates and amounts into the correct types and standardizing column names. After data prep is complete, SQL is used to compare profit performance across overall, time, product, and region levels and explore differences across discount ranges. Later, Power BI will be used to present the results, with this README documenting key SQL, results, and business interpretations.

This project uses Sample Superstore retail data to analyze the distribution of sales and profit, identify products and markets that need priority checking, and explore the relationship between discounts and profitability. By organizing data with SQL, performing grouped comparisons, and visualizing results in Power BI, the query results are turned into actionable business recommendations ready for discussion and validation.
