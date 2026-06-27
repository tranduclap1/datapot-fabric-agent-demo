# Report development lifecycle

The standard path for every report in this repo. It mirrors the Skills-for-Fabric pattern
(*discover → build data → Gold → Direct Lake model → report → verify*) adapted for a
local-first PBIP workflow where source data is pasted into the report folder.

```
 1. Scope ──► 2. Dataset contract ──► 3. Paste data ──► 4. Profile ──► 5. Model
                                                                          │
 8. Publish ◄── 7. Validate & review ◄── 6. Report ◄──────────────────────┘
```

## 1. Scope the report
Capture audience, decisions to support, KPIs, and the grain of analysis in `docs/report-spec.md`.
Decide the sub-domain and the dimensions/facts needed.

## 2. Write the dataset contract
In `dataset/DATASET-CONTRACT.md`, list every file the report needs and its columns
(`snake_case`, with type and meaning). This is the agreement between whoever produces the data
and the model. The proposed star schema in `docs/model-design.md` is built from this.

## 3. Paste the data
Drop the source files into `dataset/raw/` (and a trimmed copy into `dataset/samples/` for
profiling / CI). Files must match the contract's names and columns.

## 4. Profile the data ← do this before finalizing the model
Profile each pasted file: row count, column list & types, distinct counts / cardinality, null
rates, date ranges, and referential integrity of keys against dimensions. Reconcile any
differences against the contract and the proposed model. **Profiling before modeling prevents
rework** — column names, grain, and cardinality drive relationships, data types, and which cuts
are worth visualizing.

Useful here: the `semantic-models:power-query` skill (preview partition data) and
`pbir model -q "EVALUATE ..."` once the model loads.

## 5. Build / reconcile the semantic model (TMDL)
- Replace each table's placeholder `#table(...)` partition with a real Power Query over the pasted
  file (point it at the `DataFolder` parameter).
- Confirm column names/types match the profiled data; fix `sourceColumn` mappings.
- Verify relationships resolve (no blank/many-to-many surprises), measures evaluate, and the
  `Date` table covers the data's range.
- Validate: `tmdl-validate <file>.tmdl`.

## 6. Build the report (PBIR)
- Finalize the pages defined in `docs/report-spec.md`, building each page's visuals in the standard
  order (KPI cards → trend → breakdown → detail).
- Bind visuals to measures (never raw columns). Format through the shared theme.

## 7. Validate & review
- `python validate_pbip.py reports/NN-name/pbip --no-pbir-cli` → 0 errors.
- `jq empty` on every PBIR JSON.
- Optional: `semantic-models:review-semantic-model` and `reports:review-report` skills.

## 8. Publish to Fabric (optional)
Publish the semantic model + report to a Fabric workspace (see `docs/fabric-environment.md`).
Prefer Direct Lake when the data already lives in a Fabric Gold lakehouse; the local pasted-CSV
path is for prototyping and demos.
