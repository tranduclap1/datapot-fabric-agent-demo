# Wireframe — VanArsdel Sales Analytics

Layout sketch for the 5 report pages, **reconciled to the built report** (`pbip/VanArsdelSales.Report`).
Every measure and number here uses the exact table/column/measure names from the
[data dictionary](../dictionary/data-dictionary.md), [model design](model-design.md), and
[report spec](report-spec.md), and every figure reconciles to the [data profile](data-profile.md).

**Canvas.** 1280 × 720, theme `shared/themes/datapot-theme.json` (applied as the report's custom theme).
Title band spans the top (textbox y 24, h 72); the KPI card row and all data visuals sit at **y ≥ 120**.
Each ASCII frame below is ~100 columns wide and represents the full canvas; it is schematic, not pixel-accurate.

**Reading the annotations.** Inside or beside each box: `[VISUAL TYPE] Measure(s) by Dimension`.
- `[CARD]` single KPI · `[LINE]` time trend · `[COL]`/`[BAR]` clustered column/bar ·
  `[100%COL]` 100%-stacked column · `[MATRIX]` matrix · `[SLICER]` filter control.
- *Maps and scatter are **optional enhancements not in the current build*** — geography uses bar charts
  (maps need online basemaps; bars render reliably offline) and the margin scatter is omitted because
  `[Gross Margin %]` is a flat 27% (see below), so a margin Y-axis carries no signal.
- Dimensions cite the **model** column, e.g. `Date[Month]`, `Product Group[Category]`, `Geo[State]`.
- All numbers are **measures** on the `Key Measures` table (`discourageImplicitMeasures` is ON — no
  implicit aggregation). Fact columns stay hidden.

**Grain & data reality this layout respects** (from the [data profile](data-profile.md)):
- Sales fact = **675,368 rows**, one row per **unit sold**; `Units` is **always 1**, so `[Units Sold]`
  equals the row count. Sales span **2015-01-01 → 2020-06-30**; **282,596** distinct customers.
- Over the full Sales period: **Revenue $65,547,113**, **Cost $47,849,393**,
  **Gross Margin $17,697,721 (≈ 27.0%)**. `[Gross Margin %]` is a **uniform 27% at every grain** — the
  data uses a fixed markup (`Price = Cost ÷ 0.73`); correct, not a bug.
- Budget covers **2020 (12 mo) + 2021 (12 mo)**; Forecast covers **2021 (12 mo)**. Actuals end
  **2020-06-30**, so true Actual-vs-Budget variance exists **only for Jan–Jun 2020** — **Page 5 is
  page-filtered to that window** (attainment ≈ **97.7%**).

---

## Page 1 — Executive Overview

**Question answered:** *How are sales, margin and volume trending?*

```
+--------------------------------------------------------------------------------------------------+
|  VanArsdel Sales Analytics — Executive Overview                                                   |  <- title band (y24,h72)
+--------------------------------------------------------------------------------------------------+
| +----------+ +----------+ +-----------+ +-----------------+   +--------------------------------+   |  <- KPI card row + slicer (y112)
| | [CARD]   | | [CARD]   | | [CARD]    | | [CARD]          |   | [SLICER] Date[Year]            |   |
| | Revenue  | | Gross    | | Units     | | Distinct        |   |  [ ] 2015  [ ] 2016 ...         |   |
| | $65.5M   | | Margin   | | Sold      | | Customers       |   |  [ ] 2019  [ ] 2020            |   |
| |          | | $17.7M   | | 675,368   | | 282,596         |   |                                |   |
| +----------+ +----------+ +-----------+ +-----------------+   +--------------------------------+   |
+--------------------------------------------------------------------------------------------------+
| +--------------------------------------------------+  +------------------------------------------+|
| | [LINE]                                           |  | [COL]                                    ||
| | Revenue + Revenue PY  by  Date[Month]            |  | Revenue  by  Product Group[Category]     ||
| |   /\        /\        ___                        |  |  Urban  ##############################    ||
| |  /  \  /\  /  \   ___/   (PY overlay dashed)     |  |  Mix    ########                          ||
| | /    \/  \/    \_/  ........(Revenue PY).....    |  |  Access ####   Youth ###   Rural #        ||
| +--------------------------------------------------+  +------------------------------------------+|
| |                                                  |  +------------------------------------------+|
| | (line spans full left column height)             |  | [BAR] Revenue by Geo[State] (top states) ||
| |                                                  |  |  CA #########  TX #######  NY ######      ||
| +--------------------------------------------------+  +------------------------------------------+|
+--------------------------------------------------------------------------------------------------+
```

| Zone | Visual | Measure(s) | Dimension / axis |
|------|--------|-----------|------------------|
| KPI 1 | `[CARD]` | `Revenue` | — (filtered by Year slicer) |
| KPI 2 | `[CARD]` | `Gross Margin` | — |
| KPI 3 | `[CARD]` | `Units Sold` | — |
| KPI 4 | `[CARD]` | `Distinct Customers` | — |
| Filter | `[SLICER]` | — | `Date[Year]` (top-right, page-level) |
| Trend | `[LINE]` | `Revenue`, `Revenue PY` (overlay) | `Date[Month]` |
| Breakdown | `[COL]` | `Revenue` | `Product Group[Category]` |
| Geography | `[BAR]` | `Revenue` | `Geo[State]` *(bar; filled map = optional enhancement)* |

> The flat `[Gross Margin %]` card was **dropped from the hero row** (it is a uniform 27% — see top note)
> to give the Year slicer a clear, prominent top-right home. Card values shown are full-period totals;
> they recompute under the Year slicer. `Revenue PY` uses `SAMEPERIODLASTYEAR` — blank for 2015 (no prior
> year), meaningful from 2016 on.

---

## Page 2 — Product & Category

**Question answered:** *Which products & categories drive revenue and margin?*

```
+--------------------------------------------------------------------------------------------------+
|  Product & Category  (note: GM% is a uniform 27% by data)   [SLICER] Product Group[Category]      |  <- title band
+--------------------------------------------------------------------------------------------------+
| +----------+ +-------------+ +---------------------+        +--------------------------------+     |  <- KPI cards + slicer
| | [CARD]   | | [CARD]      | | [CARD]              |        | [SLICER]                       |     |
| | Revenue  | | Gross       | | Average Selling     |        | Product Group[Category]        |     |
| |          | | Margin %    | | Price               |        | [ ] Urban [ ] Mix [ ] Rural .. |     |
| +----------+ +-------------+ +---------------------+        +--------------------------------+     |
+--------------------------------------------------------------------------------------------------+
| +----------------------------------------------------+  +----------------------------------------+|
| | [MATRIX] rows: Product Group[Category] > [Segment] |  | [BAR] Product[Product] by Revenue      ||
| |  values: Revenue | Gross Margin | Gross Margin %   |  |  Prod A ########################        ||
| |          | Units Sold | Average Selling Price      |  |  Prod B #####################           ||
| |  Urban > Convenience  $.. $.. 27.0% .. $..          |  |  Prod C ##################              ||
| |  Urban > Moderation   $.. $.. 27.0% .. $..          |  |  ... (descending; Top-N optional)      ||
| |  Mix / Accessory / Youth / Rural ...               |  +----------------------------------------+|
| +----------------------------------------------------+  +----------------------------------------+|
| +----------------------------------------------------+  | [100%COL] Category mix of Revenue      ||
| | (matrix continues / GM% column reads 27.0% flat)   |  |  over Date[Year]                        ||
| |                                                    |  |  2015 |##U##|#M#|A|Y|R|                 ||
| |                                                    |  |  ...  2020 |##U##|#M#|A|Y|R|            ||
| |                                                    |  |  legend = Product Group[Category]      ||
| +----------------------------------------------------+  +----------------------------------------+|
+--------------------------------------------------------------------------------------------------+
```

| Zone | Visual | Measure(s) | Dimension / axis |
|------|--------|-----------|------------------|
| KPI 1 | `[CARD]` | `Revenue` | — |
| KPI 2 | `[CARD]` | `Gross Margin %` | — (uniform 27%) |
| KPI 3 | `[CARD]` | `Average Selling Price` | — |
| Detail | `[MATRIX]` | `Revenue`, `Gross Margin`, `Gross Margin %`, `Units Sold`, `Average Selling Price` | rows `Product Group[Category]` › `Product Group[Segment]` |
| Ranking | `[BAR]` | `Revenue` (sort desc; Top-N optional) | `Product[Product]` |
| Mix trend | `[100%COL]` | `Revenue` (100% stacked) | axis `Date[Year]`, legend `Product Group[Category]` |
| Filter | `[SLICER]` | — | `Product Group[Category]` (page-level) |

> The matrix and slicer are driven by **`Product Group`** (the conformed Category-Segment dimension), so
> the same Category slicer filters both actuals and the Budget on Page 5. The `Gross Margin %` column reads
> **27.0% on every row** — a fixed-markup artifact of the data, so the **$** columns carry the signal.
> *(A margin-vs-revenue scatter was considered but omitted — a flat 27% Y-axis is uninformative.)*

---

## Page 3 — Geography & Customers

**Question answered:** *Where are customers and revenue concentrated?*

```
+--------------------------------------------------------------------------------------------------+
|  Geography & Customers                                      [SLICER] Geo[Region]                  |  <- title band
+--------------------------------------------------------------------------------------------------+
| +----------+ +-------------+ +---------------------+        +--------------------------------+     |  <- KPI cards + slicer
| | [CARD]   | | [CARD]      | | [CARD]              |        | [SLICER] Geo[Region]           |     |
| | Revenue  | | Distinct    | | Revenue per         |        | [ ] East [ ] Central [ ] West  |     |
| |          | | Customers   | | Customer  $232      |        |                                |     |
| | $65.5M   | | 282,596     | |                     |        |                                |     |
| +----------+ +-------------+ +---------------------+        +--------------------------------+     |
+--------------------------------------------------------------------------------------------------+
| +--------------------------------------------------+  +------------------------------------------+|
| | [BAR] Revenue by Geo[State] (top states)         |  | [COL] Revenue by Geo[Region]             ||
| |  CA ##########################                   |  |  East    ############################     ||
| |  TX #####################                        |  |  Central #####################            ||
| |  NY #################   ... (desc)               |  |  West    ###########                      ||
| +--------------------------------------------------+  +------------------------------------------+|
| +-----------------------------------------------------------------------------------------------+ |
| | [MATRIX]  rows: Geo[Region] > [State] > [District]   values: Revenue | Units Sold | Distinct C.| |
| |  East     $..  ..  ..       Central  $..  ..  ..       West  $..  ..  ..    (drill State>District)| |
| +-----------------------------------------------------------------------------------------------+ |
+--------------------------------------------------------------------------------------------------+
```

| Zone | Visual | Measure(s) | Dimension / axis |
|------|--------|-----------|------------------|
| KPI 1 | `[CARD]` | `Revenue` | — |
| KPI 2 | `[CARD]` | `Distinct Customers` | — |
| KPI 3 | `[CARD]` | `Revenue per Customer` | — |
| Geography | `[BAR]` | `Revenue` | `Geo[State]` *(bar; filled map = optional enhancement)* |
| Breakdown | `[COL]` | `Revenue` | `Geo[Region]` |
| Detail | `[MATRIX]` | `Revenue`, `Units Sold`, `Distinct Customers` | rows `Geo[Region]` › `Geo[State]` › `Geo[District]` |
| Filter | `[SLICER]` | — | `Geo[Region]` (page-level) |

> Region splits reflect the ZIP distribution (East 18,929 / Central 14,512 / West 6,507 ZIPs), all-USA.
> `Revenue per Customer` = `DIVIDE([Revenue],[Distinct Customers])` (≈ $232 over the full period). Geography
> is ZIP-level with no lat/long, so state-level **bars** are used; a built-in-geocoded filled map is an
> optional enhancement.

---

## Page 4 — Marketing & Channel

**Question answered:** *Which acquisition channels & devices convert to revenue?*

```
+--------------------------------------------------------------------------------------------------+
|  Marketing & Channel                                       [SLICER] Campaign[Traffic Channel]     |  <- title band
+--------------------------------------------------------------------------------------------------+
| +----------+ +-------------+ +-----------+              +--------------------------------+         |  <- KPI cards + slicer
| | [CARD]   | | [CARD]      | | [CARD]    |              | [SLICER] Traffic Channel       |         |
| | Revenue  | | Digital     | | Units     |              | [ ] Organic [ ] SEM [ ] Email  |         |
| |          | | Revenue %   | | Sold      |              | [ ] Banner [ ] SEO ...         |         |
| +----------+ +-------------+ +-----------+              +--------------------------------+         |
+--------------------------------------------------------------------------------------------------+
| +--------------------------------------------------+  +------------------------------------------+|
| | [BAR] Revenue by Campaign[Traffic Channel]       |  | [COL] Revenue by Campaign[Device]        ||
| |  Organic Search #####################            |  |  Desktop ######################           ||
| |  SEM ################  Email ############         |  |  Mobile  ###################              ||
| |  Banner / SEO / Affiliate / SMO / Mail ...        |  |  Tablet  ############   Paper ##          ||
| +--------------------------------------------------+  +------------------------------------------+|
| +--------------------------------------------------+  +------------------------------------------+|
| | [LINE] Digital Revenue % over Date[Month]        |  | [MATRIX] Traffic Channel (rows)          ||
| |    ___/\___/\___  (digital share trend)          |  |   x Device (cols)  value: Revenue        ||
| |   /                                              |  |          Desktop Mobile Tablet Paper     ||
| |                                                  |  |  Organic  $..    $..    $..    $..        ||
| +--------------------------------------------------+  +------------------------------------------+|
+--------------------------------------------------------------------------------------------------+
```

| Zone | Visual | Measure(s) | Dimension / axis |
|------|--------|-----------|------------------|
| KPI 1 | `[CARD]` | `Revenue` | — |
| KPI 2 | `[CARD]` | `Digital Revenue %` | — |
| KPI 3 | `[CARD]` | `Units Sold` | — |
| Breakdown A | `[BAR]` | `Revenue` | `Campaign[Traffic Channel]` |
| Breakdown B | `[COL]` | `Revenue` | `Campaign[Device]` |
| Trend | `[LINE]` | `Digital Revenue %` | `Date[Month]` |
| Cross-tab | `[MATRIX]` | `Revenue` | rows `Campaign[Traffic Channel]` × cols `Campaign[Device]` |
| Filter | `[SLICER]` | — | `Campaign[Traffic Channel]` (page-level) |

> `Digital Revenue` = `Revenue` where `Campaign[Device] IN {Desktop, Mobile, Tablet}`; `Paper` is Offline.
> Labels show the **cleaned** values (`Affliliate → Affiliate`, `Deskop → Desktop`, trailing spaces
> trimmed in Power Query). 8 Traffic Channels and 4 cleaned Devices (Desktop, Mobile, Tablet, Paper)
> combine into **22 distinct campaign rows** — *not* a full cross-product (8 × 4 ≠ 22).

---

## Page 5 — Budget vs Actual

**Question answered:** *How does actual track to plan (where comparable)?*

```
+--------------------------------------------------------------------------------------------------+
|  Budget vs Actual — page set to the Jan-Jun 2020 overlap   [SLICER] Product Group[Category]       |  <- title band
+--------------------------------------------------------------------------------------------------+
| +-----------+ +-----------+ +---------------+ +--------------------+                               |  <- KPI cards
| | [CARD]    | | [CARD]    | | [CARD]        | | [CARD]             |                               |
| | Revenue   | | Budget    | | Budget        | | Revenue vs Budget  |                               |
| | $5.96M    | | $6.09M    | | Attain. 97.7% | | -$137K             |                               |
| +-----------+ +-----------+ +---------------+ +--------------------+                               |
+--------------------------------------------------------------------------------------------------+
| +--------------------------------------------------+  +------------------------------------------+|
| | [COL] clustered: Revenue vs Budget               |  | [MATRIX] rows: Product Group             ||
| |       by Date[Month]  (Jan-Jun 2020)             |  |   [Category] > [Segment]                 ||
| |   R B   R B   R B   R B   R B   R B              |  | values: Revenue | Budget |               ||
| |  |#|=|  |#|=|  |#|=|  |#|=|  |#|=|  |#|=|         |  |   Revenue vs Budget | Budget Attain. %  ||
| |  Jan20  Feb20  Mar20  Apr20  May20  Jun20        |  |   Urban > Convenience  $.. $.. $.. ..%    ||
| +--------------------------------------------------+  |   Urban > Moderation   $.. $.. $.. ..%    ||
| | !! NOTE (textbox): Comparison period = Jan-Jun   |  |   ... Rural > Productivity  $.. (blank)   ||
| | !! 2020 (only actuals & budget overlap). Cards/  |  |       (actuals, no budget -> blank)      ||
| | !! matrix are like-for-like (~97.7%). Change or  |  |                                          ||
| | !! clear the 'Comparison period' page filter.    |  |                                          ||
| +--------------------------------------------------+  +------------------------------------------+|
+--------------------------------------------------------------------------------------------------+
   PAGE FILTER (Filters pane): Date  is on or after 2020-01-01  AND  on or before 2020-06-30
```

| Zone | Visual | Measure(s) | Dimension / axis |
|------|--------|-----------|------------------|
| KPI 1 | `[CARD]` | `Revenue` | — (≈ $5.96M in window) |
| KPI 2 | `[CARD]` | `Budget` | — (≈ $6.09M in window) |
| KPI 3 | `[CARD]` | `Budget Attainment %` | — (≈ **97.7%**) |
| KPI 4 | `[CARD]` | `Revenue vs Budget` | — (≈ −$137K) |
| Plan-vs-actual | `[COL]` (clustered) | `Revenue`, `Budget` | `Date[Month]` (Jan–Jun 2020) |
| Caveat | `[TEXTBOX]` | — | coverage caveat + Rural-Productivity note |
| Detail | `[MATRIX]` | `Revenue`, `Budget`, `Revenue vs Budget`, `Budget Attainment %` | rows `Product Group[Category]` › `Product Group[Segment]` |
| Filter | `[SLICER]` | — | `Product Group[Category]` (page-level) |
| **Page filter** | (Filters pane) | — | `Date` ∈ **2020-01-01 … 2020-06-30** ("Comparison period") |

> **The page is filtered to Jan–Jun 2020** — the only window where `Revenue` and `Budget` both exist —
> so the cards/matrix read the **like-for-like 97.7%** attainment ($5.96M vs $6.09M; variance −$137K)
> instead of the misleading 295.8% that an unfiltered 6-years-vs-2-years comparison would give. The
> "Comparison period" filter is visible and changeable/clearable in the Filters pane. `Forecast` (2021
> only) is out of the window, so it is not shown on the month column. `Rural-Productivity` (2 products)
> has actuals but **no budget** → blank Budget/variance in the matrix.

---

## Build note

This wireframe is **kept in sync with the built report** (`pbip/VanArsdelSales.Report`): the page
structure, the measure mapped to each visual, the slicing dimension, and the Page-5 page filter all match
what Power BI Desktop renders. Frames are schematic (positions indicative, not pixel coordinates).

Build rules honored: page size **1280 × 720**, title textbox at the top, all data visuals at **y ≥ 120**,
theme formatting from `shared/themes/datapot-theme.json` (no per-visual overrides), page/visual folder
names **word-characters-or-hyphens only**. Every number is a measure on `Key Measures`
(`discourageImplicitMeasures` ON); fact columns stay hidden. All figures reconcile to the
[data profile](data-profile.md).
