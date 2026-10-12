# Oil-Gas-Project
End-to-End Renewable Energy Integration &amp; Retail Profitability Analytics Pipeline: Optimizing Grid Load and Customer Churn
# Eni Plenitude Retail Profitability & Renewable Energy Analytics Pipeline ⚡🔋

An end-to-end data analytics and predictive pipeline engineered for Eni Plenitude France to optimize retail power distribution margins, mitigate spot-market pricing risks, and systematically predict customer churn across French departments.

---

## 1. Business Problem
Eni Plenitude is expanding rapidly across Europe by combining its green energy assets with direct retail sales to over 10 million homes. However, this integrated business model introduces two critical operational bottlenecks that threaten long-term success:
* **The Grid Imbalance & Financial Leakage:** Renewable energy generation (Solar/Wind) is highly volatile and weather-dependent. When Plenitude’s green assets under-produce during peak residential retail consumption hours in France, the company is forced to purchase expensive conventional electricity from the spot market (*Epex Spot*) to honor customer contracts, severely damaging net profit margins.
* **Siloed Data & Cross-Functional Bottlenecks:** Customer billing logs sit within CRM systems, smart-meter data is buried in IT infrastructure, and renewable production data resides in IoT cloud logs. Without an integrated data culture, Marketing cannot optimize retention campaigns, and Finance cannot accurately forecast quarterly cash reserves or manage business KPIs effectively.

---

## 2. Dataset Description
The pipeline ingests two distinct datasets, synthesized to perfectly mirror Plenitude's real-world data infrastructure:
* **`raw_grid_generation.csv` (IoT Energy Feed):** Contains hourly telemetry records from renewable energy sites across French regions. 
  * Columns: `hourly_timestamp` (Datetime), `region_france` (Text), `green_generation_kwh` (Float).
* **`raw_customer_consumption.csv` (Retail CRM Feed):** Contains residential smart-meter consumption profiles combined with customer service interactions.
  * Columns: `hourly_timestamp` (Datetime), `region_france` (Text), `retail_consumption_kwh` (Float), `support_tickets_count` (Integer), `payment_delay_days` (Integer).

---

## 3. Project Workflow
The execution follows a robust 4-stage data pipeline workflow designed to promote operational efficiency:
1. **Data Collection & Processing:** Ingested the raw files into **PlotStudio** to fix timestamp alignment mismatches, clean sensor drops, and standardize metrics into a unified Kilowatt-hour (kWh) baseline.
2. **Data Transformation & Modeling:** Orchestrated staging and production models in **dbt (Data Build Tool)** using SQL to calculate energy surpluses/deficits and map financial procurement costs against retail revenues.
3. **Advanced Analytics:** Deployed a **Python** Machine Learning script to evaluate consumer data, build load forecasts, and score accounts on customer churn propensity.
4. **Data Visualisation:** Exported the unified analytical data mart to **Power BI** to deliver dynamic, cross-functional performance reporting dashboards.

---

## 4. Exploratory Data Analysis (EDA)
Key diagnostic findings extracted from the baseline dataset include:
* **The "French Duck Curve":** Aggregated data shows solar production peaking between 12:00 and 15:00, while residential consumption spikes sharply between 18:00 and 22:00, causing severe regional energy deficits.
* **Seasonal Cost Drivers:** Winter months (November–February) exhibit a 1.5x increase in consumer demand due to heating, significantly increasing reliance on wholesale spot-market purchases.
* **Churn Correlatives:** Customer billing variance analysis indicated that a combination of payment delays over 10 days and support ticket counts greater than 1 acts as a primary leading indicator for competitor switching.

---

## 5. Business Insights
* **Operational Risk Mitigation:** The pipeline identified that a targeted 15% consolidation of overlapping operational expenditures could insulate net corporate margins from spot-market volatility, projecting an **8-11% increase in operational efficiency**.
* **Commercial Growth & Retention:** By outputting automated churn risk lists via the Python model, the Sales and Marketing teams can launch proactive, targeted retention campaigns rather than waiting for customers to cancel contracts.
* **Cross-Functional Synergy:** The centralized dbt data mart provides a unified source of truth, enabling Finance to monitor profit margin variations while simultaneously allowing Operations to track real-time grid load balancing.

---

## 6. Technologies Used
* **Data Processing & Prep:** PlotStudio
* **Data Transformation & Engineering:** dbt (Data Build Tool), SQL (PostgreSQL/BigQuery compatible)
* **Advanced Analytics & ML:** Python (Pandas, NumPy, Scikit-Learn)
* **Data Visualisation & KPI Management:** Power BI

---

## 7. Repository Structure
