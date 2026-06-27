# Data profile — VanArsdel Sales Analytics

Profiled **2026-06-27** from the pasted files in `dataset/raw/` (pandas + openpyxl). This is the
ground-truth record that the dataset contract, data dictionary, model design, and report spec are
reconciled against. **Profile before modeling** ([report-lifecycle](../../../docs/report-lifecycle.md) §4).

## Source files

| File | Sheets / shape | Role |
|------|----------------|------|
| `VanArsdel_Actuals.xlsx` (32 MB) | `Date, Campaign, Customer, Product, Geo, Sales` | Star schema — 5 dimensions + `Sales` fact |
| `VanArsdel_Budget_Forecast.xlsx` (20 KB) | `Sheet1` (row 0 = title banner, header on row 1) | Budget & Forecast, **wide** (9 Category-Segment columns) |
| `Actuals_Path.txt`, `Budget_Path.txt`, `Number_Days.txt` | Power Query M snippets | Original sample's parameterized loaders (path + day-of-year helper) — not data |

> The `*_Path.txt` files show the source workbook was loaded via a `Path` + filename parameter
> pattern. We re-implement loading with this repo's `DataFolder` parameter convention.

---

## Table-by-table profile

### `Sales` — fact · 675,368 rows × 5 cols
- **Grain:** one row per **unit sold** (Product × Date × Customer × Campaign). `Units` is **always 1**
  (distinct = 1) → *units sold = row count*; any true quantity > 1 is not represented.
- **No money column** — Revenue/Cost/Margin must be **derived** via the `Product` price/cost.

| Column | Type | Nulls | Distinct | Notes |
|--------|------|-------|----------|-------|
| `ProductID` | string | 0 | 212 | → `Product` (0 orphans) |
| `Date` | date | 0 | 2,002 | **2015-01-01 → 2020-06-30** |
| `CustomerID` | string (zero-padded) | 0 | 282,596 | → `Customer` (0 orphans) |
| `CampaignID` | int | 0 | 22 | → `Campaign` (0 orphans) |
| `Units` | int | 0 | 1 | **constant = 1** |

