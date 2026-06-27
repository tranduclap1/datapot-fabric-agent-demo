# Dataset contract — VanArsdel Sales Analytics

What `dataset/raw/` must contain. This is the agreement between the data producer and the semantic
model; the model's Power Query partitions read these files relative to the `DataFolder` parameter.
Reconciled to the actual files on **2026-06-27** ([data profile](../docs/data-profile.md)).

> ✅ Status: data present and profiled. The model is built from the columns/grain below.

## Files consumed by THIS report (#01)

| File / sheet | Model table | Grain | Rows |
|--------------|-------------|-------|-----:|
| `VanArsdel_Actuals.xlsx` › `Sales` | `Sales` (fact) | one row per unit sold (Product × Date × Customer × Campaign) | 675,368 |
| `VanArsdel_Actuals.xlsx` › `Product` | `Product` (dim) | one row per product | 212 |
| `VanArsdel_Actuals.xlsx` › `Customer` | `Customer` (dim) | one row per customer | 282,597 |
| `VanArsdel_Actuals.xlsx` › `Geo` | `Geo` (dim) | one row per ZIP | 39,948 |
| `VanArsdel_Actuals.xlsx` › `Campaign` | `Campaign` (dim) | one row per traffic-channel × device | 22 |
| `VanArsdel_Actuals.xlsx` › `Date` | *(not used — replaced by calculated `CALENDAR`)* | — | 2,192 |
| `VanArsdel_Budget_Forecast.xlsx` › `Sheet1` | `Budget` (fact, after unpivot) | Type × Year × Month × Category × Segment | 36 → 324 |

### `Sales` (sheet `Sales`)
| Column | Type | Notes |
|--------|------|-------|
| `ProductID` | text | → `Product` |
| `Date` | date | 2015-01-01 → 2020-06-30 |
| `CustomerID` | text (zero-padded, 6 digits) | → `Customer` |
| `CampaignID` | int (1–22) | → `Campaign` |
| `Units` | int | **always 1** (units sold = row count) |

### `Product` (sheet `Product`)
`ProductID (text), Product (text), Category (text, 5), Segment (text, 9), ManufacturerID (text, const '7'),
Manufacturer (text, const 'VanArsdel'), Unit Cost (decimal), Unit Price (decimal)`.

### `Customer` (sheet `Customer`)
`CustomerID (text), ZipCode (text → Geo.Zip), Email Name (text composite "(email): Last, First ")`.
Model splits `Email Name` → `Email` + `Customer Name`; trims trailing whitespace.

### `Geo` (sheet `Geo`)
`Zip (text), City (text "City, ST, USA"), State (text, 2-letter, 49), Region (text: East/Central/West),
District (text, 39), Country (text, const 'USA')`. Hierarchy Country › Region › State › District › City › Zip.

### `Campaign` (sheet `Campaign`)
`CampaignID (int), TrafficChannel (text, 8 — trim + fix 'Affliliate'→Affiliate), Device (text, 5 —
fix 'Deskop'→Desktop)`.

### `Budget & Forecast` (sheet `Sheet1`)
- **Header on row 1** (row 0 is a `"Budget & Forecast"` banner — skip it).
- Columns: `Type (Budget/Forecast), Year (2020/2021), Month (Mon abbrev)` + **9 value columns** named
  `Category-Segment`: `Accessory-Accessory, Mix-All Season, Mix-Productivity, Rural-Select,
  Urban-Convenience, Urban-Extreme, Urban-Moderation, Urban-Regular, Youth-Youth`.
- The model **unpivots** the 9 value columns into `Category`, `Segment`, `Amount`, and derives a
  month-start `Date` = `#date(Year, MonthNo(Month), 1)` for the relationship to `Date`.

## Refresh rules
- Keep sheet names, header positions, and column names/order stable. If they change, update the
  matching `*.tmdl` partition and the [data dictionary](../dictionary/data-dictionary.md).
- Provide a **continuous monthly `Date`/Sales history** — note the supplied `Date` sheet starts 2016
  while Sales starts 2015; the model uses a calculated calendar covering **2015-01-01 → 2021-12-31**.
- Encoding UTF-8. To extend actuals beyond 2020-06, append rows to the `Sales` sheet (same columns).
