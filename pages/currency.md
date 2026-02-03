---
title: Currency Crises & Exchange Rate Turmoil
description: When currencies collapse - the REER story and sovereign debt crises
---

# Currency Crises: When the Local Currency Becomes Toxic

Real Effective Exchange Rate (REER) movements reveal currency crises and capital flight. When a currency depreciates sharply against a basket of trading partners' currencies, it often signals economic distress. This page correlates currency movements with crisis events.

## The 2008 Financial Crisis: Currency Contagion

```sql currency_crisis_2008
SELECT 
  year,
  countryname,
  ROUND(REER, 1) as reer_index,
  CurrencyCrisis,
  SovDebtCrisis,
  LAG(REER) OVER (PARTITION BY countryname ORDER BY year) as prev_reer,
  ROUND((REER / LAG(REER) OVER (PARTITION BY countryname ORDER BY year) - 1) * 100, 1) as reer_change_pct
FROM gmd
WHERE year >= 2005 
  AND year <= 2012
  AND REER IS NOT NULL
  AND (CurrencyCrisis = 1 OR year IN (2008, 2009, 2011))
ORDER BY year DESC, countryname
```

<DataTable 
  data={currency_crisis_2008}
  rows=60
>
  <Column id=year title="Year"/>
  <Column id=countryname title="Country"/>
  <Column id=reer_index title="REER Index"/>
  <Column id=reer_change_pct title="Annual Change (%)"/>
  <Column id=CurrencyCrisis title="Currency Crisis"/>
  <Column id=SovDebtCrisis title="Sovereign Debt Crisis"/>
</DataTable>

## Emerging Market Currency Crises: The Contagion Effect

```sql emerging_currency_stress
SELECT 
  year,
  countryname,
  ROUND(REER, 1) as reer_index,
  ROUND(infl, 1) as inflation_rate,
  ROUND(govdebt_GDP, 1) as govt_debt_pct_gdp,
  ROUND(CA_GDP, 1) as current_account_pct_gdp,
  CurrencyCrisis
FROM gmd
WHERE year >= 2013 
  AND year <= 2023
  AND REER IS NOT NULL
  AND countryname IN ('Argentina', 'Brazil', 'Turkey', 'Russia', 'Mexico', 'South Africa', 'Indonesia')
ORDER BY year DESC, countryname
```

<DataTable 
  data={emerging_currency_stress}
  rows=80
>
  <Column id=year title="Year"/>
  <Column id=countryname title="Country"/>
  <Column id=reer_index title="REER"/>
  <Column id=inflation_rate title="Inflation (%)"/>
  <Column id=govt_debt_pct_gdp title="Govt Debt (% GDP)"/>
  <Column id=current_account_pct_gdp title="Current Account (% GDP)"/>
  <Column id=CurrencyCrisis title="Currency Crisis"/>
</DataTable>

## The Carry Trade Unwind: Rising US Rates and Currency Collapse

```sql carry_trade_story
WITH rate_data AS (
  SELECT 
    year,
    countryname,
    ROUND(cbrate, 2) as policy_rate,
    ROUND(REER, 1) as reer_index,
    ROUND(infl, 1) as inflation_rate,
    LAG(REER) OVER (PARTITION BY countryname ORDER BY year) as prev_reer,
    ROUND((REER / LAG(REER) OVER (PARTITION BY countryname ORDER BY year) - 1) * 100, 1) as reer_change_pct
  FROM gmd
  WHERE year >= 2017 
    AND year <= 2023
    AND REER IS NOT NULL
)
SELECT 
  year,
  countryname,
  policy_rate,
  reer_index,
  reer_change_pct,
  inflation_rate,
  CASE 
    WHEN reer_change_pct < -10 THEN 'Sharp Depreciation'
    WHEN reer_change_pct < -5 THEN 'Moderate Depreciation'
    WHEN reer_change_pct > 5 THEN 'Appreciation'
    ELSE 'Stable'
  END as movement
FROM rate_data
WHERE countryname IN ('Turkey', 'Argentina', 'Brazil', 'Russia', 'Mexico', 'India', 'South Africa')
ORDER BY year DESC, reer_change_pct ASC
```

<DataTable 
  data={carry_trade_story}
  rows=50
>
  <Column id=year title="Year"/>
  <Column id=countryname title="Country"/>
  <Column id=policy_rate title="Policy Rate (%)"/>
  <Column id=reer_index title="REER Index"/>
  <Column id=reer_change_pct title="Annual Change (%)"/>
  <Column id=inflation_rate title="Inflation (%)"/>
  <Column id=movement title="Movement Type"/>
</DataTable>

## The "Trilemma" Problem: You Can't Have All Three

The impossible trinity states: A country cannot simultaneously maintain:
1. Free capital flows
2. Fixed exchange rate  
3. Independent monetary policy

When maintaining all three becomes impossible, currency crises occur.

```sql trilemma_countries
SELECT 
  year,
  countryname,
  ROUND(REER, 1) as exchange_rate_stability,
  ROUND(cbrate, 2) as policy_independence,
  ROUND(CA_GDP, 1) as capital_flow_measure,
  CurrencyCrisis,
  CASE 
    WHEN CurrencyCrisis = 1 THEN 'Trilemma Failed'
    WHEN REER < 95 OR REER > 105 THEN 'Exchange Pressure'
    ELSE 'Stable'
  END as status
FROM gmd
WHERE year >= 2010 
  AND year <= 2023
  AND REER IS NOT NULL
  AND (CurrencyCrisis = 1 OR countryname IN ('China', 'Brazil', 'Turkey', 'Argentina', 'Russia'))
ORDER BY year DESC, countryname
```

