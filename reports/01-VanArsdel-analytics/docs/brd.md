# Business Requirements Document — VanArsdel Sales Analytics

Report 01 of the Datapot Fabric/Power BI monorepo. This BRD states *why* the report exists, *who*
uses it, *what decisions* it supports, and the *exact metric definitions* the build must implement.
Every number here is reconciled to the profiled source data in
[data-profile.md](data-profile.md) (profiled 2026-06-27). The metric names, table
names and DAX are the canonical backbone — the data dictionary
([../dictionary/data-dictionary.md](../dictionary/data-dictionary.md)), model design and report spec
must match them verbatim.

| | |
|---|---|
| **Report** | 01 — VanArsdel Sales Analytics |
| **Subject** | List-price sales, margin, budget-vs-actual, geography and acquisition channel for VanArsdel |
| **Data** | 100% synthetic VanArsdel sample (US consumer-products manufacturer/retailer), USD |
| **Status** | Requirements — model & report to be built per backbone |
| **Source of truth** | [data-profile.md](data-profile.md), [../dataset/DATASET-CONTRACT.md](../dataset/DATASET-CONTRACT.md) |

---

## 1. Overview & background

VanArsdel is a (synthetic) US manufacturer and retailer of consumer products. The dataset is the
classic VanArsdel sales sample: **675,368** unit-level sales rows spanning **2015-01-01 to
2020-06-30**, plus a separate Budget/Forecast plan covering **2020–2021**. The supplied star schema
has five dimensions (`Date`, `Product`, `Customer`, `Geo`, `Campaign`) around a `Sales` fact, plus a
wide Budget/Forecast sheet. VanArsdel is the **only** manufacturer in the data (`Manufacturer`
constant); the currency is **USD**; the data is **100% synthetic**.

Commercial leadership at VanArsdel currently reconciles sales by hand across spreadsheets, with no
single trusted view of revenue, margin, product mix, geographic concentration, acquisition-channel
performance, or plan attainment. This report consolidates those into **five pages** on a single
governed semantic model so that every published number is an explicit, reviewable measure.

Two characteristics of the data shape the entire design and must be understood up front:

1. **There is no money column in the sales fact.** Revenue and cost are **derived** from the product's
   list `Unit Price` and `Unit Cost` — there are no discounts, returns, tax, or net/realized price in
   the data. Every monetary figure in this report is therefore **list-price (gross) revenue**.
2. **The `Units` column is always 1.** Each `Sales` row is one unit sold, so *units sold = row count*.
   There is no multi-unit order quantity, no order/basket grain, and no `OrderID`.

These are not defects to fix in the report; they are the boundary of what the data can answer, and are
carried through Sections 6 (Assumptions), 7 (Scope) and 8 (Limitations & risks).

### Verified headline totals (full Sales period, list price × units, no discounts/returns)

| Quantity | Value |
|----------|------:|
| Units sold (= rows) | **675,368** |
| Revenue = Σ Units × `Unit Price` | **$65,547,113** |
| Cost = Σ Units × `Unit Cost` | **$47,849,393** |
| Gross Margin | **$17,697,721** (≈ **27.0%**) |
| Distinct customers (in sales) | **282,596** |

---

## 2. Users / personas

Four primary personas consume this report. For each: their role, what they need from the report, and
the key decisions it supports.

### 2.1 Sales & commercial leadership (VP Sales / Commercial Director)

- **Role.** Owns the top-line revenue and margin number for the business; reports performance and
  plan attainment upward; sets commercial priorities.
- **What they need.** A single trusted top-line: Revenue, Gross Margin, Gross Margin %, Units Sold and
  Distinct Customers, with month trend and year-over-year movement; and a *caveated* read of how
  actuals track to plan. Entry point is **P1 Executive Overview** and **P5 Budget vs Actual**.
- **Key decisions.** Whether the business is on track on revenue and margin; where to focus commercial
  effort; how to frame performance against the 2020/2021 plan — accepting that a true actual-vs-budget
  comparison only exists for **Jan–Jun 2020** (see G2).

