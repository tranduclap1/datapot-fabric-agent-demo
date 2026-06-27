# Measures list — VanArsdel Sales Analytics

> The complete catalogue of every explicit measure in the `VanArsdelSales` semantic model:
> name, format, return type, the exact DAX, and a plain-language definition. Generated to match
> the model's `Key Measures` table **1:1** (source of truth is the TMDL, not this doc).

| | |
|---|---|
| **Model** | `pbip/VanArsdelSales.SemanticModel` (TMDL, Import) |
| **Home table** | `Key Measures` — a disconnected calculated table (`partition = {BLANK()}`); the hidden `Value` column is a scaffold only |
| **Count** | **20 measures** across **5 display folders** |
| **Convention** | All analysis is measure-based — `discourageImplicitMeasures` is on; fact columns are hidden and never dragged as values |
| **Money format** | `Unit Price` / `Unit Cost` are unitless sample values; `$` formatting is for readability only |

## At a glance

| # | Measure | Folder | Format | Returns | Depends on |
|---|---------|--------|--------|---------|------------|
| 1 | Units Sold | 01 Volume | `#,##0` | Integer | — *(base)* |
| 2 | Distinct Customers | 01 Volume | `#,##0` | Integer | — *(base)* |
| 3 | Distinct Products Sold | 01 Volume | `#,##0` | Integer | — *(base)* |
| 4 | Revenue | 02 Revenue & Profit | `$#,##0` | Currency | — *(base)* |
| 5 | Cost | 02 Revenue & Profit | `$#,##0` | Currency | — *(base)* |
| 6 | Gross Margin | 02 Revenue & Profit | `$#,##0` | Currency | Revenue, Cost |
| 7 | Gross Margin % | 02 Revenue & Profit | `0.0%` | Percent | Gross Margin, Revenue |
| 8 | Average Selling Price | 02 Revenue & Profit | `$#,##0.00` | Currency | Revenue, Units Sold |
| 9 | Revenue per Customer | 02 Revenue & Profit | `$#,##0` | Currency | Revenue, Distinct Customers |
| 10 | Budget | 03 Budget & Forecast | `$#,##0` | Currency | — *(base)* |
| 11 | Forecast | 03 Budget & Forecast | `$#,##0` | Currency | — *(base)* |
| 12 | Revenue vs Budget | 03 Budget & Forecast | `$#,##0` | Currency | Revenue, Budget |
| 13 | Revenue vs Budget % | 03 Budget & Forecast | `0.0%` | Percent | Revenue, Budget |
| 14 | Budget Attainment % | 03 Budget & Forecast | `0.0%` | Percent | Revenue, Budget |
| 15 | Revenue PY | 04 Time Intelligence | `$#,##0` | Currency | Revenue, Date |
| 16 | Revenue YoY | 04 Time Intelligence | `$#,##0` | Currency | Revenue, Revenue PY |
| 17 | Revenue YoY % | 04 Time Intelligence | `0.0%` | Percent | Revenue, Revenue PY |
| 18 | Revenue YTD | 04 Time Intelligence | `$#,##0` | Currency | Revenue, Date |
| 19 | Digital Revenue | 05 Marketing & Channel | `$#,##0` | Currency | Revenue, Campaign[Device] |
| 20 | Digital Revenue % | 05 Marketing & Channel | `0.0%` | Percent | Digital Revenue, Revenue |

---

## 01 Volume

| Measure | Format | DAX | Definition |
|---------|--------|-----|------------|
| **Units Sold** | `#,##0` | `SUM ( Sales[Units] )` | Total units sold. In this sample each `Sales` row is one unit, so this equals the sales row count. |
| **Distinct Customers** | `#,##0` | `DISTINCTCOUNT ( Sales[Customer ID] )` | Number of unique customers with at least one sale in the current filter context. |
| **Distinct Products Sold** | `#,##0` | `DISTINCTCOUNT ( Sales[Product ID] )` | Number of unique products that recorded a sale. |

## 02 Revenue & Profit

| Measure | Format | DAX | Definition |
|---------|--------|-----|------------|
| **Revenue** | `$#,##0` | `SUMX ( Sales, Sales[Units] * RELATED ( Product[Unit Price] ) )` | List-price (gross) sales — units × the product's list `Unit Price`. **No** discounts, returns or tax (the source has no such columns). |
| **Cost** | `$#,##0` | `SUMX ( Sales, Sales[Units] * RELATED ( Product[Unit Cost] ) )` | Cost of goods sold — units × the product's `Unit Cost`. |
| **Gross Margin** | `$#,##0` | `[Revenue] - [Cost]` | Revenue minus cost. |
| **Gross Margin %** | `0.0%` | `DIVIDE ( [Gross Margin], [Revenue] )` | Gross margin as a share of revenue. |
| **Average Selling Price** | `$#,##0.00` | `DIVIDE ( [Revenue], [Units Sold] )` | Revenue per unit sold. |
| **Revenue per Customer** | `$#,##0` | `DIVIDE ( [Revenue], [Distinct Customers] )` | Average revenue per active customer. |

