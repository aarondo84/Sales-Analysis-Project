# Sales-Analysis-Project

# Overview

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

# Strategic Recommendations

# Technical Details