### 2.2 Product / category managers

- **Role.** Own the product portfolio — which categories and segments to push, price, or rationalize.
- **What they need.** Revenue and **margin by Category and Segment**, top products by revenue, the
  margin-vs-volume trade-off per product, and how the category mix shifts over time. Entry point is
  **P2 Product & Category**.
- **Key decisions.** Which categories/segments to invest in or prune; which high-revenue products are
  low-margin (and vice versa); whether the mix is drifting toward or away from higher-margin lines.
  Note: list-price margin only — true profitability after discounts is out of scope (G4).

### 2.3 Regional sales managers

- **Role.** Own performance for a Region/State/District; manage geographic coverage and concentration.
- **What they need.** Revenue and units by **Region › State › District**, customer counts and revenue
  per customer by geography, and which states drive the business. Entry point is **P3 Geography &
  Customers**.
- **Key decisions.** Where revenue and customers concentrate; which regions/states are under- or
  over-indexed; where to add or reallocate coverage. Geography is USA-only across 3 regions and 49
  states; mapping is ZIP/State-level with no lat/long (G10).

### 2.4 Marketing / acquisition team

- **Role.** Own demand generation across traffic channels and devices; optimize the acquisition mix.
- **What they need.** Revenue by **Traffic Channel** and **Device**, the **Digital vs Offline** split,
  and how the digital share trends over time. Entry point is **P4 Marketing & Channel**.
- **Key decisions.** Which channels and devices convert to the most revenue; whether to shift spend
  toward digital; how the digital share is moving. Caveat: the data attributes each sale to the
  campaign (channel × device) on the row — it is **not** a spend/ROI or conversion-funnel dataset (G11).

---

## 3. Decisions the report supports

The report exists to drive these decisions:

1. **Revenue & margin steering** — is the business growing top line and protecting margin, and where
   is the movement coming from (P1, P2).
2. **Portfolio prioritization** — which categories, segments and products to invest in, reprice or
   prune, on a revenue-vs-margin basis (P2).
3. **Geographic coverage** — where revenue and customers concentrate and where to reallocate regional
   effort (P3).
4. **Acquisition mix** — which traffic channels and devices to favour, and how far to lean into digital
   (P4).
5. **Plan framing** — how actuals track to the 2020/2021 plan *for the comparable window only*, and
   where plan-vs-actual is not yet measurable (P5, with the G2 caveat printed on the page).

The report is **descriptive and diagnostic** (what happened, where, how it splits) — not predictive,
prescriptive, or real-time.

---

## 4. Business questions (mapped to the 5 pages)

The report is organized as one headline question per page.

| Page | Business question | How it is answered |
|------|-------------------|--------------------|
| **P1 — Executive Overview** | How are sales, margin and volume trending? | KPI cards (Revenue, Gross Margin, Units Sold, Distinct Customers); Revenue by Month line with **Revenue PY** overlay; Revenue by Category column; Revenue by State bar; Year slicer. *(GM% card dropped from the hero row — it is a uniform 27% by data; a filled map is an optional enhancement.)* |
| **P2 — Product & Category** | Which products & categories drive revenue and margin? | KPI cards (Revenue, Gross Margin %, Average Selling Price); matrix Category › Segment × {Revenue, Gross Margin, Gross Margin %, Units Sold, Average Selling Price}; top products by Revenue bar; 100% stacked category mix over Year; Category slicer. *(A margin-vs-revenue scatter is omitted — GM% is a flat 27%.)* |
| **P3 — Geography & Customers** | Where are customers and revenue concentrated? | KPI cards (Revenue, Distinct Customers, Revenue per Customer); Revenue by State bar; Revenue by Region column; matrix Region › State › District × {Revenue, Units Sold, Distinct Customers}; Region slicer. *(Filled map is an optional enhancement.)* |
| **P4 — Marketing & Channel** | Which acquisition channels & devices convert to revenue? | KPI cards (Revenue, Digital Revenue %, Units Sold); Revenue by Traffic Channel bar; Revenue by Device column; Digital Revenue % over Month line; Traffic Channel × Device revenue matrix; Traffic Channel slicer. |
| **P5 — Budget vs Actual** | How does actual track to plan (where comparable)? | Cards Revenue, Budget, Budget Attainment %, Revenue vs Budget; clustered column Revenue vs Budget by Month; matrix Category › Segment × {Revenue, Budget, Revenue vs Budget, Budget Attainment %}; Category slicer. **Page-filtered to Jan–Jun 2020** (the only actuals∩budget overlap) so attainment is the like-for-like ≈ 97.7% — see G2. |

