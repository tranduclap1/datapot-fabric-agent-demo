# <NN> — <Report Name>

> One-line purpose of this report.

| | |
|---|---|
| **Domain** | <banking sub-domain> |
| **Status** | 🔴 Scoping / 🟡 In progress / 🟢 Published |
| **Owner** | <name / team> |
| **Audience** | <who reads it and what decisions it supports> |
| **Semantic model** | `pbip/<Project>.SemanticModel` (PBIP / TMDL, Import) |
| **Report** | `pbip/<Project>.Report` (PBIR) |
| **Refresh** | <manual / scheduled; cadence> |

## Contents
- `dataset/` — source data + [DATASET-CONTRACT.md](dataset/DATASET-CONTRACT.md)
- `dictionary/` — [data dictionary](dictionary/data-dictionary.md)
- `docs/` — [report spec](docs/report-spec.md), [model design](docs/model-design.md), [build log](docs/build-log.md)
- `pbip/` — Power BI project

## How to refresh / rebuild
1. Paste source files into `dataset/raw/` per the dataset contract.
2. Profile the data; reconcile the TMDL partitions and dictionary.
3. Open `pbip/<Project>.pbip` in Power BI Desktop and refresh.
4. Validate: `python validate_pbip.py pbip --no-pbir-cli`.

## Status notes
- <what's done, what's pending>