## 03 Budget & Forecast

| Measure | Format | DAX | Definition |
|---------|--------|-----|------------|
| **Budget** | `$#,##0` | `CALCULATE ( SUM ( Budget[Amount] ), Budget[Type] = "Budget" )` | Planned revenue — `Amount` from rows where plan `Type` = "Budget". |
| **Forecast** | `$#,##0` | `CALCULATE ( SUM ( Budget[Amount] ), Budget[Type] = "Forecast" )` | Forecast revenue — `Amount` from rows where plan `Type` = "Forecast". |
| **Revenue vs Budget** | `$#,##0` | `[Revenue] - [Budget]` | Actual revenue minus budget (absolute variance). |
| **Revenue vs Budget %** | `0.0%` | `DIVIDE ( [Revenue] - [Budget], [Budget] )` | Variance as a share of budget. |
| **Budget Attainment %** | `0.0%` | `DIVIDE ( [Revenue], [Budget] )` | Actual revenue as a share of budget; 100% = exactly on plan. ⚠️ Compare only on the like-for-like overlap (see notes). |

## 04 Time Intelligence

| Measure | Format | DAX | Definition |
|---------|--------|-----|------------|
| **Revenue PY** | `$#,##0` | `CALCULATE ( [Revenue], SAMEPERIODLASTYEAR ( 'Date'[Date] ) )` | Revenue for the same period in the prior year. |
| **Revenue YoY** | `$#,##0` | `[Revenue] - [Revenue PY]` | Year-over-year revenue change (absolute). |
| **Revenue YoY %** | `0.0%` | `DIVIDE ( [Revenue] - [Revenue PY], [Revenue PY] )` | Year-over-year revenue growth rate. |
| **Revenue YTD** | `$#,##0` | `TOTALYTD ( [Revenue], 'Date'[Date] )` | Revenue accumulated from the start of the calendar year to the date in context. |

## 05 Marketing & Channel

| Measure | Format | DAX | Definition |
|---------|--------|-----|------------|
| **Digital Revenue** | `$#,##0` | `CALCULATE ( [Revenue], Campaign[Device] IN { "Desktop", "Mobile", "Tablet" } )` | Revenue from digital device channels (Desktop / Mobile / Tablet). |
| **Digital Revenue %** | `0.0%` | `DIVIDE ( [Digital Revenue], [Revenue] )` | Digital revenue as a share of total revenue. |

---

## Dependency map

```
base ── Units Sold, Distinct Customers, Distinct Products Sold,
        Revenue, Cost, Budget, Forecast

Revenue ─┬─ Gross Margin ── Gross Margin %
         ├─ Average Selling Price        (÷ Units Sold)
         ├─ Revenue per Customer         (÷ Distinct Customers)
         ├─ Revenue vs Budget ── Revenue vs Budget %     (vs Budget)
         ├─ Budget Attainment %                          (÷ Budget)
         ├─ Revenue PY ─┬─ Revenue YoY
         │              └─ Revenue YoY %
         ├─ Revenue YTD
         └─ Digital Revenue ── Digital Revenue %
Cost ───── Gross Margin
```

## Notes & caveats

- **Revenue is list-price / gross sales.** Units × list `Unit Price`, with no discount, return or tax
  adjustment (the source has no such columns). Treat it as gross, not net. See
  [data gaps & questions](../docs/data-gaps-and-questions.md) and the
  [business glossary](../docs/business-glossary.md).
- **Budget / Forecast overlap.** Actuals cover **Jan 2015 – Jun 2020**; Budget/Forecast cover **2020–2021**,
  so the only like-for-like window is **Jan–Jun 2020**. Page 5 is page-filtered to that window — read
  *Budget*, *Revenue vs Budget*, and *Budget Attainment %* there (≈ 97.7% attainment), not on the
  all-years total (which reads a meaningless ~295.8%).
- **Gross Margin % is a flat ~27%** across nearly every slice. That is a **data artifact** — `Unit Cost`
  is a near-constant ratio of `Unit Price` in the synthetic sample — **not** a model bug.
- **`Digital Revenue` vs `Channel Type`.** The **raw** `Device` column had a typo `Deskop` and an offline
  value `Paper`. **Power Query corrects `Deskop`→`Desktop`**, so the modeled devices are Mobile, Tablet,
  Desktop, Paper. `Digital Revenue` (matching `{Desktop, Mobile, Tablet}`) therefore captures **all**
  digital and excludes only offline `Paper` — consistent with the derived `Channel Type`. No under-counting.
- **Helper measures.** None are hidden — all 20 are user-facing. `Revenue PY` and `Digital Revenue` are
  intermediate but exposed because they read sensibly on their own.

> Keep this list in lock-step with `Key Measures.tmdl`. If you add or change a measure, update this file
> and the [data dictionary](data-dictionary.md) (and its `.csv`) in the same change.