---

## 5. KPIs & metric definitions

### 5.1 Headline KPIs

These are the figures leadership reads first.

| KPI | Measure | Format | Full-period value | Notes |
|-----|---------|--------|------------------:|-------|
| Revenue | `[Revenue]` | `$#,0` | $65,547,113 | List-price gross sales (no discounts/returns/tax) |
| Gross Margin | `[Gross Margin]` | `$#,0` | $17,697,721 | Revenue − Cost |
| Gross Margin % | `[Gross Margin %]` | `0.0%` | ≈ 27.0% | Margin share of revenue |
| Units Sold | `[Units Sold]` | `#,0` | 675,368 | = Sales row count (`Units` ≡ 1) |
| Distinct Customers | `[Distinct Customers]` | `#,0` | 282,596 | Unique purchasing customers |
| Average Selling Price | `[Average Selling Price]` | `$#,0.00` | ≈ $97.05 | Revenue ÷ Units Sold |
| Budget Attainment % | `[Budget Attainment %]` | `0.0%` | *context only* | Read for **Jan–Jun 2020** only — see G2 |

> ASP ≈ $97.05 is `$65,547,113 / 675,368` from the verified totals; it is shown for context and is not
> separately asserted in the profile.

### 5.2 Canonical measures — definitions and exact DAX

All measures live on the disconnected **`Key Measures`** table, grouped by numbered display folder.
`discourageImplicitMeasures` is **on** — every report number is one of these measures; no raw column is
ever dragged as a value. There are **20 measures**.

#### 01 Volume

| Measure | Format | Plain-language definition | DAX |
|---------|--------|---------------------------|-----|
| `Units Sold` | `#,0` | Total units sold (= `Sales` row count, since `Units` is always 1). | `SUM ( Sales[Units] )` |
| `Distinct Customers` | `#,0` | Unique customers who purchased. | `DISTINCTCOUNT ( Sales[Customer ID] )` |
| `Distinct Products Sold` | `#,0` | Distinct products sold. | `DISTINCTCOUNT ( Sales[Product ID] )` |

#### 02 Revenue & Profit

| Measure | Format | Plain-language definition | DAX |
|---------|--------|---------------------------|-----|
| `Revenue` | `$#,0` | List-price revenue (units × list unit price). | `SUMX ( Sales, Sales[Units] * RELATED ( Product[Unit Price] ) )` |
| `Cost` | `$#,0` | Cost of goods (units × unit cost). | `SUMX ( Sales, Sales[Units] * RELATED ( Product[Unit Cost] ) )` |
| `Gross Margin` | `$#,0` | Revenue minus cost. | `[Revenue] - [Cost]` |
| `Gross Margin %` | `0.0%` | Margin as a share of revenue. | `DIVIDE ( [Gross Margin], [Revenue] )` |
| `Average Selling Price` | `$#,0.00` | Average revenue per unit (ASP). | `DIVIDE ( [Revenue], [Units Sold] )` |
| `Revenue per Customer` | `$#,0` | Average revenue per purchasing customer. | `DIVIDE ( [Revenue], [Distinct Customers] )` |

#### 03 Budget & Forecast

