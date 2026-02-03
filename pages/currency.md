---
title: Currency Crises & Exchange Rate Turmoil
description: When currencies collapse - the REER story and emerging market crises
---

# Currency Crises: When Capital Flight Happens

Emerging market currencies are vulnerable to sudden reversals. When foreign investors panic and pull money out, currencies collapse within months. Real Effective Exchange Rate (REER) tracks this: when a currency depreciates sharply against trading partners, it signals lost competitiveness and economic crisis.

## The 2008 Financial Crisis: Global Currency Contagion

```sql currency_crisis_2008
WITH base AS (
  SELECT 
    year,
    countryname,
    ROUND(REER, 1) as reer_index,
    ROUND((REER / LAG(REER) OVER (PARTITION BY countryname ORDER BY year) - 1) * 100, 1) as reer_change_pct
  FROM gmd
  WHERE year >= 2005 
    AND year <= 2012
    AND REER IS NOT NULL
),
top_countries AS (
  SELECT countryname
  FROM (
    SELECT countryname, MAX(ABS(reer_change_pct)) as max_abs
    FROM base
    GROUP BY countryname
    ORDER BY max_abs DESC
    LIMIT 8
  )
)
SELECT *
FROM base
WHERE countryname IN (SELECT countryname FROM top_countries)
ORDER BY year, countryname
```

<BarChart
  data={currency_crisis_2008}
  x=countryname
  y=reer_change_pct
  series=year
  title="Currency Crash Around 2008: REER Annual Change (%)"
  yAxisTitle="Annual REER Change (%)"
  xAxisTitle="Country"
  sort=true
/>

**The Pattern**: When Lehman collapsed in 2008, investors fled emerging markets. Currencies depreciated 10-20% as capital fled. Iceland's króna crashed hardest; emerging Asian currencies also weakened sharply.

## Emerging Market Currency Stress: The Triangle of Vulnerability

```sql emerging_currency_stress
SELECT 
  year,
  countryname,
  ROUND(REER, 1) as reer_index,
  ROUND(infl, 1) as inflation_rate,
  ROUND(CA_GDP, 1) as current_account_pct_gdp
FROM gmd
WHERE year >= 2013 
  AND year <= 2023
  AND REER IS NOT NULL
  AND countryname IN ('Argentina', 'Brazil', 'Turkey', 'Mexico', 'South Africa', 'India')
ORDER BY year DESC, countryname
```

<BubbleChart
  data={emerging_currency_stress}
  x=current_account_pct_gdp
  y=inflation_rate
  size=reer_index
  series=countryname
  title="Emerging Markets: The Currency Crisis Triangle"
  xAxisTitle="Current Account (% GDP)"
  yAxisTitle="Inflation Rate (%)"
/>

**The Three Vulnerabilities**:
1. **Current Account Deficits**: Money flowing out faster than in (negative CA)
2. **High Inflation**: Loss of purchasing power drives currency weakness
3. **Low Reserves/Weak REER**: When the exchange rate is already low, there's nowhere left to fall before crisis

Turkey and Argentina show all three signs—no surprise both experienced severe currency crises in 2018 and 2019 respectively.

## The Carry Trade Collapse: When Rate Hikes Hit Emerging Markets

```sql carry_trade_story
SELECT 
  year,
  countryname,
  ROUND(cbrate, 2) as policy_rate,
  ROUND(REER, 1) as reer_index,
  ROUND((REER / LAG(REER) OVER (PARTITION BY countryname ORDER BY year) - 1) * 100, 1) as reer_change_pct,
  ROUND(infl, 1) as inflation_rate
FROM gmd
WHERE year >= 2017 
  AND year <= 2023
  AND REER IS NOT NULL
  AND countryname IN ('Turkey', 'Argentina', 'Brazil', 'Mexico', 'India')
ORDER BY year DESC, countryname
```

<LineChart
  data={carry_trade_story}
  x=year
  y=reer_index
  series=countryname
  title="REER During Fed Rate Hikes (2017-2023)"
  yAxisTitle="REER Index"
  xAxisTitle="Year"
/>

**What Happened**: 
- **2017-2018**: US Fed raised rates while emerging markets had low rates. Carry traders borrowed cheap money in EM currencies and invested in expensive US assets.
- **2022-2023**: Fed cut growth forecasts, rates peaked at 5.5%, and carry trades unwound. EM currencies collapsed as capital fled back to dollars.
- **Result**: Argentina's peso lost 60% of value; Turkish lira fell 40%; Brazil's real down 20%+

This shows the power asymmetry: when the Fed moves, emerging markets have no choice but to follow or face currency collapse.

## Currency Crisis Epidemic: 2010-2023

```sql crisis_count
SELECT 
  year,
  countryname,
  ROUND(REER, 1) as reer_index,
  ROUND(infl, 1) as inflation_rate,
  CurrencyCrisis
FROM gmd
WHERE (CurrencyCrisis = 1 OR year IN (2018, 2019, 2022))
  AND year >= 2010
  AND year <= 2023
  AND REER IS NOT NULL
ORDER BY year DESC, countryname
```

<BarChart
  data={crisis_count}
  x=countryname
  y=reer_index
  series=year
  title="Currency Crises: REER Index When Crises Hit"
  yAxisTitle="REER Index"
  xAxisTitle="Country"
  sort=true
/>

**Crisis Timeline**:
- **2013**: Taper tantrum (India, Brazil, Indonesia currency stress)
- **2014-2015**: Oil collapse (Russia ruble crisis)
- **2018**: Turkey currency crisis (escalating US tariffs + rate hikes)
- **2019**: Argentina peso collapse (fiscal crisis + capital controls)
- **2022-2023**: Post-Fed hikes (Brazil, Mexico stress)

## Key Vulnerability Indicators

When to worry about currency collapse:

1. **Current Account Deficit > 4% of GDP**: Capital flight risk rises
2. **Inflation > 8%**: Central bank likely to raise rates, but delayed response makes currency vulnerable
3. **REER Already Low**: If the exchange rate is already 20% below long-term average, less room to fall
4. **Foreign Debt Denominated in Foreign Currency**: When currency depreciates, debt service cost rises (vicious cycle)
5. **US Fed Hiking Cycle**: EM currencies always suffer when Fed tightens

**Brazil, India, Mexico are Relatively Safe**: Strong reserves, managed floats, diverse economies.

**Argentina & Turkey are High Risk**: Large deficits, high inflation, weak reserves—next crisis likely within 18-24 months.

---

**Data Source**: Global Macro Database (1960-2023) | **Last Updated**: <LastRefreshed/>
