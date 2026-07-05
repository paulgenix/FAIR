---
title: Who pays for the California FAIR Plan?
full_width: true
---

The California FAIR Plan (Fair Access to Insurance Requirements) is the state's insurer of last resort for property owners who can't get coverage in the regular market. It's an insurance pool created by the state of California for property owners who cannot find insurance in the state's price-regulated market, typically because their home sits in a high wildfire-risk area and private carriers have declined or non-renewed them.  
### A few key things to know:
- It isn't a government agency or taxpayer-funded program. Under Insurance Code §10091, it is an association formed by insurers licensed to write basic property insurance in California, a shared pool that all the state's property insurers are required to participate in. The California FAIR Plan Association was formed back in 1968, after brush fires and unrest in the 1960s made insurers unwilling to write policies in certain areas.
- Its coverage is limited. FAIR Plan policies provide basic fire, lightning, and smoke damage coverage, but do not include tree damage, water damage, theft, or liability coverage. Because of these gaps, most homeowners pair it with a separate "Difference in Conditions" (DIC) policy that fills in the missing protections so the combined package behaves more like a standard homeowners policy.
- It's meant to be temporary, a bridge back to the regular market rather than a permanent solution. But that's increasingly not how it functions in practice. As major carriers have pulled back from high-risk areas, the plan has grown dramatically; today it insures more homes than almost any private carrier in California, a sign of how strained the insurance market has become.

## What was designed as a last-resort option has become a major risk holder

```sql kpi
select
  (select exposure_billions from fairplan.growth_by_fiscal_year order by fy desc limit 1) as exposure_b,
  (select policies_in_force from fairplan.growth_by_fiscal_year order by fy desc limit 1) as pif,
  (select round((max(exposure_billions)/min(exposure_billions)-1)*100) from fairplan.growth_by_fiscal_year) as growth_pct,
  (select losses_incurred_millions/1000 from fairplan.fin01_income_loss_trend order by fiscal_year desc limit 1) as losses_b
```

<BigValue data={kpi} value=exposure_b fmt='$0"B"' title="Residential exposure (FY2025)"/>
<BigValue data={kpi} value=pif fmt='#,##0' title="Policies in force"/>
<BigValue data={kpi} value=growth_pct fmt='0"%"' title="Exposure growth since FY2021"/>
<BigValue data={kpi} value=losses_b fmt='$0.0"B"' title="FY2025 losses incurred"/>

<Details title="Data notes">

Source: Exposure and policies in force are FAIR residential (dwelling-fire) figures,
 FY2025 losses incurred are net of reinsurance.

</Details>

The Plan now guarantees $645 billion of housing stock on a balance sheet whose members'
equity, at its peak, amounted to under one percent of that figure. When the January 2025
Palisades and Eaton fires produced $2.0B of retained losses against $0.7B of premium, the
shortfall was socialised, by statute, across every insurer licensed to
write property coverage in California and, through a premium surcharge, across their
policyholders.

## The net transfer flow from wealthier ZIP codes to poorer ones, contrary to the common assumption

```sql net_transfer_teaser
select t.income_decile,
  'D' || cast(t.income_decile as int) || ' · $' || cast(round(s.median_income/1000.0) as int) || 'k' as decile_label,
  t.net_millions,
  case when t.net_millions >= 0 then 'Net recipient' else 'Net payer' end as position
from fairplan.net_transfer_by_decile t
join fairplan.n5_scissors_by_decile s on t.income_decile = s.income_decile
order by t.income_decile
```

<BarChart data={net_transfer_teaser} x=decile_label y=net_millions series=position sort=false
  seriesColors={{'Net recipient':'#4f7a5c','Net payer':'#b5442e'}}
  xAxisTitle="Income decile · median income" yAxisTitle="Net transfer ($M)"
  title="The transfer runs from richer to poorer deciles"/>

<Details title="Data notes">

Net = share of FAIR exposure − surcharge cost
(share of homeowners premium), each scaled by the recoupable half of the $1B assessment.

</Details>


## Overview

1. [The fires: not more common, worse](/01-the-fires)
2. [The insurance retreat](/02-the-retreat)
3. [The FAIR Plan's Growth](/03-fair-balloons)
4. [Who relies on FAIR](/04-who-relies)
5. [Who pays](/05-who-pays)
6. [The $1B bill](/06-the-bill)
7. [Explore the map](/07-map)
8. [Data Sources](/08-methods)
