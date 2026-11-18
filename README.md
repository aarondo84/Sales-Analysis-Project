# Sales-Analysis-Project

# Overview
Analysis of customer behavior, retention, and lifetime value for an e-commerce company to improve customer retention and maximize revenue.

# Business Questions
1. **Customer Segmentation**: Who are our most valuable customers?
2. **Cohort Analysis**: How do different customer groups generate revenue?
3. **Retention Analysis**: Which customers have not purchased recently?
# Analysis Approach
# 1. Customer Segmentation
-Categorize customers based on LTV(lifetime value)

-Assign customers to Low, Mid, and High value groups 
```
with customer_ltv as (
select 
customerkey,
cleaned_name,
sum(total_net_revenue) as total_ltv
from cohort_analysis ca 
group by
customerkey,
cleaned_name
), customer_segments as(

select
percentile_cont(0.25) within group (order by total_ltv) as ltv_25th_percentile,
percentile_cont(0.50) within group (order by total_ltv) as ltv_50th_percentile,
percentile_cont(0.75) within group (order by total_ltv) as ltv_75th_percentile
from customer_ltv
), segment_values as (

select c.*,
case 
	when c.total_ltv < cs.ltv_25th_percentile then '1- Low-Value'
	when c.total_ltv <= cs.ltv_75th_percentile then '2- Mid-Value'
	else '3 - High-Value'
end as customer_segment

from customer_ltv c,
customer_segments cs
)

select 
customer_segment,
sum(total_ltv) as total_ltv, 
count(customerkey) as customer_count,  
sum(total_ltv) / count(customerkey) as avg_ltv
from segment_values 
group by 
customer_segment
```
<img width="609" height="90" alt="image" src="https://github.com/user-attachments/assets/46b971ac-bf4c-4176-a040-9547daeee37d" />


<img width="1080" height="1014" alt="image" src="https://github.com/user-attachments/assets/b6b3a6ff-bf01-46d9-a0b8-c8d1af091912" />

Key Findings:

1.Low Value(25% of customers) accounts for 2% of revenue(4.3M)
-12,372 customers (25% of 49,487 total customers)

-4.3M/ 206.4M total revenue =2%

-$350 is average LTV for low-value customer

2.Mid Value(50% of customers) account for 32% of revenue(66.6M)

-24,743 customers (50% of 49,487 total customers)

-66.6M/206.4M total revenue= 32%

-$2700 is the average LTV for mid-value customer

3.High Value( 25% of customer) accounts for 66% of revenue(135.4M)

-12,372 customers (25% of 49,487 total customers)

-135.4/206.4 total revenue= 66%

-$10946 is the average LTV for high-value customer

Business Insights

1.Low Value 

-Work on engagement campaigns to increase customer frequency

-Focus on converting more customers into mid value customers

-Look into price promotions or deals to increase frequency of purchases

2.Mid Value

-Create personalized promotions or discounts for customers

-Put product recommendations based on purchasing patterns

-Potential revenue increase to high value customers

3.High Value

-Provide early access to new products and dedicated support

-focus on customer retention to maintain revenue 

-Look into Membership programs or special deals for high value customers

# 2. Cohort Analysis
-Tracked revenue and customer count per cohorts

-Cohorts were grouped by year of first purchase

-Analyzed customer retention at cohort level
```
SELECT
    cohort_year,
    SUM(total_net_revenue) AS total_revenue,
    COUNT(DISTINCT customerkey) AS total_customers,
    SUM(total_net_revenue) / COUNT(DISTINCT customerkey) AS customer_revenue
FROM cohort_analysis
GROUP BY 
    cohort_year
```
<img width="1980" height="1180" alt="image" src="https://github.com/user-attachments/assets/2e2ab1f0-cb13-4d3c-993d-c63bbc3596e6" />

Key Finding: Revenue per customer shows a decrease over time, more recent cohorts are performing worse than previous cohorts. Net revenue in increasing due to customer base but not reflective of customer value.

Business Insight: Value extracted from specific customers are decreasing over time. Customer retention rates need to be looked into more, with lowering LTC and customer acquisition this could lead to potential revenue declines

# 3. Customer Retention
```
WITH customer_last_purchase AS (
    SELECT
        customerkey,
        orderdate,
        ROW_NUMBER() OVER (PARTITION BY customerkey ORDER BY orderdate DESC) AS rn,
        first_purchase_date,
        cohort_year
    FROM cohort_analysis
),
churned_customers AS (
    SELECT
        customerkey,
        orderdate AS last_purchase_date,
        cohort_year,
        CASE
            WHEN orderdate < (SELECT MAX(orderdate) FROM sales) - INTERVAL '6 months' THEN 'Churned'
            ELSE 'Active'
        END AS customer_status
    FROM customer_last_purchase
    WHERE rn = 1
        AND first_purchase_date < (SELECT MAX(orderdate) FROM sales) - INTERVAL '6 months'
)
SELECT
    cohort_year,
    customer_status,
    COUNT(customerkey) AS num_customers,
    SUM(COUNT(customerkey)) OVER(PARTITION BY cohort_year) AS total_customers,
    ROUND(COUNT(customerkey) / SUM(COUNT(customerkey)) OVER(PARTITION BY cohort_year), 2) AS cohort_percentage
FROM churned_customers
GROUP BY
    cohort_year,
    customer_status
ORDER BY
    cohort_year,
    customer_status;
```
-Identified customers at risk of churning

-Analyzed last purchase patterns 

-Calculated customer specific metrics

<img width="1387" height="1097" alt="image" src="https://github.com/user-attachments/assets/e1f46772-6e19-4304-9ba9-61fe33d03554" />

Key Findings: Cohort churn stabilizes at 90% after 2-3 years, shows a long term retention pattern, retention rates are low at 8-10% across all cohorts, future cohorts will follow the same pattern without intervention.

Business Insights: look into early engagement strategies to target first 2 years with incentives, engage more with high value customers, be more proactive with intervening at risk users before losing retention.


# Strategic Recommendations

1. Customer Value Optimization (Customer Segmentation)

-Create personalized upgrade paths for mid-value segment

-Design price-sensitive promotions for low-value segment to increase purchase frequency


2. Cohort Performance Strategy (Customer Revenue by Cohort)

-Target 2022-2024 cohorts with personalized re-engagement offers

-Apply successful strategies from high-spending 2016-2018 cohorts to newer customers

-Potential revenue increase to high value customers

3. Retention & Churn Prevention (Customer Retention)

-Strengthen first 1-2 year engagement with onboarding incentives

-re-engage with high value customers

-Implement proactive intervention system for at-risk customers

# Technical Details
-PostgreSQL, Dbeaver, Chatgpt
