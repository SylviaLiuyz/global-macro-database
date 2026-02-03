---
title: Inflation & Employment
description: Modern macroeconomic challenges in the 21st century
---

# Inflation & Employment: The Phillips Curve in Modern Times

One of macroeconomics' most famous relationships is the **Phillips Curve**—the trade-off between inflation and unemployment. This page explores whether this relationship still holds in modern times.

## Recent Inflation Trends (2000-2023)

```sql inflation_trends
SELECT 
  year,
  countryname,
  infl,
  LAG(infl) OVER (PARTITION BY countryname ORDER BY year) as prev_year_infl
FROM gmd
WHERE year >= 2000 
  AND year <= 2023
  AND infl IS NOT NULL
  AND countryname IN ('United States', 'United Kingdom', 'Canada', 'Australia', 'Japan', 'Germany', 'France')
ORDER BY year DESC, countryname
```

```sql inflation_by_country
SELECT 
  year,
  MAX(CASE WHEN countryname = 'United States' THEN infl END) as usa,
  MAX(CASE WHEN countryname = 'United Kingdom' THEN infl END) as uk,
  MAX(CASE WHEN countryname = 'Canada' THEN infl END) as canada,
  MAX(CASE WHEN countryname = 'Japan' THEN infl END) as japan,
  MAX(CASE WHEN countryname = 'Germany' THEN infl END) as germany,
  MAX(CASE WHEN countryname = 'France' THEN infl END) as france,
  MAX(CASE WHEN countryname = 'Australia' THEN infl END) as australia
FROM gmd
WHERE year >= 2000 
  AND year <= 2023
  AND infl IS NOT NULL
GROUP BY year
ORDER BY year
```

<LineChart 
  data={inflation_by_country}
  x=year
  y=usa
  title="US Inflation Rates (2000-2023)"
  yAxisTitle="Inflation Rate (%)"
  xAxisTitle="Year"
  yFmt="#,##0.0"
/>

## The Great Inflation of 2021-2023

```sql inflation_recent_table
SELECT 
  year,
  countryname,
  ROUND(infl, 1) as inflation_rate,
  ROUND(unemp, 1) as unemployment_rate
FROM gmd
WHERE year >= 2018 
  AND year <= 2023
  AND infl IS NOT NULL
  AND unemp IS NOT NULL
  AND countryname IN ('United States', 'United Kingdom', 'Canada', 'Euro Area', 'Japan', 'Australia')
ORDER BY year DESC, countryname
```

<DataTable 
  data={inflation_recent_table}
  rows=50
>
  <Column id=year title="Year"/>
  <Column id=countryname title="Country"/>
  <Column id=inflation_rate title="Inflation (%)" fmt="#,##0.0" contentType=delta/>
  <Column id=unemployment_rate title="Unemployment (%)"/>
</DataTable>

## The Phillips Curve Today: Does It Still Work?

```sql phillips_curve_usa
SELECT 
  year,
  ROUND(unemp, 1) as unemployment,
  ROUND(infl, 1) as inflation
FROM gmd
WHERE year >= 2010 
  AND year <= 2023
  AND infl IS NOT NULL
  AND unemp IS NOT NULL
  AND countryname = 'United States'
ORDER BY year
```

<LineChart 
  data={phillips_curve_usa}
  x=year
  y=unemployment
  title="US Unemployment Rate (2010-2023)"
  yAxisTitle="Rate (%)"
  xAxisTitle="Year"
/>

**Finding**: The traditional Phillips Curve relationship (low unemployment → high inflation) appears **broken in recent years**. In 2023, the US had **both low unemployment (~3.8%) and high inflation (~3.4%)**—defying the historical relationship that held for decades.

## Unemployment Trends

```sql unemployment_trends
SELECT 
  year,
  MAX(CASE WHEN countryname = 'United States' THEN unemp END) as usa,
  MAX(CASE WHEN countryname = 'United Kingdom' THEN unemp END) as uk,
  MAX(CASE WHEN countryname = 'Canada' THEN unemp END) as canada,
  MAX(CASE WHEN countryname = 'Euro Area' THEN unemp END) as euro_area,
  MAX(CASE WHEN countryname = 'Japan' THEN unemp END) as japan
FROM gmd
WHERE year >= 2000 
  AND year <= 2023
  AND unemp IS NOT NULL
GROUP BY year
ORDER BY year
```

<LineChart 
  data={unemployment_trends}
  x=year
  y=usa
  title="Unemployment Rates: US (2000-2023)"
  yAxisTitle="Unemployment Rate (%)"
  xAxisTitle="Year"
  yFmt="#,##0.0"
/>

## Inflation Volatility: Emerging Markets

```sql emerging_inflation
SELECT 
  year,
  countryname,
  ROUND(infl, 1) as inflation_rate
FROM gmd
WHERE year >= 2010 
  AND year <= 2023
  AND infl IS NOT NULL
  AND countryname IN ('Turkey', 'Brazil', 'Argentina', 'Mexico', 'South Africa', 'Russia')
ORDER BY year DESC
```

<DataTable 
  data={emerging_inflation}
  rows=80
>
  <Column id=year title="Year"/>
  <Column id=countryname title="Country"/>
  <Column id=inflation_rate title="Inflation (%)"/>
</DataTable>

## Key Insights

- **2021-2023 Shock**: Advanced economies experienced the largest inflation surge since the 1980s, driven by pandemic stimulus, supply chain disruptions, and energy shocks
- **Broken Phillips Curve**: The traditional unemployment-inflation trade-off appears to have weakened, suggesting supply-side shocks dominated over demand in recent inflation
- **Emerging Market Volatility**: Emerging markets face significantly more volatile inflation, with countries like Turkey and Argentina experiencing double-digit rates
- **Labor Market Resilience**: Despite inflation, unemployment rates in most advanced economies remained surprisingly low, highlighting a tight labor market
- **Divergent Responses**: Central banks responded very differently—some tightened aggressively (US Fed), while others moved more cautiously (ECB, BoJ)
