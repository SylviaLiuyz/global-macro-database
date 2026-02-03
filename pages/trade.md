---
title: Trade Wars & Global Supply Chains
description: How tariffs and protectionism reshape global commerce
---

# Trade Wars: The Return of Protectionism

After 50 years of gradual trade liberalization, the 2018 U.S.-China trade war marked a historic reversal. Tariffs on $370 billion of Chinese goods triggered retaliation and fundamentally restructured global supply chains. This shift reveals the geopolitical tensions underlying modern trade.

## The Trade War Timeline

- **2018 Q1**: Trump imposes steel (25%) and aluminum (10%) tariffs
- **2018 Q2**: $34B Chinese goods hit with 25% tariffs; China retaliates
- **2018 Q4**: Tariffs expanded to $200B—peak tension
- **2019 Q1**: Phase One deal signed; tensions ease temporarily
- **2020-2023**: Tariffs remain in place under Biden; focus shifts to semiconductors and critical minerals

## Who Gains, Who Loses?

```sql winners_losers
WITH country_performance AS (
  SELECT 
    countryname,
    year,
    ROUND(exports_USD, 0) as exports_billions
  FROM gmd
  WHERE year IN (2017, 2023)
    AND exports_USD IS NOT NULL
)
SELECT 
  countryname,
  MAX(CASE WHEN year = 2017 THEN exports_billions END) as exports_2017,
  MAX(CASE WHEN year = 2023 THEN exports_billions END) as exports_2023,
  ROUND(100 * (MAX(CASE WHEN year = 2023 THEN exports_billions END) / 
               MAX(CASE WHEN year = 2017 THEN exports_billions END) - 1), 1) as growth_pct
FROM country_performance
WHERE countryname IN ('United States', 'China', 'Vietnam', 'Mexico', 'Germany', 'India', 'South Korea')
GROUP BY countryname
ORDER BY growth_pct DESC
```

<BarChart
  data={winners_losers}
  x=countryname
  y=exports_2023
  title="Export Winners: 2017 vs 2023"
  yAxisTitle="Exports (Billions USD)"
  xAxisTitle="Country"
  sort=true
  />

**Vietnam surged** as manufacturers fled China via the "China+1" strategy. **Mexico thrived** under USMCA. But **China's growth slowed** from both tariff pressure and internal economic weakness.

## Global Trade Flows: The Slowdown

```sql global_trade_flows
SELECT 
  year,
  ROUND(SUM(exports_USD) / 1000, 2) as global_exports_trillions
FROM gmd
WHERE year >= 2015 
  AND year <= 2023
  AND exports_USD IS NOT NULL
GROUP BY year
ORDER BY year
```

<LineChart
  data={global_trade_flows}
  x=year
  y=global_exports_trillions
  title="Global Exports (2015-2023)"
  yAxisTitle="Exports (Trillions USD)"
  xAxisTitle="Year"
  yFmt="#,##0.0"
  />

Trade growth stalled after 2018, suggesting tariffs and uncertainty dampened global commerce. COVID-19 recovery pushed volume higher, but pre-2018 growth rates weren't restored.

## Supply Chain Shift: From China to ASEAN

```sql manufacturing_hubs
SELECT 
  year,
  countryname,
  ROUND(exports_USD, 1) as exports_billions
FROM gmd
WHERE year >= 2017 
  AND year <= 2023
  AND countryname IN ('China', 'Vietnam', 'Thailand', 'Mexico')
ORDER BY year, countryname
```

<LineChart
  data={manufacturing_hubs}
  x=year
  y=exports_billions
  series=countryname
  title="China vs Alternative Manufacturing Hubs"
  yAxisTitle="Exports (Billions USD)"
  xAxisTitle="Year"
  />

Vietnam and other Southeast Asian economies captured market share as multinational firms pursued **nearshoring** (Mexico for US markets) and **friend-shoring** (NATO-allied suppliers for Western firms).

## The Trade Deficit Debate

```sql trade_balance_major
SELECT 
  year,
  countryname,
  ROUND(exports_USD, 1) as exports_billions,
  ROUND(imports_USD, 1) as imports_billions,
  ROUND(exports_USD - imports_USD, 1) as trade_balance_billions
FROM gmd
WHERE year >= 2017 
  AND year <= 2023
  AND exports_USD IS NOT NULL
  AND countryname IN ('United States', 'China', 'Germany')
ORDER BY year DESC, countryname
```

<BarChart
  data={trade_balance_major}
  x=countryname
  y=trade_balance_billions
  series=year
  title="Trade Balances: US Deficit, China Surplus, Germany Surplus"
  yAxisTitle="Trade Balance (Billions USD)"
  xAxisTitle="Country"
  />

**The US runs chronic deficits**, used to justify protectionist policies. But these reflect deeper forces: low US savings, capital inflows seeking dollar assets, and global demand for US services. **Germany maintains surpluses** (particularly within the EU), creating trade tensions. **China's surplus compressed** post-2018 as demand weakened and supply chains diversified.

## Looking Forward: Fragmented Trade

The data suggests three possible futures:

**Selective Decoupling**: Strategic industries (semiconductors, pharma, batteries) fragment into Western vs. China supply chains, while consumer goods remain globally integrated.

**Regional Blocs**: A Western alliance (US, EU, Japan, South Korea) competes with a China-led bloc (BRICS) for influence in unaligned markets like ASEAN and Africa.

**Automation Wildcard**: Rising robotics reduce labor-cost advantages of offshoring, potentially reshoring manufacturing to developed economies regardless of trade policy.

Current data shows tariffs persist, supply chains diversify, and global trade growth remains subdued—consistent with a fragmented, lower-growth trade environment.

---

**Data Source**: Global Macro Database (1960-2023) | **Last Updated**: <LastRefreshed/>
