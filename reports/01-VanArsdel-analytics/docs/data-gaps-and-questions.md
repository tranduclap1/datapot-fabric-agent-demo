# Data gaps & stakeholder questions — VanArsdel Sales Analytics

Eleven gaps surfaced when the pasted `dataset/raw/` files were profiled (see [data-profile.md](data-profile.md))
against the report's analytical intent. Four are **High** priority because they shape model boundaries
(date coverage, the actual-vs-budget window, the unit grain, and what "Revenue" actually means); the rest
are Medium/Low scoping or data-quality items that we can mitigate in Power Query or defer. Every finding
below is reconciled to the profiled numbers; nothing here contradicts the ground truth. None of these
gaps blocks the build — the model ships with documented assumptions — but each carries a question that the
business should answer before the numbers are treated as decision-grade.

## Prioritized summary

| ID | Gap | Priority |
|----|-----|----------|
| G1 | Supplied `Date` sheet (2016–2021) misses 2015 actuals (16.61% of rows) and over-extends to 2021 with no source data | **H** |
| G2 | Actuals end 2020-06-30; Budget/Forecast span 2020–2021 → true variance only Jan–Jun 2020 | **H** |
| G3 | `Sales[Units]` is always 1 — count grain, no true order quantity | **H** |
| G4 | No money column in `Sales`; Revenue/Cost derived from list price/cost (no discounts/returns/tax) | **H** |
| G5 | No competitor data — `Manufacturer` is VanArsdel-only → no market share | M |
| G6 | Budget grain (9 Category-Segment combos) vs products (10 combos): Rural-Productivity has no budget | M |
| G7 | Data-quality defects: `Affliliate`, `Deskop`, trailing spaces, composite `Email Name` (PII) | M |
| G8 | Thin customer dimension — only ZIP + email/name, no demographics or tenure | M |
| G9 | Two Budget years + one Forecast year — which is the official plan, and how do they relate for 2021? | L |
| G10 | Geo is ZIP-level with composite `City` and no lat/long | L |
| G11 | No order/basket grain — no OrderID or time-of-day | L |

---

## High priority

### G1 — Date coverage: supplied `Date` sheet misses 2015 and over-extends to 2021

**Finding.** `Sales[Date]` spans **2015-01-01 → 2020-06-30** (2,002 distinct dates over 675,368 rows),
but the supplied `Date` dimension is daily **2016-01-01 → 2021-12-31** (2,192 rows). The sheet therefore
**excludes all of 2015**, which is **112,202 sales rows = 16.61%** of the fact, while simultaneously
extending two-and-a-half years (2020 H2 + all of 2021) beyond the last actual.

**Impact on analysis.** Joining `Sales` to the supplied `Date` would silently drop 1 in 6 actual rows
(2015 falls out of the filter context), understating `[Revenue]`, `[Units Sold]`, and every time-intelligence
measure for the earliest year — and making `[Revenue YoY %]` for 2016 meaningless (no valid prior year).
The over-extension to 2021 is benign for a calendar but means most of the table has no actuals.

**Question for stakeholders.** Is 2015 in scope for actuals reporting? Why does the supplied `Date` sheet
start in 2016 when sales begin in January 2015 — is 2015 considered a partial/ramp year to be excluded,
or an oversight in the date table?

**Proposed resolution.** Per repo convention, **replace** the supplied sheet with a calculated
`CALENDAR(DATE(2015,1,1), DATE(2021,12,31))` marked as the model's date table. This covers every actual
date and the full Budget/Forecast horizon, opens without source data, and removes the silent-drop risk.
2015 is then included by default; if the business excludes it, do so explicitly with a `Date[Year]` slicer,
not by an incomplete date table.

### G2 — Actuals-vs-Budget overlap is one half-year

**Finding.** Actuals end **2020-06-30**. The Budget/Forecast plan covers **Budget 2020 (12 mo) + Budget 2021
(12 mo) + Forecast 2021 (12 mo)** — i.e. plan starts where two-thirds of it has no actual to compare against.
The only window where a true actual **and** a budget both exist is **January–June 2020**. Plan totals for
context: Budget 2020 ≈ **$11.17M**, Budget 2021 ≈ **$10.99M**, Forecast 2021 ≈ **$10.99M**.

**Impact on analysis.** `[Revenue vs Budget]`, `[Revenue vs Budget %]`, and `[Budget Attainment %]` are only
defensible for **Jan–Jun 2020**. Computed over a full year, 2020 attainment would read ~50% purely because
actuals stop mid-year — an artifact, not under-performance. For 2020 H2 and all of 2021, "Actual" is `$0`
(no rows), so any variance card is misleading. This is why `Budget Attainment %` is a **context-only** KPI.

