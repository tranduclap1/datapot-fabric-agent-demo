# Data dictionary — <Report Name>

Keep in lock-step with the TMDL semantic model. One section per table, then a measures section.

## Conventions
- **Model Name** = user-facing name in the model. **Source Column** = physical `snake_case` column pasted in `dataset/raw/`.
- Types: `int64`, `double`, `string`, `dateTime`, `boolean`.

---

## Table: `<Table Name>`  *(dimension | fact)*
- **Grain:** <one row per …>
- **Source file:** `dataset/raw/<file>.csv`

| Model Name | Source Column | Type | Key | Hidden | Description | Example |
|------------|---------------|------|-----|--------|-------------|---------|
| <Name> | <source_col> | <type> | PK/FK/– | yes/no | <meaning> | <example> |

*(repeat per table)*

---

## Measures  *(all on `Key Measures`)*

| Measure | Folder | Format | DAX | Definition |
|---------|--------|--------|-----|------------|
| <Name> | <NN Folder> | <format> | `<DAX>` | <business meaning> |
