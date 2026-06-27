# Model design — VanArsdel Sales Analytics

Semantic model: `pbip/VanArsdelSales.SemanticModel` (TMDL, Import). Reconciled to the profiled
VanArsdel data — see [data-profile.md](data-profile.md) and the [dataset contract](../dataset/DATASET-CONTRACT.md).
All numbers below trace back to the profile; the table/column/measure names are the canonical model
names and must not be renamed.

## Star schema

```
  Facts join (many-side) to dimensions (one-side); every relationship is single-direction.

  Sales   (fact · 675,368 rows · grain: one unit sold)
    ├─ Sales[Date]         → Date       (dim · calculated calendar 2015–2021 · marked Time)
    ├─ Sales[Product ID]   → Product    (dim · 212)
    ├─ Sales[Customer ID]  → Customer   (dim · 282,597)
    └─ Sales[Campaign ID]  → Campaign   (dim · 22)

  Budget  (fact · 324 rows · grain: Type × Year × Month × Category × Segment)
    ├─ Budget[Date]             → Date           (month-start key — join at month grain or coarser)
    └─ Budget[Category-Segment] → Product Group

  Dimension-to-dimension hops (conformed + snowflake; many → one):
    Product[Category-Segment]  → Product Group  (conformed dim · ~10 rows)  ←  Budget[Category-Segment]
    Customer[ZIP]              → Geo            (snowflake · 39,948 ZIPs · Geo is the one-side lookup)

  Disconnected:
    Key Measures  (calculated table · holds all 20 measures · no relationships)

  Filter flow: a Category/Segment slicer on Product Group fans out to BOTH Product→Sales and Budget;
  a Region/State slicer on Geo flows Geo → Customer → Sales.
```

- **Facts:** `Sales` (one row per unit sold) and `Budget` (Type × Year × Month × Category × Segment,
  produced by unpivoting the wide budget sheet).
- **Dimensions:** `Date` (marked Date table, calculated calendar), `Product`, `Product Group`
  (conformed), `Customer`, `Geo`, `Campaign`. `Customer → Geo` is a **snowflake** (ZIP joins on the
  customer, not the fact); everything else is a clean star.
- **`Key Measures`** is a disconnected calculated table that holds all **20** measures, grouped by
  numbered `displayFolder` (`01 Volume` … `05 Marketing & Channel`).
- **Import** with the repo `DataFolder` parameter; `discourageImplicitMeasures` is on — every report
  number is an explicit measure, all fact columns and surrogate keys are hidden.

## Why a star schema here