### `Date` — dimension (as supplied) · 2,192 rows × 7 cols
- Daily, **2016-01-01 → 2021-12-31**. Columns: `Date, MonthNo, MonthName, MonthID (YYYYMM), Month (Mon-YY), Quarter, Year`.
- ⚠️ **Does not cover 2015** — see [date-coverage gap](#-critical-findings--drive-the-model--the-gaps-list).
  We will **replace** it with a calculated `CALENDAR(2015-01-01, 2021-12-31)` per repo convention.

### `Product` — dimension · 212 rows × 8 cols
- `ProductID` (key), `Product`, `Category`, `Segment`, `ManufacturerID`, `Manufacturer`, `Unit Cost`, `Unit Price`.
- **Category** (5): Urban 169, Mix 20, Accessory 10, Youth 10, Rural 3.
- **Segment** (9): Convenience 79, Moderation 72, Extreme 17, Productivity 12, Accessory 10, All Season 10, Youth 10, Regular 1, Select 1.
- `Unit Cost` 15.31–149.46 · `Unit Price` 20.97–204.74 (margin embedded in every product).
- ⚠️ `Manufacturer` / `ManufacturerID` are **constant** (`VanArsdel` / `7`) → **no competitor data**.
- `Product` name has 173 distinct values over 212 IDs (some names repeat across IDs).

### `Customer` — dimension · 282,597 rows × 3 cols
- `CustomerID` (key, zero-padded string), `ZipCode` (→ `Geo`, 29,190 distinct, 0 orphans), `Email Name`.
- ⚠️ `Email Name` is a **composite** `"(first.last@xyza.com): Last, First "` with **trailing whitespace on
  every row** → split into `Email` + `Customer Name` in Power Query. Synthetic PII (`@xyza.com`).
- Only attributes are zip + email/name → **thin** customer dimension (no age/gender/segment/tenure).

### `Geo` — dimension · 39,948 rows × 6 cols
- `Zip` (key), `City` (composite `"City, ST, USA"`, 28,575 distinct), `State` (49), `Region` (3:
  East 18,929 / Central 14,512 / West 6,507), `District` (39), `Country` (USA only).
- Hierarchy: **Country › Region › State › District › City › Zip**. No lat/long.

### `Campaign` — dimension · 22 rows × 3 cols
- `CampaignID` (key 1–22), `TrafficChannel` (8), `Device` (5). Campaign = TrafficChannel × Device.
- ⚠️ DQ: `TrafficChannel` has trailing spaces on 6 values and the misspelling **`Affliliate`** (→ Affiliate);
  values: Organic Search, SEO, Banner, Affliliate[sic], SEM, Email, SMO, Mail.
- ⚠️ DQ: `Device` has the typo **`Deskop`** (→ Desktop) plus `Paper` (1) — values: Mobile, Tablet, Desktop, Deskop[sic], Paper.

### `Budget & Forecast` — fact (aggregate) · 36 rows raw → 324 after unpivot
- Header on **row 1** (row 0 is a `"Budget & Forecast"` banner). Columns: `Type, Year, Month` + **9
  `Category-Segment` value columns**.
- After unpivot → grain = **Type × Year × Month × Category × Segment** (`Amount`).
- `Type` × `Year`: **Budget 2020 (12 mo) + Budget 2021 (12 mo) + Forecast 2021 (12 mo)**.
- Totals: Budget 2020 ≈ **$11.17M**, Budget 2021 ≈ **$10.99M**, Forecast 2021 ≈ **$10.99M**.
- The 9 budget combos: Accessory-Accessory, Mix-All Season, Mix-Productivity, **Rural-Select**,
  Urban-Convenience, Urban-Extreme, Urban-Moderation, Urban-Regular, Youth-Youth.

---

## Referential integrity (all clean)

| Relationship | Child keys | Parent keys | Orphan rows |
|--------------|-----------:|------------:|------------:|
| `Sales.ProductID` → `Product.ProductID` | 212 | 212 | **0** |
| `Sales.CustomerID` → `Customer.CustomerID` | 282,596 | 282,597 | **0** |
| `Sales.CampaignID` → `Campaign.CampaignID` | 22 | 22 | **0** |
| `Customer.ZipCode` → `Geo.Zip` | 29,190 | 39,948 | **0** |

→ A clean **snowflake on the customer side** (`Sales → Customer → Geo`); star elsewhere.

---

## Derived measures sanity (full Sales period, list price × units, no discounts/returns)

| Quantity | Value |
|----------|------:|
| Units sold (= rows) | 675,368 |
| Revenue = Σ Units × Unit Price | **$65,547,113** |
| Cost = Σ Units × Unit Cost | $47,849,393 |
| Gross Margin | **$17,697,721** (≈ 27.0%) |

---

## ⚠️ Critical findings → drive the model & the gaps list

1. **Date coverage.** Supplied `Date` is 2016–2021 but **Sales starts 2015-01-01** → **112,202 rows
   (16.61%) fall outside** it; meanwhile 2020 H2 + 2021 have **no actuals**. Fix: calculated
   `CALENDAR(2015-01-01, 2021-12-31)`.
2. **Actuals vs Budget overlap is tiny.** Actuals end **2020-06-30**; Budget/Forecast are 2020–2021.
   True Actual-vs-Budget variance exists only for **Jan–Jun 2020**. (→ stakeholder question.)
3. **`Units` ≡ 1 & no money column.** Quantity is effectively a count; Revenue/Cost/Margin are derived
   from list price/cost — **no discounts, returns, or tax** in the data.
4. **No competitor data.** `Manufacturer` is VanArsdel-only → no market-share analysis.
5. **Budget is at Category-Segment grain**, and **`Rural-Productivity` (2 products) has no budget column**
   → needs conformed Category-Segment handling for budget-vs-actual.
6. **DQ cleanups** (Power Query): trim `TrafficChannel`/`Email Name`; fix `Affliliate`→Affiliate,
   `Deskop`→Desktop; split `Email Name` into `Email` + `Customer Name`.
7. **Synthetic seasonality artifact.** Monthly row counts repeat identically year-over-year
   (e.g., every Jan = 5,358; every Apr = 14,394) — fine for a demo; not "real" trend.
8. **Uniform gross margin (markup artifact).** **Every one of the 212 products has exactly 27.0%
   margin** — `Unit Price = Unit Cost ÷ 0.73` for all rows. So `[Gross Margin %]` is a flat **27.0%**
   at *every* grain (product, category, segment, region, channel). This is **correct** behaviour given
   the data, not a model bug — only the Revenue/Margin **$** amounts vary. (P2 notes this in its subtitle.)

All downstream docs (dictionary, model design, BRD, report spec) must reconcile to the numbers above.
