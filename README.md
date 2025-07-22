## Customer Lifetime Value (CLV) Analysis

### Project Overview

This project presents a refined Customer Lifetime Value (CLV) analysis using a cohort-based approach. Unlike previous CLV methods based on the Shopify formula, which only considers purchasers, this analysis includes all website visitors and tracks their purchasing behavior over a 12-week period. This provides a more complete and realistic view of customer value and long-term engagement.

### Data Source

- Dataset: `turing_data_analytics.raw_events`
- Cohort Period: 12 weeks
- Reference Date: 2021-01-24 (last weekly cohort in the dataset)

### Key Analysis Steps

1. **Registration Cohort Creation**  
   Extracted each user’s first event week (registration week) using `user_pseudo_id`.

2. **Revenue Data Extraction**  
   Filtered for purchase events and ensured all event dates were properly formatted.

3. **Cohort CLV Calculation**  
   Calculated Average Revenue Per User (ARPU) for each cohort on a weekly basis.

4. **Cumulative CLV Calculation**  
   Tracked cumulative revenue per user over 12 weeks to assess long-term value.

5. **Future CLV Projection**  
   Applied cumulative growth rates to forecast revenue behavior for each cohort beyond the current window.

### Tools Used

- BigQuery SQL for querying and transforming raw event data
- Excel for ARPU calculations, cumulative CLV, and forecasting
- PowerPoint for creating a professional presentation of key findings

### Key Metrics

- Average Revenue Per User (ARPU) by cohort and week
- Cumulative Revenue Per User over a 12-week horizon
- Projected future revenue based on historical cohort growth trends

### Visualizations

- Weekly ARPU by cohort
- Cumulative CLV tables with running totals
- Forecasted revenue projections based on observed growth rates

### SQL Query Used

```sql
-- SQL Query for CLV Analysis
WITH
  FirstVisit AS (
    SELECT
      DISTINCT user_pseudo_id AS User, 
      MIN(PARSE_DATE('%Y%m%d', event_date)) AS RegDate, 
      MIN(event_timestamp) AS FirstVisit 
    FROM
      `tc-da-1.turing_data_analytics.raw_events`
    GROUP BY
      1  
  ),

  Purchases AS (
    SELECT
      user_pseudo_id AS User,  
      PARSE_DATE('%Y%m%d', event_date) AS PurchaseDate,  
      purchase_revenue_in_usd AS Revenue  
    FROM
      `tc-da-1.turing_data_analytics.raw_events`
    WHERE
      event_name = 'purchase'  
      AND purchase_revenue_in_usd > 0  
  ),

  CohortData AS (
    SELECT
      f.User AS FUser,  
      DATE_TRUNC(RegDate, week) AS RegWeek,  
      p.User AS PUser,  
      DATE_TRUNC(PurchaseDate, week) AS PurchaseWeek,  
      p.Revenue  
    FROM
      FirstVisit f  
    LEFT JOIN
      Purchases p  
    ON
      f.User = p.User 
  )

SELECT
  RegWeek,  
  COUNT(FUser) AS Registrations,  
  SUM(CASE WHEN PurchaseWeek = RegWeek THEN Revenue ELSE 0 END) AS Week0,  
  SUM(CASE WHEN PurchaseWeek = DATE_ADD(RegWeek, INTERVAL 1 WEEK) THEN Revenue ELSE 0 END) AS Week1,  
  SUM(CASE WHEN PurchaseWeek = DATE_ADD(RegWeek, INTERVAL 2 WEEK) THEN Revenue ELSE 0 END) AS Week2,  
  SUM(CASE WHEN PurchaseWeek = DATE_ADD(RegWeek, INTERVAL 3 WEEK) THEN Revenue ELSE 0 END) AS Week3,  
  SUM(CASE WHEN PurchaseWeek = DATE_ADD(RegWeek, INTERVAL 4 WEEK) THEN Revenue ELSE 0 END) AS Week4,  
  SUM(CASE WHEN PurchaseWeek = DATE_ADD(RegWeek, INTERVAL 5 WEEK) THEN Revenue ELSE 0 END) AS Week5,  
  SUM(CASE WHEN PurchaseWeek = DATE_ADD(RegWeek, INTERVAL 6 WEEK) THEN Revenue ELSE 0 END) AS Week6,  
  SUM(CASE WHEN PurchaseWeek = DATE_ADD(RegWeek, INTERVAL 7 WEEK) THEN Revenue ELSE 0 END) AS Week7,  
  SUM(CASE WHEN PurchaseWeek = DATE_ADD(RegWeek, INTERVAL 8 WEEK) THEN Revenue ELSE 0 END) AS Week8,  
  SUM(CASE WHEN PurchaseWeek = DATE_ADD(RegWeek, INTERVAL 9 WEEK) THEN Revenue ELSE 0 END) AS Week9,  
  SUM(CASE WHEN PurchaseWeek = DATE_ADD(RegWeek, INTERVAL 10 WEEK) THEN Revenue ELSE 0 END) AS Week10,  
  SUM(CASE WHEN PurchaseWeek = DATE_ADD(RegWeek, INTERVAL 11 WEEK) THEN Revenue ELSE 0 END) AS Week11,
  SUM(CASE WHEN PurchaseWeek = DATE_ADD(RegWeek, INTERVAL 12 WEEK) THEN Revenue ELSE 0 END) AS Week12  
FROM
  CohortData  
WHERE
  RegWeek < '2021-01-31'  
GROUP BY
  RegWeek 
ORDER BY
  RegWeek;


### Excel File: `CLV_Analysis.xlsx`

This Excel workbook includes:

- Registration week cohorts and user counts  
- Weekly revenue values from Week 0 to Week 12  
- Weekly ARPU per cohort  
- Cumulative revenue and projected revenue per user  

### PDF File: `CLV_Analysis.pdf`

This presentation highlights:

- Revenue trends and drop-offs after initial registration  
- Cohort-level differences in retention and revenue growth  
- A slowdown in cumulative growth after Week 7  
- Strategic recommendations for improving engagement and CLV outcomes  

### Insights

- Most revenue is generated in the first week after registration, with sharp declines in later weeks  
- Early user engagement is a key driver of long-term value  
- Newer cohorts underperformed older ones, indicating a potential shift in acquisition quality or onboarding  
- Forecasting revealed diminishing returns beyond Week 7, emphasizing the importance of early re-engagement  

### Future Improvements

- Apply segmentation based on acquisition source, geography, or device type  
- Incorporate Customer Acquisition Cost (CAC) to assess net profitability  
- Experiment with machine learning models to predict future customer value more accurately  
