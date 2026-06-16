# Build log — Branch & Channel Performance

Dated decisions and changes. Newest on top. (Repo created 2026-06-16.)

## 2026-06-16 — Built Pages 2 & 3 (design confirmed with stakeholder first)
- Confirmed business purpose / story / layout with the user before building.
- **Channel Performance** (12 visuals): 4 KPI cards (Digital Adoption %, Digital Transactions,
  Avg Service Time, Fee Revenue); hero 100% stacked column = transaction mix by Channel Type over
  months (digital shift); Transactions by Channel (bar); Avg Service Time by Channel (column);
  Channel × Transaction Category matrix; Year/Region/Channel Type slicers.
- **Branch Scorecard** (11 visuals): 4 KPI cards (Total Transactions, Fee Revenue, NPS, Active
  Customers); scorecard matrix (Region▸City▸Branch rows × Transactions, Gross Value, Fee Revenue,
  Revenue Target [context only — no attainment %, per decision], Digital Adoption %, NPS, Avg Service
  Time); NPS by Branch (bar); Detractor reasons (bar, visual-level filter NPS Category = Detractor);
  Year/Region/Branch Type slicers.
- All visuals bound to real measures/columns; folders word-char compliant; `validate_pbip` 0 errors;
  31 visuals across 3 pages. (pbir CLI flags report.json schema 3.3.0 as unknown — Desktop wrote that
  version; CLI v0.9.19 lag, harmless.)
- Deferred (optional polish): conditional formatting on the scorecard matrix; Top/Bottom-N on NPS by Branch.

## 2026-06-16 — Fixed report "no pages" (PBIR folder naming)
- Power BI Desktop reported "ActivePageName not found / The report has no pages" even though
  `pbir validate` and `validate_pbip` both saw 3 pages. Root cause: page/visual folders used the
  `.Page` / `.Visual` dotted suffix. Per Microsoft's PBIR docs, page/visual folder names must be
  **word characters or hyphens only**; Desktop *silently ignores* folders with dots (treats them as
  private user files) → no pages. Renamed `Overview.Page`→`Overview`, `Title.Visual`→`Title`, etc.
  (the `.Report`/`.SemanticModel` item folders keep their suffix — different rule). Recorded in
  CLAUDE.md and CONVENTIONS.md.
- Also added the required `reportVersionAtImport` on both report.json themes (see below).

## 2026-06-16 — Fixed PBIP load errors found in Power BI Desktop
- TMDL parse: removed an orphaned `///` description (blank line after it) in relationships.tmdl;
  removed all root-level `//`/`///` comments (Desktop rejects `//` at the TMDL root); removed
  blank-after-table-header; normalized all TMDL to LF/UTF-8/no-BOM. Recorded rules in CLAUDE.md.
- PBIR: `report.json` themes were missing the required `reportVersionAtImport` property, which made
  Desktop reject report.json and cascade to "no pages / activePageName not found". Added it to both
  baseTheme and customTheme. Pages themselves were correctly wired.

## 2026-06-16 — Reconciled model to real pasted data
- Sample dataset (BankDIAD, 2023–2025 monthly) was pasted into `dataset/raw/`. Profiled it
  ([data-profile.md](data-profile.md)) — real schema differs from the initial assumption.
- **Rebuilt the semantic model** to bind to the real files:
  - Dimensions `Date`, `Branch`, `Product`, `Customer` from `BankDIAD_Dimensions.xlsx` sheets.
  - Derived `Channel` and `Transaction Type` dims from `Transactions`.
  - Facts `Transactions` and `NPS Surveys` via folder-combine; `Branch Targets` unpivoted from the
    targets workbook.
  - 22 measures reconciled to real columns (inflow/outflow for signed amounts, NPS, digital adoption,
    time intelligence). Dropped assumed measures with no backing data (wait time, staffed hours,
    cross-sell, cost-to-serve).
  - Cleaned trailing spaces in `DimBranch[City]`.
- Scoped report #1 to branch/channel data; reserved `AccountSnapshots`, `CustomerProductMonth`,
  `LoanOriginations` for future reports #2/#3 (documented in the dataset contract).
- Updated `report.json` card binding `Transaction Value` → `Gross Transaction Value`.
- Validation: `tmdl-validate` ✓ (14 files), `validate_pbip.py --no-pbir-cli` → 0 errors,
  cross-reference check (relationships / measure DAX / report bindings) ✓.

## 2026-06-16 — Initial scaffold
- Created the multi-report repo structure and report #1 folder (dataset / dictionary / docs / pbip).
- Built an initial **proposed** star schema from banking-domain knowledge (placeholder empty tables)
  plus a minimal Overview report and the Datapot theme. Superseded by the reconciliation above once
  real data arrived.
- Decision: **profile before finalizing** the model and report — confirmed correct, as the real
  schema differed from the proposal.

## Next
- Build Page 2 (Channel Performance) and Page 3 (Branch Scorecard) per [report-spec.md](report-spec.md).
- Resolve the revenue-vs-target definition (open question 1 in [model-design.md](model-design.md)).
- Open `pbip/BranchChannelPerformance.pbip` in Power BI Desktop, refresh, and sanity-check visuals.
