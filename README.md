# Sales-Analysis-Project

# Overview

# Business Questions
1.
2. **Cohort Analysis**: How do different customer groups generate revenue
3.  
# Analysis Approach

# Cohort Analysis
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

# Strategic Recommendations

# Technical Details
