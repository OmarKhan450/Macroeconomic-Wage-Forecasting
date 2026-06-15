# Macroeconomic-Wage-Forecasting

📈 Macroeconomic Wage & Statutory Limit Forecasting
Power BI Forecasting Dashboard

📌 **Project Overview**
This project is an interactive modernization of a static Excel projection model I originally built at the Canada Revenue Agency in 2021. 

The goal of this dashboard is to allow corporate finance and HR teams to forecast when Canadian statutory payroll and retirement limits (YMPE, YAMPE) will hit six figures. By evaluating historical inflation and median wage growth, this tool utilizes dynamic user parameters and predictive DAX logic to estimate future corporate payroll tax impacts.


📂 **The Data Sources**
Statistics Canada Web Data Service (1980 - 2026):
Direct CSV API connections to historical public data detailing Annual Consumer Price Index (CPI) and National Income (Average and Median).

Canada Revenue Agency (CRA) Statutory Limits:
Historical HTML tables detailing RRSP, TFSA, ALDA, YMPE, and YAMPE dollar limits per year.


🛠️ **Architecture & Workflow**
1. Data Engineering & ETL (Power Query / M Code)
Web Extraction & API Connections:
Pulled live database CSV links and scraped HTML tables directly into Power BI to ensure the dashboard can refresh automatically without local file dependencies.

**Data Cleansing & Reshaping:**
Pivoted and unpivoted raw StatsCan database rows into a clean tabular structure. Cleaned legacy text characters, handled nulls for unpublished future years, and resolved type mismatch errors caused by raw data spacing.

**Custom M-Code Logic:**
Engineered conditional columns to segment data. Built a "CPP Era" column mapping years to specific legislative phases (e.g., Pre-Enhancement vs. Phase 2) and flagged periods as Historical Actuals versus Future Projections.

**2. Data Modeling**
**Star Schema (1-to-Many):**
Architected an optimized database model connecting three distinct transactional fact tables (Limits, Income, Inflation) to a single central dimension table.

**Rolling Calendar:**
Generated a custom integer-based calendar table (1980 to 2040) natively in Power Query using list generation logic to support continuous timeline projections.

**3. DAX & Advanced Analytics**
**What-If Parameters:**
Built a numeric range parameter allowing users to slide an expected inflation/growth rate. Engineered the parameter to automatically default to the mathematically calculated historical Compound Annual Growth Rate (CAGR) if left untouched.

**Predictive Forecasting & Business Rules:**
Wrote Future Value compounding logic to project limits forward. Utilized SWITCH(TRUE) logic to map the YAMPE rollout perfectly to federal legislation (did not exist before 2024, capped at 107% of YMPE in 2024, and 114% thereafter).

**Algebraic Target Solving:**
Utilized logarithmic functions (LOG) in DAX to dynamically solve for the exact future calendar year when limits cross the $100,000 threshold based on the selected growth rate.

Backtesting:
Created secondary baseline measures tracking my original 2021 projections against actual 2024 to 2026 data to visually prove the historical accuracy of the model.


📊 **Dashboard Features**
**Executive Forecast:**
High-level KPIs, user-driven "What-If" growth sliders, and a main projection chart showing historical actuals transitioning into predictive trend lines against a static $100k ceiling.

**Macro Drivers:**
An analysis showing the growing gap between average and median income, alongside a 100% stacked column chart detailing exactly which categories (Food, Shelter, Transport) drove inflation during specific legislative eras.

**Corporate Wealth & Benefit Limits:**
A clean, matrix-based tracking sheet designed for HR directors to track historical and current TFSA, RRSP, and ALDA limits.

**Custom Tooltip:**
A custom report page tooltip designed so that hovering over the main inflation chart generates a dynamic mini-dashboard detailing that specific year's inflation breakdown.


📈 Dashboard Previews
[Screenshot 1 Placeholder] 
[Screenshot 2 Placeholder] 
[Screenshot 3 Placeholder] 
[Screenshot 4 Placeholder]


**📂 Repository Contents**
01_Raw_Data: Raw backup CSV and Excel files.
02_Assets: Custom logos and navigation icons.
03_PBIX_File: The complete Power BI Desktop project file.


**🧮 Key DAX Measures**

**Historical YMPE CAGR = **
VAR MinYear = MIN(Dim_Year[Year])
VAR MaxYear = MAX(Dim_Year[Year])
VAR StartValue = CALCULATE([Average YMPE], FILTER(ALL(Dim_Year), Dim_Year[Year] = MinYear))
VAR EndValue = CALCULATE([Average YMPE], FILTER(ALL(Dim_Year), Dim_Year[Year] = MaxYear))
VAR Years = MaxYear - MinYear
RETURN
IF( Years > 0 && NOT(ISBLANK(StartValue)) && NOT(ISBLANK(EndValue)), (EndValue / StartValue) ^ (1 / Years) - 1, BLANK() )


**Projected YAMPE =**
VAR CurrentYear = MAX(Dim_Year[Year])
RETURN
SWITCH( TRUE(),
    CurrentYear < 2024, BLANK(),
    CurrentYear = 2024, [Projected YMPE] * 1.07,
    CurrentYear >= 2025, [Projected YMPE] * 1.14,
    BLANK()
)


**Target Year YAMPE Hits 100k =**
VAR TargetAmount = 100000
VAR BaseYear = CALCULATE(MAX(Fact_StatutoryLimits[Year]), REMOVEFILTERS(Dim_Year), NOT(ISBLANK(Fact_StatutoryLimits[YMPE])))
VAR BaseYMPE = CALCULATE([Average YMPE], REMOVEFILTERS(Dim_Year), Dim_Year[Year] = BaseYear)
VAR BaseYAMPE = BaseYMPE * 1.14
VAR GrowthRate = 'Expected Growth Rate'[Expected Growth Rate Value]
VAR YearsToTarget = IF( GrowthRate > 0, LOG(TargetAmount / BaseYAMPE, 1 + GrowthRate), BLANK() )
RETURN
IF( NOT(ISBLANK(YearsToTarget)), FORMAT(ROUNDUP(BaseYear + YearsToTarget, 0), "0"), "Never" )
