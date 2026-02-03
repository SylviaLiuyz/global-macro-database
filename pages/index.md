---
title: Global Macro Database Dashboard
description: Interactive analysis of global macroeconomic trends and insights
---

# Global Macroeconomic Dashboard
<LastRefreshed prefix="Data last updated"/>

Welcome to the Global Macro Database Dashboard. This comprehensive analysis explores fascinating patterns in global economics over the past century, from economic convergence between nations to inflation dynamics and government debt trends.

## Dashboard Overview

This dashboard presents three key insights:

1. **[Economic Convergence →](convergence)** - How emerging markets are catching up to developed economies
2. **[Inflation & Employment →](inflation)** - Modern macroeconomic challenges in the 21st century  
3. **[Government Debt →](debt)** - The rising debt burden in advanced economies

## Global GDP Growth

```sql global_gdp_by_year
SELECT 
    year,
    SUM(nGDP_USD) as total_gdp_usd_billions,
    COUNT(DISTINCT countryname) as countries_with_data,
    ROUND(SUM(nGDP_USD) / 1000, 2) as total_gdp_trillions
FROM gmd 
WHERE year >= 1970 
    AND nGDP_USD IS NOT NULL 
    AND nGDP_USD > 0
GROUP BY year 
ORDER BY year
```

<LineChart 
    data={global_gdp_by_year}
    x=year
    y=total_gdp_trillions
    title="Global GDP Growth Since 1970"
    yAxisTitle="GDP (Trillions USD)"
    xAxisTitle="Year"
    yFmt="#,##0.0"
/>

## Top 10 Economies Today

```sql top_economies
SELECT 
    countryname,
    year,
    ROUND(nGDP_USD / 1000, 2) as gdp_trillions,
    ROW_NUMBER() OVER (ORDER BY nGDP_USD DESC) as rank
FROM gmd
WHERE year = (SELECT MAX(year) FROM gmd WHERE nGDP_USD IS NOT NULL)
    AND nGDP_USD IS NOT NULL
ORDER BY gdp_trillions DESC
LIMIT 10
```

<BarChart 
    data={top_economies}
    x=countryname
    y=gdp_trillions
    title="Top 10 Economies by GDP"
    yAxisTitle="GDP (Trillions USD)"
    sort=true
/>

---

Explore the detailed analysis pages above to discover key insights about global economic trends.