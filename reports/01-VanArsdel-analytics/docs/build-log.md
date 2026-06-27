# Build log — VanArsdel Sales Analytics

Dated decisions & changelog. Newest on top.

## 2026-06-27 — Post-open fixes (after first Desktop refresh)
Opened in Power BI Desktop (refresh OK once `DataFolder` was pointed at the local `dataset/raw`).
Three issues surfaced and were fixed:
- **P5 Budget vs Actual — misleading attainment (the big one).** Unfiltered, the page compared 6 years
  of actuals ($65.5M) against 2 years of budget ($22.2M) → a meaningless **295.8%**. Added a **page-level
  filter** (`Date` ∈ 2020-01-01 … 2020-06-30 — the only actuals∩budget overlap), so cards/matrix now show
  the **like-for-like 97.7%** (Revenue $5.96M vs Budget $6.09M; variance −$137K). This is what report-spec
  §P5 / gap G2 prescribed; the default simply hadn't been applied at build time. Year slicer removed from
  P5 (the page filter defines the period); Category slicer kept; year-column changed to **Revenue vs Budget
  by Month**; caveat reworded. The "Comparison period" filter is changeable/clearable in the filter pane.
- **P2 GM% = 27.0% on every row — confirmed *not* a bug.** All 212 products carry a uniform 27.0% markup
  (`Price = Cost ÷ 0.73`), so `[Gross Margin %]` is flat at every grain. Documented in
  [data-profile.md](data-profile.md) finding #8 and noted in the P2 subtitle.
- **Slicer visibility.** P1 Year slicer moved from a cramped bottom strip to a clear top-right box (flat
  GM% card dropped from the P1 hero row to make room).
- **Re-validated:** TOM parse PASS (model), report JSON + 76 bindings resolve, P5 `filterConfig` authored
  per the official PBIR `filterConfiguration`/`semanticQuery` schemas (Advanced, ComparisonKind 2/4,
  `datetime'…'` literals). *(Note: Desktop re-saves some report JSON with CRLF; harmless — Desktop reads it.)*


## 2026-06-27 — PBIP project built (semantic model + report)
- **Semantic model** (`pbip/VanArsdelSales.SemanticModel`, TMDL, Import) built per
  [model-design.md](model-design.md): 2 facts (`Sales`, `Budget`), 6 dimensions
  (`Date` calculated calendar 2015–2021, `Product`, `Product Group` conformed, `Customer`, `Geo`,
  `Campaign`), disconnected `Key Measures` (20 measures), 8 single-direction relationships.
  Power Query cleanups implemented (trim, `Affliliate`→Affiliate, `Deskop`→Desktop, `Email Name`
  split, budget banner-skip + unpivot + month-start `Date`, derived `Category-Segment` + `Channel Type`).
- **Report** (`pbip/VanArsdelSales.Report`, PBIR) built per [report-spec.md](report-spec.md) and
  [wireframe.md](wireframe.md): 5 pages (Executive Overview, Product & Category, Geography & Customers,
  Marketing & Channel, Budget vs Actual), **43 visuals**. Shared theme registered
  (`datapot-theme.json` + base `CY24SU10`).
- **Validation (engine-level, performed 2026-06-27):** the repo's `tmdl-validate` / `validate_pbip.py` /
  `pbir` CLIs were unavailable in this environment, so verification used Power BI Desktop's own libraries
  plus a Power Query replay:
  1. **TOM parse PASS** — `Microsoft.AnalysisServices.Tabular.TmdlSerializer` (from the installed
     `Microsoft.PowerBI.Tabular.dll`, the same library Desktop uses) deserialized the model folder with
     **no errors**: 9 tables, 8 relationships, 20 measures, all partitions (incl. the calculated `Date`
     calendar, calculated `Product Group`, and `Key Measures`). → the model **opens** in Desktop.
  2. **Refresh simulation PASS** — every table's Power Query (M) logic replayed in pandas against the real
     files: declared columns/rows produced; `Product Group` key unique; **all 8 relationships join with
     0 orphans** (`Sales→Date` now 0, vs 112,202 against the supplied 2016 sheet; `Budget→Date` 0);
     measures reconcile (Revenue $65,547,113, Gross Margin $17,697,721).
  3. **DAX reference PASS** — all 20 measures reference only existing tables/columns/measures.
  4. **Report PASS** — all PBIR JSON valid; **79 visual bindings** resolve to real visible columns /
     existing measures; page-identity rule holds (folder == `page.json` name == `pageOrder`); 45 unique
     visual names; LF/no-BOM throughout.
  Residual (needs a human glance in Desktop): visual *formatting/aesthetics* and a true engine DAX
  compile. Open `VanArsdelSales.pbip`, set `DataFolder` to your clone's `dataset/raw`, and Refresh.

## 2026-06-27 — Part 1 & Part 2 deliverables
- Profiled the VanArsdel data ([data-profile.md](data-profile.md)) and filled the
  [dataset contract](../dataset/DATASET-CONTRACT.md).
- Authored Part 1 (business glossary, [data dictionary](../dictionary/data-dictionary.md),
  [data gaps & questions](data-gaps-and-questions.md)) and Part 2 ([BRD](brd.md),
  [report spec](report-spec.md), [wireframe](wireframe.md)) — each adversarially verified against the
  profile (numbers, measure names/DAX, conventions).

## 2026-06-27 — Scaffold
- Scaffolded the report folder from `shared/templates/`, mirroring report-01 structure
  (`dataset/`, `dictionary/`, `docs/`, `pbip/`). Folder later renamed to `01-VanArsdel-analytics`.
- Domain: manufacturing & retail sales analytics (VanArsdel sample — actuals vs budget/forecast).
