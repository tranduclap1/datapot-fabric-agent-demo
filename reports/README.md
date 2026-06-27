# Reports registry

Every report in this repo, newest decisions on top. One folder per report under `reports/`.

| # | Report | Domain | Status | Model | Owner | Notes |
|---|--------|--------|--------|-------|-------|-------|
| 01 | [VanArsdel Sales Analytics](01-VanArsdel-analytics/README.md) | Manufacturing & Retail — sales | 🟢 Built | Import (TMDL) over VanArsdel Actuals + Budget/Forecast XLSX | — | Part 1 + Part 2 docs done; PBIP opens & refreshes in Desktop — model (2 facts, 6 dims, 20 measures) + 5-page/43-visual report, Datapot theme; P5 page-filtered to the Jan–Jun 2020 overlap |

Legend: 🔴 scoping · 🟡 in progress · 🟢 published.

## Adding a report
1. `cp -r shared/templates` content into `reports/NN-name/` and create `dataset/`, `dictionary/`, `docs/`, `pbip/`.
2. Fill `dataset/DATASET-CONTRACT.md`, then follow [docs/report-lifecycle.md](../docs/report-lifecycle.md).
3. Add a row here.
