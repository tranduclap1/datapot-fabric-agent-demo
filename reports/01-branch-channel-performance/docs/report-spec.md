# Report spec — Branch & Channel Performance

## Purpose & audience
- **Audience:** retail-banking COO / channel & branch operations leads / branch managers.
- **Decisions supported:** where to shift volume to digital, which branches/channels lag on service
  and NPS, how branches track to monthly targets.
- **Refresh:** monthly (drop new monthly files into `dataset/raw/…`; folder-combine picks them up).

## Key questions
1. How many transactions / how much value flow through each channel and branch, and how is it trending?
2. What share of activity is digital, and is digital adoption growing?
3. Which branches/regions have the best and worst NPS and service times?
4. How do branches track against their monthly revenue targets?

## KPIs
| KPI | Measure | Notes |
|-----|---------|-------|
| Total Transactions | `[Total Transactions]` | headline volume |
| Gross Transaction Value | `[Gross Transaction Value]` | inflow + outflow magnitude (VND) |
| Digital Adoption % | `[Digital Adoption %]` | digital share of transactions |
| NPS | `[NPS]` | by branch / region |
| Fee Revenue | `[Fee Revenue]` | vs `Revenue Target` (context — see model-design open Q1) |
| Avg Service Time (min) | `[Avg Service Time (min)]` | read per channel |

## Grain & dimensions
- **Fact grain:** transaction-level (`Transactions`); survey-level (`NPS Surveys`); branch×month (`Branch Targets`).
- **Slicers / dimensions:** Date (Year/Quarter/Month), Branch (Region → City → Branch), Channel,
  Transaction Type, Customer Segment, Product Category.

## Pages

### Page 1 — Overview  *(built)*
KPI cards: Total Transactions, Gross Transaction Value, NPS, Digital Adoption %.
Trend: Transactions by Month (line). Breakdown: Transactions by Channel (column). Slicer: Year.

### Page 2 — Channel Performance  *(built)*
- Digital vs Physical/Assisted transaction mix (100% stacked column by month).
- Transactions & Gross Value by Channel (bar); Avg Service Time by Channel.
- Channel × Transaction Category matrix. Fee Revenue by Channel.

### Page 3 — Branch Scorecard  *(built)*
- Matrix: branch rows × {Total Transactions, Gross Value, Digital Adoption %, NPS, Avg Service Time,
  Fee Revenue, Revenue Target} with conditional formatting.
- Top / bottom branches by NPS; map or bar by Region; NPS reason breakdown (`Primary Reason`).

## Filters
- Report-level: Year (default latest), Region.
- Page-level (P2): Channel Type. (P3): Branch Type, Segment.

## Design notes
- Theme `shared/themes/datapot-theme.json` (registered). Page 1280×720. Format via theme, not per-visual.
- All three pages are built. Pending polish (optional): conditional formatting (data bars / colour
  scale) on the Branch Scorecard matrix, and a Top/Bottom-N filter on "NPS by Branch" (50 branches).
