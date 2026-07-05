---
title: "The fires: not more common, worse"
full_width: true
---

California does not have more wildfires than it used to, fire counts have been flat for
25 years. What changed is the intensity: four of the six worst acreage years on record are
2017 or later, 13 of the 20 largest fires ever came in 2017 or later, and the January 2025 fires
destroyed more insured value than any wildfire in US history. The trend is rising severity, not rising frequency and severity is what destabilizes insurance markets. Insurance absorbs frequent,
independent, modest losses; it struggles with rare, correlated, enormous ones.

## 2025: a record year by destruction

```sql season_kpis
with a as (select * from fairplan.context_acres_burned_by_year),
     s as (select * from fairplan.context_structures_destroyed_by_year)
select
  (select fires from a where year = 2025) as fires_2025,
  (select fires from a where year = 2025) - (select round(avg(fires)) from a where year between 2020 and 2024) as fires_delta,
  (select acres from a where year = 2025) as acres_2025,
  (select acres from a where year = 2025) - (select round(avg(acres)) from a where year between 2020 and 2024) as acres_delta,
  (select structures from s where year = 2025) as structures_2025,
  (select structures from s where year = 2025) - (select round(avg(structures)) from s where year between 2020 and 2024) as structures_delta
```

<BigValue data={season_kpis} value=fires_2025 fmt='#,##0' title="Fires (2025)"
  comparison=fires_delta comparisonFmt='+#,##0;-#,##0' comparisonTitle="vs 5-yr avg" downIsGood=true/>
<BigValue data={season_kpis} value=acres_2025 fmt='#,##0' title="Acres burned (2025)"
  comparison=acres_delta comparisonFmt='+#,##0;-#,##0' comparisonTitle="vs 5-yr avg" downIsGood=true/>
<BigValue data={season_kpis} value=structures_2025 fmt='#,##0' title="Structures destroyed (2025)"
  comparison=structures_delta comparisonFmt='+#,##0;-#,##0' comparisonTitle="vs 5-yr avg" downIsGood=true/>

<Details title="Data notes">

Source:  vs 5-yr avg compares 2025 against the 2020–2024 mean. 2025 structure counts are
preliminary and structures-destroyed definitions vary slightly by year

</Details>

2025 had a below-average number of fires and a below-average acreage year yet still
destroyed 16,512 structures, more than four times the five-year average, because two of
those fires burned into dense Los Angeles neighborhoods.

## The land burned is growing even as fire counts stay steady

<Tabs>

<Tab label="#Fires vs acres (annual)">

**The number of ignitions is stable; the land they burn is not.**

The line (number of fires) barely moves across a quarter-century. The bars (acres burned)
shift after 2016: four of the five seasons from 2017 to 2021 burned more than the
2000–2016 average and 2020 alone burned 4.4 million acres.

```sql fires_vs_acres
select year, fires, acres_millions
from fairplan.context_acres_burned_by_year
order by year
```

<BarChart data={fires_vs_acres} x=year y=acres_millions
  y2=fires y2SeriesType=line
  yAxisTitle="Acres burned (millions)" y2AxisTitle="Number of fires"
  title="Same number of fires, far more land burned (2000–2025)"/>

<Details title="Data notes">

Fire counts (the line) and acreage (the bars,
in millions of acres) are from the same statewide annual series. The gap between ignition count and acreage is real in the data, not just a result of measurement or definition divergences. 

</Details>

> The 2020 season burned 4.4M acres more than six times the 2000–2016 average from a
> fire count less than 20% above the series median.

</Tab>

<Tab label="5-year average trend">

**There is a change happening, not just a few extreme fire seasons.**

Big fire years are noisy, 2020 sits next to a lull year like 2023. But the five-year average shows a real shift after 2017 that hasn't returned to earlier levels. For an insurer pricing on historical data, this is the
situation where backward-looking rates understate forward-looking risk. 

```sql acres_5yr
select year, acres_millions,
  avg(acres_millions) over (order by year rows between 4 preceding and current row) as five_year_avg
from fairplan.context_acres_burned_by_year
order by year
```

<BarChart data={acres_5yr} x=year y=acres_millions
  y2=five_year_avg y2SeriesType=line yMax=4.5 y2Max=4.5
  yAxisTitle="Acres burned (millions)" y2AxisTitle="5-year average"
  title="Annual acreage and the 5-year average"/>

<Details title="Data notes">

Source: The line is a trailing five-year mean (current year plus the
four prior), so the earliest years of the series average over fewer than five points.

</Details>

</Tab>

</Tabs>

## Before 2017, a bad year destroyed ~3,000 structures; since then, four years have topped 10,000

Acres burn in wilderness; structures burn in urban areas, and structures are
what insurers pay for. From 2013 to 2016 the worst year on record destroyed 3,159 structures.
Then 2017 destroyed 10,280, 2018 destroyed 24,226 (the Camp Fire year), 2020 destroyed
11,116, and 2025 destroyed 16,512.

