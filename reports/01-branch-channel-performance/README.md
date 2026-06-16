# 01 — Branch & Channel Performance

> How transactions and customers flow across branches and channels, how digital adoption is trending,
> and how branches perform on service quality (NPS) and monthly targets. Domain: **retail banking**.

| | |
|---|---|
| **Domain** | Banking — branch & channel operations |
| **Status** | 🟢 All 3 pages built (Overview, Channel Performance, Branch Scorecard); opens & refreshes in Desktop |
| **Audience** | COO / channel & branch operations leads / branch managers |
| **Semantic model** | `pbip/BranchChannelPerformance.SemanticModel` (TMDL, Import) |
| **Report** | `pbip/BranchChannelPerformance.Report` (PBIR) |
| **Data** | BankDIAD sample, monthly 2023-01 → 2025-12, in `dataset/raw/` |
| **Refresh** | Monthly — drop new files into `dataset/raw/…`; folder-combine queries pick them up |

## Contents
- `dataset/` — source files + [DATASET-CONTRACT.md](dataset/DATASET-CONTRACT.md)
- `dictionary/` — [data-dictionary.md](dictionary/data-dictionary.md) · [.csv](dictionary/data-dictionary.csv)
- `docs/` — [report-spec](docs/report-spec.md) · [model-design](docs/model-design.md) · [data-profile](docs/data-profile.md) · [build-log](docs/build-log.md)
- `pbip/` — Power BI project (`BranchChannelPerformance.pbip`)

## Model at a glance
- **Facts:** `Transactions`, `NPS Surveys`, `Branch Targets`.
- **Dimensions:** `Date`, `Branch`, `Channel`, `Transaction Type`, `Product`, `Customer`.
- **22 measures** on `Key Measures` (volume, revenue & targets, service & quality, digital, time intelligence).

## How to open / refresh
1. Ensure `dataset/raw/` has the files in [DATASET-CONTRACT.md](dataset/DATASET-CONTRACT.md) (already present).
2. Open `pbip/BranchChannelPerformance.pbip` in Power BI Desktop.
3. If your clone path differs, update the **`DataFolder`** parameter (Transform data → Parameters), then Refresh.
4. Validate: `python validate_pbip.py pbip --no-pbir-cli` → 0 errors.

## Status notes
- ✅ Data profiled; model reconciled and bound to the real schema.
- ✅ **Overview** — 4 KPI cards + monthly trend + channel breakdown + year slicer.
- ✅ **Channel Performance** — digital-adoption KPIs, channel-mix-over-time (100% stacked), channel
  volume & service time, Channel × Transaction Category matrix.
- ✅ **Branch Scorecard** — KPI cards, per-branch scorecard matrix, NPS by Branch, detractor reasons.
- ⏳ Optional polish: conditional formatting on the scorecard matrix; Top/Bottom-N on NPS by Branch.
- ❓ Revenue-vs-target definition open — Revenue Target shown as context only (see [model-design.md](docs/model-design.md)).