**Question for stakeholders.** Will 2020 H2 and/or 2021 actuals be supplied (refreshed monthly into the
`Sales` sheet)? What comparison period does the business actually intend for budget-vs-actual — the
H1-2020 overlap only, or a planned full-year once data lands?

**Proposed resolution.** Build **page P5 (Budget vs Actual)** with a **prominent caveat printed on the page**
and in the spec: *"Actuals end 2020-06-30; true Actual-vs-Budget variance exists only for Jan–Jun 2020;
2020 H2 and 2021 are plan-only."* Default P5 slicers to Year = 2020 so the comparison lands on the valid
overlap. The contract already documents the append path (add rows to `Sales` to extend actuals), so the
page becomes fully meaningful as soon as later months arrive — no model change needed.

### G3 — `Sales[Units]` is always 1 (count grain, not quantity)

**Finding.** `Sales[Units]` has **distinct count = 1** across all 675,368 rows: every row carries the value
`1`. The fact grain is therefore **one row per unit sold** (Product × Date × Customer × Campaign), and
`[Units Sold] = SUM(Sales[Units])` equals the row count. There is no column expressing a multi-unit order
quantity.

**Impact on analysis.** "Units Sold" is reliable as a volume count, but the data cannot distinguish
"282,596 customers each bought a few units" from any larger basket — every purchase event is a single unit.
Average-basket, units-per-order, and any quantity-weighted analysis are out of scope by construction. ASP
(`[Average Selling Price] = DIVIDE([Revenue],[Units Sold])`) is still valid because both numerator and
denominator are per-unit.

**Question for stakeholders.** Is the one-row-per-unit grain correct and intended? Is true order quantity
(e.g. 3 of SKU X on one order) available in a source upstream of this sample, or is single-unit the actual
selling model?

**Proposed resolution.** Keep `[Units Sold] = SUM(Sales[Units])` and document the grain in the
[data dictionary](../dictionary/data-dictionary.md) and model design as *units sold = row count*. Hide
`Sales[Units]` (all fact columns are hidden; analysis is via measures only). If a real quantity column
later appears, the measure already generalizes — `SUM` over a non-constant `Units` needs no rewrite.

### G4 — No money column in `Sales`; Revenue/Cost are list-price derivations

**Finding.** The `Sales` fact has **no amount/price column** — its only columns are `ProductID`, `Date`,
`CustomerID`, `CampaignID`, `Units`. Revenue and Cost are **derived** from the `Product` dimension's list
values: `[Revenue] = SUMX(Sales, Sales[Units] * RELATED(Product[Unit Price]))` and
`[Cost] = SUMX(Sales, Sales[Units] * RELATED(Product[Unit Cost]))`. Over the full Sales period this yields
Revenue = **$65,547,113**, Cost = **$47,849,393**, Gross Margin = **$17,697,721 (≈ 27.0%)**. `Unit Price`
ranges 20.97–204.74 and `Unit Cost` 15.31–149.46, so margin is embedded in every product at list.

**Impact on analysis.** Every monetary figure is **gross, list-price revenue** — there are **no discounts,
promotions, returns, tax, or net/realized price** anywhere in the data. Reported Revenue and Gross Margin
are therefore an upper bound on realized economics; campaign "performance" measured this way reflects
gross sales attributed to a channel, not net contribution. The headline 27.0% margin is a list-price margin.

**Question for stakeholders.** Is realized/net revenue (after discounts and returns), or a discount/returns
feed, relevant to the decisions this report supports — and is it available? If so, list-price measures should
be relabeled and a net-revenue measure added; if not, "Revenue" must be understood as gross list-price sales.

**Proposed resolution.** Define `[Revenue]` / `[Cost]` / `[Gross Margin]` explicitly as **list-price /
gross-sales** in the [business glossary](business-glossary.md) and dictionary, with the
no-discount/return/tax caveat stated next to the KPI definitions. Keep the SUMX-over-`RELATED` pattern.
Should a net feed arrive, add it as a parallel fact column or table and expose a separate `Net Revenue`
measure rather than overloading `[Revenue]`.

---

## Medium priority

### G5 — No competitor data → no market share

**Finding.** `Product[Manufacturer]` (and `ManufacturerID`) are **constant**: `VanArsdel` / `7` on all 212
products. The data contains only VanArsdel's own sales — there are no competitor units, prices, or revenue.

