---
title: The billion-dollar bill
full_width: true
---

FY2025 is what an extreme event looks like on a residual insurer's books: $2.02B of losses
against $0.70B of premium, a combined ratio of 333.5%, a net loss of $1.58B, and
members' equity swinging from +$374M to −$352M in a single year. For the first time in 30 years the Plan assessed its member insurers $1 billion allocated by market share, with
half recoupable from policyholders statewide as a percentage of premium.

## FY2025 in three views: losses overran premiums, the combined ratio surged, and profit turned to a $1.58B loss

<Tabs>

<Tab label="Premiums vs losses">

**FY2025 losses ran nearly three times premiums earned.**

Through the growth years the Plan looked profitable: losses of $54M, $90M and
$212M against rising premiums. In FY2025, losses incurred ($2,025M) overwhelmed premiums earned ($698M) almost three times
over.

```sql loss_trend
select fiscal_year, net_premiums_earned_millions, losses_incurred_millions
from fairplan.fin01_income_loss_trend
order by fiscal_year
```

<BarChart data={loss_trend} x=fiscal_year sort=false
  y={['net_premiums_earned_millions','losses_incurred_millions']} type=grouped
  yAxisTitle="$ millions" title="Losses overtook premiums in FY2025"/>

<Details title="Data notes">

From the FAIR Plan's audited Statement of Operations (fiscal years
ending September 30). Both series are accrual figures, net premiums earned and losses
incurred, net of reinsurance not cash collected or paid. FY2025 losses incurred ($2,025M) are the
retained loss after reinsurance recoveries, not the ~$3.5B gross or ~$4B ultimate claim value.

</Details>

</Tab>

<Tab label="Combined ratio">

**The combined ratio detonated: 41% → 49% → 64% → 334%.**

The combined ratio: losses plus expenses as a share of premium. Below 100%, every premium dollar covers claims and costs with room to
spare; the Plan sat at 40.8% in FY2022 and only 64.2% by FY2024. In FY2025 the loss-and-LAE
ratio alone went from 37.7% to 300.5%: the Plan paid out more than three dollars for
every premium dollar earned.
```sql ratios
select fiscal_year, Metric, value_pct from fairplan.fin02_key_ratios order by fiscal_year
```

<LineChart data={ratios} x=fiscal_year y=value_pct series=Metric sort=false
  yFmt='#,##0"%"' yAxisTitle="%" title="Combined ratio blew past 100% in FY2025"/>

<Details title="Data notes">

The combined ratio is losses + loss-adjustment + other expenses as a share
of earned premium; above 100% the Plan pays out more than it earns. These are statutory accrual ratios
from the audited financials the FY2025 loss-and-LAE ratio alone reached 300.5%.

</Details>

</Tab>

<Tab label="Net income">

**One season erased three years of profit: net income swung from +$284M to −$1.58B.**

The Plan earned $214M, $245M and $284M in FY2022–24, rebuilding its capital base from a
members' deficit to +$374M of equity. FY2025 consumed all of it AND the billion-dollar
assessment on top leaving members' equity at −$352M. The Plan enters the next fire
season undercapitalised, which is why the thesis reads the 2025 assessment as *structural*
rather than exceptional.

```sql net_income
select fiscal_year, net_income_millions, underwriting_income_millions
from fairplan.fin01_income_loss_trend
order by fiscal_year
```

<LineChart data={net_income} x=fiscal_year sort=false
  y={['net_income_millions','underwriting_income_millions']}
  yAxisTitle="$ millions" title="Net and underwriting income swing negative"/>

<Details title="Data notes">

 **Underwriting income** is premiums minus losses and expenses only;
**net income** additionally includes investment gains and the assessment's income effect. Both are
accrual figures; the FY2025 net loss of −$1.58B is struck *after* the $1B assessment is recognised.

</Details>

</Tab>

</Tabs>

## The cash flow that forced the call: only a $1B assessment balances FY2025

Premiums covered only about a third of the losses; investment income and a
draw-down of the surplus built in the good years closed part of the gap; the $1B member
assessment is what balances the diagram. 

```sql sankey
select source, target, value from fairplan.fin03_sankey_2025
```

<SankeyDiagram data={sankey} sourceCol=source targetCol=target valueCol=value
  title="FAIR Plan FY2025 cash flow"/>

<Details title="Data notes">

These are accrual income-statement figures, not cash despite the
"cash flow" title, every line item is a statutory operations number that ties to the FAIR Plan's audited
Statement of Operations for the year ended September 30, 2025. On the uses side, losses incurred
($2,024.8M), loss-adjustment expense ($73.8M) and other underwriting expense ($230.2M) sum to
$2,328.8M of underwriting deductions. On the sources side, net premiums earned ($698.4M), net
investment gain ($47.1M) and the $1,000M assessment total $1,745.5M so the "drawdown of surplus"
line (~$581M) is a derived balancing figure, exactly the FY2025 net loss ($1,581.0M) minus the $1B
assessment

The $1B assessment was recorded directly in members'
equity when billed rather than flowing through operations, so pooling it with earned premium and
losses is a simplification for a "where did the money come from" view, not a literal cash
movement. A small $2.3M "other income" inflow is left out, which is why the drawdown lands at $581M rather
than $583M. A true cash-basis version would
shrink the drawdown from ~$581M to $257M.

</Details>

## Three decades of paying money out, then a $1,000M call: the first in 30 years

