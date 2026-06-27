# CONVENTIONS.md — standards for this repo

Conventions that keep every report consistent and reviewable. Applies to all reports under `reports/`.

## 1. Folder & naming

| Thing | Convention | Example |
|-------|------------|---------|
| Report folder | `reports/NN-kebab-case/` (zero-padded ordinal) | `reports/01-VanArsdel-analytics/` |
| PBIP project files | PascalCase, no spaces | `VanArsdelSales.pbip` |
| Semantic model item | `<Project>.SemanticModel` | `VanArsdelSales.SemanticModel` |
| Report item | `<Project>.Report` | `VanArsdelSales.Report` |
| Page folder | `<PageName>` — **word characters/hyphens only, NO `.Page` suffix** | `ExecutiveOverview` |
| Visual folder | `<VisualName>` — **word characters/hyphens only, NO `.Visual` suffix** | `Card_Revenue` |
| Source data files | as named in the dataset contract (`snake_case.csv` or workbook sheets) | `VanArsdel_Actuals.xlsx` |

**PBIR identity rule:** `pages.json` pageOrder entry == page folder name == `page.json` `name`, and the
folder name must be **word characters or hyphens only (no dots/spaces)**. Power BI Desktop *silently
ignores* any page/visual folder whose name isn't compliant — a `.Page`/`.Visual` dotted suffix makes
Desktop drop the page → "The report has no pages" (even though `pbir`/`validate_pbip` accept the suffix).

## 2. Per-report folder structure (the contract)

```
reports/NN-name/
├── README.md                 # overview, status, owners, how to refresh
├── dataset/
│   ├── DATASET-CONTRACT.md    # the files + columns this report expects (source of truth for paste/profiling)
│   ├── raw/                   # paste source files here (gitignored except .gitkeep, by policy below)
│   └── samples/               # small committed samples used for profiling/CI
├── dictionary/
│   ├── data-dictionary.md     # human-readable: every table, column, measure
│   └── data-dictionary.csv    # machine-readable version of the same
├── docs/
│   ├── report-spec.md         # audience, KPIs, pages, visuals, filters, refresh
│   ├── model-design.md        # star-schema rationale, grain, relationships, measure logic
│   └── build-log.md           # dated decisions & changelog
└── pbip/                       # the Power BI project
    ├── <Project>.pbip
    ├── <Project>.SemanticModel/ (TMDL)
    └── <Project>.Report/        (PBIR)
```

## 3. Semantic model (TMDL) standards

- **Star schema.** Dimensions on the one-side filter facts on the many-side; relationships single-direction
  unless a documented reason requires bidirectional.
- **Friendly model names, snake_case sources.** Model columns are Title Case with spaces (`Customer Name`);
  each maps to a `sourceColumn` (`customer_name`) — the physical column you paste. The dataset
  contract lists the source names.
- **Surrogate keys hidden.** `*_key` columns are `isHidden` + `isKey` on dimensions.
- **Fact columns hidden.** Facts expose nothing directly; all analysis is via measures.
- **All measures live on `Key Measures`**, a disconnected calculated table, grouped by numbered
  `displayFolder` (`01 Volume`, `02 Revenue & Cost`, …). Helper measures are `isHidden`.
- **One `Date` table** marked `dataCategory: Time`, used for all time intelligence. Keep it a calculated
  CALENDAR table so the model opens without source data.
- **Import with `DataFolder` parameter.** Table partitions read pasted files relative to the `DataFolder`
  parameter. Until data is pasted, partitions are empty typed `#table(...)` placeholders so the model
  opens and refreshes to 0 rows. Replace each placeholder with a real query during reconciliation.
- **`discourageImplicitMeasures`** is on — never drag a raw column as a value; use a measure.

## 4. Report (PBIR) standards

- Page size **1280×720**; title textbox at the top (y 24, h 72); place visuals at **y ≥ 120**; no overlaps.
- Prefer **theme formatting** (`shared/themes/datapot-theme.json`, registered per report) over per-visual overrides.
- KPI cards → trend (line/area) → breakdown (bar/column) → detail (table/matrix). Slicers grouped together.
- Visual titles state the *differentiator* ("by Category", "by Month"); the page title states the subject.

## 5. Data dictionary format

`data-dictionary.md` documents, per table: grain, source file, and a row per column
(`Model Name | Source Column | Type | Description | Example`). A separate section lists every measure
(`Measure | Display Folder | Format | DAX | Definition`). `data-dictionary.csv` mirrors the columns for
tooling. Keep the dictionary in lock-step with the TMDL.

## 6. Validation gate

A report is "green" when: `tmdl-validate` passes on every `.tmdl`, `validate_pbip.py … --no-pbir-cli`
reports 0 errors, and `jq empty` passes on every PBIR JSON. Run before every commit.