| Measure | Format | Plain-language definition | DAX |
|---------|--------|---------------------------|-----|
| `Budget` | `$#,0` | Planned revenue (Budget rows). | `CALCULATE ( SUM ( Budget[Amount] ), Budget[Type] = "Budget" )` |
| `Forecast` | `$#,0` | Forecast revenue. | `CALCULATE ( SUM ( Budget[Amount] ), Budget[Type] = "Forecast" )` |
| `Revenue vs Budget` | `$#,0` | Actual minus budget (variance amount). | `[Revenue] - [Budget]` |
| `Revenue vs Budget %` | `0.0%` | Variance as a share of budget. | `DIVIDE ( [Revenue] - [Budget], [Budget] )` |
| `Budget Attainment %` | `0.0%` | Actual as a share of budget. | `DIVIDE ( [Revenue], [Budget] )` |

#### 04 Time Intelligence

| Measure | Format | Plain-language definition | DAX |
|---------|--------|---------------------------|-----|
| `Revenue PY` | `$#,0` | Revenue for the same period last year. | `CALCULATE ( [Revenue], SAMEPERIODLASTYEAR ( Date[Date] ) )` |
| `Revenue YoY` | `$#,0` | Year-over-year revenue change. | `[Revenue] - [Revenue PY]` |
| `Revenue YoY %` | `0.0%` | Year-over-year change percent. | `DIVIDE ( [Revenue] - [Revenue PY], [Revenue PY] )` |
| `Revenue YTD` | `$#,0` | Year-to-date revenue. | `TOTALYTD ( [Revenue], Date[Date] )` |

#### 05 Marketing & Channel

| Measure | Format | Plain-language definition | DAX |
|---------|--------|---------------------------|-----|
| `Digital Revenue` | `$#,0` | Revenue from digital devices. | `CALCULATE ( [Revenue], Campaign[Device] IN { "Desktop", "Mobile", "Tablet" } )` |
| `Digital Revenue %` | `0.0%` | Digital share of revenue. | `DIVIDE ( [Digital Revenue], [Revenue] )` |

> Time-intelligence measures require the **calculated `Date` table** (`CALENDAR(2015-01-01,
> 2021-12-31)`, marked as a date table) because the supplied `Date` sheet starts in 2016 and would
> drop 2015 actuals — see G1.

---

## 6. Assumptions

These assumptions are baked into the model and must be stated wherever the numbers appear.

1. **Revenue is list-price (gross) revenue.** Revenue = Σ units × `Unit Price`; Cost = Σ units ×
   `Unit Cost`. There is **no discount, return, tax, or net/realized price** in the data (G4). All
   monetary measures are gross by construction.
2. **`Units` is always 1.** Every `Sales` row is one unit; therefore *units sold = row count*
   (`[Units Sold] = SUM(Sales[Units])`). There is no multi-unit order quantity (G3).
3. **100% synthetic data.** Figures are for demonstration; they are not real VanArsdel results.
   Monthly row counts repeat identically year-over-year (e.g. every Jan = 5,358, every Apr = 14,394),
   so the "seasonality" is a synthetic artifact, not an observed trend.
4. **Calendar 2015–2021.** The supplied `Date` sheet is 2016–2021 and **misses 2015** — 112,202 sales
   rows (**16.61%**) fall outside it. The model **replaces** it with a calculated `CALENDAR(DATE(2015,1,1),
   DATE(2021,12,31))` marked as a date table (G1). Sales actuals run **2015-01-01 → 2020-06-30**.
5. **VanArsdel is the only manufacturer.** `Manufacturer`/`ManufacturerID` are constant — no
   competitor sales and therefore no market share (G5).
6. **Single currency (USD).** No FX conversion.
7. **Budget at Category-Segment grain.** Budget/Forecast is planned at 9 Category-Segment combinations,
   not per product; a conformed **`Product Group`** dimension lets one Category/Segment slicer filter
   both actuals (via `Product`) and Budget. `Rural-Productivity` (2 products) has **no** budget column
   (G6).