<DataTable 
  data={trilemma_countries}
  rows=70
>
  <Column id=year title="Year"/>
  <Column id=countryname title="Country"/>
  <Column id=exchange_rate_stability title="REER Index"/>
  <Column id=policy_independence title="Policy Rate (%)"/>
  <Column id=capital_flow_measure title="Current Account (% GDP)"/>
  <Column id=status title="Trilemma Status"/>
</DataTable>

## Specific Crisis Episodes

```sql crisis_episodes
SELECT 
  year,
  countryname,
  CASE 
    WHEN year = 2008 AND countryname = 'Iceland' THEN 'Icelandic Banking Crisis'
    WHEN year BETWEEN 2010 AND 2012 AND countryname IN ('Greece', 'Ireland', 'Portugal') THEN 'European Sovereign Crisis'
    WHEN year = 2013 AND countryname IN ('India', 'Brazil', 'Indonesia', 'Turkey') THEN 'Taper Tantrum'
    WHEN year BETWEEN 2014 AND 2015 AND countryname IN ('Russia', 'Brazil') THEN 'Commodity Collapse'
    WHEN year = 2018 AND countryname = 'Turkey' THEN 'Turkey Currency Crisis'
    WHEN year = 2019 AND countryname = 'Argentina' THEN 'Argentine Default'
    WHEN year = 2022 AND countryname = 'UK' THEN 'UK Gilts Crisis'
    ELSE 'Normal'
  END as crisis_name,
  ROUND(REER, 1) as reer_index,
  ROUND(infl, 1) as inflation_rate,
  ROUND(unemp, 1) as unemployment_rate,
  CurrencyCrisis,
  SovDebtCrisis
FROM gmd
WHERE (CurrencyCrisis = 1 OR SovDebtCrisis = 1)
  AND year >= 2007
  AND year <= 2023
ORDER BY year DESC, countryname
```

<DataTable 
  data={crisis_episodes}
  rows=50
>
  <Column id=year title="Year"/>
  <Column id=countryname title="Country"/>
  <Column id=crisis_name title="Crisis Event"/>
  <Column id=reer_index title="REER"/>
  <Column id=inflation_rate title="Inflation (%)"/>
  <Column id=unemployment_rate title="Unemployment (%)"/>
</DataTable>

## Modern Vulnerabilities: Which Emerging Markets Are at Risk?

```sql risk_assessment
SELECT 
  year,
  countryname,
  ROUND(CA_GDP, 1) as current_account_pct_gdp,
  ROUND(infl, 1) as inflation_rate,
  ROUND(REER, 1) as reer_index,
  ROUND(govdebt_GDP, 1) as government_debt_pct_gdp,
  ROUND(M2, 0) as m2_level,
  CASE 
    WHEN CA_GDP < -5 AND infl > 10 THEN 'High Risk'
    WHEN CA_GDP < -3 OR infl > 8 THEN 'Elevated Risk'
    ELSE 'Lower Risk'
  END as vulnerability_assessment
FROM gmd
WHERE year = 2023 
  AND CA_GDP IS NOT NULL
  AND infl IS NOT NULL
  AND countryname IN ('Argentina', 'Turkey', 'Brazil', 'Mexico', 'India', 'South Africa', 'Indonesia', 'Thailand', 'Philippines', 'Vietnam', 'Poland')
ORDER BY vulnerability_assessment DESC, infl DESC
```

<DataTable 
  data={risk_assessment}
  rows=20
>
  <Column id=year title="Year"/>
  <Column id=countryname title="Country"/>
  <Column id=current_account_pct_gdp title="Current Account (% GDP)"/>
  <Column id=inflation_rate title="Inflation (%)"/>
  <Column id=reer_index title="REER"/>
  <Column id=government_debt_pct_gdp title="Govt Debt (% GDP)"/>
  <Column id=vulnerability_assessment title="Risk Level"/>
</DataTable>

## Key Insights

- **2008 Crisis Spread Through Currencies**: Countries dependent on external financing saw sharp REER depreciation
  - Iceland's currency collapsed 50%+
  - Emerging markets suffered "sudden stop" in capital flows

- **European Crisis (2010-2012)**: REER movements revealed divergence
  - Southern Europe (Greece, Ireland, Portugal) couldn't devalue to restore competitiveness
  - This caused the crisis to deepen—trapped in euro without escape valve

- **Taper Tantrum (2013)**: When US began signaling rate hikes
  - Carry trade unwound (borrow in low-rate countries, invest in US)
  - Emerging market currencies crashed simultaneously

- **Commodity Shock (2014-2015)**: Oil/commodity collapse hit commodity exporters
  - Russia, Brazil saw sharp depreciation
  - Both experienced currency crises

- **2022 UK Gilts Crisis**: Unlike typical EM crises, but illustrates:
  - Even advanced economies vulnerable if policy credibility lost
  - Currency reflected gilt market stress

- **2023 Emerging Market Vulnerabilities**:
  - Argentina: In currency crisis regime with 140%+ inflation
  - Turkey: Repeated crises, REER volatility persistent
  - India: Relatively insulated despite Fed tightening

## The Global Connection

Currency crises are no longer isolated. In a world of:
- High capital mobility
- Dollar dominance (75% of forex reserves)
- Low interest rates in developed markets

...emerging markets face perpetual vulnerability to:
- Capital flight when US rates rise
- Currency depreciation reducing dollar debt servicing ability
- Inflation from import prices (when using dollar-denominated debt)

This creates a debt trap: borrow in dollars, depreciate when rates rise, struggle to repay.
