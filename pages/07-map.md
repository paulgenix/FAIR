---
title: Explore the map
full_width: true
---

<script>
  // Prepend the deployment base path (e.g. /FAIR) so static assets in
  // static/ resolve under GitHub Pages. Empty string when no base path.
  import { base } from '$app/paths';
</script>

FAIR
penetration averages a single-digit share of homes; in the foothill interior it becomes the
primary market (Tuolumne reaches ~50% at the county level, and the worst ZIPs exceed 70%).

## Reliance is concentrated, not scattered

Reliance clusters in connected high-hazard regions rather than scattering at random (Moran's
I = 0.645). That proximity is what defeats the diversification ordinary insurance depends
on: a single wind-driven event can ignite a large fraction of the Plan's book at once, which
is exactly how January 2025 turned a few burning canyons into a bill for every policyholder
in the state.  
Hover any county for its full profile:penetration, FAIR exposure, wildfire
risk, median income, and non-renewal rate.

```sql county_map
select lpad(cast(m.fips as varchar), 5, '0') as fips, m.county,
  m.fair_penetration, m.fair_exposure_millions, m.median_income, m.nonrenew_rate,
  n.wildfire_risk_score
from fairplan.map_county m
left join fairplan.n9_county_fair_vs_income n on m.county = n.county
```

<AreaMap data={county_map} geoJsonUrl={`${base}/ca_counties.geojson`} geoId=fips
  areaCol=fips value=fair_penetration valueFmt='0.0%'
    height=600
  tooltip={[
    {id: 'county', showColumnName: false, valueClass: 'font-bold text-sm'},
    {id: 'fair_penetration', title: 'FAIR penetration', fmt: '0.0%'},
    {id: 'fair_exposure_millions', title: 'FAIR exposure', fmt: '$#,##0"M"'},
    {id: 'wildfire_risk_score', title: 'Wildfire risk', fmt: '0"th pctl"'},
    {id: 'median_income', title: 'Median income', fmt: '$#,##0'},
    {id: 'nonrenew_rate', title: 'Non-renewal rate', fmt: '0.0%'}
  ]}
  title="FAIR penetration by county"/>



Reading suggestions: compare the dark interior band against the county non-renewal ranking on
[the retreat page](/02-the-retreat): they are the same counties. Then compare median incomes
in the tooltip against the darkest penetration values: the counties leaning hardest on the
Plan are, almost without exception, below the state's median income.

---

How the numbers were built, and what they can and cannot support →
[Methods, data & caveats](/08-methods)
