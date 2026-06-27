# Data dictionary — VanArsdel Sales Analytics

Lock-step with the TMDL model in `../pbip/VanArsdelSales.SemanticModel` (built — the TMDL is reconciled
to this dictionary). Currency = **USD**.
Reconciled to [`../docs/data-profile.md`](../docs/data-profile.md) and the
[`../dataset/DATASET-CONTRACT.md`](../dataset/DATASET-CONTRACT.md) on **2026-06-27**. All data is
synthetic (the classic VanArsdel sales sample).

## Conventions

- **Model Name** = user-facing name in the model. **Source Column** = physical column as supplied in
  `dataset/raw/` (the sheet column name).
- **Types** use TMDL naming: `int64`, `double`, `string`, `dateTime`, `boolean`.
- **Key** = `PK` (table key, dimension one-side), `FK` (foreign key into another table), or `–`.
- **Hidden** columns are not shown to report users (surrogate keys and all fact columns). Per the
  star-schema standard, **facts expose nothing directly** — every report number comes from a measure on
  `Key Measures`, and `discourageImplicitMeasures` is on.
- **Derived** columns are produced in Power Query or DAX, not pasted; their description says so.

> **Scope.** ~675K unit-level sales rows span **2015-01-01 → 2020-06-30**; a Budget/Forecast plan covers
> **2020–2021**. Manufacturer is **VanArsdel only** (no competitors → no market share). Revenue and Cost
> are **derived from list Unit Price / Unit Cost** — there are no discounts, returns, or tax in the data.

---

## Table: `Sales`  *(fact)*

- **Grain:** one row per **unit sold** (Product × Date × Customer × Campaign). `Units` is **always 1**, so
  units sold = row count; any true order quantity > 1 is not represented.
- **Rows:** 675,368 · 282,596 distinct customers · 212 distinct products.
- **Source file:** `dataset/raw/VanArsdel_Actuals.xlsx` › sheet `Sales`.
- **No money column** — Revenue / Cost / Gross Margin are derived via `Product[Unit Price]` / `[Unit Cost]`.
- All columns are **hidden**; analysis is via measures only.

| Model Name | Source Column | Type | Key | Hidden | Description | Example |
|------------|---------------|------|-----|--------|-------------|---------|
| Product ID | ProductID | string | FK | yes | → `Product[Product ID]` (0 orphans) | `1000` |
| Date | Date | dateTime | FK | yes | Transaction date → `Date[Date]` (2015-01-01 → 2020-06-30) | `2018-04-15` |
| Customer ID | CustomerID | string | FK | yes | → `Customer[Customer ID]`; zero-padded (0 orphans) | `045821` |
| Campaign ID | CampaignID | int64 | FK | yes | → `Campaign[Campaign ID]` (0 orphans) | `7` |
| Units | Units | int64 | – | yes | Units sold; **constant = 1** → row count = units | `1` |

---

## Table: `Budget`  *(fact)*

- **Grain:** one row per **Type × Year × Month × Category × Segment**, after unpivoting the wide sheet.
- **Rows:** 36 raw rows → **324** after unpivot (9 Category-Segment value columns × 36 rows).
- **Source file:** `dataset/raw/VanArsdel_Budget_Forecast.xlsx` › sheet `Sheet1` (row 0 is a
  `"Budget & Forecast"` banner; the real header is on **row 1**).
- **Coverage:** Budget 2020 (12 mo) + Budget 2021 (12 mo) + Forecast 2021 (12 mo). Totals: Budget 2020
  ≈ **$11.17M**, Budget 2021 ≈ **$10.99M**, Forecast 2021 ≈ **$10.99M**.
- All columns are **hidden**; analysis is via the `Budget` / `Forecast` measures.

