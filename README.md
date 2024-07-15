# AtliQ Mart’s Diwali & Sankranti Promotions!

This repository contains the SQL scripts used to analyze the performance of promotional campaigns run by AtliQ Mart during Diwali 2023 and Sankranti 2024. The project addresses various business requests related to identifying high-value discounted products, store distribution, campaign effectiveness, and product performance in terms of incremental sales and revenue.

## Introduction

Promotional campaigns play a crucial role in the retail industry, driving sales and attracting customers during festive seasons. This project aims to analyze the performance of promotional campaigns conducted by AtliQ Mart during Diwali 2023 and Sankranti 2024. By leveraging data analytics, we seek to gain insights into the effectiveness of these campaigns and provide recommendations for optimizing future marketing strategies.

## Data Sources

The analysis is based on data obtained from AtliQ Mart's internal databases. The main datasets used include fact_events, dim_products, dim_stores, and sales_summary. These datasets contain information about product sales, store locations, promotional events, and campaign revenues.

## Project Overview:

1. Analyzed data from AtliQ Mart's internal databases.
2. Performed SQL queries to fulfill five business requests.
3. Insights are intended to inform future promotional strategies and resource allocation.

## Business Requests

### 1. High-Value Products in 'BOGOF' Promotion

**Objective:** Identify high-value products featured in the 'BOGOF' (Buy One Get One Free) promotion.

```sql
select 
      p.product_code,
      p.product_name,
      p.category,
      fe.promo_type
from fact_events fe
join dim_products p
on
      fe.product_code=p.product_code
where fe.base_price>500 
and promo_type="BOGOF";
```

### 2. Store Presence Overview

**Objective:** Provide an overview of the number of stores in each city.

```sql
select
      city,
      count(store_id) as store_count
from dim_stores
group by city
order by store_count desc;
```

### 3. Promotional Campaign Revenue Analysis
**Objective:** Display total revenue generated before and after each promotional campaign.

```sql
with CampaignRevenueCTE as (SELECT 
        c.campaign_name,
        sum(fe.base_price * fe.quantity_sold_before_promo) AS revenue_before_promo,
        sum(fe.base_price * fe.quantity_sold_after_promo) AS revenue_after_promo
FROM dim_campaigns c
JOIN fact_events fe 
ON 
         c.campaign_id = fe.campaign_id
group by c.campaign_name)

select 
      campaign_name,
      (revenue_before_promo)/1000000 as Total_revenue_before_promo,
      (revenue_after_promo)/1000000 as Total_revenue_after_promo
from CampaignRevenueCTE;
```

### 4. Incremental Sold Quantity Analysis during Diwali Campaign
**Objective:** Calculate Incremental Sold Quantity (ISU%) for each category during the Diwali campaign.

```sql
with  DiwaliCategoryMetrics as (select 
     p.category,
     sum(fe.quantity_sold_after_promo - fe.quantity_sold_before_promo) as incremental_sold_quantity,
     sum(fe.quantity_sold_before_promo) as baseline_sold_quantity
from dim_campaigns c
join fact_events fe
on 
	   c.campaign_id=fe.campaign_id
join dim_products p
on 
	   p.product_code=fe.product_code
where c.campaign_name="diwali"
group by p.category)

select 
     category,
     round((incremental_sold_quantity)/(baseline_sold_quantity)*100,2) as ISU_pct,
     RANK() OVER (ORDER BY ROUND((incremental_sold_quantity / baseline_sold_quantity) * 100, 2)) AS rank_order
from DiwaliCategoryMetrics
order by rank_order;
```

### 5. Top 5 Products by Incremental Revenue Percentage
**Objective:** Identify the top 5 products ranked by Incremental Revenue Percentage (IR%) across all campaigns.

```sql
SELECT 
    product_name,
    category,
    ROUND((SUM(CASE
            WHEN promo_type = 'BOGOF' THEN base_price * 0.5 * (`quantity_sold_after_promo` * 2)
            WHEN promo_type = '500 Cashback' THEN (base_price - 500) * `quantity_sold_after_promo`
            WHEN promo_type = '50% OFF' THEN base_price * 0.5 * `quantity_sold_after_promo`
            WHEN promo_type = '33% OFF' THEN base_price * 0.67 * `quantity_sold_after_promo`
            WHEN promo_type = '25% OFF' THEN base_price * 0.75 * `quantity_sold_after_promo`
            ELSE 0
           END) 
           - SUM(base_price * `quantity_sold_before_promo`)) / SUM(base_price * `quantity_sold_before_promo`) * 100,2) AS `IR%`
      FROM
      fact_events
      JOIN
      dim_products USING (product_code)
      GROUP BY product_name , category
      ORDER BY `IR%` DESC
      LIMIT 5;
```

## Limitations and Challenges

One significant limitation encountered during the analysis is related to the handling of promotions with the 'BOGOF' (Buy One Get One Free) promotion type. The dataset does not accurately account for the quantity of the free item provided as part of the promotion. This limitation may lead to some discrepancies or misunderstandings in the analysis, particularly when evaluating the effectiveness of 'BOGOF' promotions and comparing them with other promotion types.

## Results and Insights

The analysis revealed several key insights:

- High-value products featured in 'BOGOF' promotions.
- Distribution of stores across different cities.
- Total revenue generated before and after each promotional campaign.
- Incremental sold quantity and revenue percentage during the Diwali campaign.
- Top 5 products ranked by incremental revenue percentage.

These insights can help AtliQ Mart make informed decisions for future promotional activities, optimize resource allocation, and improve overall sales performance.

## Conclusion

Overall, the analysis provides valuable insights into the performance of promotional campaigns conducted by AtliQ Mart during Diwali 2023 and Sankranti 2024. By leveraging data analytics, AtliQ Mart can enhance its marketing strategies, attract more customers, and drive higher sales during festive seasons.

## Additional Insights

In addition to the main business requests, the following recommended insights were explored during the analysis:

### Store Performance Analysis

- **Top 10 Stores by Incremental Revenue (IR):** Identify the top-performing stores in terms of incremental revenue generated from promotions.
- **Bottom 10 Stores by Incremental Sold Units (ISU):** Identify the stores with the lowest performance in terms of incremental sold units during the promotional period.
- **City-wise Store Performance:** Analyze how store performance varies by city and identify any common characteristics among top-performing stores.

### Promotion Type Analysis

- **Top 2 Promotion Types by Incremental Revenue:** Determine the top-performing promotion types that resulted in the highest incremental revenue.
- **Bottom 2 Promotion Types by Incremental Sold Units:** Identify the least effective promotion types in terms of their impact on incremental sold units.
- **Comparison of Promotion Types:** Analyze the performance differences between discount-based promotions, BOGOF (Buy One Get One Free), and cashback promotions.
- **Optimal Promotion Type:** Determine which promotions strike the best balance between incremental sold units and maintaining healthy margins.

### Product and Category Analysis

- **High-Lifting Product Categories:** Identify product categories that saw significant increases in sales from the promotions.
- **Product Responsiveness to Promotions:** Analyze specific products that respond exceptionally well or poorly to promotions.
- **Correlation between Product Category and Promotion Type Effectiveness:** Investigate the relationship between product categories and the effectiveness of different promotion types.

## Visualizations

Explore the live Power BI dashboard for interactive visualizations:

## Future Work

Future work could include:
- Exploring additional datasets to gain deeper insights into customer behavior and preferences.
- Conducting more granular analysis on specific product categories or regions.
- Implementing machine learning models for predictive analytics to forecast sales and optimize promotional strategies.


