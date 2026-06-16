# Reports registry

Every report in this repo, newest decisions on top. One folder per report under `reports/`.

| # | Report | Domain | Status | Model | Owner | Notes |
|---|--------|--------|--------|-------|-------|-------|
| 01 | [Branch & Channel Performance](01-branch-channel-performance/README.md) | Banking — branch & channel ops | 🟢 Built | Import (TMDL) over BankDIAD CSVs/XLSX | — | All 3 pages built (Overview, Channel Performance, Branch Scorecard); opens in Desktop |

Legend: 🔴 scoping · 🟡 in progress · 🟢 published.

## Adding a report
1. `cp -r shared/templates` content into `reports/NN-name/` and create `dataset/`, `dictionary/`, `docs/`, `pbip/`.
2. Fill `dataset/DATASET-CONTRACT.md`, then follow [docs/report-lifecycle.md](../docs/report-lifecycle.md).
3. Add a row here.