| Model Name | Source Column | Type | Key | Hidden | Description | Example |
|------------|---------------|------|-----|--------|-------------|---------|
| Type | Type | string | – | yes | Plan type: `Budget` or `Forecast` | `Budget` |
| Year | Year | int64 | – | yes | Plan year (2020 / 2021) | `2020` |
| Month Name | Month | string | – | yes | Month abbreviation as supplied | `Jan` |
| Date | *(derived)* | dateTime | FK | yes | Month-start `#date(Year, MonthNo(Month), 1)` → `Date[Date]` | `2020-01-01` |
| Category-Segment | *(derived)* | string | FK | yes | Unpivoted value-column name → `Product Group[Category-Segment]` | `Urban-Convenience` |
| Category | *(derived)* | string | – | yes | Category portion of the unpivoted key | `Urban` |
| Segment | *(derived)* | string | – | yes | Segment portion of the unpivoted key | `Convenience` |
| Amount | *(unpivoted value)* | double | – | yes | Planned revenue for the cell (USD) | `931250` |

---

## Table: `Date`  *(dimension)*

- **Grain:** one row per calendar date.
- **Source:** **calculated** `CALENDAR(DATE(2015,1,1), DATE(2021,12,31))` — replaces the supplied `Date`
  sheet, which starts **2016** and misses 2015 (112,202 sales rows = 16.61% would fall outside it). Marked
  as the model's Date table (`dataCategory: Time`).
- **Hierarchy `Calendar`:** Year › Quarter › Month.

| Model Name | Source Column | Type | Key | Hidden | Description | Example |
|------------|---------------|------|-----|--------|-------------|---------|
| Date | *(calculated)* | dateTime | PK | no | Calendar date (relationship key) | `2018-04-15` |
| Year | *(derived)* | int64 | – | no | Calendar year | `2018` |
| Quarter | *(derived)* | string | – | no | `Q1`–`Q4` | `Q2` |
| Quarter No | *(derived)* | int64 | – | yes | 1–4, sort key for Quarter | `2` |
| Month Number | *(derived)* | int64 | – | yes | 1–12, sort key for Month Name | `4` |
| Month Name | *(derived)* | string | – | no | `Jan`…`Dec`, sorted by Month Number | `Apr` |
| Month | *(derived)* | string | – | no | `Mon-YY` chronological month label (trend axis) | `Apr-18` |
| MonthID | *(derived)* | int64 | – | yes | `YYYYMM`, sort key for Month | `201804` |

---

## Table: `Product`  *(dimension)*

- **Grain:** one row per product. **Rows:** 212 (173 distinct `Product` names — some repeat across IDs).
- **Source file:** `dataset/raw/VanArsdel_Actuals.xlsx` › sheet `Product`.
- **Category** (5): Urban 169, Mix 20, Accessory 10, Youth 10, Rural 3. **Segment** (9): Convenience 79,
  Moderation 72, Extreme 17, Productivity 12, Accessory 10, All Season 10, Youth 10, Regular 1, Select 1.
- `Manufacturer` / `ManufacturerID` are **constant** (`VanArsdel` / `7`) → no competitor data.

| Model Name | Source Column | Type | Key | Hidden | Description | Example |
|------------|---------------|------|-----|--------|-------------|---------|
| Product ID | ProductID | string | PK | yes | Product key | `1000` |
| Product | Product | string | – | no | Product display name | `VanArsdel Maximus UE-44` |
| Manufacturer | Manufacturer | string | – | yes | Constant `VanArsdel` (no competitors) | `VanArsdel` |
| Category-Segment | *(derived)* | string | FK | yes | `Category` & `-` & `Segment` → `Product Group[Category-Segment]` | `Urban-Convenience` |
| Category | Category | string | – | yes | Product category (surfaced via `Product Group`) | `Urban` |
| Segment | Segment | string | – | yes | Product segment (surfaced via `Product Group`) | `Convenience` |
| Unit Cost | Unit Cost | double | – | yes | List unit cost, USD (range 15.31–149.46) | `42.18` |
| Unit Price | Unit Price | double | – | yes | List unit price, USD (range 20.97–204.74) | `58.40` |

