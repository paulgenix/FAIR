---
title: The insurance retreat
full_width: true
---

After the 2017–18 fire seasons the insurance market repriced and retreated. Under Proposition
103, insurers cannot raise rates without prior approval, and rates have historically been
justified on backward-looking loss experience so when expected losses jumped, approved prices could not follow. An insurer that cannot charge for a risk has one
lever left: decline to renew the policy at expiry. Insurer-initiated non-renewals surged in
the highest-risk counties, the coverage that remained got more expensive per dollar insured,
and when an owner exhausts the admitted market, the FAIR Plan is what's left.

## Most non-renewals are normal churn but the insurer-initiated band is the retreat signal, and it grew to ~242,000 by 2021

The blue band is consumers switching carriers of their own: normal market churn.
The red band is insurers refusing to renew: the real retreat signal, and the one that
pushes homes onto FAIR. It climbs from ~165,000 in 2017–18 to roughly 242,000 in 2021, the
last year the insurer-vs-consumer split is reported.  
**Note**: After major wildfires for which a state of emergency is declared, California law (SB 824) imposes a one-year moratorium preventing insurers from cancelling or non-renewing residential policies in ZIP codes within or near the fire perimeter based solely on wildfire risk.

```sql nonrenew_split
select year, insurer_initiated, consumer_initiated
from fairplan.n10_nonrenewal_insurer_vs_consumer order by year
```

<BarChart data={nonrenew_split} x=year y={['insurer_initiated','consumer_initiated']}
  type=stacked yAxisTitle="Non-renewals" title="Who ends the policy?"/>

<Details title="Data notes">

The
insurer-vs-consumer split is only reported through 2021 CDI stopped publishing the breakdown
afterward which is why the series ends there. Post-fire SB 824 moratoria freeze non-renewals in
affected ZIP-years, so a dip can reflect a legal freeze rather than a actual market recovery.

</Details>

## Where insurers walked away: the fifteen hardest-hit counties are all foothill and mountain interior

Statewide averages conceal where the pressure actually sits. The fifteen hardest-hit counties
are the Sierra foothill and mountain interior: Tuolumne (18.1%, roughly one policy in six),
Nevada (17.2%), Lake, Calaveras, Plumas.

```sql county_nonrenew
select e.county, e.nonrenew_rate, e.one_in_x,
  case when m.fair_penetration >= 0.25 then 'FAIR covers > 25% of homes'
       when m.fair_penetration >= 0.10 then 'FAIR covers 10–25%'
       else 'FAIR covers < 10%' end as fair_reliance
from fairplan.ex05b_nonrenewal_rate_by_county_2023 e
left join fairplan.map_county m on e.county = m.county
order by e.nonrenew_rate desc limit 15
```

<BarChart data={county_nonrenew} x=county y=nonrenew_rate swapXY=true
  series=fair_reliance
  seriesColors={{'FAIR covers > 25% of homes':'#b5442e','FAIR covers 10–25%':'#dd7f3a','FAIR covers < 10%':'#a7a099'}}
  seriesOrder={['FAIR covers > 25% of homes','FAIR covers 10–25%','FAIR covers < 10%']}
  yFmt='0.0%' yAxisTitle="Non-renewal rate (2023)"
  title="Top-15 counties by 2023 non-renewal rate, colored by FAIR reliance"/>

<Details title="Data notes">

This is the combined 2023 non-renewal rate: insurer- and consumer-initiated together so
it blends the retreat signal with ordinary churn; the pure insurer-initiated series ends in 2021.
Reliance bands are FAIR penetration = policies ÷ (policies + admitted policies) in the county.

</Details>

## Insurers drop by hazard, not income: non-renewal is flat across income deciles but climbs step by step across risk deciles

Drop rates are nearly flat across income, the income story emerges in who lands on FAIR.
Average ZIP non-renewal rates is about 11.2% in
the poorest decile and 10.7% in the richest. Insurers
drop policies by hazard, not by income. The distributional sorting happens after the
drop: households that can afford surplus-lines coverage, higher premiums, or mitigation
investments find their way back into the private market, while those that cannot land on
FAIR.


