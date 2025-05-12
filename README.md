# 📊 Customer Lifetime Value (CLV) Analysis

## 🔍 Project Overview

This project focuses on a refined Customer Lifetime Value (CLV) analysis using a cohort-based approach. Unlike previous CLV calculations using the Shopify formula, this analysis considers all website visitors (not just purchasers) and tracks their spending behavior over a 12-week period, providing more reliable insights into customer value.

## 📂 Data Source

* **Dataset:** `turing_data_analytics.raw_events`
* **Cohort Period:** 12 weeks
* **Reference Date:** 2021-01-24 (last weekly cohort in the dataset)

## 🚀 Key Analysis Steps

1. **Registration Cohort Creation:**

   * Extracted each user's first event week (registration week) using `user_pseudo_id`.

2. **Revenue Data Extraction:**

   * Collected weekly user purchases, ensuring date format consistency.

3. **Cohort CLV Calculation:**

   * Calculated Average Revenue per User (ARPU) for each cohort week.

4. **Cumulative CLV Calculation:**

   * Calculated cumulative revenue per user over 12 weeks for each cohort.

5. **Future CLV Projection:**

   * Used cumulative growth % to predict future revenue for 12 weeks.

## 🛠️ Tools Used

* **BigQuery SQL:** For data extraction, transformation, and cohort creation.
* **Excel:** For calculating and visualizing cohort revenue and ARPU.
* **PowerPoint:** For creating a clean and visually engaging presentation of findings.

## 📊 Key Metrics

* **Average Revenue Per User (ARPU)** for each cohort.
* **Cumulative Revenue per User** over time.
* **Projected Future Revenue** using cumulative growth.

## 📊 Visualizations

* Cohort Table with Average Revenue per User.
* Cumulative CLV Table with running totals.
* Projected CLV Table showing predicted values for each cohort.

## 📌 SQL Query Used

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
  Where RegWeek < '2021-01-31'  
GROUP BY
  RegWeek 
ORDER BY
  RegWeek; 
```

## 📂 Excel File: CVL Analysis.xlsx

* **Content:** This file tracks revenue per cohort across 12 weeks, including:

  * `RegWeek`: Registration week of each cohort.
  * `Registrations`: Number of users in each cohort.
  * `Week0 - Week12`: Total revenue per cohort in each week.
  * `Week0.1 - Week12.1`: Average Revenue Per User (ARPU) per cohort.

## 📂 PDF File: CVL Analysis.pdf

* **Content:** Visual presentation of CLV analysis, highlighting:

  * Revenue decline trends, especially after Week 0.
  * Cohort performance differences (strong early cohorts, weak recent ones).
  * Cumulative growth slowing down beyond Week 7.
  * Actionable recommendations for improving user engagement and retention.

## 💡 Insights

* Identified trends in revenue growth across user cohorts.
* Analyzed how initial customer engagement influences long-term value.
* Provided a forecast of future user revenue using cumulative growth rates.

## 🚀 Future Improvements

* Explore advanced segmentation techniques for better CLV prediction.
* Compare CLV across different marketing channels.
* Integrate customer acquisition cost for more accurate profitability analysis.



