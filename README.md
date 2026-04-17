# data-analysis-portfolio
Data analysis project : SQL for data extraction and cleaning, combined with Tableu for interactive dashboarding. Focusing on perfomance metrics analysis and data-driven insights
![SQL](https://img.shields.io/badge/SQL-4479A1?style=for-the-badge&logo=postgresql&logoColor=white)
![Tableau](https://img.shields.io/badge/Tableau-E97627?style=for-the-badge&logo=tableau&logoColor=white)


Customer Retention & Acquisition Analysis
Project Overview
This analysis provides a comprehensive understanding of customer behavior,focusing on Cohort Retention and Acquisition Channel Effectiveness.
The aim was to identify trends in customer loyalty all over time and determine which marketing channel deliver the highest long term revenue.
--

Technical Stack
* Database: SQL( for data extraction,transformation and optimization)
* Visualizaton: Tableau (for dashboard design,cohort heatmaps)
* Data Methodolgy: Ravenue formula (Current MRR / Initial Month 0 MRR) * 100
  --


Step 1: Define the Cohort for each account on their signup date 
```
WITH account_cohorts As (
  Select 
    account_id,
    referral_source,
    DATE_FORMAT(signup_date , '%Y-%m-01')  -- Normalize signup_date to the first month of the year
as cohort_month
   From ravenstack_accounts r 
   )
 Select * from account_cohorts ;
```
<img width="545" height="293" alt="Снимок экрана 2026-04-17 в 19 37 11" src="https://github.com/user-attachments/assets/95994d77-7aff-4282-b206-3a40178c0c1b" />


Step 2: Calculate the lifecycle month for each subscription payment
```
monthly_revenue_lifecycle AS (
    SELECT 
        ac.cohort_month,
        ac.referral_source,
        PERIOD_DIFF(
            DATE_FORMAT(s.start_date, '%Y%m'),
            DATE_FORMAT(ac.cohort_month, '%Y%m')
        ) AS month_number,
        s.mrr_amount,
        ac.account_id
    FROM account_cohorts ac
    JOIN ravenstack_subscriptions s 
        ON ac.account_id = s.account_id
    WHERE s.mrr_amount > 0
)

SELECT * 
FROM monthly_revenue_lifecycle;
```
<img width="807" height="293" alt="Снимок экрана 2026-04-17 в 19 38 21" src="https://github.com/user-attachments/assets/3637912d-87b7-4749-8f52-0f63e1451632" />

Step 3: Aggregate revenue metrics by cohort and month
```
cohort_summary AS (
    SELECT
        cohort_month,
        month_number,
        referral_source,
        SUM(mrr_amount) AS total_mrr,
        COUNT(DISTINCT account_id) AS active_customers
    FROM monthly_revenue_lifecycle
    WHERE month_number >= 0
    GROUP BY cohort_month, month_number, referral_source
)

SELECT *
FROM cohort_summary;

<img width="822" height="375" alt="Снимок экрана 2026-04-17 в 18 26 09" src="https://github.com/user-attachments/assets/ed918140-74b6-47bb-803f-3cfad1c251c1" />
```
<img width="822" height="375" alt="Снимок экрана 2026-04-17 в 18 26 09" src="https://github.com/user-attachments/assets/b612578c-9c1c-4f3e-9fde-17f6937612d1" />

Step 4: Final output with Revenue Retention percantage
```

SELECT 
    cs.cohort_month,
    cs.month_number,
    cs.referral_source,
    cs.total_mrr,
    cs.active_customers,

    ROUND(
        100.0 * cs.total_mrr /             
        FIRST_VALUE(cs.total_mrr) OVER (
            PARTITION BY cs.cohort_month, cs.referral_source   
            ORDER BY cs.month_number
        ),
        2
    ) AS revenue_retention_pct

FROM cohort_summary cs
ORDER BY cs.cohort_month DESC, cs.month_number ASC;
```

<img width="822" height="375" alt="Снимок экрана 2026-04-17 в 13 18 27" src="https://github.com/user-attachments/assets/86a95ebb-8244-40ec-976a-a0619e723304" />
