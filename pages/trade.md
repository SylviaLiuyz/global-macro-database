---
title: Trade Wars & Global Supply Chains
description: How trade flows reveal geopolitical tensions and economic decoupling
---

# The Trade War Era: Tariffs, Decoupling, and Supply Chains

Trade flows (exports and imports as % of GDP) reveal fundamental economic relationships. Sudden disruptions in trade often signal economic or geopolitical shocks. This page traces recent trade wars and supply chain stress.

## The US-China Trade War (2018-2023)

```sql us_china_trade
SELECT 
  year,
  countryname,
  ROUND(exports_GDP, 1) as exports_pct_gdp,
  ROUND(imports_GDP, 1) as imports_pct_gdp,
  ROUND(exports_GDP - imports_GDP, 1) as trade_balance_pct_gdp,
  CASE 
    WHEN year < 2018 THEN 'Pre-Trade War'
    WHEN year BETWEEN 2018 AND 2019 THEN 'Trade War Phase 1-2'
    WHEN year = 2020 THEN 'COVID-19'
    WHEN year BETWEEN 2021 AND 2022 THEN 'Post-COVID'
    WHEN year >= 2023 THEN 'De-Risking'
    ELSE 'Normal'
  END as period
FROM gmd
WHERE year >= 2015 
  AND year <= 2023
  AND exports_GDP IS NOT NULL
  AND imports_GDP IS NOT NULL
  AND countryname IN ('United States', 'China')
ORDER BY year DESC, countryname
```

<DataTable 
  data={us_china_trade}
  rows=20
>
  <Column id=year title="Year"/>
  <Column id=countryname title="Country"/>
  <Column id=exports_pct_gdp title="Exports (% GDP)"/>
  <Column id=imports_pct_gdp title="Imports (% GDP)"/>
  <Column id=trade_balance_pct_gdp title="Trade Balance (% GDP)"/>
  <Column id=period title="Period"/>
</DataTable>

**Key Events:**
- **2018**: Trump administration imposes 25% tariffs on Chinese goods
- **2019**: Retaliatory tariffs; negotiations continue
- **2020**: COVID-19 disrupts trade flows, but relative positions stable
- **2021-2022**: Supply chain "re-shoring" initiatives announced
- **2023**: Focus on "de-risking" (not complete decoupling, but reducing China exposure)

## Global Export Collapse During COVID (2020)

```sql trade_during_covid
SELECT 
  year,
  countryname,
  ROUND(exports_GDP, 1) as exports_pct_gdp,
  ROUND(imports_GDP, 1) as imports_pct_gdp,
  LAG(exports_GDP) OVER (PARTITION BY countryname ORDER BY year) as prev_exports_pct,
  ROUND((exports_GDP - LAG(exports_GDP) OVER (PARTITION BY countryname ORDER BY year)), 1) as exports_change,
  CASE 
    WHEN year = 2019 THEN 'Pre-Pandemic'
    WHEN year = 2020 THEN 'Lockdowns'
    WHEN year = 2021 THEN 'Recovery'
    ELSE 'Normalization'
  END as covid_phase
FROM gmd
WHERE year >= 2018 
  AND year <= 2023
  AND exports_GDP IS NOT NULL
  AND countryname IN ('Germany', 'Japan', 'South Korea', 'Vietnam', 'Mexico', 'Thailand')
ORDER BY year DESC, countryname
```

<DataTable 
  data={trade_during_covid}
  rows=40
>
  <Column id=year title="Year"/>
  <Column id=countryname title="Country"/>
  <Column id=exports_pct_gdp title="Exports (% GDP)"/>
  <Column id=exports_change title="Change from Prior Year"/>
  <Column id=covid_phase title="COVID Phase"/>
</DataTable>

## Supply Chain Resilience: Export Concentration