---

## Table: `Product Group`  *(dimension · conformed)*

- **Grain:** one row per **Category-Segment** combination. **Rows:** ~10 combinations.
- **Source:** **derived** — DISTINCT union of the Category-Segment combos in `Product` and in `Budget`.
- **Purpose:** a conformed dimension so **one** Category / Segment slicer filters **both** actuals (via
  `Product`) and plan (via `Budget`). Products span **10** combos; Budget covers only **9** — combo
  `Rural-Productivity` (2 products) has **no budget**, so it appears in actuals but is plan-blank.

| Model Name | Source Column | Type | Key | Hidden | Description | Example |
|------------|---------------|------|-----|--------|-------------|---------|
| Category-Segment | *(derived)* | string | PK | yes | Conformed key on the one-side of `Product` and `Budget` | `Urban-Convenience` |
| Category | *(derived)* | string | – | no | Category (slicer / matrix rows) | `Urban` |
| Segment | *(derived)* | string | – | no | Segment (slicer / matrix rows) | `Convenience` |

---

## Table: `Customer`  *(dimension)*

- **Grain:** one row per customer. **Rows:** 282,597 (282,596 appear in `Sales`).
- **Source file:** `dataset/raw/VanArsdel_Actuals.xlsx` › sheet `Customer`.
- `Email Name` is a **composite** `"(first.last@xyza.com): Last, First "` with trailing whitespace on
  every row → split into `Customer Name` (`Last, First`) and `Email`, trimmed, in Power Query. Synthetic
  PII (`@xyza.com`). **Thin** dimension — only ZIP + email/name (no age, gender, segment, tenure).

| Model Name | Source Column | Type | Key | Hidden | Description | Example |
|------------|---------------|------|-----|--------|-------------|---------|
| Customer ID | CustomerID | string | PK | yes | Customer key; zero-padded | `045821` |
| Customer Name | *(parsed from Email Name)* | string | – | no | `Last, First`, trimmed | `Nguyen, An` |
| Email | *(parsed from Email Name)* | string | – | yes | Parsed e-mail, trimmed (synthetic PII) | `an.nguyen@xyza.com` |
| ZIP | ZipCode | string | FK | yes | → `Geo[ZIP]` (29,190 distinct, 0 orphans) | `98101` |

---

## Table: `Geo`  *(dimension)*

- **Grain:** one row per ZIP code. **Rows:** 39,948 ZIPs (29,190 used by customers).
- **Source file:** `dataset/raw/VanArsdel_Actuals.xlsx` › sheet `Geo`.
- **Region** (3): East 18,929 / Central 14,512 / West 6,507. **State** (49), **District** (39),
  **Country** USA only. `City` is a composite `"City, ST, USA"`. No lat/long.
- **Hierarchy `Geography`:** Region › State › District › City.

| Model Name | Source Column | Type | Key | Hidden | Description | Example |
|------------|---------------|------|-----|--------|-------------|---------|
| ZIP | Zip | string | PK | yes | ZIP key (one-side of `Customer`) | `98101` |
| City | City | string | – | no | City label (composite `City, ST, USA`) | `Seattle, WA, USA` |
| State | State | string | – | no | 2-letter state (49 distinct) | `WA` |
| Region | Region | string | – | no | East / Central / West | `West` |
| District | District | string | – | no | District (39 distinct) | `Pacific Northwest` |
| Country | Country | string | – | no | Constant `USA` | `USA` |

---

## Table: `Campaign`  *(dimension)*

- **Grain:** one row per **Traffic Channel × Device**. **Rows:** 22 (Campaign IDs 1–22).
- **Source file:** `dataset/raw/VanArsdel_Actuals.xlsx` › sheet `Campaign`.
- **DQ cleanups (Power Query):** trim `TrafficChannel`; fix `Affliliate` → `Affiliate`; fix `Deskop` →
  `Desktop`. Traffic channels: Organic Search, SEO, Banner, Affiliate, SEM, Email, SMO, Mail.
  Devices: Mobile, Tablet, Desktop, Paper.