8. **Plan structure.** Budget covers 2020 (12 mo) **and** 2021 (12 mo); Forecast covers 2021 (12 mo).
   Totals: Budget 2020 ≈ **$11.17M**, Budget 2021 ≈ **$10.99M**, Forecast 2021 ≈ **$10.99M** (G9).
9. **Data-quality cleanups applied in Power Query.** `TrafficChannel` trimmed and `Affliliate` →
   `Affiliate`; `Device` `Deskop` → `Desktop`; `Email Name` trimmed and split into `Email` +
   `Customer Name`; `Channel Type` derived as `Digital` (Desktop/Mobile/Tablet) else `Offline` (G7).
10. **Referential integrity is clean.** 0 orphan rows on every key (`Sales`→`Product`/`Customer`/
    `Campaign`; `Customer`→`Geo`), so no rows are dropped or bucketed to "Unknown".

---

## 7. Scope

### 7.1 In scope

- **List-price revenue, cost, and gross margin** by Category, Segment, Product, geography, channel and
  device, and over time (`[Revenue]`, `[Cost]`, `[Gross Margin]`, `[Gross Margin %]`,
  `[Average Selling Price]`).
- **Volume** — Units Sold, Distinct Customers, Distinct Products Sold, Revenue per Customer.
- **Time intelligence** — month trend, Revenue PY overlay, YoY (amount and %), YTD — over the actuals
  window 2015-01 → 2020-06.
- **Geography** — Revenue/Units/Customers by Region › State › District (USA only, 3 regions, 49 states).
- **Acquisition** — Revenue by Traffic Channel and Device, and the Digital vs Offline split.
- **Budget vs Actual** — Revenue vs Budget/Forecast at Category-Segment grain, **for the comparable
  window only** (see G2), with the caveat printed on P5.

### 7.2 Out of scope

Each item below is out of scope because the data cannot support it — tied to the gaps in Section 8.

| Out of scope | Why (gap) |
|--------------|-----------|
| **Competitor / market-share analysis** | Manufacturer is VanArsdel-only; no competitor sales (G5). |
| **Net / discounted / realized revenue** | No money column in `Sales`; revenue is list-price only — no discounts, returns or tax (G4). |
| **Returns / refunds** | Not represented in the data (G4). |
| **Order / basket analysis** (order value, basket size, items per order) | No `OrderID`, no time-of-day; `Units` ≡ 1 so no quantity grain (G3, G11). |
| **Customer segmentation / CLV / cohorts / retention** | Thin customer dimension — only ZIP + email/name; no age, gender, segment, tenure or acquisition date (G8). |
| **Marketing spend / ROI / conversion funnel** | Campaign is channel × device attribution only; no spend, impressions or funnel data. |
| **Real-time / intraday** | Daily grain at coarsest; no event/streaming source. Refresh is batch. |
| **Precise point-level mapping** | Geo is ZIP/State-level with no lat/long; `City` is a composite string (G10). |
| **2020 H2 / 2021 actual-vs-budget variance** | Actuals end 2020-06-30; those periods are plan-only (G2). |

---

## 8. Known limitations & risks

The four **high-priority** gaps (G1–G4) materially shape what the report can and cannot claim and must
be communicated to consumers. G5–G11 are documented for completeness; the full register lives in the
model design / build log.

### High priority

**G1 — Date coverage [H].** The supplied `Date` sheet runs 2016–2021 and **misses 2015**; 112,202
sales rows (**16.61%**) fall outside it, while 2020 H2 and 2021 carry no actuals.
*Impact:* using the supplied date table would silently drop 1/6 of actuals and break time
intelligence. *Resolution:* replace with a calculated `CALENDAR(2015-01-01, 2021-12-31)` marked as a
date table. *Question to stakeholder:* is 2015 in scope, and why does the supplied `Date` start in
2016?

