# Call Center Workforce Management Analytics

**Author:** Christos Papakostas  
**Tools Used:** Google BigQuery (SQL), Tableau Public  
**Project Type:** Data Analysis & Visualization  
**Focus Areas:** Workforce Optimization, SLA Monitoring, Abandonment Rate, Staffing Simulation

---

## Overview

This project explores and analyzes workforce performance data from a call center using SQL for modeling and Tableau for data storytelling. The analysis focuses on key performance indicators (KPIs) such as SLA%, Abandonment Rate, Wait Time, Staffing Efficiency, and Simulation of Increased Workload.

The final output includes a comprehensive dashboard with executive-level KPIs, operational charts, and insights generated through more than 25 SQL queries.

---

## 1. Data Preparation & Import

- Dataset: `Call Center Data.csv`
- Tool: Google BigQuery
- Steps:
  1. Created a new project in BigQuery: `wfm-call-center`
  2. Created a dataset: `call_center_data`
  3. Cleaned the CSV file to remove unsupported characters and formatted column headers using underscores.
  4. Imported the data using the BigQuery UI (CSV format, with header row).

### Final Table: `call_center_data.raw_call_data`

---

## 2. SQL Modeling – Queries Q01–Q25

Below is the complete list of SQL queries used in the project with short descriptions:

### Q01 – Calculate the average SLA over the entire period.
```sql
SELECT ROUND(AVG(Service_Level_20S), 4) AS avg_service_level FROM call_center_data.raw_call_data;
```

### Q02 – Calculate the average abandonment rate across all days.
```sql
SELECT ROUND(AVG(abandonment_rate), 4) AS avg_abandonment_rate FROM call_center_data.raw_call_data;
```

### Q03 – Extract SLA per day to visualize SLA trend.
```sql
SELECT Index, Service_Level_20S FROM call_center_data.raw_call_data;
```

### Q04 – Categorize each day based on SLA value into four quality levels.
```sql
SELECT Index, CASE WHEN Service_Level_20S >= 0.80 THEN 'Excellent' WHEN Service_Level_20S >= 0.65 THEN 'Acceptable' WHEN Service_Level_20S >= 0.50 THEN 'Low' ELSE 'Critical' END AS sla_category FROM call_center_data.raw_call_data;
```

### Q05 – List daily abandonment rates in descending order.
```sql
SELECT Index, abandonment_rate FROM call_center_data.raw_call_data ORDER BY abandonment_rate DESC;
```

### Q06 – Classify days into waiting time buckets for better understanding of hold times.
```sql
SELECT Index, CASE WHEN Waiting_Time_Sec <= 20 THEN '0–20 sec' WHEN Waiting_Time_Sec <= 60 THEN '21–60 sec' ELSE '60+ sec' END AS wait_time_bucket FROM call_center_data.raw_call_data;
```

### Q07 – Calculate how many agents were required daily, assuming 1800s effort per agent.
```sql
SELECT Index, ROUND((Talk_Duration_Sec + Waiting_Time_Sec) / 1800, 2) AS required_agents FROM call_center_data.raw_call_data;
```

### Q08 – Compute correlation between SLA and talk duration to check for relationship.
```sql
SELECT CORR(Talk_Duration_Sec, Service_Level_20S) AS correlation_talk_vs_sla FROM call_center_data.raw_call_data;
```

### Q09 – Calculate 7-day rolling average of SLA to smooth out volatility.
```sql
SELECT Index, ROUND(AVG(Service_Level_20S) OVER (ORDER BY Index ROWS BETWEEN 6 PRECEDING AND CURRENT ROW), 4) AS rolling_sla_7d FROM call_center_data.raw_call_data;
```

### Q10 – Rank all days by SLA performance to identify top and bottom performers.
```sql
SELECT Index, Service_Level_20S, RANK() OVER (ORDER BY Service_Level_20S DESC) AS sla_rank FROM call_center_data.raw_call_data;
```

### Q11 – Detect days with service level below a critical threshold.
```sql
SELECT Index, Service_Level_20S FROM call_center_data.raw_call_data WHERE Service_Level_20S < 0.50;
```

### Q12 – Compute the efficiency ratio between talk time and total effort time.
```sql
SELECT Index, ROUND(Talk_Duration_Sec / (Talk_Duration_Sec + Waiting_Time_Sec), 4) AS efficiency_ratio FROM call_center_data.raw_call_data;
```

### Q13 – Calculate number of calls handled per total talk hour to assess productivity.
```sql
SELECT Index, ROUND(Incoming_Calls / (Talk_Duration_Sec / 3600), 2) AS calls_per_talk_hour FROM call_center_data.raw_call_data;
```

### Q14 – Create a custom workload index based on call volume and effort time.
```sql
SELECT Index, ROUND(Incoming_Calls * (Talk_Duration_Sec + Waiting_Time_Sec) / 60, 2) AS workload_index FROM call_center_data.raw_call_data;
```