```sql drop_by_decile
select r.income_decile,
  'D' || cast(r.income_decile as int) || ' · $' || cast(round(s.median_income/1000.0) as int) || 'k' as decile_label,
  avg(r.total_nonrenew_rate_2021) as avg_nonrenew_rate
from fairplan.n4_retreat_reliance r
join fairplan.n5_scissors_by_decile s on r.income_decile = s.income_decile
where r.total_nonrenew_rate_2021 is not null
group by r.income_decile, s.median_income
order by r.income_decile
```

```sql risk_retreat_decile
with scored as (
  select fair_risk_score, nonrenew_rate,
    ntile(10) over (order by fair_risk_score) as risk_decile
  from fairplan.map_zip
  where fair_risk_score is not null and nonrenew_rate is not null
)
select risk_decile, avg(nonrenew_rate) as avg_nonrenew
from scored
group by risk_decile
order by risk_decile
```

<Grid cols=2>

<BarChart data={drop_by_decile} x=decile_label y=avg_nonrenew_rate sort=false
  yFmt='0.0%' xAxisTitle="Income decile · median income" yAxisTitle="Avg ZIP non-renewal rate (2021)"
  title="Non-renewal rates by income decile"/>

<BarChart data={risk_retreat_decile} x=risk_decile y=avg_nonrenew
  yFmt='0.0%' xAxisTitle="ZIP wildfire-risk decile (10 = highest risk)"
  yAxisTitle="Avg non-renewal rate"
  title="Insurers drop policies where the fire risk is"/>

</Grid>

<Details title="Data notes">

Deciles are equal-count groups of ZIPs ranked by ACS median income (D1 = poorest
tenth), not of households: a poor household in a rich ZIP is averaged into that ZIP's decile. The
rate is the combined total. ZIPs missing
either a risk score or a non-renewal rate are excluded from the risk-decile chart.

</Details>



```sql county_risk_retreat
select c.county, n.wildfire_risk_score, c.nonrenew_rate, c.fair_exposure_millions,
  c.median_income, c.fair_penetration,
  case when c.fair_penetration >= 0.25 then 'FAIR covers > 25% of homes'
       when c.fair_penetration >= 0.10 then 'FAIR covers 10–25%'
       else 'FAIR covers < 10%' end as fair_reliance
from fairplan.map_county c
join fairplan.n9_county_fair_vs_income n on c.county = n.county
where c.nonrenew_rate is not null and n.wildfire_risk_score is not null
```

```sql county_risk_labels
select c.county, n.wildfire_risk_score, c.nonrenew_rate
from fairplan.map_county c
join fairplan.n9_county_fair_vs_income n on c.county = n.county
where c.nonrenew_rate >= 0.14 or c.county in ('Los Angeles', 'San Francisco')
```

<BubbleChart data={county_risk_retreat} x=wildfire_risk_score y=nonrenew_rate
  size=fair_exposure_millions tooltipTitle=county chartAreaHeight=380
  series=fair_reliance
  seriesColors={{'FAIR covers > 25% of homes':'#b5442e','FAIR covers 10–25%':'#dd7f3a','FAIR covers < 10%':'#a7a099'}}
  seriesOrder={['FAIR covers > 25% of homes','FAIR covers 10–25%','FAIR covers < 10%']}
  scaleTo=1.5 sizeFmt='$#,##0"M"' opacity=0.75
  yFmt='0.0%' xAxisTitle="County wildfire-risk percentile" yAxisTitle="Non-renewal rate"
  title="Risk and retreat, county by county">
  <ReferencePoint data={county_risk_labels} x=wildfire_risk_score y=nonrenew_rate label=county
    labelPosition=right symbolSize=1 color=#736c63/>
</BubbleChart>

<Details title="Data notes">

Counties missing a rate or a risk score are dropped

</Details>

## Staying insured costs more on FAIR: its rate per $1,000 has run well above the voluntary market in every year but 2023