**G2 — Actuals vs Budget overlap (the budget caveat) [H].** Actuals end **2020-06-30**; Budget covers
2020 + 2021 and Forecast covers 2021. **A true Actual-vs-Budget variance therefore exists only for
Jan–Jun 2020.** Outside that window the actual is zero against a non-zero plan, so any full-year
attainment figure is misleading.
*Impact:* `[Budget Attainment %]`, `[Revenue vs Budget]` and `[Revenue vs Budget %]` are only valid for
**Jan–Jun 2020**; 2020 H2 and 2021 are **plan-only**. *Resolution:* P5 carries a **prominent caveat**
on the page and in the spec — *"actuals end 2020-06-30, so true Actual-vs-Budget variance exists only
for Jan–Jun 2020; 2020 H2 and 2021 are plan-only"* — and `Budget Attainment %` is treated as a
context KPI, not a headline. *Question to stakeholder:* will 2020 H2 / 2021 actuals be supplied, and
what comparison period is intended?

**G3 — `Units` always = 1 [H].** Each `Sales` row is a single unit; there is no multi-unit quantity.
*Impact:* "units sold" equals row count; basket size / order quantity cannot be measured. *Resolution:*
define `[Units Sold]` as `SUM(Sales[Units])` (= row count) and document the grain. *Question to
stakeholder:* confirm the unit grain; is true order quantity available?

**G4 — No money column in `Sales` [H].** Revenue and cost are derived from the product's **list**
`Unit Price` / `Unit Cost`. *Impact:* every monetary figure is **gross/list-price** — no discounts,
returns, tax, or net/realized revenue. Margin reflects list margin, not realized profitability.
*Resolution:* derive `[Revenue]`/`[Cost]` via `SUMX … RELATED(Product[…])` and label all monetary
measures as list-price. *Question to stakeholder:* are realized/net revenue, discounting or returns
relevant and available?

### Medium / low priority (summary)

| Gap | Pri | Impact | Resolution / handling |
|-----|-----|--------|-----------------------|
| **G5 — No competitor data** | M | No market share; VanArsdel-only manufacturer. | Out of scope; flag if competitive analysis is requested. |
| **G6 — Budget grain mismatch** | M | Budget at 9 Category-Segment combos; products span 10 — `Rural-Productivity` (2 products) has no budget. | Conformed `Product Group` dim; `Rural-Productivity` actuals show against blank budget. |
| **G7 — Data quality** | M | `TrafficChannel` `Affliliate`/trailing spaces; `Device` `Deskop`; composite `Email Name`. | Clean/trim/split/fix in Power Query. |
| **G8 — Thin customer dimension** | M | 282,597 customers but only ZIP + email/name — no demographics/tenure. | No segmentation/CLV; request richer master data. |
| **G9 — Two Budget years + one Forecast year** | L | Budget 2020 & 2021, Forecast 2021 — ambiguity on the official plan for 2021. | Expose Budget and Forecast separately; confirm the official plan per year. |
| **G10 — Geo granularity** | L | ZIP-level, composite `City`, no lat/long. | State/ZIP mapping only; geocode if precise mapping is required. |
| **G11 — No order/basket grain** | L | No `OrderID` or time-of-day. | Order/basket analysis out of scope. |

### Cross-cutting risks

- **Misreading gross as net.** Consumers may treat list-price revenue/margin as realized — every
  monetary visual and definition must label it gross/list-price (G4).
- **Misreading the budget comparison.** Without the P5 caveat, a full-year `Budget Attainment %` reads
  as failure when it is just an actuals-coverage artifact (G2). The caveat is a **build requirement**,
  not optional polish.
- **Synthetic data mistaken for real signal.** Identical year-over-year monthly counts can be read as a
  genuine seasonal pattern; the report is a demonstration build.

---

*All figures reconcile to [data-profile.md](data-profile.md). Metric names, table/column
names and DAX are canonical and shared with the [data dictionary](../dictionary/data-dictionary.md),
model design and report spec.*
