
# Finance Risk & Delinquency Analysis Project

This is a finance analytics project where I tried to understand how loan delinquency behaves month by month and what kind of early warning signs we can catch before the portfolio worsens.  
The idea was to make it feel as close to a real credit risk monitoring setup as possible, using **SQL**, **Excel**, and **Power BI** together.

---

## Project Summary

I built this project to track how loans move across delinquency stages like 30 DPD, 60 DPD, and 90+ DPD.  
The dataset is fully simulated but looks realistic enough to behave like an actual portfolio of around 10,000 monthly loan observations.

I wanted to see:
- How delinquency rates change month to month  
- Which products or regions are driving the spike  
- How many accounts roll from 30→60 or 60→90 (roll rate)  
- And how branch performance compares to the average  

Later I used Power BI to visualize all these things and add early warning indicators.

---

## Tools Used

| Tool | Purpose |
|------|----------|
| **Excel** | Quick data checks and pivots |
| **MySQL Workbench** | Data import and main analysis |
| **Power BI** | Final reports and dashboards |

---

## Dataset Info

**File:** `loan_month_snapshots.csv`  
**Rows:** ~10k  
**Period:** Nov 2024 – Oct 2025  

Main columns:
- month  
- loan_id  
- product (Auto, Home, Personal, Gold, Education)  
- region (North, South, East, West, Central)  
- branch  
- disbursed_amount, outstanding_amount  
- dpd (Days Past Due)  
- due_date, payment_date  
- is_paid  
- customer_segment  
- charge_off_flag  

The data was designed with some real patterns:
- Auto Loans in the South region show a visible delinquency spike around May–July 2025  
- Home Loans stay stable through the year  
- Personal Loans in Central and East rise a bit early in 2025  
This helps make the analysis meaningful when drilling down in Power BI.

---

## Steps I Followed

### 1. Loading the Data
I used MySQL Workbench’s import wizard to load the CSV into a new database called `finance_project`.


CREATE DATABASE finance_project;
USE finance_project;
Then I just checked:

SELECT COUNT(*) FROM loan_month_snapshots;

***2. Monthly Delinquency Rate ***

### To find how many accounts were 30+ DPD each month:

SELECT 
    month,
    ROUND(SUM(CASE WHEN dpd >= 30 THEN 1 ELSE 0 END) / COUNT(*), 4) AS delq_rate_30plus
FROM loan_month_snapshots
GROUP BY month
ORDER BY month;


#### This helped to spot months where delinquency went up. I noticed a clear jump around May–June 2025.

 3. Roll Rate Analysis (30 → 60)
This part shows how accounts are transitioning between stages. If 30 DPD accounts are rolling faster into 60+, it’s a red flag.

WITH current_month AS (
    SELECT loan_id, month, dpd FROM loan_month_snapshots
),
next_month AS (
    SELECT loan_id, month AS next_month, dpd AS next_dpd FROM loan_month_snapshots
)
SELECT 
    c.month,
    ROUND(SUM(CASE WHEN c.dpd=30 AND n.next_dpd>=60 THEN 1 ELSE 0 END) /
          SUM(CASE WHEN c.dpd=30 THEN 1 ELSE 0 END), 4) AS roll_rate_30_to_60
FROM current_month c
JOIN next_month n ON c.loan_id = n.loan_id
WHERE DATE_ADD(c.month, INTERVAL 1 MONTH) = n.next_month
GROUP BY c.month;


#### When I saw a jump in the roll rate before the delinquency spike, that gave me a good early warning indicator.

 4. Product & Region Analysis

To check where delinquencies are highest:

SELECT 
    month, product, region,
    ROUND(SUM(CASE WHEN dpd >= 30 THEN 1 ELSE 0 END) / COUNT(*), 4) AS delq_rate
FROM loan_month_snapshots
GROUP BY month, product, region
ORDER BY month;


#### This helped confirm that Auto Loans in the South region were driving the problem.

 5. Branch Level Comparison

To see which branches are performing worse:

SELECT 
    branch,
    ROUND(AVG(CASE WHEN dpd >= 30 THEN 1 ELSE 0 END), 4) AS avg_delinquency
FROM loan_month_snapshots
GROUP BY branch
ORDER BY avg_delinquency DESC;

###  6. Vintage Analysis

#### Just to see how different batches of loans perform over time:

SELECT 
    LEFT(loan_id, 4) AS vintage_year,
    month,
    ROUND(SUM(CASE WHEN dpd >= 30 THEN 1 ELSE 0 END) / COUNT(*), 4) AS delq_rate
FROM loan_month_snapshots
GROUP BY vintage_year, month;

### Excel & PowerBI Dashboard

In Power BI, I created visuals for:

Trend line of 30+, 60+, and 90+ delinquency

Product × Region matrix

Roll rate over months

Branch performance comparison

Vintage curve

I also added a simple trigger logic in DAX:

if 30→60 roll rate > 0.2 for two months = flag it as “Early Warning”

That helped simulate how risk teams might set alerts.

### Findings

Auto Loans (South) had the sharpest rise in delinquencies mid-2025

30→60 roll rates jumped first, before the actual 60+ spike

Two branches underperformed compared to the average

Home Loans were consistently the best-performing product

### What I Learned

This project helped me understand how delinquency rates, roll rates, and segment-level breakdowns all connect.
It also gave me a good end-to-end idea of how banks monitor portfolio risk.
Using Power BI to visualize early warnings was the most fun part for me.