```sql supply_chain_stress
SELECT 
  year,
  countryname,
  ROUND(exports_GDP, 1) as exports_pct_gdp,
  ROUND(imports_GDP, 1) as imports_pct_gdp,
  ROUND(exports_USD, 0) as exports_billions,
  ROUND(imports_USD, 0) as imports_billions,
  nGDP_USD as gdp_usd,
  CASE 
    WHEN exports_GDP > 40 THEN 'Export Dependent'
    WHEN exports_GDP > 30 THEN 'Trade Exposed'
    WHEN exports_GDP > 15 THEN 'Moderate Trade'
    ELSE 'Closed Economy'
  END as trade_profile
FROM gmd
WHERE year >= 2018 
  AND year <= 2023
  AND exports_GDP IS NOT NULL
  AND exports_USD IS NOT NULL
  AND countryname IN ('Vietnam', 'Belgium', 'Netherlands', 'South Korea', 'Germany', 'Japan', 'United States', 'China', 'India')
ORDER BY year DESC, exports_GDP DESC
```

<DataTable 
  data={supply_chain_stress}
  rows=60
>
  <Column id=year title="Year"/>
  <Column id=countryname title="Country"/>
  <Column id=exports_pct_gdp title="Exports (% GDP)"/>
  <Column id=imports_pct_gdp title="Imports (% GDP)"/>
  <Column id=exports_billions title="Exports (Billions USD)"/>
  <Column id=trade_profile title="Trade Profile"/>
</DataTable>

## Current Account Imbalances: Who Runs Deficits?

```sql current_account_story
SELECT 
  year,
  countryname,
  ROUND(CA_GDP, 1) as current_account_pct_gdp,
  ROUND(exports_GDP - imports_GDP, 1) as trade_balance_pct_gdp,
  CASE 
    WHEN CA_GDP < -5 THEN 'Large Deficit'
    WHEN CA_GDP < -2 THEN 'Moderate Deficit'
    WHEN CA_GDP > 2 THEN 'Surplus'
    ELSE 'Balanced'
  END as ca_position,
  LAG(CA_GDP) OVER (PARTITION BY countryname ORDER BY year) as prev_ca_pct
FROM gmd
WHERE year >= 2015 
  AND year <= 2023
  AND CA_GDP IS NOT NULL
  AND countryname IN ('United States', 'China', 'Japan', 'Germany', 'India', 'Australia', 'Canada', 'Mexico')
ORDER BY year DESC, CA_GDP ASC
```

<DataTable 
  data={current_account_story}
  rows=80
>
  <Column id=year title="Year"/>
  <Column id=countryname title="Country"/>
  <Column id=current_account_pct_gdp title="Current Account (% GDP)"/>
  <Column id=trade_balance_pct_gdp title="Trade Balance (% GDP)"/>
  <Column id=ca_position title="Position"/>
</DataTable>

## Semiconductor Supply Chain: The Critical Chokepoint

The semiconductor industry perfectly illustrates modern supply chain fragility:
- Design: USA (Intel, Qualcomm, Nvidia)
- Manufacturing: Taiwan (TSMC), South Korea (Samsung), China (SMIC)
- Assembly: Malaysia, Thailand, Vietnam, Philippines
- Materials/Equipment: Japan, Netherlands, Germany

```sql semiconductor_countries_trade
SELECT 
  year,
  countryname,
  ROUND(exports_GDP, 1) as exports_pct_gdp,
  ROUND(imports_GDP, 1) as imports_pct_gdp,
  ROUND(exports_GDP - imports_GDP, 1) as trade_balance_pct,
  CASE 
    WHEN countryname IN ('Taiwan', 'South Korea') THEN 'Manufacturer'
    WHEN countryname IN ('Malaysia', 'Thailand', 'Vietnam', 'Philippines') THEN 'Assembly Hub'
    WHEN countryname IN ('Japan', 'Netherlands') THEN 'Equipment/Materials'
    WHEN countryname = 'United States' THEN 'Design/R&D'
    ELSE 'Other'
  END as supply_chain_role
FROM gmd
WHERE year >= 2015 
  AND year <= 2023
  AND exports_GDP IS NOT NULL
  AND countryname IN ('Taiwan', 'South Korea', 'Malaysia', 'Thailand', 'Vietnam', 'Philippines', 'Japan', 'Netherlands', 'United States')
ORDER BY year DESC, countryname
```