| Model Name | Source Column | Type | Key | Hidden | Description | Example |
|------------|---------------|------|-----|--------|-------------|---------|
| Campaign ID | CampaignID | int64 | PK | yes | Campaign key (1–22) | `7` |
| Traffic Channel | TrafficChannel | string | – | no | Acquisition channel, trimmed + `Affliliate`→`Affiliate` | `Affiliate` |
| Device | Device | string | – | no | Device, `Deskop`→`Desktop` | `Mobile` |
| Channel Type | *(derived)* | string | – | no | `Digital` if Device ∈ {Desktop, Mobile, Tablet}, else `Offline` | `Digital` |

---

## Table: `Key Measures`  *(measure host)*

- A **disconnected** calculated table holding all measures — it has no analytical columns; see the
  [Measures](#measures-all-on-key-measures) section. Per standard, `discourageImplicitMeasures` is on so
  every report number is an explicit measure.

---

## Measures  *(all on `Key Measures`)*

The canonical model defines the measures below, grouped by numbered display folder. DAX is verbatim.
Reconcile values to [`../docs/data-profile.md`](../docs/data-profile.md) (full-period Revenue
**$65,547,113** · Cost **$47,849,393** · Gross Margin **$17,697,721** ≈ **27.0%**).

| Measure | Display Folder | Format | DAX | Definition |
|---------|----------------|--------|-----|------------|
| Units Sold | 01 Volume | `#,0` | `SUM ( Sales[Units] )` | Total units sold (= `Sales` row count, since `Units` is always 1). |
| Distinct Customers | 01 Volume | `#,0` | `DISTINCTCOUNT ( Sales[Customer ID] )` | Unique customers who purchased. |
| Distinct Products Sold | 01 Volume | `#,0` | `DISTINCTCOUNT ( Sales[Product ID] )` | Distinct products sold in context. |
| Revenue | 02 Revenue & Profit | `$#,0` | `SUMX ( Sales, Sales[Units] * RELATED ( Product[Unit Price] ) )` | List-price revenue (gross sales; no discounts / returns / tax). |
| Cost | 02 Revenue & Profit | `$#,0` | `SUMX ( Sales, Sales[Units] * RELATED ( Product[Unit Cost] ) )` | Cost of goods sold (units × list unit cost). |
| Gross Margin | 02 Revenue & Profit | `$#,0` | `[Revenue] - [Cost]` | Revenue minus cost. |
| Gross Margin % | 02 Revenue & Profit | `0.0%` | `DIVIDE ( [Gross Margin], [Revenue] )` | Margin as a share of revenue (full period ≈ 27.0%). |
| Average Selling Price | 02 Revenue & Profit | `$#,0.00` | `DIVIDE ( [Revenue], [Units Sold] )` | Average revenue per unit (ASP). |
| Revenue per Customer | 02 Revenue & Profit | `$#,0` | `DIVIDE ( [Revenue], [Distinct Customers] )` | Average revenue per purchasing customer. |
| Budget | 03 Budget & Forecast | `$#,0` | `CALCULATE ( SUM ( Budget[Amount] ), Budget[Type] = "Budget" )` | Planned revenue (Budget rows only). |
| Forecast | 03 Budget & Forecast | `$#,0` | `CALCULATE ( SUM ( Budget[Amount] ), Budget[Type] = "Forecast" )` | Forecast revenue (Forecast rows only). |
| Revenue vs Budget | 03 Budget & Forecast | `$#,0` | `[Revenue] - [Budget]` | Actual minus budget (variance amount). |
| Revenue vs Budget % | 03 Budget & Forecast | `0.0%` | `DIVIDE ( [Revenue] - [Budget], [Budget] )` | Variance as a share of budget. |
| Budget Attainment % | 03 Budget & Forecast | `0.0%` | `DIVIDE ( [Revenue], [Budget] )` | Actual as a share of budget. |
| Revenue PY | 04 Time Intelligence | `$#,0` | `CALCULATE ( [Revenue], SAMEPERIODLASTYEAR ( Date[Date] ) )` | Revenue for the same period last year. |
| Revenue YoY | 04 Time Intelligence | `$#,0` | `[Revenue] - [Revenue PY]` | Year-over-year revenue change (amount). |
| Revenue YoY % | 04 Time Intelligence | `0.0%` | `DIVIDE ( [Revenue] - [Revenue PY], [Revenue PY] )` | Year-over-year change percent. |
| Revenue YTD | 04 Time Intelligence | `$#,0` | `TOTALYTD ( [Revenue], Date[Date] )` | Year-to-date revenue. |
| Digital Revenue | 05 Marketing & Channel | `$#,0` | `CALCULATE ( [Revenue], Campaign[Device] IN { "Desktop", "Mobile", "Tablet" } )` | Revenue from digital devices. |
| Digital Revenue % | 05 Marketing & Channel | `0.0%` | `DIVIDE ( [Digital Revenue], [Revenue] )` | Digital share of revenue. |

> **Note on count.** The `Key Measures` table holds exactly **20 measures** — the complete, named
> definitions listed above. The CSV mirrors these 20 rows.

---

## Relationships

All relationships are **single-direction** (filter flows from the one-side dimension to the many-side fact),
with the `*` (many) on the fact side. Referential integrity is clean — **0 orphans** on every key
([data profile](../docs/data-profile.md)).

| # | From (one-side) | To (many-side) | Cardinality | Direction | Notes |
|---|-----------------|----------------|-------------|-----------|-------|
| 1 | `Date[Date]` | `Sales[Date]` | 1 → * | Single | Calendar drives the sales fact (time intelligence). |
| 2 | `Product[Product ID]` | `Sales[Product ID]` | 1 → * | Single | Brings Unit Price / Unit Cost to Revenue / Cost. |
| 3 | `Customer[Customer ID]` | `Sales[Customer ID]` | 1 → * | Single | Customer attribution. |
| 4 | `Campaign[Campaign ID]` | `Sales[Campaign ID]` | 1 → * | Single | Traffic Channel / Device / Channel Type. |
| 5 | `Geo[ZIP]` | `Customer[ZIP]` | 1 → * | Single | Snowflake on the customer side (`Sales → Customer → Geo`). |
| 6 | `Date[Date]` | `Budget[Date]` | 1 → * | Single | Same calendar drives the plan fact (month-start dates). |
| 7 | `Product Group[Category-Segment]` | `Product[Category-Segment]` | 1 → * | Single | Conformed dimension over actuals. |
| 8 | `Product Group[Category-Segment]` | `Budget[Category-Segment]` | 1 → * | Single | Same conformed dimension over plan → one slicer filters both. |

---

## Known limitations (reconciled to the data profile)

- **Date coverage:** the supplied `Date` sheet is 2016–2021 and misses **2015** (112,202 rows = 16.61%);
  resolved by the calculated calendar `2015–2021`.
- **Actuals-vs-Budget overlap:** actuals end **2020-06-30**; Budget = 2020 + 2021, Forecast = 2021 → true
  Actual-vs-Budget variance exists **only for Jan–Jun 2020**. 2020 H2 and 2021 are plan-only.
- **`Units` ≡ 1 & no money column:** quantity is effectively a count; Revenue / Cost / Margin are derived
  from **list** price/cost — no discounts, returns, or tax.
- **Budget grain mismatch:** plan is at 9 Category-Segment combos; products span 10 — `Rural-Productivity`
  has no budget. The conformed `Product Group` dimension makes this gap explicit (plan-blank rows).
- **No competitor data:** Manufacturer is VanArsdel-only → no market-share analysis.
- **Synthetic seasonality:** monthly row counts repeat identically year-over-year — fine for a demo, not a
  real trend.
