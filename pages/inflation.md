---
title: Inflation & Employment
description: Modern macroeconomic challenges in the 21st century
---

# Inflation & Employment: The Phillips Curve Breaks

For 60 years, economists knew the rule: **low unemployment → high inflation**. Then 2023 happened. The US had both low unemployment (3.8%) and high inflation (3.4%), breaking the relationship that defined monetary policy. This signals a fundamental shift in how inflation works.

## The 2021-2023 Inflation Shock

```sql inflation_panel
SELECT
  year,
  countryname,
  infl
FROM gmd
WHERE year >= 2018
  AND year <= 2023
  AND infl IS NOT NULL
  AND countryname IN ('United States', 'United Kingdom', 'Euro Area', 'Japan')
ORDER BY year, countryname
```

<LineChart
  data={inflation_panel}
  x=year
  y=infl
  series=countryname
  title="Inflation Shock: Advanced Economies (2018-2023)"
  yAxisTitle="Inflation Rate (%)"
  xAxisTitle="Year"
  yFmt="#,##0.0"
/>

**The Shock**: After 30 years of low, stable inflation, advanced economies experienced their worst inflation surge since the 1980s:
- Supply chain disruptions from COVID-19 lockdowns
- Massive fiscal and monetary stimulus 
- Energy price shocks from Ukraine war
- Tight labor markets reducing wage flexibility

## The Broken Phillips Curve

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
  x=unemployment
  y=inflation
  title="The Phillips Curve: Does It Still Work?"
  yAxisTitle="Inflation (%)"
  xAxisTitle="Unemployment (%)"
/>

**The Break**: Historically, this curve should be downward-sloping (low unemployment = high inflation). Instead, 2023 shows both low unemployment AND high inflation—violating 60 years of economic theory. This suggests **supply shocks dominated demand**, rather than labor market tightness driving prices.

## Emerging Market Instability

```sql emerging_inflation_vol
SELECT
  countryname,
  ROUND(AVG(infl), 1) as avg_inflation,
  ROUND(STDDEV_POP(infl), 2) as infl_volatility
FROM gmd
WHERE year >= 2010
  AND year <= 2023
  AND infl IS NOT NULL
  AND countryname IN ('Turkey', 'Brazil', 'Argentina', 'Mexico', 'India', 'South Africa')
GROUP BY countryname
ORDER BY infl_volatility DESC
```

<BarChart
  data={emerging_inflation_vol}
  x=countryname
  y=infl_volatility
  title="Inflation Volatility: Emerging Markets Face Chaos"
  yAxisTitle="Std Dev of Inflation (%)"
  sort=true
/>

Emerging markets face far worse instability. Turkey and Argentina experience double-digit inflation and extreme volatility, while developed markets have more tools to manage inflation risks.

## What Changed?

1. **Supply Matters More**: Global supply chains fragmented post-COVID, limiting traditional demand-side policy effectiveness

2. **Expectations Shifted**: Inflation became unanchored from central bank targets—workers demanded wage increases, locking in price pressures

3. **Labor Market Rigidities**: Even with high unemployment in the 2010s, wages stayed suppressed. Now the relationship reversed

4. **Oil & Commodity Shocks**: Geopolitical events (Ukraine war) created supply shocks outside central bank control

The Phillips Curve isn't dead—it's been overwhelmed by supply-side factors that traditional demand management can't address.

---

**Data Source**: Global Macro Database (1960-2023) | **Last Updated**: <LastRefreshed/>
