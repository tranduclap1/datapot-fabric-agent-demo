# 01 — VanArsdel Sales Analytics

> Manufacturing & retail sales performance for **VanArsdel** — units, revenue & margin across
> products, geography and marketing channels, with budget/forecast context.

| | |
|---|---|
| **Domain** | Manufacturing & Retail — sales analytics |
| **Status** | 🟢 Built — opens & refreshes in Power BI Desktop (parameterized `DataFolder`); Datapot theme applied |
| **Owner** | — |
| **Audience** | Sales & commercial leadership, product/category managers, regional sales & marketing |
| **Semantic model** | `pbip/VanArsdelSales.SemanticModel` (PBIP / TMDL, Import) — 2 facts, 6 dims, 20 measures |
| **Report** | `pbip/VanArsdelSales.Report` (PBIR) — 5 pages, 43 visuals, Datapot theme |
| **Refresh** | Monthly (drop refreshed `VanArsdel_Actuals.xlsx` / `VanArsdel_Budget_Forecast.xlsx` into `dataset/raw/`) |

## Contents
- `dataset/` — source data + [DATASET-CONTRACT.md](dataset/DATASET-CONTRACT.md)
- `dictionary/` — [data dictionary](dictionary/data-dictionary.md) · [measures list](dictionary/measures-list.md)
- `docs/` — [business glossary](docs/business-glossary.md) · [data gaps & questions](docs/data-gaps-and-questions.md) · [BRD](docs/brd.md) · [report spec](docs/report-spec.md) · [model design](docs/model-design.md) · [data profile](docs/data-profile.md) · [wireframe](docs/wireframe.md) · [build log](docs/build-log.md)
- `pbip/` — Power BI project *(built — TMDL semantic model + PBIR report)*

## Deliverables
**Part 1 — understand & translate the data**
- [Business glossary](docs/business-glossary.md) — VanArsdel business concepts.
- [Data dictionary](dictionary/data-dictionary.md) — every table, column & relationship.
- [Data gaps & stakeholder questions](docs/data-gaps-and-questions.md).

**Part 2 — spec for handoff**
- [BRD](docs/brd.md) — users, decisions, questions, KPIs + metric definitions, assumptions & scope.
- [Report spec](docs/report-spec.md) — build-ready page/visual specification.
- [Wireframe / mockup](docs/wireframe.md) — page layouts annotated by metric & dimension.

## How to refresh / rebuild
1. Files in `dataset/raw/` per the [dataset contract](dataset/DATASET-CONTRACT.md).
2. Profile (done — see [data profile](docs/data-profile.md)); reconcile the TMDL partitions and dictionary.
3. Open `pbip/VanArsdelSales.pbip` in Power BI Desktop and refresh.
4. Validate: `python validate_pbip.py pbip --no-pbir-cli`.

## Status notes
- Data profiled 2026-06-27 (see [data profile](docs/data-profile.md)). Actuals cover Jan 2015 – Jun 2020;
  Budget/Forecast cover 2020–2021 → Actual-vs-Budget overlap is Jan–Jun 2020 only (Page 5 is page-filtered
  to that window so attainment is the like-for-like ~97.7%).
- Part 1 & Part 2 deliverables complete and verified; PBIP semantic model + report **built, opened and
  refreshed in Power BI Desktop**, with the Datapot theme applied (see [build log](docs/build-log.md)).
- To open: `pbip/VanArsdelSales.pbip` → **Edit parameters** → set `DataFolder` to your clone's
  `dataset/raw` → **Refresh**.
