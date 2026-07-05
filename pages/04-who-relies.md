---
title: Who relies on FAIR
full_width: true
---

Two different distributions matter and they point in opposite directions:
**exposure** (dollars insured) skews to high-income areas, because their homes are worth
more, while **reliance** (the share of homes on FAIR) skews to low-income, high-hazard
areas, where a larger fraction of households have nowhere else to turn. Every distributional
claim about the FAIR Plan turns on which of these you weight: as a *subsidy* its dollar value
flows upward, as a *safety net* its protective value flows downward. Both statements are
true at once.

## Exposure is pro-rich (+0.29), reliance is pro-poor (−0.21)

```sql indices
select ci_exposure_by_income, ci_penetration_by_income,
  suits_cost_vs_income, kakwani_benefit_vs_income
from fairplan.headline_indices
```

<BigValue data={indices} value=ci_exposure_by_income fmt='+0.000;-0.000' title="Concentration index: exposure"/>
<BigValue data={indices} value=ci_penetration_by_income fmt='+0.000;-0.000' title="Concentration index: reliance"/>




A concentration index is positive when a quantity piles up among richer ZIPs and negative
when it piles up among poorer ones. Exposure dollars concentrate among the richer ZIPs
(**+0.29**); reliance concentrates among the poorer (**−0.21**). 

## Exposure rises with income while reliance falls

```sql scissors
select income_decile,
  'D' || cast(income_decile as int) || ' · $' || cast(round(median_income/1000.0) as int) || 'k' as decile_label,
  exposure_share_pct, penetration_pct
from fairplan.n5_scissors_by_decile
order by income_decile
```

```sql incidence
select i.income_decile,
  'D' || cast(i.income_decile as int) || ' · $' || cast(round(s.median_income/1000.0) as int) || 'k' as decile_label,
  i.exposure_share_pct, i.mean_penetration*100 as mean_penetration_pct
from fairplan.incidence_by_income_decile i
join fairplan.n5_scissors_by_decile s on i.income_decile = s.income_decile
order by i.income_decile
```

<Tabs>

<Tab label="Exposure share vs reliance">

**The top decile holds ten times the exposure share of the bottom but relies on FAIR far less.**