Last-resort cover is deliberately basic and not cheap. FAIR's rate per $1,000 of coverage
has run roughly 1.3–1.7× the voluntary homeowners rate across the period (they converged
only in 2023, before FAIR's 2024 rate action re-opened the gap). The Plan's statutory mandate
is access, not competitiveness: it is meant to be the option you take when there is no
other.

```sql rate_compare
select cast(year as int)::varchar as year, segment, rate_per_1000 from fairplan.n7_premium_rate_vs_voluntary order by year
```

<LineChart data={rate_compare} x=year y=rate_per_1000 series=segment sort=false
  yAxisTitle="Rate per $1,000 of coverage" title="FAIR costs more per dollar insured"/>

<Details title="Data notes">

FAIR vs voluntary-market rate per $1,000 of coverage, by year.
This is a rate per dollar insured, not an average premium. The two segments buy different coverage
(FAIR is basic dwelling-fire, the voluntary market is a full homeowners policy), so the raw ratio
understates FAIR's real cost gap once the missing perils are priced back in.

</Details>

## FAIR's share more than doubled after 2018 (1.6% → 3.8%) reaching 32.6% in the ten riskiest counties

FAIR's share of the residential market more than doubled after 2018: from 1.6% to 3.8%…
For two decades the residual market was below 2% of the state's residential policies. The
inflection lands precisely between the 2017–18 fire seasons and the non-renewal surge above
the timing that identifies FAIR's growth as displaced demand from a contracting private
market, not the Plan competing business away.

10% above-median risk, 32.6% in the ten riskiest counties.
FAIR covers one home in ten in counties above median wildfire risk, and nearly **one
home in three in the ten riskiest counties**. In the places that matter, the insurer of last
resort has become a primary market and its book is concentrated in exactly the geography
where a single wind event correlates every policy at once.

```sql fair_share
select year, fair_share_pct from fairplan.n11_fair_market_share_by_year order by year
```

```sql share_tier
select tier, fair_share_2023_pct from fairplan.n11b_fair_share_by_risk_tier_2023
order by fair_share_2023_pct
```

<Grid cols=2>

<LineChart data={fair_share} x=year y=fair_share_pct yFmt='0.0"%"' yAxisTitle="FAIR share (%)"
  title="FAIR's share of the market is climbing"/>

<BarChart data={share_tier} x=tier y=fair_share_2023_pct swapXY=true sort=false
  yFmt='0.0"%"' yAxisTitle="FAIR share 2023 (%)" title="Reliance concentrates in high-risk counties"/>

</Grid>

<Details title="Data notes">

This is a statewide average and understates concentration: the same figure split by county risk
tier (the right-hand chart) shows reliance is far higher where the hazard is.

</Details>



## Where insurers pulled out in 2021, FAIR reliance had risen by 2023

Each bubble is a ZIP code non-renewal intensity in 2021 on the horizontal, FAIR penetration
two years later on the vertical, sized by exposure and colored by income decile.

```sql retreat
select total_nonrenew_rate_2021, fair_penetration_2023, exposure_millions,
  'D' || cast(income_decile as int) as decile
from fairplan.n4_retreat_reliance
where total_nonrenew_rate_2021 is not null and fair_penetration_2023 is not null
order by income_decile
```

<BubbleChart data={retreat} x=total_nonrenew_rate_2021 y=fair_penetration_2023
  size=exposure_millions series=decile sort=false
  seriesOrder={['D1','D2','D3','D4','D5','D6','D7','D8','D9','D10']}
  xAxisTitle="Non-renewal rate 2021" yAxisTitle="FAIR penetration 2023"
  xFmt='0.0%' yFmt='0.0%' title="Retreat drives reliance"/>

<Details title="Data notes">

ZIPs missing either measure are excluded.

</Details>

**How to read the legend:** D1–D10 are statewide *income deciles* of ZIP codes. Every
FAIR-Plan ZIP is ranked by its median household income (from the American Community Survey)
and the ranking is cut into ten equal groups: D1 is the poorest tenth of ZIPs, D10 the
richest the same deciles used throughout the reliance and incidence pages. Toggle a
decile in the legend to isolate it: the low deciles ride higher on the vertical axis at any
given non-renewal rate, which is the income gradient in reliance that the next page measures.

---

Where did those non-renewed homeowners go? → [The FAIR Plan's Growth](/03-fair-balloons)