### Q15 – Flag potential outliers where SLA performance is unexpectedly low.
```sql
WITH cte AS (SELECT Index, Service_Level_20S, AVG(Service_Level_20S) OVER () AS mean_sla, STDDEV(Service_Level_20S) OVER () AS stddev_sla FROM call_center_data.raw_call_data) SELECT Index, CASE WHEN Service_Level_20S < mean_sla - 2 * stddev_sla THEN 'Anomaly' ELSE 'Normal' END AS anomaly_flag FROM cte;
```

### Q16 – Bucket abandonment rates into performance tiers.
```sql
SELECT Index, CASE WHEN abandonment_rate < 0.10 THEN 'Good' WHEN abandonment_rate < 0.20 THEN 'Moderate' ELSE 'High' END AS abandonment_category FROM call_center_data.raw_call_data;
```

### Q17 – Compute volume-adjusted SLA by multiplying SLA by incoming calls.
```sql
SELECT Index, Service_Level_20S * Incoming_Calls AS volume_adjusted_sla FROM call_center_data.raw_call_data;
```

### Q18 – Simulate required agents under increased workload (+50%).
```sql
SELECT Index, ((Talk_Duration_Sec + Waiting_Time_Sec) * 1.5) / 1800 AS required_agents_simulated FROM call_center_data.raw_call_data;
```

### Q19 – Estimate staffing gap under simulation vs. 8 available agents.
```sql
SELECT Index, 8 - ((Talk_Duration_Sec + Waiting_Time_Sec) * 1.5) / 1800 AS staffing_gap_simulated FROM call_center_data.raw_call_data;
```

### Q20 – Flag days as understaffed under simulated load if gap < 0.
```sql
SELECT Index, CASE WHEN 8 - ((Talk_Duration_Sec + Waiting_Time_Sec) * 1.5) / 1800 < 0 THEN 'Understaffed' ELSE 'OK' END AS simulated_flag FROM call_center_data.raw_call_data;
```

### Q21 – Identify high waiting time but low abandonment days.
```sql
SELECT Index FROM call_center_data.raw_call_data WHERE Waiting_Time_Sec > 300 AND abandonment_rate < 0.10;
```

### Q22 – Detect maximum call volume day.
```sql
SELECT Index, Incoming_Calls FROM call_center_data.raw_call_data ORDER BY Incoming_Calls DESC LIMIT 1;
```

### Q23 – Simulate SLA change if 1 more agent was added per day.
```sql
SELECT Index, ROUND(Service_Level_20S + 0.015, 4) AS simulated_sla_plus_agent FROM call_center_data.raw_call_data;
```

### Q24 – Calculate SLA efficiency index based on workload and result.
```sql
SELECT Index, ROUND(Service_Level_20S / ((Talk_Duration_Sec + Waiting_Time_Sec) / 1800), 2) AS sla_efficiency_index FROM call_center_data.raw_call_data;
```

### Q25 – Estimate how many agents would be needed to reach 80% SLA.
```sql
SELECT Index, CEIL(((Talk_Duration_Sec + Waiting_Time_Sec) / 1800) + 1) AS suggested_agents_for_80 FROM call_center_data.raw_call_data WHERE Service_Level_20S < 0.80;
```



---

## 3. Tableau Dashboard

Tool: Tableau Public  
Dataset: Exported CSV with clean + simulated fields  
Final file: `clean_call_data_ready.csv`

### Dashboard Elements:

- **Top KPI Zone:**
  - Average SLA
  - Average Abandonment Rate
  - Understaffed Days (Simulated)

![KPI Zone](screenshots/kpi_zone.png)

- SLA Over Time (Line)

![SLA Over Time](screenshots/sla_over_time.png)

- SLA Category Distribution (Bar)

![SLA Category Distribution](screenshots/sla_category_distribution.png)

- Abandonment Rate with Color by Wait Time (Bar)

![Abandonment Rate](screenshots/abandonment_rate_colored.png)

- Simulated Staffing Gap (Line)

![Simulated Staffing Gap](screenshots/simulated_staffing_gap.png)

---

## 4. Key Insights

- The call center maintains SLA performance at ~71% on average.
- Abandonment rate averaged ~7.3%, with wait times impacting performance.
- Staffing remained sufficient under both actual and simulated scenarios.
- Even with a 50% workload increase, the simulated staffing gap remained positive.
- Tableau was used to visually communicate operational insights clearly.

---

## 📂 Repository Structure

```
call-center-wfm/
├── data/
│   └── clean_call_data_ready.csv
├── screenshots/
│   ├── sla_over_time.png
│   ├── sla_category_distribution.png
│   ├── abandonment_rate_colored.png
│   ├── simulated_staffing_gap.png
│   └── kpi_zone.png
├── README.md
```

---

## Contact

Feel free to comment or contact me for collaboration.