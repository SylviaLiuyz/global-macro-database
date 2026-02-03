---
title: Government Debt Crisis
description: The rising debt burden in advanced economies
---

# Government Debt: An Unsustainable Trend?

The past two decades have witnessed an unprecedented accumulation of government debt in advanced economies. From the 2008 financial crisis through the COVID-19 pandemic, government debt as a share of GDP has reached historic levels.

## Government Debt as % of GDP

```sql govt_debt_trends
SELECT 
  year,
  MAX(CASE WHEN countryname = 'United States' THEN govdebt_GDP END) as usa,
  MAX(CASE WHEN countryname = 'Japan' THEN govdebt_GDP END) as japan,
  MAX(CASE WHEN countryname = 'United Kingdom' THEN govdebt_GDP END) as uk,
  MAX(CASE WHEN countryname = 'Germany' THEN govdebt_GDP END) as germany,
  MAX(CASE WHEN countryname = 'France' THEN govdebt_GDP END) as france,
  MAX(CASE WHEN countryname = 'Italy' THEN govdebt_GDP END) as italy,
  MAX(CASE WHEN countryname = 'Spain' THEN govdebt_GDP END) as spain
FROM gmd
WHERE year >= 1980 
  AND year <= 2023
  AND govdebt_GDP IS NOT NULL
GROUP BY year
ORDER BY year
```

<LineChart 
  data={govt_debt_trends}
  x=year
  y=usa
  title="Government Debt as % of GDP: US (1980-2023)"
  yAxisTitle="Debt as % of GDP"
  xAxisTitle="Year"
  yFmt="#,##0"
/>

## The 2008-2020 Debt Explosion

```sql debt_increase
WITH base_year AS (
  SELECT 
    countryname,
    year,
    govdebt_GDP,
    LAG(govdebt_GDP, 1) OVER (PARTITION BY countryname ORDER BY year) as prev_year_debt
  FROM gmd
  WHERE year >= 2000 AND year <= 2023 AND govdebt_GDP IS NOT NULL
)
SELECT 
  year,
  countryname,
  ROUND(govdebt_GDP, 1) as debt_pct_gdp,
  ROUND(govdebt_GDP - FIRST_VALUE(govdebt_GDP) OVER (PARTITION BY countryname ORDER BY year), 1) as increase_from_2000
FROM base_year
WHERE countryname IN ('United States', 'United Kingdom', 'Japan', 'France', 'Spain', 'Italy', 'Germany', 'Canada')
  AND year IN (2000, 2008, 2012, 2020, 2023)
ORDER BY year, countryname
```

<DataTable 
  data={debt_increase}
  rows=50
>
  <Column id=year title="Year"/>
  <Column id=countryname title="Country"/>
  <Column id=debt_pct_gdp title="Debt (% GDP)"/>
  <Column id=increase_from_2000 title="Increase from 2000"/>
</DataTable>

## COVID-19's Fiscal Impact

```sql covid_fiscal_response
SELECT 
  year,
  countryname,
  ROUND(govdebt_GDP, 1) as debt_pct_gdp,
  ROUND(LAG(govdebt_GDP) OVER (PARTITION BY countryname ORDER BY year), 1) as prev_year,
  ROUND(govdebt_GDP - LAG(govdebt_GDP) OVER (PARTITION BY countryname ORDER BY year), 1) as annual_change
FROM gmd
WHERE year >= 2018 
  AND year <= 2023
  AND govdebt_GDP IS NOT NULL
  AND countryname IN ('United States', 'United Kingdom', 'Canada', 'Germany', 'France', 'Japan', 'Australia')
ORDER BY year DESC, countryname
```

<DataTable 
  data={covid_fiscal_response}
  rows=50
>
  <Column id=year title="Year"/>
  <Column id=countryname title="Country"/>
  <Column id=debt_pct_gdp title="Debt (% GDP)"/>
  <Column id=annual_change title="Annual Change"/>
</DataTable>

**Key Observation**: Government debt jumped dramatically in 2020-2021 due to massive fiscal stimulus programs responding to the pandemic.

## Government Revenue vs Expenditure

