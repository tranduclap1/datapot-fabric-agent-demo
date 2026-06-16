# CLAUDE.md — how to work in this repo

This is a **multi-report Power BI / Microsoft Fabric monorepo**. Read this before making changes.

## What this repo is

- A place to develop **many** Power BI reports over time, one self-contained folder each under `reports/`.
- Each report carries its own **dataset**, **documents**, and **data dictionary** (see [README.md](README.md)).
- Power BI artifacts use **PBIP** format (TMDL semantic model + PBIR report) so everything is diff-able and reviewable.

## Tooling available (installed plugins)

- **skills-for-fabric** (`@fabric-collection`) — Fabric data-plane work: workspaces, lakehouses,
  warehouses, notebooks, pipelines, semantic-model authoring/consumption via REST. Orchestrator
  agents: `FabricDataEngineer`, `FabricIQ`, `FabricAdmin`, `FabricAppDev`. Use for anything that
  touches a Fabric workspace or the medallion (Bronze/Silver/Gold) layers feeding a report.
- **power-bi-agentic-development** — local PBIP development:
  - `pbip` / `tmdl` / `pbir-format` skills — file-format authoring and validation.
  - `semantic-models` skills — modeling, DAX, Power Query, reviews.
  - `reports` skills — `create-pbi-report`, `pbir-cli`, theme, visuals, report review.

When a task spans Fabric (data engineering) **and** Power BI (modeling/report), follow the
skills-for-fabric workflow: discover → build data → **Gold lakehouse → Direct Lake semantic model → report**.

## Golden rules

1. **Never hardcode** workspace IDs, item IDs, connection strings, or absolute paths in committed
   artifacts. Parameterize (see the `DataFolder` parameter in the semantic model) and discover IDs at runtime.
2. **One report = one folder** under `reports/NN-name/`. Don't share datasets or models across report folders
   unless a model is explicitly promoted to `shared/`.
3. **Profile data before modeling.** A model authored from domain assumptions is a *proposal*. Once real
   (or sample) data is pasted into `dataset/raw/`, profile it (row counts, distinct values, types, nulls,
   date ranges) and reconcile the TMDL partitions + dictionary before building the full report.
4. **Measures, not implicit aggregation.** All report numbers come from explicit measures on the
   `Key Measures` table. Fact columns stay hidden.
5. **Validate after every PBIP change** (see below). Keep `validate_pbip.py` at 0 errors.
6. **Format via the theme**, not per-visual overrides. The shared theme is `shared/themes/datapot-theme.json`.

## Validating Power BI artifacts

```bash
tmdl-validate <file>.tmdl                                   # structural TMDL lint
python validate_pbip.py reports/NN-name/pbip --no-pbir-cli  # whole-project check
jq empty <file>.json                                        # JSON syntax for PBIR files
```

PBIP page rule: **`pages.json` entry == page folder name == `page.json` `name`** — and page/visual
folder names must be **word characters or hyphens only (NO `.Page`/`.Visual` dotted suffix, no spaces)**.
Power BI Desktop *silently ignores* non-compliant folders → "The report has no pages" — even though
`pbir validate`/`validate_pbip` accept the `.Page` suffix. (Folders `Overview/`, `Overview/visuals/Title/`,
NOT `Overview.Page/`, `Title.Visual/`.) The `.Report`/`.SemanticModel` *item* folders DO keep their suffix —
that rule is only for page/visual/bookmark folders inside `definition/`.

### TMDL authoring rules (Power BI Desktop's parser is stricter than `tmdl-validate`)

The bundled `tmdl-validate` linter is lightweight and misses several rules that Power BI Desktop
**rejects** with `TMDL Format Error: InvalidLineType`. Hand-authored TMDL must obey:
- **No `//` comments at the TMDL root** — Desktop reports line type `Other`. (M `//` comments *inside*
  a `source =` query body are fine; they're part of the captured expression.)
- **`///` descriptions must be immediately followed by the declaration** they describe — never a blank
  line, never EOF, never a `//`. A blank after `///` → `InvalidLineType: Empty`.
- A block **header is immediately followed by a property or child**, not a blank line
  (e.g. `table X` then `column …`, or `table X` then `lineageTag: …`).
- **LF line endings, UTF-8, no BOM** — match the exported reference; a BOM breaks line 1.
- Quote names with spaces/parens/`%`/leading digit: `'Amount (VND)'`, `'NPS Surveys'`, `'Digital Adoption %'`.
- Tabs only for indentation. To keep model docs, put descriptions in the data dictionary, not inline.

After any TMDL edit, run `tmdl-validate` **and** re-read for the rules above (the linter won't catch them).

### PBIR (report.json) rules Desktop enforces but `validate_pbip` does not

- **`themeCollection.baseTheme` and `.customTheme` each REQUIRE `reportVersionAtImport`**
  (`{ "visual": "...", "report": "...", "page": "..." }`). Omitting it makes Desktop reject
  `report.json`, which **cascades** to "ActivePageName not found" / "The report has no pages"
  even though the pages are fine. Keep the property present.
- Any referenced theme file (base + custom) must exist under `StaticResources/`.
- Page wiring must be self-consistent: every `pages.json` `pageOrder` entry has a matching
  `<slug>.Page` folder whose `page.json` `name` equals the slug; `activePageName` ∈ `pageOrder`.

## ⚠️ Known environment caveat (non-ASCII path)

This clone is under `C:\Users\HoàngTôMạnh\...`. The pbip plugin's PostToolUse hooks
(`validate-pbir.sh`, `validate-report-binding.sh`) can fail to read files on paths with
diacritics and then report **false** "missing required fields / no byPath" errors when you
`Write`/`Edit` `.pbir` or `.Report/**` files.

- These are **false positives** — the file is fine if `jq empty <file>` passes and
  `validate_pbip.py` reports 0 errors.
- The hooks are **not** registered for the `Bash` tool, so writing PBIR JSON via a shell heredoc
  avoids the noise. TMDL files validate correctly either way.
- To silence globally (affects all repos), set `all_hooks_enabled: false` in the pbip plugin's
  `hooks/config.yaml`. Leave enabled unless the noise blocks you.

## Report lifecycle

See [docs/report-lifecycle.md](docs/report-lifecycle.md) and [CONVENTIONS.md](CONVENTIONS.md).