```sql structures_destroyed
select year, structures
from fairplan.context_structures_destroyed_by_year
order by year
```

<BarChart data={structures_destroyed} x=year y=structures
  yAxisTitle="Structures destroyed"
  title="Structure destruction increased after 2017"/>

<Details title="Data notes">

Definitions vary slightly by year (destroyed vs damaged + destroyed); 2016 covers the six major fires only; 2025 is preliminary.
The series starts in 2013 because no consistent statewide series exists before then.

</Details>

## The largest and most notable fires cluster after 2017

<Tabs>

<Tab label="20 largest fires">

**13 of the 20 largest fires in recorded history have burned since 2017.**

A century of fire records, and two-thirds of the very largest events are packed into the last
eight years including the million-acre August Complex (2020) and the near-million-acre
Dixie (2021), each more
than twice the size of anything recorded before 2017. Size alone doesn't destroy homes, but
it marks the same underlying shift: fires that outrun suppression entirely.

```sql largest_fires
select fire, year, acres/1000000.0 as acres_m,
  case when year >= 2017 then '2017 or later' else 'Before 2017' end as era
from fairplan.context_largest_fires
```

```sql largest_labels_right
select fire, year, acres/1000000.0 as acres_m
from fairplan.context_largest_fires
where fire not in ('Creek', 'North Complex', 'Caldor', 'Witch', 'River Complex')
```

```sql largest_labels_left
select fire, year, acres/1000000.0 as acres_m
from fairplan.context_largest_fires
where fire in ('Creek', 'North Complex', 'Caldor', 'Witch', 'River Complex')
```

<BubbleChart data={largest_fires} x=year y=acres_m series=era
  size=acres_m sizeFmt='0.00"M"' opacity=0.75
  tooltipTitle=fire chartAreaHeight=420
  seriesColors={{'2017 or later':'#b5442e','Before 2017':'#a7a099'}}
  yAxisTitle="Acres (millions)"
  title="13 of the 20 largest fires in California history burned since 2017">
  <ReferencePoint data={largest_labels_right} x=year y=acres_m label=fire
    labelPosition=right symbolSize=1 color=#736c63/>
  <ReferencePoint data={largest_labels_left} x=year y=acres_m label=fire
    labelPosition=left symbolSize=1 color=#736c63/>
</BubbleChart>

</Tab>

<Tab label="137 years of notable fires">

**137 years of notable fires: catastrophe used to be episodic, now it clusters**

Every fire on CAL FIRE's largest, deadliest, or most-destructive top-20 lists. Height is size,
bubble is deaths, color is structures destroyed. For a century the catastrophic events are
scattered: one Tunnel Fire (1991), one Cedar Fire (2003) per decade. From 2017 the dark-red
bubbles arrive nearly every year. The right edge of this chart is the reason this dashboard
exists: correlated, repeated tail events are the one thing a thinly capitalised risk pool
cannot survive without outside money.

```sql notable_fires
select fire, year, acres, structures, deaths,
  greatest(deaths, 1) as deaths_size,
  case when structures >= 10000 then '10,000+ structures'
       when structures >= 1000 then '1,000–9,999'
       when structures >= 100 then '100–999'
       else 'Under 100' end as impact
from fairplan.context_notable_fires
```

```sql fire_labels
select fire, year, acres
from fairplan.context_notable_fires
where acres >= 300000 or structures >= 5000 or deaths >= 25
```

<BubbleChart data={notable_fires} x=year y=acres size=deaths_size series=impact
  tooltipTitle=fire
    chartAreaHeight=500
  seriesColors={{'10,000+ structures':'#7c2410','1,000–9,999':'#b5442e','100–999':'#dd7f3a','Under 100':'#e9c46a'}}
  xAxisTitle="Year" yAxisTitle="Acres burned" yFmt='#,##0'
  title="California's largest, deadliest, and most destructive fires">
  <ReferencePoint data={fire_labels} x=year y=acres label=fire
    labelPosition=right symbolSize=1 color=#736c63/>
</BubbleChart>

Labels mark fires that are the largest on at least one dimension (≥300k acres, ≥5,000 structures,
or ≥25 deaths); hover for the rest.

> Bubble size floors at 1 so zero-death fires stay visible. Union of the three CAL FIRE top-20
> lists (42 fires); Thomas Fire deaths include the Montecito debris flow as listed by the source.

</Tab>

</Tabs>

## The worst fires ranked two ways: most destructive by structures, most costly by dollars

**Seven of the ten most destructive fires have come since 2015** and January 2025 recorded
the two costliest wildfires in US history. Palisades ($23.0B) and Eaton ($17.5B) each
individually exceeded Camp (2018), the previous US record, and together drove roughly $40B of
insured losses. About $4 billion of those claims landed on the FAIR Plan.

