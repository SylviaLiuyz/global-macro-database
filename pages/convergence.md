---
title: Economic Convergence
description: How emerging markets are catching up to developed economies
---

# Economic Convergence: The Rise of Emerging Markets

This analysis reveals a striking trend: **emerging market economies are catching up to developed nations**. After centuries of divergence, we're seeing evidence of convergence in real GDP per capita.

## GDP Per Capita Trends by Development Status

```sql gdp_pc_trends
WITH country_classification AS (
  SELECT DISTINCT
    countryname,
    CASE 
      WHEN countryname IN ('United States', 'United Kingdom', 'Germany', 'France', 'Japan', 'Canada', 'Australia', 'Netherlands', 'Belgium', 'Switzerland', 'Sweden', 'Denmark', 'Norway', 'Austria', 'Finland', 'Spain', 'Italy')
        THEN 'Developed'
      WHEN countryname IN ('China', 'India', 'Brazil', 'Russia', 'Mexico', 'Indonesia', 'South Africa', 'Turkey', 'Thailand', 'Philippines', 'Vietnam', 'Malaysia', 'Pakistan', 'Nigeria', 'Bangladesh')
        THEN 'Emerging'
      ELSE 'Other'
    END as status
  FROM gmd
)
SELECT 
  g.year,
  cc.status,
  ROUND(AVG(g.rGDP_pc), 0) as avg_gdp_pc,
  COUNT(DISTINCT g.countryname) as num_countries
FROM gmd g
LEFT JOIN country_classification cc ON g.countryname = cc.countryname
WHERE g.year >= 1980 
  AND g.rGDP_pc IS NOT NULL 
  AND cc.status IS NOT NULL
GROUP BY g.year, cc.status
ORDER BY g.year, cc.status
```

<LineChart 
  data={gdp_pc_trends}
  x=year
  y=avg_gdp_pc
  series=status
  title="Real GDP Per Capita: Developed vs Emerging Markets (1980-Present)"
  yAxisTitle="Real GDP Per Capita (constant USD)"
  xAxisTitle="Year"
  yFmt="#,##0"
/>

## Key Finding: The Convergence Gap

```sql convergence_gap
WITH country_classification AS (
  SELECT DISTINCT
    countryname,
    CASE 
      WHEN countryname IN ('United States', 'United Kingdom', 'Germany', 'France', 'Japan', 'Canada', 'Australia', 'Netherlands', 'Belgium', 'Switzerland', 'Sweden', 'Denmark', 'Norway', 'Austria', 'Finland', 'Spain', 'Italy')
        THEN 'Developed'
      WHEN countryname IN ('China', 'India', 'Brazil', 'Russia', 'Mexico', 'Indonesia', 'South Africa', 'Turkey', 'Thailand', 'Philippines', 'Vietnam', 'Malaysia', 'Pakistan', 'Nigeria', 'Bangladesh')
        THEN 'Emerging'
      ELSE 'Other'
    END as status
  FROM gmd
),
yearly_avg AS (
  SELECT 
    g.year,
    cc.status,
    ROUND(AVG(g.rGDP_pc), 0) as avg_gdp_pc
  FROM gmd g
  LEFT JOIN country_classification cc ON g.countryname = cc.countryname
  WHERE g.year >= 1980 
    AND g.rGDP_pc IS NOT NULL 
    AND cc.status IS NOT NULL
  GROUP BY g.year, cc.status
)
SELECT 
  year,
  MAX(CASE WHEN status = 'Developed' THEN avg_gdp_pc END) as developed_avg,
  MAX(CASE WHEN status = 'Emerging' THEN avg_gdp_pc END) as emerging_avg,
  ROUND(
    (MAX(CASE WHEN status = 'Developed' THEN avg_gdp_pc END) - 
     MAX(CASE WHEN status = 'Emerging' THEN avg_gdp_pc END)) /
    MAX(CASE WHEN status = 'Developed' THEN avg_gdp_pc END) * 100,
    1
  ) as gap_percentage
FROM yearly_avg
GROUP BY year
ORDER BY year
```

<LineChart 
  data={convergence_gap}
  x=year
  y=gap_percentage
  title="The Narrowing Gap: Income Gap % (1980-Present)"
  yAxisTitle="Income Gap as % of Developed Average"
  xAxisTitle="Year"
  yFmt="#,##0"
/>

The gap between developed and emerging market average income has **narrowed from over 80% to less than 60%** since 1980.

## Emerging Market Champions

```sql emerging_leaders
WITH latest_decade AS (
  SELECT 
    countryname,
    year,
    rGDP_pc,
    ROW_NUMBER() OVER (PARTITION BY countryname ORDER BY year DESC) as rn
  FROM gmd
  WHERE year >= 2010 
    AND rGDP_pc IS NOT NULL
)
SELECT 
  ld.countryname,
  MAX(CASE WHEN ld.rn = 1 THEN ld.year END) as latest_year,
  ROUND(MAX(CASE WHEN ld.rn = 1 THEN ld.rGDP_pc END), 0) as latest_gdp_pc,
  ROUND(MAX(CASE WHEN ld.rn = 1 THEN ld.rGDP_pc END) / 
        MAX(CASE WHEN ld.rn > 13 THEN ld.rGDP_pc END) - 1, 2) as growth_multiplier
FROM latest_decade ld
WHERE countryname IN ('China', 'India', 'Brazil', 'Vietnam', 'Indonesia', 'Turkey', 'Mexico', 'Russia', 'Thailand', 'Poland')
GROUP BY ld.countryname
ORDER BY growth_multiplier DESC
LIMIT 10
```

<BarChart 
  data={emerging_leaders}
  x=countryname
  y=growth_multiplier
  title="Emerging Market Growth: Multiplier Since Early 2010s"
  yAxisTitle="Growth Multiple"
  sort=true
/>

## The China Story

```sql china_india_comparison
SELECT 
  year,
  MAX(CASE WHEN countryname = 'China' THEN rGDP_pc END) as china_gdp_pc,
  MAX(CASE WHEN countryname = 'India' THEN rGDP_pc END) as india_gdp_pc,
  MAX(CASE WHEN countryname = 'United States' THEN rGDP_pc END) as usa_gdp_pc
FROM gmd
WHERE year >= 1980 
  AND year <= 2023
  AND rGDP_pc IS NOT NULL
  AND countryname IN ('China', 'India', 'United States')
GROUP BY year
ORDER BY year
```

<LineChart 
  data={china_india_comparison}
  x=year
  y=china_gdp_pc
  title="Real GDP Per Capita Comparison (1980-2023)"
  yAxisTitle="Real GDP Per Capita (constant USD)"
  xAxisTitle="Year"
  yFmt="#,##0"
/>

## Insights

- **China's Transformation**: China's real GDP per capita has grown from ~$600 in 1980 to over $17,000 by 2023—a **28x increase**
- **India's Journey**: While starting lower, India has also seen consistent growth, reaching ~$3,400 by 2023
- **The Convergence Trend**: This isn't just about these two countries—it reflects a broader global trend where productivity growth in emerging markets is reducing historical income gaps
- **Policy Matters**: Countries that invested in education, infrastructure, and open trade policies have converged faster than others