```sql fiscal_balance
SELECT 
  year,
  countryname,
  ROUND(govexp_GDP, 1) as expenditure_pct_gdp,
  ROUND(govrev_GDP, 1) as revenue_pct_gdp,
  ROUND(govexp_GDP - govrev_GDP, 1) as deficit_pct_gdp
FROM gmd
WHERE year >= 2000 
  AND year <= 2023
  AND govexp_GDP IS NOT NULL
  AND govrev_GDP IS NOT NULL
  AND countryname IN ('United States', 'United Kingdom', 'Germany', 'France', 'Japan', 'Canada')
ORDER BY year DESC, countryname
```

<DataTable 
  data={fiscal_balance}
  rows=80
>
  <Column id=year title="Year"/>
  <Column id=countryname title="Country"/>
  <Column id=expenditure_pct_gdp title="Expenditure (% GDP)"/>
  <Column id=revenue_pct_gdp title="Revenue (% GDP)"/>
  <Column id=deficit_pct_gdp title="Deficit (% GDP)" contentType=delta fmt="+#,##0.0;-#,##0.0"/>
</DataTable>

## Interest Rates and Debt Sustainability

```sql interest_rates
SELECT 
  year,
  countryname,
  ROUND(ltrate, 2) as long_term_rate,
  ROUND(cbrate, 2) as central_bank_rate
FROM gmd
WHERE year >= 2000 
  AND year <= 2023
  AND (ltrate IS NOT NULL OR cbrate IS NOT NULL)
  AND countryname IN ('United States', 'United Kingdom', 'Germany', 'Japan', 'Euro Area')
ORDER BY year DESC, countryname
LIMIT 80
```

<DataTable 
  data={interest_rates}
  rows=80
>
  <Column id=year title="Year"/>
  <Column id=countryname title="Country"/>
  <Column id=long_term_rate title="Long-Term Rate (%)"/>
  <Column id=central_bank_rate title="Central Bank Rate (%)"/>
</DataTable>

## The Debt Sustainability Question

```sql debt_sustainability
WITH debt_metrics AS (
  SELECT 
    year,
    countryname,
    govdebt_GDP as debt_ratio,
    govdef_GDP as deficit_ratio,
    LAG(govdebt_GDP) OVER (PARTITION BY countryname ORDER BY year) as prev_debt_ratio
  FROM gmd
  WHERE year >= 2010 AND year <= 2023
    AND govdebt_GDP IS NOT NULL
    AND govdef_GDP IS NOT NULL
)
SELECT 
  year,
  countryname,
  ROUND(debt_ratio, 1) as debt_pct_gdp,
  ROUND(deficit_ratio, 1) as deficit_pct_gdp,
  CASE 
    WHEN debt_ratio > 90 THEN 'High Risk'
    WHEN debt_ratio > 60 THEN 'Elevated Risk'
    WHEN debt_ratio > 40 THEN 'Moderate Risk'
    ELSE 'Low Risk'
  END as risk_category
FROM debt_metrics
WHERE countryname IN ('United States', 'Japan', 'Italy', 'Spain', 'Germany', 'France')
ORDER BY year DESC, debt_ratio DESC
```

<DataTable 
  data={debt_sustainability}
  rows=100
>
  <Column id=year title="Year"/>
  <Column id=countryname title="Country"/>
  <Column id=debt_pct_gdp title="Debt (% GDP)"/>
  <Column id=deficit_pct_gdp title="Deficit (% GDP)"/>
  <Column id=risk_category title="Risk Level"/>
</DataTable>

## Key Insights

- **Historic Debt Levels**: Government debt in advanced economies has reached levels not seen since World War II
  - Japan leads at ~250% of GDP
  - US at ~120% of GDP
  - Italy and Spain at ~140% and ~110% respectively

- **The Deficit Problem**: Persistent budget deficits mean debt continues to grow even during economic expansions

- **Interest Rate Sensitivity**: Rising interest rates increase debt servicing costs, creating a vicious cycle

- **Political Constraints**: Addressing debt requires either raising taxes, cutting spending, or achieving faster growth—all politically difficult

- **Emerging Question**: Can advanced economies sustain current debt levels? This is the defining economic policy question of the 2020s

- **Divergent Paths**: Some countries (Germany, Australia) maintained lower debt, while others had to issue massive amounts (US, Japan, UK)