**Impact on analysis.** Market share, share-of-shelf, win/loss, and competitive price positioning **cannot
be computed**. All "share" framing in the report is internal share (e.g. category mix of *VanArsdel's own*
revenue, digital share of *VanArsdel's own* revenue), never share of a total market.

**Question for stakeholders.** Is competitive analysis in scope for this report? If yes, it requires an
external competitor-sales or market-size dataset that this sample does not contain — should that be sourced?

**Proposed resolution.** Keep `Manufacturer` as a hidden constant attribute and scope the report to
**internal performance** (revenue, margin, volume, mix, budget). Label mix/share visuals as internal share.
Defer any market-share page until a competitor/market dataset is contracted.

### G6 — Budget grain mismatch: Rural-Productivity has no budget

**Finding.** Budget is planned at **9 Category-Segment combos**: Accessory-Accessory, Mix-All Season,
Mix-Productivity, Rural-Select, Urban-Convenience, Urban-Extreme, Urban-Moderation, Urban-Regular,
Youth-Youth. Products, however, span **10 combos** — the extra one, **Rural-Productivity (2 products)**,
has **no budget column** at all.

**Impact on analysis.** Any actuals attributed to **Rural-Productivity** have **no budget to compare to**:
on page P5, that combo will show Revenue with a blank/zero Budget, so `[Budget Attainment %]` and
`[Revenue vs Budget %]` (`DIVIDE` by Budget) return blank for it. Conversely, totaling Attainment across
all combos will dilute the denominator unless the unbudgeted combo is handled deliberately.

**Question for stakeholders.** How should **Rural-Productivity** be treated in budget-vs-actual — excluded
from attainment, rolled up to a parent (e.g. all Rural), or assigned a $0 plan that flags it as unplanned
sales? Was its omission from the budget intentional?

**Proposed resolution.** Introduce a **conformed `Product Group` dimension** keyed on `Category-Segment`,
built from the **DISTINCT union** of the Product and Budget combos, so one Category/Segment slicer filters
**both** actuals (via `Product`) and `Budget`. Rural-Productivity then appears in the dimension; on P5 it
reads as Revenue with no Budget — visually flagging "unplanned" sales rather than dropping it. Confirm the
intended rollup with the business and document it in the model design.

### G7 — Data-quality defects in Campaign and Customer

**Finding.** Profiling found, verbatim: `TrafficChannel` carries **trailing spaces on 6 of 8 values** and the
misspelling **`Affliliate`** (values: Organic Search, SEO, Banner, Affliliate[sic], SEM, Email, SMO, Mail);
`Device` carries the typo **`Deskop`** plus a `Paper` value (values: Mobile, Tablet, Desktop, Deskop[sic],
Paper); and `Customer[Email Name]` is a **composite** `"(first.last@xyza.com): Last, First "` with **trailing
whitespace on every row** and embedded synthetic PII (`@xyza.com`).

**Impact on analysis.** Untreated, `Affliliate` vs `Affiliate` and `Deskop` vs `Desktop` would **split one
real channel/device into two** in every grouping, slicer, and matrix — corrupting `[Revenue]` by Traffic
Channel/Device and the `[Digital Revenue]` classification (`Campaign[Device] IN {"Desktop","Mobile","Tablet"}`
would miss every `Deskop` row). Trailing spaces cause duplicate-looking slicer entries; the composite
`Email Name` is unusable as a clean customer name and exposes PII.

**Question for stakeholders.** Confirm the **canonical spellings**: `Affliliate` → **Affiliate**,
`Deskop` → **Desktop**. `Channel Type` is derived from **`Device` only** — confirm the `Device` value
**`Paper`** classifies as **Offline** under that Device-only rule (Desktop/Mobile/Tablet → Digital,
else Offline). Separately (and **not** coupled to the Device-derived `Channel Type`): is a
**TrafficChannel-level** online/offline grouping useful — e.g. treating `Mail`/`Email` as offline/online
acquisition channels — and if so, what is the intended mapping? And is the synthetic email PII acceptable
to retain (split out) or should it be dropped?

**Proposed resolution.** Clean in **Power Query**: trim all text; fix `Affliliate` → `Affiliate` and
`Deskop` → `Desktop`; derive `Channel Type` = "Digital" if `Device` ∈ {Desktop, Mobile, Tablet} else
"Offline" (so `Paper` falls to Offline); split `Email Name` into `Email` (hidden) + `Customer Name`
("Last, First", trimmed). Record the fixes in the dictionary so refreshes stay consistent.

### G8 — Thin customer dimension

**Finding.** `Customer` has **282,597 rows** but only **three columns**: `CustomerID`, `ZipCode`, and the
composite `Email Name`. There are **no demographics** (age, gender), **no behavioral segment, tenure, or
acquisition date**. (For context, 282,596 distinct customers actually appear in `Sales`.)

**Impact on analysis.** Customer cuts are limited to **geography (via ZIP → Geo)** and identity. We can report
`[Distinct Customers]` and `[Revenue per Customer] = DIVIDE([Revenue],[Distinct Customers])`, but **cannot**
build demographic segmentation, cohort/tenure analysis, retention, or true CLV — there is no attribute to
segment on beyond location.

**Question for stakeholders.** Is a **richer customer master** (demographics, segment, signup/acquisition
date, loyalty tier) available to join on `CustomerID`? Segmentation and CLV are the most-requested customer
analyses and are blocked without it.

**Proposed resolution.** Ship the customer view scoped to **count + value + geography** for now
(`[Distinct Customers]`, `[Revenue per Customer]`, Region/State/District/City via the Geo hierarchy). Keep
`CustomerID` as the stable join key so a future customer-master table can extend the dimension without
remodeling. Defer segmentation/CLV pages until richer attributes are contracted.

---

## Low priority

### G9 — Two Budget years plus one Forecast year

**Finding.** The plan contains **Budget 2020, Budget 2021, and Forecast 2021** — so **2021 is covered by both
a Budget and a Forecast** (each ≈ **$10.99M**), while 2020 has a Budget only (≈ **$11.17M**). The model
separates them with `[Budget] = CALCULATE(SUM(Budget[Amount]), Budget[Type]="Budget")` and
`[Forecast] = CALCULATE(SUM(Budget[Amount]), Budget[Type]="Forecast")`.

**Impact on analysis.** For 2021 the report can show Budget, Forecast, or both. Without guidance it's unclear
which is the **official plan of record** for 2021, or whether Forecast is a mid-year reforecast that should
supersede Budget. The two 2021 numbers being near-identical (~$10.99M) suggests little reforecasting, but
that should be confirmed rather than assumed.

**Question for stakeholders.** Which series is the **official plan per year** (Budget vs Forecast)? How do
Budget and Forecast relate for 2021 — is Forecast a later revision of the 2021 Budget, and which should
attainment be measured against once 2021 actuals exist?

**Proposed resolution.** Both `[Budget]` and `[Forecast]` measures exist. P5 is page-filtered to the
**Jan–Jun 2020** actuals∩budget overlap (where only Budget applies), so its month column plots **Revenue
vs Budget**; widening or clearing the "Comparison period" filter into 2021 surfaces **Forecast** for that
plan-only horizon. `[Budget Attainment %]` is anchored to **Budget** as the default baseline; revisit once
the business names the plan of record.

### G10 — Geo granularity: ZIP-level, composite City, no coordinates

**Finding.** `Geo` is **ZIP-level** (39,948 ZIPs; 29,190 of them actually used by customers), with `City`
stored as a **composite `"City, ST, USA"`** (28,575 distinct), `State` (49), `Region` (3 regions; ZIP
counts: East 18,929 / Central 14,512 / West 6,507), `District` (39), `Country` (USA only). There are
**no latitude/longitude** columns.

**Impact on analysis.** Maps must rely on Power BI's **built-in geocoding** of State/Region (and ZIP), which
is adequate for the planned filled/State maps but imprecise for ZIP-point plotting and can mis-resolve the
composite `City` string. There is no way to plot exact customer coordinates or compute distance.

**Question for stakeholders.** Is **precise point-level geocoding** (lat/long per ZIP) needed, or are
State/Region-level filled maps sufficient for the geographic decisions this report supports?

**Proposed resolution.** Use the **Geography hierarchy** (Region › State › District › City) for the
State/Region maps and matrices specified on P3; rely on built-in geocoding. If point precision is later
required, join a ZIP→lat/long reference table on `Geo[Zip]` — no remodeling needed.

### G11 — No order/basket grain

**Finding.** `Sales` has **no OrderID, transaction ID, or time-of-day** — only Date (day granularity),
Product, Customer, Campaign, and the constant `Units`. There is no field that groups multiple line items
into a single order/basket.

**Impact on analysis.** **Basket size, order value, items-per-order, and time-of-day patterns cannot be
measured** — the data has no concept of an order above the per-unit row, and dates have no clock component.
Combined with G3 (Units ≡ 1), every purchase event is an isolated single unit on a given day.

**Question for stakeholders.** Is **order-level data** (an OrderID grouping line items, plus timestamps)
available upstream? Basket and order-value analysis depends on it.

**Proposed resolution.** Scope the report to **product, customer, geography, channel, and time at day grain
or coarser** — all of which the current data supports cleanly. Defer basket/order-value analysis until an
order grain with timestamps is contracted; the `Sales` fact can be re-grained to line-item level later
without disturbing the dimensions.

---

*All figures above are reconciled to [data-profile.md](data-profile.md) (profiled 2026-06-27). Measure names,
formats, and DAX match the canonical Key Measures definitions in the
[data dictionary](../dictionary/data-dictionary.md) and model design.*
