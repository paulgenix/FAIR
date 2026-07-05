---
title: The FAIR Plan's growth
full_width: true
---

The plan of last resort became a pillar of the market in four years: residential exposure
rose from **$155.7B to $645.1B** (a 4.1× expansion) while policies in force grew from
**236,000 to 621,000**. Growth is not spread evenly: it concentrates in a handful of
high-risk ZIP codes and the standard FAIR policy leaves a coverage gap most
policyholders never fill.

## Exposure quadrupled in four years ($156B → $645B) but policies grew only half as fast, so the homes arriving are markedly more valuable

Exposure grew roughly 30% a year through FY2023, then accelerated sharply: FY2024 was the
steepest proportional jump (+59%), immediately after the largest private withdrawals were
announced in 2023, and FY2025 added the most dollars in any single year (+$214B).

Policies in force rose 163% over the same window that exposure rose 314%. When insured value
grows twice as fast as policy count, the average home entering the Plan is bigger-ticket than
the ones it already covered: evidence that private insurers retreated from high-value
coastal and hillside markets, not merely modest rural ones.

```sql exposure_growth
select cast(fy as int)::varchar as fy, exposure_billions, policies_in_force
from fairplan.growth_by_fiscal_year
order by fy
```

```sql pif_growth
select cast(fy as int)::varchar as fy, policies_in_force
from fairplan.growth_by_fiscal_year
order by fy
```

<Grid cols=2>

<AreaChart data={exposure_growth} x=fy y=exposure_billions sort=false
  yAxisTitle="Exposure ($B)"
  title="FAIR Plan residential exposure exploded, FY2021–FY2025"/>

<LineChart data={pif_growth} x=fy y=policies_in_force yFmt='#,##0' sort=false
  yAxisTitle="Policies in force" title="Policies in force, FY2021–FY2025"/>

</Grid>

<Details title="Data notes">

Commercial-line files are excluded; this series is residential only.

</Details>



## The growth is not in safe places: the top-20 growth ZIPs sit at a median 80th percentile of wildfire risk

FAIR is absorbing exactly the exposure the voluntary market is shedding. Truckee (96161)
alone added **$4.9B** of FAIR exposure between 2021 and 2024 its book nearly tripled and
the rest of the list reads like a WUI gazetteer: Tahoe, the Sierra foothills, the Los Angeles
hills. Note the last column: many of these ZIPs pair explosive FAIR growth with elevated
non-renewal rates. 

```sql top_zips
with risk as (
  select zip, fair_risk_score, nonrenew_rate,
    round(100 * percent_rank() over (order by fair_risk_score)) as risk_pctl
  from fairplan.map_zip
  where fair_risk_score is not null
)
select t.rank, t.region, t.zip, t.exposure_2024, t.change_2021_2024,
  r.risk_pctl, r.nonrenew_rate
from fairplan.ex02_top20_zips_exposure t
left join risk r on t.zip = r.zip
order by t.rank
```

<Tabs>

<Tab label="Top-20 growth ZIPs (table)">

<DataTable data={top_zips} rows=20>
  <Column id=rank/>
  <Column id=region/>
  <Column id=zip/>
  <Column id=exposure_2024 fmt='$#,##0,,"M"' contentType=colorscale/>
  <Column id=change_2021_2024 fmt='$#,##0,,"M"' contentType=colorscale/>
  <Column id=risk_pctl title="Fire-risk percentile" fmt='0"th"' contentType=colorscale scaleColor=#b5442e/>
  <Column id=nonrenew_rate title="Non-renewal rate" fmt='0.0%'/>
</DataTable>

</Tab>

<Tab label="Growth vs fire risk (chart)">





The chart below plots the same twenty ZIPs: growth on the vertical, fire-risk percentile on
the horizontal, bubble size the ZIP's total 2024 FAIR exposure, and color its 2021
non-renewal intensity. Nearly
everything sits in the right half of the chart. A pool whose fastest growth is also its most
fire-exposed is accumulating precisely the correlated tail risk that ordinary insurance
pooling cannot diversify away.

```sql growth_vs_risk
select t.region, t.change_2021_2024/1000000000.0 as growth_b,
  r.risk_pctl, t.exposure_2024,
  case when r.nonrenew_rate >= 0.15 then 'Non-renewals > 15%'
       when r.nonrenew_rate >= 0.10 then 'Non-renewals 10–15%'
       else 'Non-renewals < 10%' end as retreat_intensity
from fairplan.ex02_top20_zips_exposure t
join (
  select zip, nonrenew_rate,
    round(100 * percent_rank() over (order by fair_risk_score)) as risk_pctl
  from fairplan.map_zip where fair_risk_score is not null
) r on t.zip = r.zip
```

<BubbleChart data={growth_vs_risk} x=risk_pctl y=growth_b
  size=exposure_2024 sizeFmt='$#,##0,,"M"' opacity=0.75
  series=retreat_intensity
  seriesColors={{'Non-renewals > 15%':'#b5442e','Non-renewals 10–15%':'#dd7f3a','Non-renewals < 10%':'#a7a099'}}
  seriesOrder={['Non-renewals > 15%','Non-renewals 10–15%','Non-renewals < 10%']}
  tooltipTitle=region chartAreaHeight=380
  xAxisTitle="ZIP wildfire-risk percentile" yAxisTitle="Exposure growth 2021–24 ($B)"
  title="FAIR's fastest-growing ZIPs are its most fire-exposed">
  <ReferencePoint data={growth_vs_risk} x=risk_pctl y=growth_b label=region
    labelPosition=right symbolSize=1 color=#736c63/>
</BubbleChart>

</Tab>

</Tabs>



## FAIR is fire-only cover and roughly 115,000 households have no DIC to complete it

FAIR's basic dwelling-fire policy excludes perils a standard homeowner policy covers:
liability, theft, water damage. A separate difference-in-conditions (DIC) wrap fills the gap.
The ratio of DIC to FAIR policies has improved (47% in 2020 to 64% in 2023), but because
FAIR enrolment keeps climbing, the *absolute* number of households with bare fire-only cover
has stayed stuck around 115,000–117,000 a group the thesis finds is disproportionately
lower-income. The Plan's growth is therefore also a growth in quiet underinsurance against
the very perils that accompany wildfire.

```sql dic_gap
select year, fair_policies, dic_policies, dic_to_fair_ratio_pct
from fairplan.n12_dic_underinsurance_gap order by year
```

<BarChart data={dic_gap} x=year y={['fair_policies','dic_policies']} type=grouped
  y2=dic_to_fair_ratio_pct y2SeriesType=line y2Fmt='#,##0"%"' y2AxisTitle="DIC / FAIR (%)"
  title="Difference-in-conditions coverage lags FAIR growth"/>

<Details title="Data notes">

DIC counts are market estimates (DIC wraps are sold by private carriers, not
the Plan), so the ratio is approximate. The ~115,000 households with bare fire-only cover is the
residual (FAIR − DIC), and stays roughly flat because DIC growth only keeps pace with FAIR's.

</Details>

---

Who is actually in it? → [Who relies on FAIR](/04-who-relies)
