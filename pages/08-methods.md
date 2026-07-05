---
title: Data Sources
full_width: true
---


<Tabs>

<Tab label="Data dictionary">

```sql dictionary
select * exclude (notes) from fairplan.data_dictionary
```

<DataTable data={dictionary} search=true/>



</Tab>

<Tab label="Data sources">

```sql sources_raw
select source, provider, format, coverage, files
from fairplan.sources_raw
```

<DataTable data={sources_raw} search=true rows=all>
  <Column id=source wrap=true/>
  <Column id=provider/>
  <Column id=format/>
  <Column id=coverage/>
  <Column id=files title="Raw files"/>
</DataTable>






</Tab>

</Tabs>

## Notes

- **Two scope conditions bound every claim here.** *Incidence, not welfare:* deciles are of
  ZIPs, not households, so a low-income household in an affluent ZIP is averaged into that ZIP's
  decile. *Pass-through is modelled:* the funding-side results assume a premium-proportional
  pass-through
- Insurer-group assessment figures are implied from participation shares and illustrate the
  allocation logic rather than approved amounts (State Farm's implied ~$182M vs its stated
  ~$165M+); recoupment for the 2025 call is capped at 50%, and filed surcharges run below the cap.
- The insurer-vs-consumer non-renewal split ends in 2021 (CDI stopped reporting it); total
  non-renewal rates continue but mix retreat with ordinary churn. Post-fire SB 824 moratoria
  freeze non-renewals in affected ZIPs for a year dips in the series are legal, not recovery.
- Fire-history context: fire *counts* are flat since 2000; the trend claims concern acreage,
  structure destruction, and insured losses. Structures-destroyed definitions vary slightly by
  year; 2016 covers the six major fires only; 2025 figures are preliminary.
- Costliest-fires table: pre-2025 losses inflation-adjusted to 2025$ by Aon; 2025 fires in
  current dollars. Maui (HI) is included in the source table for US-wide context but filtered
  out of the California chart.
- The "season right now" tables come from CAL FIRE's live incident feed and are a snapshot
  (refreshed when I rebuild the dashboard). The data covers CAL FIRE-tracked incidents
  only, incident counts are not comparable across
  years; acreage comparisons are.


### This data project was built alongside my Bachelor's thesis: Who Pays for the Fair Plan? A Distributional Incidence Analysis of the Insurer of Last Resort and its 2025 One-Billion-Dollar Assessment   
### Contact: paul.genix@gmail.com