The supplied `VanArsdel_Actuals.xlsx` already ships as a near-textbook star: one `Sales` fact plus five
lookup sheets (`Product`, `Customer`, `Geo`, `Campaign`, `Date`). The profile confirmed the model is
safe to build directly on it — **0 orphan keys on every relationship** (`Sales→Product/Customer/Campaign`,
`Customer→Geo`). A star keeps the fact narrow (just keys + `Units`), pushes all descriptive attributes
out to conformed dimensions, and lets one slicer fan out across visuals via single-direction filters.
The only two deliberate departures from a pure star are documented below: the **snowflaked `Customer → Geo`**
(forced by the data — `Geo` joins on the customer's ZIP, not the sale) and the **conformed `Product Group`**
dimension (added so a single Category/Segment slicer can drive both the actuals and the budget facts at once).

## The two facts and their grain

### `Sales` (fact · 675,368 rows)
- **Grain: one row per unit sold** (Product × Date × Customer × Campaign). The profile found
  `Units` has exactly **one distinct value (always 1)**, so **units sold = row count** and any true
  order quantity > 1 is not represented in the source ([gap G3](#open-questions)).
- **No money column.** `Sales` carries only keys + `Units`; there is no amount, discount, return, or
  tax field. Revenue and Cost are **derived** at query time from `Product[Unit Price]` / `Product[Unit Cost]`
  via `SUMX … RELATED`, so the model never stores a redundant amount ([gap G4](#open-questions)).
- All columns are **hidden** (`Product ID [ProductID]`, `Date [Date]`, `Customer ID [CustomerID]`,
  `Campaign ID [CampaignID]`, `Units [Units]`); analysis is via measures only.
- Date range **2015-01-01 → 2020-06-30**; **282,596** distinct customers appear in `Sales`.

### `Budget` (fact · 36 raw rows → 324 after unpivot)
- **Grain: Type × Year × Month × Category × Segment.** The source sheet is **wide** — 3 key columns
  (`Type`, `Year`, `Month`) plus **9 `Category-Segment` value columns**. Unpivoting the 9 value columns
  turns each raw row into 9 long rows (36 × 9 = **324**), giving one `Amount` per plan combination.
- Columns (all hidden): `Type [Type]`, `Year [Year]`, `Month Name [Month]`, `Date` (derived month start),
  `Category-Segment` (derived key), `Category`, `Segment`, `Amount` (the unpivoted value).
- Coverage: **Budget 2020 (12 mo) + Budget 2021 (12 mo) + Forecast 2021 (12 mo)**. Totals reconcile to
  the profile: **Budget 2020 ≈ $11.17M, Budget 2021 ≈ $10.99M, Forecast 2021 ≈ $10.99M**.

## Relationships (all single-direction, * on the fact side)

| From (many) | To (one) | Notes |
|---|---|---|
| `Sales[Date]` | `Date[Date]` | actuals on the calendar |
| `Sales[Product ID]` | `Product[Product ID]` | |
| `Sales[Customer ID]` | `Customer[Customer ID]` | |
| `Sales[Campaign ID]` | `Campaign[Campaign ID]` | |
| `Customer[ZIP]` | `Geo[ZIP]` | **snowflake** — Geo joins on the customer, not the fact |
| `Budget[Date]` | `Date[Date]` | budget on the calendar at **month start** |
| `Product[Category-Segment]` | `Product Group[Category-Segment]` | conformed key |
| `Budget[Category-Segment]` | `Product Group[Category-Segment]` | conformed key |

`Geo` is reached **only** through `Customer`; there is no `Sales → Geo` relationship. Region/State/City
filters therefore propagate `Geo → Customer → Sales`, which is correct because geography is an attribute
of the customer in this data.

## The conformed `Product Group` dimension — and why

`Product Group` is a small (~10-row) conformed dimension whose key is `Category-Segment` and whose
attributes are `Category` and `Segment`. It is sourced from the **DISTINCT union of the Category-Segment
combinations found in `Product` and in `Budget`** — so it is a superset of both sides.

**Why it exists.** The two facts live at different grains: `Sales` rolls up to **product** (which carries
Category + Segment), while `Budget` is published directly at **Category-Segment** (it has no product). If
Category/Segment lived only on `Product`, a Category slicer would filter actuals but **not** the budget,
and budget-vs-actual visuals would silently mis-slice. Promoting Category-Segment to a conformed dimension
that joins to **both** `Product` and `Budget` means a **single Category (or Segment) slicer filters both
actuals and plan at once** — the core requirement of the Budget-vs-Actual page (P5) and the Product matrix
(P2). Both `Product` and `Budget` carry hidden `Category-Segment` foreign keys; their own `Category`/`Segment`
columns are hidden so users always slice through the conformed `Product Group` dimension.

**How `Rural-Productivity` is handled.** Products span **10** Category-Segment combinations, but the budget
sheet has only **9** value columns. The missing one is **`Rural-Productivity` (2 products), which has no
budget at all** — the budget's "Rural" line is `Rural-Select`, not `Rural-Productivity`. Because `Product Group`
is the **union** of both sides, `Rural-Productivity` exists in the dimension and its actuals are fully
attributed; it simply has **no matching `Budget` rows**, so `[Budget]` is blank for that group and
`[Revenue vs Budget]` / `[Budget Attainment %]` are undefined there. This is the correct, non-misleading
behaviour — the gap is surfaced, not hidden ([gap G6](#open-questions)).

## The snowflake: `Customer → Geo`

The profile showed ZIP is an attribute of the **customer** (`Customer[ZipCode] → Geo[Zip]`,
**0 orphans**, 29,190 distinct ZIPs used out of 39,948 in `Geo`), not of the sale. Modelling it that way —
`Sales → Customer → Geo` — keeps the fact narrow and avoids duplicating ZIP onto 675K fact rows. The cost
is one extra hop for geography filters, which is acceptable at this row count. `Geo` exposes a
**Geography hierarchy: Region > State > District > City** (USA only; 49 states; Region split East 18,929 /
Central 14,512 / West 6,507). No lat/long is supplied, so maps bind to `State` ([gap G10](#open-questions)).

## The calculated `Date` calendar (2015–2021) — and why it replaces the supplied sheet

`Date` is **not** the supplied `Date` sheet. It is a calculated table
`CALENDAR ( DATE ( 2015, 1, 1 ), DATE ( 2021, 12, 31 ) )`, **marked as a Date table** (`dataCategory: Time`).

**Why replace the sheet.** The supplied `Date` sheet runs **2016-01-01 → 2021-12-31** and **misses 2015
entirely**, while `Sales` starts **2015-01-01**. That leaves **112,202 sales rows (16.61%) with no matching
date row** — they would drop out of every time-sliced visual. A calculated calendar starting **2015**
covers all actuals; extending it to **2021** covers all Budget/Forecast plan rows. Keeping it calculated
also means the model opens and time intelligence works **without any source data loaded**, per repo
convention ([gap G1](#open-questions)).

Columns: `Date` (key), `Year`, `Quarter` ("Q1".."Q4"), `Quarter No` (hidden), `Month Number` (hidden sort),
`Month Name` (Jan..Dec, **sorted by Month Number**), `Month` ("Mon-YY"), `MonthID` (YYYYMM, hidden sort).
Hierarchy **Calendar: Year > Quarter > Month**.

## The budget month-start `Date` key (and the month-grain constraint)

`Budget` has no day-level date — only `Year` + `Month Name`. To join it to the shared `Date` table, the
model derives a **month-start key**: `Date = #date ( Year, MonthNo ( Month ), 1 )`, i.e. the **1st of each
month**. Each budget row therefore contributes its `Amount` on a **single calendar day** (the month start).

**Constraint — budget-vs-actual visuals must aggregate at month grain or coarser.** Because every budget
amount lands on the 1st, a day-level filter (e.g. "2020-03-15") returns the full `[Revenue]` but **zero
`[Budget]`** — the budget for March sits on 2020-03-01 only. At **month grain or higher** (month, quarter,
year) the relationship rolls up correctly: the month's single budget row aligns with that month's actuals.
P5 (Budget vs Actual) and any budget visual therefore use `Date[Month]` / `Date[Quarter]` / `Date[Year]`,
never `Date[Date]`.

**The overlap caveat (must be printed).** Actuals end **2020-06-30**; Budget covers 2020 + 2021 and Forecast
covers 2021. **True Actual-vs-Budget variance exists only for Jan–Jun 2020** — 2020 H2 and all of 2021 are
**plan-only**. `[Budget Attainment %]` is KPI/context only on the headline, and P5 carries a prominent
caveat to that effect ([gap G2](#open-questions)).

## Power Query cleanups

The profile flagged several data-quality issues; each is fixed in the table's Power Query partition so the
model loads clean (raw source files are never edited):

- **Trim trailing whitespace.** `Campaign[TrafficChannel]` (6 values had trailing spaces) and the
  `Customer` `Email Name` composite (trailing whitespace on **every** row) are trimmed.
- **`TrafficChannel` spelling.** `"Affliliate"` → **`"Affiliate"`** (and trimmed) so the channel slicer
  shows one clean value ([gap G7](#open-questions)).
- **`Device` typo.** `"Deskop"` → **`"Desktop"`** so device aggregation and the `Channel Type` derivation
  don't split the desktop value in two.
- **Email Name split.** `Customer[Email Name]` is the composite `"(first.last@xyza.com): Last, First "`.
  It is split into **`Email`** (hidden, parsed) and **`Customer Name`** (the `"Last, First"` display name).
  `@xyza.com` is synthetic PII; `Email` stays hidden.
- **Budget unpivot.** The 9 `Category-Segment` value columns are unpivoted into `Category`, `Segment`,
  and `Amount` (36 → 324 rows), then a `Category-Segment` key and a month-start `Date` are derived.
- **Skip the banner row.** The budget sheet's **row 0 is a `"Budget & Forecast"` title banner**; the real
  header is on **row 1**. The partition skips the banner so `Type`/`Year`/`Month` promote correctly.
- **`Channel Type` derivation.** `Campaign[Channel Type]` = `"Digital"` when `Device` ∈ {Desktop, Mobile,
  Tablet}, else `"Offline"` (covers `Paper` and any non-digital device).

Friendly model names map to `snake_case`-style source columns per [CONVENTIONS.md](../../../CONVENTIONS.md) §3;
the exact mapping lives in the [data dictionary](../dictionary/data-dictionary.md).

## Measure logic highlights

All 20 measures live on `Key Measures` (disconnected), grouped by numbered display folder. Highlights:

- **Volume from the fact.** `Units Sold = SUM ( Sales[Units] )` equals the `Sales` row count because
  `Units` is always 1. `Distinct Customers = DISTINCTCOUNT ( Sales[Customer ID] )` and
  `Distinct Products Sold = DISTINCTCOUNT ( Sales[Product ID] )` count unique participation.
- **Revenue/Cost are derived, not stored.** `Revenue = SUMX ( Sales, Sales[Units] * RELATED ( Product[Unit Price] ) )`
  and `Cost = SUMX ( Sales, Sales[Units] * RELATED ( Product[Unit Cost] ) )` walk each fact row to the
  product's list price/cost. Over the full period these reconcile to the profile: **Revenue $65,547,113,
  Cost $47,849,393, Gross Margin $17,697,721 (≈ 27.0%)**. `Gross Margin = [Revenue] - [Cost]`;
  `Gross Margin % = DIVIDE ( [Gross Margin], [Revenue] )`. These are **list-price gross sales** — no
  discounts, returns, or tax.
- **Per-unit / per-customer economics.** `Average Selling Price = DIVIDE ( [Revenue], [Units Sold] )`;
  `Revenue per Customer = DIVIDE ( [Revenue], [Distinct Customers] )`.
- **Budget & variance filter the `Type` column.** `Budget = CALCULATE ( SUM ( Budget[Amount] ), Budget[Type] = "Budget" )`
  and `Forecast = CALCULATE ( SUM ( Budget[Amount] ), Budget[Type] = "Forecast" )`. Variance:
  `Revenue vs Budget = [Revenue] - [Budget]`, `Revenue vs Budget % = DIVIDE ( [Revenue] - [Budget], [Budget] )`,
  `Budget Attainment % = DIVIDE ( [Revenue], [Budget] )`. `DIVIDE` returns blank (not an error) where there
  is no budget — e.g. for `Rural-Productivity` or any out-of-overlap period.
- **Time intelligence on the marked Date table.** `Revenue PY = CALCULATE ( [Revenue], SAMEPERIODLASTYEAR ( Date[Date] ) )`;
  `Revenue YoY = [Revenue] - [Revenue PY]`; `Revenue YoY % = DIVIDE ( [Revenue] - [Revenue PY], [Revenue PY] )`;
  `Revenue YTD = TOTALYTD ( [Revenue], Date[Date] )`. (The synthetic data repeats monthly counts
  year-over-year, so YoY is structurally near-flat — fine for a demo, not a real trend.)
- **Channel mix via the device flag.** `Digital Revenue = CALCULATE ( [Revenue], Campaign[Device] IN { "Desktop", "Mobile", "Tablet" } )`;
  `Digital Revenue % = DIVIDE ( [Digital Revenue], [Revenue] )`.

## Validation

Build to green: `tmdl-validate` passes on every `.tmdl`; `python validate_pbip.py … --no-pbir-cli`
reports **0 errors**; `jq empty` passes on every PBIR JSON. A cross-reference check confirms every
relationship, measure DAX reference, and report binding resolves before commit.

## Open questions

Resolve with the business. Cross-referenced to the gaps in [data-profile.md](data-profile.md) and the
[dataset contract](../dataset/DATASET-CONTRACT.md).

1. **G1 — Date coverage [H].** Supplied `Date` is 2016–2021 and misses 2015 (**112,202 rows = 16.61%** of
   actuals); it also extends to 2021 where there are no actuals. **Resolution:** calculated calendar
   2015–2021. *Q:* Is 2015 in scope, and why does the supplied Date sheet start in 2016?
2. **G2 — Actuals vs Budget overlap [H].** Actuals end **2020-06-30**; Budget = 2020 + 2021, Forecast = 2021,
   so true variance exists only **Jan–Jun 2020**. *Q:* Will 2020 H2 / 2021 actuals be supplied, and what
   comparison period is intended?
3. **G3 — `Units` always = 1 [H].** Each row is one unit; there is no multi-unit quantity. *Q:* Confirm the
   grain — is true order quantity available?
4. **G4 — No money column in `Sales` [H].** Revenue/Cost are derived from **list** Unit Price/Cost — no
   discounts, returns, tax, or net price. *Q:* Is realized/net revenue (discounting, returns) relevant and
   available?
5. **G5 — No competitor data [M].** Manufacturer is **VanArsdel only** → no market share. *Q:* Is competitive
   analysis in scope (would need competitor sales)?
6. **G6 — Budget grain mismatch [M].** Budget is at **9** Category-Segment combos; products span **10**.
   **`Rural-Productivity` (2 products) has no budget column.** *Q:* How should `Rural-Productivity` be treated
   in budget-vs-actual?
7. **G7 — Data quality [M].** `TrafficChannel` `"Affliliate"` + trailing spaces; `Device` `"Deskop"` typo;
   `Email Name` composite with trailing spaces and embedded PII. **Resolution:** clean/trim/split in Power
   Query. *Q:* Confirm the canonical channel/device spellings.
8. **G8 — Thin customer dimension [M].** 282K customers but only ZIP + email/name — no age, gender, segment,
   tenure, or acquisition date. *Q:* Is richer customer master data available for segmentation / CLV?
9. **G9 — Two Budget years + one Forecast year [L].** Budget 2020 & 2021, Forecast 2021. *Q:* Which is the
   official plan per year, and how do Budget vs Forecast relate for 2021?
10. **G10 — Geo granularity [L].** ZIP-level; `City` is the composite `"City, ST, USA"`; no lat/long. *Q:* Is
    geocoding needed for precise mapping?
11. **G11 — No order/basket grain [L].** No OrderID or time-of-day → cannot measure basket size or order
    value. *Q:* Is order-level data available?
```
