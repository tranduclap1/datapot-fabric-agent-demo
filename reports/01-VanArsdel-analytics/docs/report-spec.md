# Report spec — VanArsdel Sales Analytics

Build-ready specification for report **01 — VanArsdel Sales Analytics**. Reconciled to the ground-truth
[data profile](data-profile.md); names, measures and DAX match the [model design](model-design.md) and
[data dictionary](../dictionary/data-dictionary.md). A DA should be able to build every page directly from
this document — each visual names its **type**, exact **measure(s)**, **dimension(s)/axis**, and any **slicer**.

> ✅ **Status:** built as described below (5 pages, 43 visuals) and opens/refreshes in Power BI Desktop.
> Where the build deviates from an earlier draft it is called out inline as *(build note)*.

> ⚠️ **Read first — Budget-vs-Actual coverage caveat.** Actuals (the `Sales` fact) end **2020-06-30**.
> The plan (`Budget`) covers **2020 (12 mo) + 2021 (12 mo)** and `Forecast` covers **2021 (12 mo)**.
> A *true* Actual-vs-Budget variance therefore exists **only for Jan–Jun 2020**; 2020 H2 and all of 2021
> are **plan-only** (no actuals to compare). **Page 5 is page-filtered to Jan–Jun 2020 by default** so its
> attainment reads the like-for-like **~97.7%** (see [Filters](#filters) and gap G2). Do not present
> full-year 2020 or 2021 "attainment" without this disclosure.

## Purpose & audience
- **Audience:** VanArsdel commercial leadership — VP Sales / Sales Operations, Category & Product
  managers, Regional sales leads, and the FP&A / planning team that owns the Budget & Forecast.
- **Decisions supported:** where revenue and margin concentrate by **category, product and geography**;
  which **acquisition channels and devices** convert to revenue; and — within the comparable window —
  how **actuals track to plan**. Drives category mix and pricing reviews, regional focus, channel-spend
  allocation, and the budget-vs-actual conversation with FP&A.
- **Refresh / latency:** the source is the static VanArsdel sample (`dataset/raw/`). To extend actuals
  beyond 2020-06, append rows to the `Sales` sheet (same columns) and refresh — see the
  [dataset contract](../dataset/DATASET-CONTRACT.md). 100% synthetic data; USD.

## Key questions
1. How are **revenue, gross margin and unit volume** trending month over month, and versus the prior year?
2. Which **categories, segments and products** drive revenue and margin, and where is margin thin?
3. **Where** (region / state) are customers and revenue concentrated, and what is revenue per customer?
4. Which **acquisition channels and devices** convert to revenue, and how large is the **digital** share?
5. Within the comparable window, how does **actual revenue track to the plan** (budget / forecast)?

## KPIs
Headline KPIs surfaced as cards. Targets/benchmarks are stated where a defensible plan exists; for most
operational KPIs the benchmark is the **prior-year / prior-period** read, since the only formal plan in the
data is the revenue **Budget** (Category-Segment grain, 2020–2021).

| KPI | Measure | Target / benchmark |
|-----|---------|--------------------|
| Revenue | `[Revenue]` | vs `[Budget]` (comparable window only — see caveat); vs `[Revenue PY]` |
| Gross Margin | `[Gross Margin]` | trend vs prior year; full-period ≈ **$17.7M** |
| Gross Margin % | `[Gross Margin %]` | full-period blended ≈ **27.0%** — *uniform by data* (fixed markup; flat at every grain) |
| Units Sold | `[Units Sold]` | trend vs prior year; full-period **675,368** units |
| Distinct Customers | `[Distinct Customers]` | trend vs prior year; full-period **282,596** |
| Average Selling Price (ASP) | `[Average Selling Price]` | trend / mix watch |
| Budget Attainment % | `[Budget Attainment %]` | **100%** — read only within the comparable window (Page 5 ≈ **97.7%**) |

> Full-period reference totals (entire `Sales` range, list price × units, no discounts/returns/tax):
> **Revenue $65,547,113 · Cost $47,849,393 · Gross Margin $17,697,721 (≈27.0%)** — from
> [data-profile.md](data-profile.md). These are sanity anchors, not page filters.

## Grain & dimensions
- **Fact grain — `Sales`:** one row per **unit sold** (Product × Date × Customer × Campaign). `Units` is
  **always 1**, so units sold = row count (675,368). All analysis is via measures; fact columns are hidden.
- **Fact grain — `Budget`:** one row per **Type × Year × Month × Category × Segment** (after unpivot of the
  wide budget sheet: 36 raw rows → **324** rows). Holds both Budget and Forecast (`Type` discriminates).
- **Dimensions / slicers:**
  - **Date** — `Date[Year]`, `Date[Quarter]`, `Date[Month Name]`, `Date[Month]`; hierarchy **Calendar**
    (Year › Quarter › Month). Calculated `CALENDAR(DATE(2015,1,1), DATE(2021,12,31))`, marked as date table.
  - **Product** — `Product[Product]`; **Product Group** (conformed) `Product Group[Category]`,
    `Product Group[Segment]` — one Category/Segment slicer filters **both** actuals (via `Product`) and
    `Budget`. (`Product Group[Category-Segment]` is the hidden conformed key, not used directly in visuals.)
  - **Geo** — `Geo[Region]`, `Geo[State]`, `Geo[District]`, `Geo[City]`; hierarchy **Geography**
    (Region › State › District › City).
  - **Campaign** — `Campaign[Traffic Channel]`, `Campaign[Device]`, `Campaign[Channel Type]` (Digital/Offline).

---

## Pages

Five canonical pages, 1280×720 each. Layout convention (CONVENTIONS §4): title band at the top
(y 24, h 72), all visuals at **y ≥ 120**, no overlaps; slicers grouped together. Card stack reads
KPI cards → trend → breakdown → detail.

### Page 1 — Executive Overview
- **Purpose / question:** *"How are sales, margin and volume trending?"* — the at-a-glance health of the
  business across revenue, margin, volume and customers, with prior-year context.
- **Visuals:**

  | # | Visual type | Measure(s) | Dimension / axis | Notes |
  |---|-------------|------------|------------------|-------|
  | 1 | KPI card | `[Revenue]` | — | format `$#,0` |
  | 2 | KPI card | `[Gross Margin]` | — | `$#,0` |
  | 3 | KPI card | `[Units Sold]` | — | `#,0` |
  | 4 | KPI card | `[Distinct Customers]` | — | `#,0` |
  | 5 | Line chart | `[Revenue]`, `[Revenue PY]` (overlay) | Axis: `Date[Month]` ("Mon-YY") | Revenue with prior-year overlay for trend & YoY read |
  | 6 | Clustered column | `[Revenue]` | Axis: `Product Group[Category]` | Revenue by category |
  | 7 | Bar chart | `[Revenue]` | Axis: `Geo[State]` | Revenue by state *(build note: bar; a filled map by `Geo[State]` is an optional enhancement — maps need online basemaps so a bar was chosen for reliable offline rendering)* |
- **Slicer:** `Date[Year]` (top-right). *(build note: the flat `[Gross Margin %]` card was dropped from the hero row — see GM% note — to give the slicer a clear, prominent spot.)*

### Page 2 — Product & Category
- **Purpose / question:** *"Which products and categories drive revenue and margin?"* — category/segment
  mix, the top revenue products, and the category mix of revenue over time.
- **Visuals:**

  | # | Visual type | Measure(s) | Dimension / axis | Notes |
  |---|-------------|------------|------------------|-------|
  | 1 | KPI card | `[Revenue]` | — | `$#,0` |
  | 2 | KPI card | `[Gross Margin %]` | — | `0.0%` — uniform 27% (fixed markup) |
  | 3 | KPI card | `[Average Selling Price]` | — | `$#,0.00` |
  | 4 | Matrix | `[Revenue]`, `[Gross Margin]`, `[Gross Margin %]`, `[Units Sold]`, `[Average Selling Price]` | Rows: `Product Group[Category]` › `Product Group[Segment]` | Category drill to segment; `[Gross Margin %]` is a uniform 27% (see note) so the $ columns carry the signal |
  | 5 | Bar chart | `[Revenue]` | Axis: `Product[Product]` | Top products by Revenue (sorted desc; apply a Top-N filter to taste) |
  | 6 | 100% stacked column | `[Revenue]` | Axis: `Date[Year]` · Legend: `Product Group[Category]` | Category mix of revenue over time |
  | — | Scatter chart *(optional — not built)* | X `[Revenue]` · Y `[Gross Margin %]` · Size `[Units Sold]` | `Product[Product]` | Omitted: `[Gross Margin %]` is flat 27% for every product (fixed markup), so the Y-axis carries no signal for this dataset |
- **Slicer:** `Product Group[Category]` (top-right). Subtitle notes GM% is a uniform 27% by data.

### Page 3 — Geography & Customers
- **Purpose / question:** *"Where are customers and revenue concentrated?"* — regional and state
  concentration of revenue and customers, and revenue per customer.
- **Visuals:**

  | # | Visual type | Measure(s) | Dimension / axis | Notes |
  |---|-------------|------------|------------------|-------|
  | 1 | KPI card | `[Revenue]` | — | `$#,0` |
  | 2 | KPI card | `[Distinct Customers]` | — | `#,0` |
  | 3 | KPI card | `[Revenue per Customer]` | — | `$#,0` |
  | 4 | Bar chart | `[Revenue]` | Axis: `Geo[State]` | Revenue by state *(build note: bar; a filled map by `Geo[State]` is an optional enhancement)* |
  | 5 | Clustered column | `[Revenue]` | Axis: `Geo[Region]` | Region totals; East / Central / West |
  | 6 | Matrix | `[Revenue]`, `[Units Sold]`, `[Distinct Customers]` | Rows: `Geo[Region]` › `Geo[State]` › `Geo[District]` | Geography hierarchy drill-down |
- **Slicer:** `Geo[Region]` (top-right).

### Page 4 — Marketing & Channel
- **Purpose / question:** *"Which acquisition channels and devices convert to revenue?"* — channel and
  device contribution to revenue and the digital share, over time.
- **Visuals:**

  | # | Visual type | Measure(s) | Dimension / axis | Notes |
  |---|-------------|------------|------------------|-------|
  | 1 | KPI card | `[Revenue]` | — | `$#,0` |
  | 2 | KPI card | `[Digital Revenue %]` | — | `0.0%` — digital share of revenue |
  | 3 | KPI card | `[Units Sold]` | — | `#,0` |
  | 4 | Bar chart | `[Revenue]` | Axis: `Campaign[Traffic Channel]` | 8 channels (cleaned; "Affliliate"→Affiliate) |
  | 5 | Clustered column | `[Revenue]` | Axis: `Campaign[Device]` | Devices (cleaned; "Deskop"→Desktop) |
  | 6 | Line chart | `[Digital Revenue %]` | Axis: `Date[Month]` | Digital share trend over time |
  | 7 | Matrix | `[Revenue]` | Rows: `Campaign[Traffic Channel]` · Columns: `Campaign[Device]` | Channel × device revenue grid |
- **Slicer:** `Campaign[Traffic Channel]` (top-right).

### Page 5 — Budget vs Actual
- **Purpose / question:** *"How does actual track to plan (where comparable)?"* — variance of actual
  revenue against the planned `Budget`, at month and Category-Segment grain.
- **🔴 Default comparison window (applied as a page filter):** a page-level **"Comparison period"** filter
  pins `Date` to **2020-01-01 … 2020-06-30** — the only window where actuals **and** budget exist. With it,
  the cards read **Revenue $5.96M · Budget $6.09M · Budget Attainment % 97.7% · Revenue vs Budget −$137K**
  (like-for-like). The filter is visible in the Filters pane and can be changed/cleared. A caveat textbox
  on the page restates that actuals end 2020-06-30 and 2020 H2 / 2021 are plan-only.
- **Visuals:**

  | # | Visual type | Measure(s) | Dimension / axis | Notes |
  |---|-------------|------------|------------------|-------|
  | 1 | KPI card | `[Revenue]` | — | `$#,0` (≈ $5.96M in window) |
  | 2 | KPI card | `[Budget]` | — | `$#,0` (≈ $6.09M in window) |
  | 3 | KPI card | `[Budget Attainment %]` | — | `0.0%` — ≈ **97.7%** in window |
  | 4 | KPI card | `[Revenue vs Budget]` | — | `$#,0` — variance (≈ −$137K) |
  | 5 | Clustered column | `[Revenue]`, `[Budget]` | Axis: `Date[Month]` | Actual vs Budget by month within the comparison window (Jan–Jun 2020) |
  | 6 | Caveat textbox | — | — | Coverage caveat + Rural-Productivity note |
  | 7 | Matrix | `[Revenue]`, `[Budget]`, `[Revenue vs Budget]`, `[Budget Attainment %]` | Rows: `Product Group[Category]` › `Product Group[Segment]` | Variance by plan grain. Rural-Productivity has actuals but no budget → blank Budget/variance |
- **Slicer:** `Product Group[Category]` (top-right). *(build note: no Year slicer — the page-level "Comparison period" filter defines the period; `[Forecast]` is dropped from visual 5 since it is 2021-only and out of the window.)*

---

## Filters

- **Report-level (recommended):**
  - `Date[Year]` — primary time filter on pages 1–4 (per-page Year slicers). A single synced Year slicer
    across pages is an optional enhancement.
  - `Product Group[Category]` — because Category lives on the **conformed** `Product Group` dimension, one
    selection filters **both** `Sales` (via `Product`) **and** `Budget`.
- **Page-level:**
  - **P1 Executive Overview:** `Date[Year]` slicer.
  - **P2 Product & Category:** `Product Group[Category]` slicer (Top-N on the product bar is optional, visual-level).
  - **P3 Geography & Customers:** `Geo[Region]` slicer.
  - **P4 Marketing & Channel:** `Campaign[Traffic Channel]` slicer.
  - **P5 Budget vs Actual:** a **page-level "Comparison period" filter** pins `Date` to
    **2020-01-01 … 2020-06-30** (the only actuals∩budget overlap) so the page opens on the valid
    like-for-like comparison (~97.7% attainment); changeable/clearable in the filter pane. Plus a
    `Product Group[Category]` slicer.
- **Implicit / structural filters:** Manufacturer is **VanArsdel only** (no competitor cut); Country is
  **USA only**; data is synthetic. No security/RLS is applied in this demo.

## Design notes
- **Theme:** `shared/themes/datapot-theme.json` (registered + applied as the report's custom theme).
  **Format via the theme, not per-visual overrides** — colours, fonts, card backgrounds/borders come from
  the theme; numeric formats come from the measure format strings (`$#,0`, `0.0%`, `#,0`, `$#,0.00`).
- **Page size 1280×720.** Title band (textbox) at the top (y 24, h 72) stating the page subject; all
  visuals at **y ≥ 120**; no overlaps. Visual titles state the *differentiator* ("by Category", "by
  Month"); the page title states the subject.
- **Measures, not implicit aggregation.** `discourageImplicitMeasures` is ON; every number on every
  visual is an explicit measure from the `Key Measures` table (folders 01 Volume → 05 Marketing & Channel).
  Never drag a raw fact column as a value.
- **Reading order per page:** KPI cards → trend (line) → breakdown (bar/column) → detail (matrix), with
  the slicer(s) grouped top-right.

---

## Open questions / data gaps

These shape what the report can and cannot claim; the full set of 11 gaps with impact, stakeholder question
and proposed resolution lives in [data-gaps-and-questions.md](data-gaps-and-questions.md) (high-priority ones
summarized in the BRD Section 8). The two highest-impact gaps that directly constrain this report:

- **G2 — Actuals vs Budget overlap [H].** Actuals end **2020-06-30**; Budget = 2020 + 2021, Forecast =
  2021 → **true variance only for Jan–Jun 2020**. *Question:* will 2020 H2 / 2021 actuals be supplied,
  and what comparison period is intended? *(Drives the Page 5 "Comparison period" page filter.)*
- **G1 — Date coverage [H].** The supplied `Date` sheet is 2016–2021 and **misses 2015**
  (112,202 sales rows = **16.61%** fall outside it). Resolved in the model with a calculated
  `CALENDAR(2015-01-01, 2021-12-31)`. *Question:* is 2015 in scope, and why does the supplied calendar
  start in 2016?

Also material to interpretation: `[Revenue]`/`[Cost]` are **list-price** figures with **no discounts,
returns or tax** (G4); `[Gross Margin %]` is a **uniform 27%** (fixed markup) at every grain — correct, not
a bug; `Units` is always 1 so there is **no multi-unit order quantity** (G3); there is **no competitor
data**, so no market share (G5); and `Rural-Productivity` (2 products) **has no budget column**, so it
appears in actuals but is plan-blank in budget-vs-actual (G6).
