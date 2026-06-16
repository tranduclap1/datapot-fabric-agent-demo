# datapot-fabric-agent-demo

> ### 🔬 Under development · research & educational use only
> This repository is an **experimental, research-purpose** project. It is **not production-tier** and
> has **not** passed Datapot's own production standards / QA workflow. Structure, models, conventions,
> and data may change at any time without notice. Do not rely on this repository, its model, or its
> numbers for real decisions. No warranty; use at your own risk.
>
> **Data notice:** the bundled banking dataset (“BankDIAD”) is **100% dummy, AI-generated data**.
> Every name, date of birth, balance, branch, and value is fabricated for demonstration. It contains
> **no real, private, or personal data**, represents no real customers or institution, and violates
> no one's privacy.

A **multi-report Power BI / Microsoft Fabric development monorepo**, built for agent-assisted
development with Claude Code and the *Skills for Fabric* + *Power BI Agentic Development* plugins.

Every report is **self-contained in its own folder** and carries a separate **dataset**,
**document(s)**, and **data dictionary**, so each report is fully reproducible and understandable
on its own.

## About Datapot

[**Datapot Analytics**](https://datapot.edu.vn) is a Vietnam-based data-analytics training and
consulting company. We help people and teams become fluent in modern analytics — SQL, Power BI,
data modelling, and the Microsoft data platform.

This repo is part of Datapot's **applied research into agentic, AI-assisted analytics engineering**:
using coding agents (Claude Code) together with the Microsoft Fabric & Power BI skill ecosystem to
design semantic models and reports as **reviewable, version-controlled code** (PBIP / TMDL / PBIR)
rather than click-through artifacts. We're sharing it openly to help the community learn the
workflow — what works, what breaks, and why. Feedback, issues, and contributions are welcome.

## Repository layout

```
datapot-fabric-agent-demo/
├── README.md                  ← you are here
├── CLAUDE.md                  ← how agents should work in this repo (read first)
├── CONVENTIONS.md             ← naming, folder, PBIP/TMDL & dictionary standards
├── .gitignore
├── docs/                      ← repo-wide documentation
│   ├── report-lifecycle.md    ← the design → dataset → model → report → deploy workflow
│   ├── fabric-environment.md  ← target workspaces, capacity, deployment (fill in per tenant)
│   └── business-glossary.md   ← banking terms shared across all reports
├── shared/                    ← assets reused by every report
│   ├── themes/                ← the Datapot brand Power BI theme
│   └── templates/             ← starter files for a new report
└── reports/                   ← ALL reports live here, one folder each
    ├── README.md              ← report registry / index (status of every report)
    └── 01-branch-channel-performance/   ← Report #1 (banking)
        ├── README.md
        ├── dataset/           ← raw source data you paste (+ a contract describing it)
        ├── dictionary/        ← the data dictionary (markdown + CSV)
        ├── docs/              ← report spec, model design, build log
        └── pbip/              ← the Power BI project: TMDL semantic model + PBIR report
```

## The per-report contract

Each report folder always contains these four things:

| Component        | Folder         | What it is |
|------------------|----------------|------------|
| **Folder**       | `reports/NN-name/` | One numbered folder per report; nothing leaks between reports |
| **Dataset**      | `dataset/`     | The raw source data (you paste it) + `DATASET-CONTRACT.md` describing the expected files/columns |
| **Document**     | `docs/`        | `report-spec.md` (requirements & visuals), `model-design.md` (star schema rationale), `build-log.md` (decisions) |
| **Dictionary**   | `dictionary/`  | `data-dictionary.md` + `data-dictionary.csv` — every table, column, and measure defined |

The Power BI artifacts (semantic model + report) live in `pbip/` in source-controllable
**PBIP** format (TMDL semantic model + PBIR report).

## Reports

| # | Report | Domain | Status |
|---|--------|--------|--------|
| 01 | [Branch & Channel Performance](reports/01-branch-channel-performance/README.md) | Banking | 🟢 All 3 pages built (Overview, Channel Performance, Branch Scorecard); opens in Desktop |

See [reports/README.md](reports/README.md) for the full registry.

## Adding a new report

See [CONVENTIONS.md](CONVENTIONS.md) and [docs/report-lifecycle.md](docs/report-lifecycle.md).
In short: copy `shared/templates/` into a new `reports/NN-name/`, fill in the dataset contract,
paste & profile data, build the TMDL model, then the PBIR report, validate, and publish.

## Validating Power BI artifacts

```bash
# TMDL structural lint (bundled with the pbip plugin)
tmdl-validate <file>.tmdl

# Full PBIP project validation
python validate_pbip.py reports/NN-name/pbip --no-pbir-cli
```

> **Windows note:** this clone lives under a path with non-ASCII characters
> (`HoàngTôMạnh`). The pbip plugin's PostToolUse PBIR hook can mis-read such paths
> and emit *false* "missing field" errors on `.pbir`/`.Report` writes. The files are
> fine — confirm with `jq empty <file>` and `validate_pbip.py`. See [CLAUDE.md](CLAUDE.md).
