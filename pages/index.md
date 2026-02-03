---
title: Global Macro Database Dashboard
description: Interactive analysis of global macroeconomic trends and insights
---

# Global Macroeconomic Dashboard
<LastRefreshed prefix="Data last updated"/>

Welcome to the Global Macro Database Dashboard. This comprehensive analysis explores fascinating patterns in global economics over the past century, from economic convergence between nations to inflation dynamics and government debt trends.

## Dashboard Overview

### Foundation Analysis
1. **[Economic Convergence →](convergence)** - How emerging markets are catching up to developed economies
2. **[Inflation & Employment →](inflation)** - Modern macroeconomic challenges in the 21st century  
3. **[Government Debt →](debt)** - The rising debt burden in advanced economies

### Advanced Topics: Crises & Instability
4. **[Housing Bubbles →](housing)** - When real estate booms precede financial crises
5. **[Monetary Policy Explosion →](monetary)** - How money printing leads to delayed inflation
6. **[Currency Crises →](currency)** - Exchange rate turmoil and emerging market vulnerabilities
7. **[Trade Wars →](trade)** - Tariffs, supply chains, and economic decoupling

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

## About This Dashboard

This dashboard goes beyond typical GDP and inflation analysis to explore the **complex system of global finance**. 

**Key themes:**
- **Crisis Contagion**: How problems in housing, debt, or currencies spread across borders
- **Policy Trade-offs**: Central banks face impossible choices—inflation control vs financial stability
- **Emerging Market Vulnerability**: The debt trap of dollar-denominated borrowing in volatile currencies
- **Supply Chain Fragility**: Trade disruptions expose dependencies on vulnerable chokepoints
- **Historical Patterns**: Modern crises rhyme with 2008, 1998 (Asian crisis), and 1980s (debt crises)

**Data Source**: Global Macro Database (1800-2027) with 200+ countries
**Last Updated**: <Value data={global_gdp_by_year} column=year agg=max/>

Start with the foundational analyses, then explore the advanced topics to understand how these different systems interact.