```sql destructive
select fire || ' (' || cast(year as int) || ')' as fire_label, structures_destroyed, era
from fairplan.context_destructive_fires
order by structures_destroyed desc
```

```sql costliest
select fire || ' (' || cast(year as int) || ')' as fire_label,
  insured_loss_millions_2025usd/1000.0 as loss_b,
  case when year = 2025 then '2025 fires' else 'Earlier (2025$)' end as vintage
from fairplan.context_costliest_fires
order by loss_b desc
```

<Grid cols=2>

<BarChart data={destructive} x=fire_label y=structures_destroyed series=era swapXY=true
  seriesColors={{'2015 or later':'#b5442e','Before 2015':'#a7a099'}}
  yAxisTitle="Structures destroyed"
  title="7 of the 10 most destructive fires: 2015 or later"/>

<BarChart data={costliest} x=fire_label y=loss_b series=vintage swapXY=true
  seriesColors={{'2025 fires':'#b5442e','Earlier (2025$)':'#a7a099'}}
  yAxisTitle="Insured loss ($B, 2025 dollars)"
  title="January 2025: the two costliest wildfires in US history"/>

</Grid>

> Camp (2018) is California's most destructive fire on record: 18,804 structures, roughly double
> the next-worst (Eaton, 2025). Only three of the ten are before 2015: Tunnel/Oakland (1991),
> Cedar (2003), and Witch (2007).


<Details title="Data notes">

Pre-2025 insured losses are
inflation-adjusted to 2025 dollars.
Losses are insured (not total economic) losses.

</Details>

---

## The fire season is spreading: the dark band of August–October is now going into mid-summer and winter

Each cell is the acreage of fires starting in that month. The dark band once sat in
August–October; it now goes into mid-summer and January 2025 shows a winter cell that
would have been unthinkable a decade ago. A longer season means less reinsurance-friendly
seasonality and fewer safe months for a concentrated book.

```sql monthly_heat
select year, month, month_name, acres
from fairplan.context_incident_acres_monthly
where year >= 2016
```

<Heatmap data={monthly_heat} x=month_name y=year value=acres
  xSort=month ySort=year
  colorScale={['#f6ead6','#a1341e']}
  title="Acres burned by start month × year"/>

<Details title="Data notes">

Each cell is the acreage of fires starting
in that month, not acres actively burning during it a large fire is booked entirely to its ignition
month. 
</Details>

# The season right now

```sql now_kpis
select
  (select acres_to_date from fairplan.context_season_to_date where year = 2026) as acres_2026,
  (select count(*) from fairplan.context_current_incidents where active = 'Active') as active_now,
  (select max(cutoff) from fairplan.context_season_to_date) as cutoff,
  cast(
    cast(cast((select max(year) from fairplan.context_season_to_date) as integer) as varchar)
    || '-' || (select max(cutoff) from fairplan.context_season_to_date)
  as date) as refreshed
```

Live data from CAL FIRE's incident feed. Data as of <Value data={now_kpis} column=refreshed fmt='mmmm d, yyyy'/>

<BigValue data={now_kpis} value=acres_2026 fmt='#,##0' title="Acres burned, 2026 season to date"/>
<BigValue data={now_kpis} value=active_now fmt='#,##0' title="Incidents active right now"/>

<Details title="Data notes">

Covers CAL FIRE tracked incidents only, so it is not the full
statewide count. 2026 figures are season-to-date through the cutoff, not a full-year total.

</Details>

## How this season compares, same date each year

```sql season_to_date
select year, acres_to_date,
  case when year = 2026 then 'This season' else 'Prior years' end as highlight
from fairplan.context_season_to_date
order by year
```

<BarChart data={season_to_date} x=year y=acres_to_date series=highlight
  seriesColors={{'This season':'#b5442e','Prior years':'#a7a099'}}
  yAxisTitle="Acres burned by this date"
  title="Season-to-date acres: 2026 vs the same date in prior years"/>

> CAL FIRE-tracked incidents only. Acres are representative (big fires are always tracked),
> incident counts are not comparable across years.

<Details title="Data notes">

Every year is measured to the same calendar cutoff (month-day). Acreage is comparable across years because large fires are always
tracked; incident counts are not.

</Details>



## The largest incidents of 2026

```sql current_incidents
select fire, county, acres, containment_pct, started, active
from fairplan.context_current_incidents
order by acres desc
```

<DataTable data={current_incidents} rows=15 search=true>
  <Column id=fire/>
  <Column id=county/>
  <Column id=acres fmt='#,##0' contentType=colorscale/>
  <Column id=containment_pct fmt='0"%"'/>
  <Column id=started/>
  <Column id=active/>
</DataTable>

<Details title="Data notes">

Acres and containment are CAL FIRE's reported
figures at capture time, not final post-season values.

</Details>

---

Insurers watched these same loss numbers accumulate — and where regulated rates couldn't
follow the risk, they responded the only way left: by leaving. That response is the next
page → [The insurance retreat](/02-the-retreat)