<DataTable 
  data={semiconductor_countries_trade}
  rows=60
>
  <Column id=year title="Year"/>
  <Column id=countryname title="Country"/>
  <Column id=supply_chain_role title="Supply Chain Role"/>
  <Column id=exports_pct_gdp title="Exports (% GDP)"/>
  <Column id=exports_pct_gdp title="Imports (% GDP)"/>
</DataTable>

## Trade Reversal After COVID: Container Ships and Shipping Prices

```sql post_covid_trade_dynamics
SELECT 
  year,
  countryname,
  ROUND(exports_GDP, 1) as exports_pct_gdp,
  ROUND(imports_GDP, 1) as imports_pct_gdp,
  ROUND(nGDP_USD, 0) as gdp_billions,
  LAG(exports_GDP) OVER (PARTITION BY countryname ORDER BY year) as prev_exports_pct,
  ROUND((exports_GDP - LAG(exports_GDP) OVER (PARTITION BY countryname ORDER BY year)), 1) as exports_change_pct_points
FROM gmd
WHERE year >= 2019 
  AND year <= 2023
  AND exports_GDP IS NOT NULL
  AND countryname IN ('China', 'Vietnam', 'Germany', 'Japan', 'South Korea', 'Mexico')
ORDER BY year DESC, exports_GDP DESC
```

<DataTable 
  data={post_covid_trade_dynamics}
  rows=30
>
  <Column id=year title="Year"/>
  <Column id=countryname title="Country"/>
  <Column id=exports_pct_gdp title="Exports (% GDP)"/>
  <Column id=exports_change_pct_points title="Change from Prior Year (pp)"/>
</DataTable>

## Key Insights

### The Trade War Impact (2018-2019)
- **Limited Direct Impact**: Trade war tariffs slowed growth but didn't collapse trade
- **Expectation Effect**: Uncertainty about escalation likely hurt investment more than actual tariffs
- **Adaptation**: Companies sought alternative sourcing (Vietnam benefited)

### COVID Supply Chain Breakdown (2020)
- **Initial Shock**: Global trade fell ~10-15% in 2020
- **Rapid Recovery**: Unlike 2008-2009, trade recovered within 6 months
- **But**: Structural fragility revealed—single chokepoints (Taiwan, Malaysia) matter enormously

### Post-COVID Dynamics (2021-2023)
- **2021**: Trade surged to record levels (pent-up demand + stimulus)
- **2022**: Import slowdown as consumer demand shifted from goods to services
- **2023**: Normalization but at elevated levels vs pre-2020

### De-Risking Trend (2023+)
- **Friend-Shoring**: Companies moving production to politically aligned countries
- **Less China Dependence**: Vietnam, India, Mexico gaining share
- **Higher Costs**: "Just-in-time" replaced by "just-in-case" (inventory buildup)
- **Regionalization**: Supply chains becoming more regional (Europe, USMCA, Asia)

### Current Account Sustainability
- **US Deficits**: Running 3-4% of GDP (structural twin deficits problem)
- **Chinese Surpluses**: Declining but still significant at 1-2%
- **German Surpluses**: Persistent 6%+ despite surplus criticism
- **Risk**: Current account deficits require capital inflows; if these reverse, currency crises can follow

### Winners and Losers
**Winners from Supply Chain Shift**:
- Vietnam (manufacturing shift from China)
- Mexico (nearshoring from US)
- India (cost + scale advantage)

**Losers**:
- China (export growth slowing)
- Advanced economies' manufacturing competitiveness

**Vulnerable**:
- Countries dependent on single export (Malaysia—semiconductors; Vietnam—electronics)
- Assembly-focused economies (face competition from automation)
