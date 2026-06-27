# datapot-fabric-agent-demo

### Power BI reports, built by an AI agent — as reviewable code, not clicks.

[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
![Status](https://img.shields.io/badge/status-research%20%26%20educational-orange)
![Built with Claude Code](https://img.shields.io/badge/built%20with-Claude%20Code-d97757)
![Power BI PBIP](https://img.shields.io/badge/Power%20BI-PBIP%20(TMDL%2BPBIR)-F2C811)
![Microsoft Fabric](https://img.shields.io/badge/Microsoft-Fabric-0078D4)
![Data: 100% synthetic](https://img.shields.io/badge/data-100%25%20synthetic-brightgreen)

A Microsoft Fabric / Power BI monorepo where every semantic model and report is authored by
**Claude Code** in diff-able **PBIP** format (TMDL + PBIR) — then validated, documented, and
version-controlled like software. The current report is **VanArsdel Sales Analytics**, a
manufacturing &amp; retail **sales** dashboard built end-to-end on a 100% synthetic dataset.

📖 **See exactly how it was built** — process, prompts, and cost: **[read the build recap](docs/session-recap.html)** *(open `docs/session-recap.html` in a browser for the full rendered recap).*

> 🔬 **Research &amp; educational use only · 100% synthetic data.** Experimental; **not production-tier**
> and not through Datapot's QA. The bundled "VanArsdel" dataset is entirely synthetic — no real
> people, customers, or company. Don't use the model or numbers for real decisions. No warranty.
> [Full notice ↓](#-data--status-notice)

---

## What it produces

A clean, conventional **star schema** — **6 dimensions**, **2 facts**, and a disconnected measures table:

- **Facts:** `Sales` (one row per unit sold) · `Budget` (Type × Year × Month × Category × Segment).
- **Dimensions:** `Date` (calculated 2015–2021 calendar) · `Product` · `Product Group` (conformed) ·
  `Customer` · `Geo` · `Campaign`.
- **`Key Measures`** — a disconnected table holding **20 explicit measures** (volume · revenue &amp; profit ·
  budget &amp; forecast · time intelligence · marketing &amp; channel). See [model design](reports/01-VanArsdel-analytics/docs/model-design.md).

The report has **five pages** — **Executive Overview**, **Product &amp; Category**, **Geography &amp; Customers**,
**Marketing &amp; Channel**, and **Budget vs Actual** (43 visuals). Open `VanArsdelSales.pbip` to explore them
([Getting Started ↓](#getting-started--open--explore-the-report)), or skim the
[build recap](docs/session-recap.html) for the visuals and story.

## What you'll learn

A worked, end-to-end example of **agentic analytics engineering** on the Microsoft data platform:

- 🧱 **PBIP as code** — how a Power BI model + report look as reviewable TMDL/PBIR files, not a binary `.pbix`.
- ⭐ **Star-schema modeling** — 6 dimensions (incl. a conformed `Product Group` and a snowflaked `Geo`),
  2 facts, hidden surrogate keys, explicit measures only.
- 🧮 **DAX measure design** — 20 measures in display folders (volume · revenue &amp; profit · budget &amp; forecast ·
  time intelligence · marketing &amp; channel).
- 📒 **Spec-driven, documented builds** — dataset contract → data profile → business glossary → data dictionary →
  data gaps → BRD → model design → report spec → wireframe → build log.
- 🤖 **The agent workflow** — the actual process, prompts, and cost ([recap](docs/session-recap.html)).
- 🧰 **Reusable conventions** — folder, naming, TMDL &amp; PBIR rules that keep a multi-report repo consistent ([CONVENTIONS.md](CONVENTIONS.md)).

## Getting Started — open &amp; explore the report

**Prerequisites**
- **Power BI Desktop** (latest; free from the Microsoft Store).
- The **PBIR preview features** enabled (one-time, below) — required, or the report won't open.
- *(Optional)* Python 3 + `openpyxl` — only to profile/regenerate the data; **not** needed to open/refresh the report.

**1. Clone**
```bash
git clone https://github.com/DatapotAnalytics/datapot-fabric-agent-demo.git
cd datapot-fabric-agent-demo
```

**2. Enable the PBIR preview (one-time, required)** — in Power BI Desktop:
*File ▸ Options and settings ▸ Options ▸ Preview features* → tick **Power BI Project (.pbip) save option**
and **Store reports using enhanced metadata format (PBIR)** → **OK** → **restart Desktop**.
*(Skip this and Desktop may refuse to open the project or show "The report has no pages".)*

**3. Open the project** — `reports/01-VanArsdel-analytics/pbip/VanArsdelSales.pbip`

**4. Point the model at your clone (required)** — the data path is parameterized. On the ribbon:
**Home ▸ Transform data ▸ Edit parameters**, set **`DataFolder`** to the absolute path of the
raw-data folder in *your* clone, e.g.
```
C:\Users\<you>\src\datapot-fabric-agent-demo\reports\01-VanArsdel-analytics\dataset\raw
```

**5. Refresh** — **Home ▸ Refresh**. The synthetic data loads — `Sales` actuals **Jan 2015 → Jun 2020**
plus a 2020–2021 budget/forecast plan. It's 100% synthetic, so open and refresh with no privacy concerns.

### The five pages
1. **Executive Overview** — headline KPI cards (Revenue, Gross Margin, Units Sold, Distinct Customers), a Revenue-by-month line with a prior-year overlay, Revenue by category, and Revenue by state. *How are sales, margin and volume trending?*
2. **Product &amp; Category** — a Category › Segment matrix (Revenue, Gross Margin, GM%, Units, ASP), top products by Revenue, and a 100% stacked category mix over time. *Which products &amp; categories drive revenue and margin?*
3. **Geography &amp; Customers** — Revenue by state and region, Distinct Customers &amp; Revenue per Customer, and a Region › State › District matrix. *Where are customers and revenue concentrated?*
4. **Marketing &amp; Channel** — Revenue by Traffic Channel and Device, Digital Revenue %, a Channel × Device matrix, and a digital-share trend. *Which acquisition channels &amp; devices convert to revenue?*
5. **Budget vs Actual** — Actual vs Budget cards and matrix, page-filtered to the **Jan–Jun 2020** actuals∕budget overlap (the only like-for-like window). *How does actual track to plan, where comparable?*

## Repository layout

```
datapot-fabric-agent-demo/
├── README.md                  ← you are here
├── CLAUDE.md                  ← how an agent (or human) should work in this repo
├── CONVENTIONS.md             ← naming, folder, PBIP/TMDL & dictionary standards
├── CONTRIBUTING.md · SECURITY.md · CODE_OF_CONDUCT.md · CHANGELOG.md · LICENSE
├── docs/                      ← repo-wide docs
│   ├── report-lifecycle.md    ← scope → dataset → profile → model → report → deploy
│   ├── fabric-environment.md  ← target workspaces / deployment (fill in per tenant)
│   ├── assets/                ← brand assets (logo, mark)
│   └── session-recap.html     ← how this build actually went (process, prompts, cost)
├── shared/                    ← assets reused by every report
│   ├── themes/                ← the Datapot brand Power BI theme
│   └── templates/             ← starter files for a new report
└── reports/                   ← ALL reports live here, one folder each
    ├── README.md              ← report registry / index
    └── 01-VanArsdel-analytics/
        ├── dataset/  dictionary/  docs/  pbip/   (+ README.md)
```

## The per-report contract

Each report folder always carries these four things — nothing leaks between reports:

| Component      | Folder             | What it is |
|----------------|--------------------|------------|
| **Folder**     | `reports/NN-name/` | One numbered folder per report |
| **Dataset**    | `dataset/`         | Source data + `DATASET-CONTRACT.md` describing the expected files/columns |
| **Document**   | `docs/`            | `report-spec.md`, `model-design.md`, `build-log.md`, `data-profile.md`, plus glossary / gaps / BRD / wireframe |
| **Dictionary** | `dictionary/`      | `data-dictionary.md` + `.csv` — every table, column, and measure defined |

Power BI artifacts (semantic model + report) live in `pbip/` as source-controllable **PBIP**
(TMDL semantic model + PBIR report).

## Reports

| # | Report | Domain | Status |
|---|--------|--------|--------|
| 01 | [VanArsdel Sales Analytics](reports/01-VanArsdel-analytics/README.md) | Manufacturing &amp; Retail | 🟢 Built — 5 pages; opens &amp; refreshes in Power BI Desktop |

New reports are added as self-contained folders over time — ⭐ **star / watch** to follow along.
Legend: 🔴 planned · 🟡 in progress · 🟢 built. Full registry: [reports/README.md](reports/README.md).

## Adding a new report

See [CONVENTIONS.md](CONVENTIONS.md) and [docs/report-lifecycle.md](docs/report-lifecycle.md): copy
`shared/templates/` into a new `reports/NN-name/`, write the dataset contract, paste &amp; **profile**
data, build the TMDL model, then the PBIR report, validate, and (optionally) publish to Fabric.
Contributions welcome — start with [CONTRIBUTING.md](CONTRIBUTING.md).

<details>
<summary><b>Local development &amp; validation (contributors)</b></summary>

```bash
tmdl-validate <file>.tmdl                                   # TMDL structural lint
python validate_pbip.py reports/NN-name/pbip --no-pbir-cli  # whole-project check (0 errors)
jq empty <file>.json                                        # PBIR JSON syntax
```

**Windows / Claude Code note:** if your clone path contains non-ASCII characters or spaces, the
pbip plugin's PostToolUse PBIR hook may mis-read it and emit *false* "missing field" errors on
`.pbir` / `.Report` writes — the files are fine if `jq empty` and `validate_pbip.py` pass. See
[CLAUDE.md](CLAUDE.md) for the full TMDL/PBIR rule set (folder naming, LF/no-BOM, theme metadata).
</details>

## About Datapot

[**Datapot Analytics**](https://datapot.edu.vn) is a Vietnam-based data-analytics training and
consulting company. We help people and teams become fluent in modern analytics — SQL, Power BI,
data modelling, and the Microsoft data platform. This repo is part of our **applied research into
agentic, AI-assisted analytics engineering**, shared openly so the community can learn the workflow
— what works, what breaks, and why.

## Explore &amp; contribute

- ⭐ **Star this repo** if the agentic-PBIP workflow is useful — it helps others find it.
- 🔎 **Explore the report** → [VanArsdel Sales Analytics](reports/01-VanArsdel-analytics/README.md)
- 📖 **Read the build story** → [session recap](docs/session-recap.html)
- 🎓 **Learn analytics with us** → [datapot.edu.vn](https://datapot.edu.vn)
- 💬 **Bug or idea?** → [open an issue](https://github.com/DatapotAnalytics/datapot-fabric-agent-demo/issues/new/choose) · see [CONTRIBUTING.md](CONTRIBUTING.md)

## License

Code is released under the [MIT License](LICENSE). The bundled VanArsdel dataset is 100% synthetic and
free to reuse for learning. Documentation and report content © Datapot Analytics — reuse encouraged
with attribution.

## 🔬 Data &amp; status notice

This repository is an **experimental, research-purpose** project. It is **not production-tier** and
has **not** passed Datapot's own production standards / QA workflow. Structure, models, conventions,
and data may change at any time without notice. Do not rely on this repository, its model, or its
numbers for real decisions. **No warranty; use at your own risk.**

**Data:** the bundled "VanArsdel" dataset is **100% synthetic data** — every customer, product, sale,
and value is fabricated for demonstration. It contains **no real, private, or personal data**,
represents no real customers or company, and violates no one's privacy.