Bars (each decile's share of total FAIR exposure) climb from 2.1% in the poorest decile to
20.2% in the richest: a $2M hillside home simply embodies more insured value than a modest
foothill one. The line (penetration: the share of homes actually on FAIR) runs the other
way.

<BarChart data={scissors} x=decile_label y=exposure_share_pct sort=false
  y2=penetration_pct y2SeriesType=line
  yFmt='0.0"%"' y2Fmt='0.0"%"'
  xAxisTitle="Income decile · median income"
  yAxisTitle="Exposure share (%)" y2AxisTitle="FAIR penetration (%)"
  title="Exposure rises with income; reliance falls"/>

</Tab>

<Tab label="Incidence detail">

**Mean penetration falls from 13.1% in decile 1 to 4.2% in decile 10.**

In
proportion, the poorest communities depend on the insurer of last resort about three
times as heavily as the richest, even as the richest hold the largest slice of its balance
sheet.
<BarChart data={incidence} x=decile_label y=exposure_share_pct sort=false
  y2=mean_penetration_pct y2SeriesType=line
  yFmt='0.0"%"' y2Fmt='0.0"%"'
  xAxisTitle="Income decile · median income"
  yAxisTitle="Exposure share (%)" y2AxisTitle="Mean penetration (%)"/>

</Tab>

</Tabs>

## Premiums take 3.4% of income in the poorest decile, 2.4% in the richest, even though richer ZIPs pay 2.6× more in dollars

The average annual FAIR premium climbs from about $1,660 in decile 1 to $4,391 in decile 10:
wealthier homes cost more to insure in absolute dollars. But scale each premium by the income
available to pay it and the burden inverts: the poorest ZIPs spend **3.4% of median income**
for the Plan's bare fire-only cover, against **2.4%** at the top.

```sql premium_burden
select income_decile,
  'D' || cast(income_decile as int) || ' · $' || cast(round(median_income/1000.0) as int) || 'k' as decile_label,
  premium_pct_of_income, avg_annual_premium
from fairplan.n1_premium_burden_by_decile
order by income_decile
```

<BarChart data={premium_burden} x=decile_label y=premium_pct_of_income sort=false
  yFmt='0.0"%"' xAxisTitle="Income decile · median income"
  yAxisTitle="Premium as % of income" title="Premium burden is regressive"/>

<Details title="Data notes">

 Absolute premiums rise with income (richer
homes cost more to insure); the burden ratio inverts because income rises faster than premium.
Deciles are of ZIPs, so the ratio is a geographic average, not a household bill.

</Details>



## Hold fire risk constant and the income gradient survives: at the same hazard, poorer ZIPs lean on FAIR more

The natural objection is that this is all just geography: poor foothill towns happen to sit
in fire country. The heatmap answers it: reading up any column, penetration rises with
hazard (as it should); but reading across the high-hazard row, reliance stays elevated in
the lower-income deciles. Two communities facing the same fire risk still differ in their
dependence on the Plan according to income. Doubling a ZIP's median income cuts expected penetration by
roughly 4.6 percentage points against a statewide average near 10%. Reliance is not purely an
environmental fact; it carries an income dimension that hazard cannot explain.

```sql pen_heatmap
select cast(cast(income_decile as int) as varchar) as decile, income_decile, hazard_bucket, penetration
from fairplan.n3_penetration_heatmap_long
order by case hazard_bucket when 'High' then 1 when 'Medium' then 2 else 3 end,
  income_decile
```

<Heatmap data={pen_heatmap} x=decile y=hazard_bucket value=penetration
  xSort=income_decile valueFmt='0.0%' colorScale={['#faf6ef','#a1341e']}
  title="FAIR penetration by wildfire hazard × income decile"/>



## Exposure bows below the equality line (pro-rich), reliance bows above it (pro-poor)

Concentration curves plot the cumulative share of each quantity as you walk the ZIPs from
poorest to richest. A curve qround the 45° line means the quantity is spread evenly across
the income distribution; bowing *below* means it accumulates among the rich, bowing *above*
means among the poor. 

```sql lorenz
select cum_zip_share_poor_to_rich, cum_exposure_share, cum_penetration_share
from fairplan.fig07_concentration_curves
order by cum_zip_share_poor_to_rich
```

<LineChart data={lorenz} x=cum_zip_share_poor_to_rich
  y={['cum_exposure_share','cum_penetration_share']}
  xAxisTitle="Cumulative share of ZIPs (poorest → richest)"
  title="Concentration curves: exposure vs reliance">
  <ReferenceLine x=0 y=0 x2=1 y2=1 label="Equality"/>
</LineChart>

<Details title="Data notes">

Each curve is the cumulative share of a quantity as ZIPs are
walked poorest → richest. Bowing below the 45° equality line means the quantity accumulates among
the rich (exposure); bowing above means among the poor (penetration). These are the curves behind
the +0.29 / −0.21 concentration indices at the top of the page.

</Details>

## The counties leaning hardest on FAIR are lower-income and least ready to absorb a surcharge

The highest-penetration
counties (Tuolumne at ~50%, Mariposa, Nevada, Calaveras) sit at the lower-income end of the
distribution, while the affluent metros barely touch the Plan in proportion.

```sql county_scatter
select county, median_income, fair_penetration, fair_exposure_millions, sovi_pctl
from fairplan.n9_county_fair_vs_income
where median_income is not null
```

<BubbleChart data={county_scatter} x=median_income y=fair_penetration
  size=fair_exposure_millions xFmt=usd yFmt='0.0%'
  xAxisTitle="Median income" yAxisTitle="FAIR penetration"
  title="County reliance vs income"/>



---

[Who pays](/05-who-pays)
