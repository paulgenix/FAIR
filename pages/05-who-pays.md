---
title: Who pays
full_width: true
---

The $1B assessment is billed to insurers by market
share, insurers recoup up to half from policyholders by premium, and the implicit
backstop benefit is allocated back to ZIPs by the exposure FAIR guarantees there. Net
those two flows against each other, decile by decile, and deciles 1–6 are net recipients and
7–10 net payers.

## Who nets out



**Deciles 1–6 receive, 7–10 pay**

```sql net_transfer
select t.income_decile,
  'D' || cast(t.income_decile as int) || ' · $' || cast(round(s.median_income/1000.0) as int) || 'k' as decile_label,
  t.net_millions,
  case when t.net_millions >= 0 then 'Net recipient' else 'Net payer' end as position
from fairplan.net_transfer_by_decile t
join fairplan.n5_scissors_by_decile s on t.income_decile = s.income_decile
order by t.income_decile
```

<BarChart data={net_transfer} x=decile_label y=net_millions series=position sort=false
  seriesColors={{'Net recipient':'#4f7a5c','Net payer':'#b5442e'}}
  xAxisTitle="Income decile · median income" yAxisTitle="Net transfer ($M)"
  title="Deciles 1–6 are net recipients; 7–10 are net payers"/>

<Details title="Data notes">

net = backstop benefit − surcharge cost. Cost = each decile's share of statewide homeowners premium
(proxied by admitted policies × median home value); benefit = its share of FAIR exposure; both scale
the recoupable half of the $1B assessment (baseline 50%).

</Details>







```sql mc_bands
with bands as (
  select income_decile, 'p5' as stat, p5_millions as value from fairplan.fig09_mc_net_transfer_bands
  union all select income_decile, 'median', median_millions from fairplan.fig09_mc_net_transfer_bands
  union all select income_decile, 'p95', p95_millions from fairplan.fig09_mc_net_transfer_bands
)
select b.income_decile,
  'D' || cast(b.income_decile as int) || ' · $' || cast(round(s.median_income/1000.0) as int) || 'k' as decile_label,
  b.stat, b.value
from bands b
join fairplan.n5_scissors_by_decile s on b.income_decile = s.income_decile
order by b.income_decile, case b.stat when 'p5' then 1 when 'median' then 2 else 3 end
```





## Design choices swing tens of millions per decile: a hazard-based surcharge flattens the transfer, a low-income rebate steepens it

 **Baseline** is the actual
2025 mechanism: premium-proportional recoupment and makes the mildly progressive
pattern above. **Hazard-based** allocates the surcharge by wildfire risk instead of premium:
actuarially cleaner (those whose risk is backstopped pay for the backstop), but it largely
erases the income gradient and shifts the burden onto middle deciles that combine modest
incomes with elevated fire risk. **Low-income rebate** keeps premium-based recoupment but
returns part of the surcharge to the bottom deciles: the most progressive design (decile 3
gains $25M; the top decile pays $24M), though it does nothing about the exposure growth that
produced the assessment in the first place. No design dominates on every criterion.

```sql reform_long
select income_decile, 'Baseline' as scenario, baseline_millions as net_millions from fairplan.reform_counterfactuals
union all select income_decile, 'Hazard-based', hazard_based_millions from fairplan.reform_counterfactuals
union all select income_decile, 'Low-income rebate', low_income_rebate_millions from fairplan.reform_counterfactuals
```

<Dropdown name=scenario data={reform_long} value=scenario defaultValue="Baseline"/>

```sql reform_pick
select r.income_decile,
  'D' || cast(r.income_decile as int) || ' · $' || cast(round(s.median_income/1000.0) as int) || 'k' as decile_label,
  r.net_millions,
  case when r.net_millions >= 0 then 'Net recipient' else 'Net payer' end as position
from ${reform_long} r
join fairplan.n5_scissors_by_decile s on r.income_decile = s.income_decile
where r.scenario = '${inputs.scenario.value}'
order by r.income_decile
```

<BarChart data={reform_pick} x=decile_label y=net_millions series=position sort=false
  seriesColors={{'Net recipient':'#4f7a5c','Net payer':'#b5442e'}}
  xAxisTitle="Income decile · median income" yAxisTitle="Net transfer ($M)" title="Net transfer under {inputs.scenario.value}"/>

<Details title="Data notes">

The same incidence engine is re-run under
three surcharge designs: **Baseline** (premium-proportional, the actual 2025 mechanism),
**Hazard-based** (cost allocated by exposure × risk score), and **Low-income rebate** (deciles 1–3
exempt, their share redistributed by premium). These are counterfactual *model* outputs, not observed
outcomes, and no design dominates on every criterion.

</Details>

---

That's the steady-state incidence. Then January 2025 happened → [The $1B bill](/06-the-bill)