For most of its history the Plan distributed surplus back to its members: the small green
bars. The last assessments before 2025 were roughly $260M spread across 1993–95, after the
Altadena/Malibu fires and the Northridge earthquake. The 2025 call is not just the first in
thirty years; at $1B it is roughly four times the entire 1990s episode, in a single year.

```sql assessment_history
select fiscal_year,
  distribution_millions + assessment_millions as signed_millions,
  case when distribution_millions + assessment_millions >= 0 then 'Distribution' else 'Assessment' end as flow
from fairplan.n13_assessment_distribution_history
order by fiscal_year
```

<BarChart data={assessment_history} x=fiscal_year y=signed_millions series=flow xFmt='0'
  seriesColors={{'Distribution':'#4f7a5c','Assessment':'#b5442e'}}
  yAxisTitle="$ millions" title="Distributions (+) and assessments (−)"/>

<Details title="Data notes">

The plotted value is `distribution + assessment`, so
green bars are net surplus distributions to members (positive) and red bars are assessments
(negative). The pre-2025 assessments: roughly $260M spread across 1993–95 follow the
Altadena/Malibu fires and the Northridge earthquake; the $1B 2025 call is the first since.

</Details>

## Who pays the $1B, by insurer group — and how participation shares are already migrating

<Tabs>

<Tab label="Who pays the $1B (by group)">

**The allocation key is market share, not wildfire exposure so the biggest homeowners writers get the biggest bills.**

Because the assessment is
apportioned by each insurer's share of the property market not by how fire-exposed its book
is, and emphatically not by its FAIR policyholders the largest writers of *ordinary*
homeowners business carry the largest bills, and may pass half of those bills to customer
bases that mostly live nowhere near fire country. That is how the
assessment socialises catastrophe risk: it converts a concentrated loss in a few foothill
canyons into a small charge on everyone in the state.  
 The figures are implied from
participation shares illustrative of the allocation logic, not the approved amounts (State
Farm's implied ~$182M compares with its own stated figure of just over $165M).

```sql assessment_group
select insurer_group, implied_2025_assessment_millions
from fairplan.n15_assessment_by_insurer_group
order by implied_2025_assessment_millions desc limit 12
```

<BarChart data={assessment_group} x=insurer_group y=implied_2025_assessment_millions
  swapXY=true yAxisTitle="Implied assessment ($M)"
  title="Who pays the $1B, by insurer group"/>

<Details title="Data notes">

Figures are implied from each group's
share of the California homeowners market (NAIC line 04, base year 2023), illustrating the allocation
logic *not* the approved per-insurer amounts (State Farm's implied ~$182M vs its stated ~$165M+).
The key is market share, not FAIR exposure or wildfire risk, which is the point the chart makes.

</Details>

</Tab>

<Tab label="Participation shares migrating (2024→2026)">

**Participation shares are already migrating as carriers rebalance their books.**

An insurer's share of the next assessment tracks its share of the market — so as carriers
shrink or grow their California books, tomorrow's bill shifts with them. Watching the top-6
groups' participation shares move from 2024 to 2026 is watching the industry reprice its
appetite for the state in real time.

```sql part_shift
with top6 as (
  select * from fairplan.n15_assessment_by_insurer_group
  order by pct_2025 desc limit 6
)
select insurer_group, '2024' as fy, dwelling_2024*100 as participation_pct from top6
union all select insurer_group, '2025', dwelling_2025*100 from top6
union all select insurer_group, '2026', dwelling_2026*100 from top6
```

<BarChart data={part_shift} x=insurer_group y=participation_pct series=fy type=grouped
  seriesOrder={['2024','2025','2026']}
  swapXY=true yFmt='#,##0.0"%"' yAxisTitle="Participation share (%)"
  title="Top-6 groups' participation shares, 2024 → 2026"/>

<Details title="Data notes">

Source: `n15_assessment_by_insurer_group`, top-6 groups by 2025 share. Each bar is a group's dwelling
participation share for 2024, 2025 and 2026 — the share that would set its slice of a future
assessment. Shares move as carriers grow or shrink their California books; the statutory assessment
base year is the second preceding calendar year (2023 for the 2025 call).

</Details>

</Tab>

</Tabs>

## Find your insurer — and what its share of the bill implies for your premium

Because the 2025 call totalled exactly $1B, recoupment is capped at **50%** of each insurer's
assessment; the surcharges actually filed run well below even that (State Farm, the largest
writer, filed 1.13% of premium over two renewal periods). Your expected surcharge depends
not on where you live or your own wildfire risk, but on which insurer you buy from and how
much premium you pay.

```sql participation_top
select company, "group" as insurer_group, pct_2025, implied_2025_assessment_millions
from fairplan.n14_top_member_participation order by pct_2025 desc
```

<DataTable data={participation_top} rows=20 search=true>
  <Column id=company/>
  <Column id=insurer_group/>
  <Column id=pct_2025 fmt='0.00"%"'/>
  <Column id=implied_2025_assessment_millions fmt='$0.0"M"' contentType=colorscale/>
</DataTable>

<Details title="Data notes">

Recoupment
from policyholders is capped at 50% of the assessment and filed surcharges run well below the cap
(State Farm filed 1.13% of premium over two renewal periods)
</Details>

<Details title="Full member list (334 companies)">

```sql participation_all
select * from fairplan.n16_participation_by_company
```

<DataTable data={participation_all} search=true/>

</Details>

---

[Explore the map](/07-map)
