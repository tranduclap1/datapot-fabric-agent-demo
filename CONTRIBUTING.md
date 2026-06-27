# Contributing to datapot-fabric-agent-demo

Thanks for your interest! This is a **research / educational** monorepo of agent-built
Power BI / Microsoft Fabric reports in PBIP (TMDL + PBIR) format. It is **not** production-grade
— see the notice in [README.md](README.md).

## Ground rules
- **Synthetic data only.** Never commit real, personal, or client data. Everything here is
  100% AI-generated. PRs that add real data will be rejected.
- **One report = one folder** under `reports/NN-name/`, each with its own dataset, dictionary,
  docs, and PBIP project. Don't share models across report folders unless promoted to `shared/`.
- **No secrets, no machine paths.** Never hardcode workspace IDs, item IDs, connection strings, or
  absolute paths. Parameterize (see the `DataFolder` parameter) and discover IDs at runtime.

## The validation gate — run before every PR
A change is mergeable only when all of these pass (see [CONVENTIONS.md](CONVENTIONS.md) §6):

```bash
tmdl-validate <file>.tmdl                                   # every changed .tmdl
python validate_pbip.py reports/NN-name/pbip --no-pbir-cli  # 0 errors
jq empty <file>.json                                        # every PBIR JSON
```

## Authoring rules tools won't catch (Power BI Desktop is stricter)
- **TMDL must be LF, UTF-8, no BOM.** A `.gitattributes` enforces LF — don't override it.
- **Page & visual folder names = word characters or hyphens only** — NO `.Page` / `.Visual`
  dotted suffix and no spaces. Desktop *silently drops* non-compliant folders → "The report has no pages".
- **`report.json` themes need `reportVersionAtImport`** on both `baseTheme` and `customTheme`.
- **Measures, not implicit aggregation** — all numbers come from explicit measures on
  `Key Measures`; fact columns stay hidden.
- The full rule set lives in [CLAUDE.md](CLAUDE.md).

## Workflow
1. Fork and branch from `main` (e.g. `report-02-inventory`, `fix/theme-contrast`).
2. For a new report, follow [docs/report-lifecycle.md](docs/report-lifecycle.md).
3. Run the validation gate above; keep it at **0 errors**.
4. Update the registry in [reports/README.md](reports/README.md) for new reports.
5. Open a PR (the template will prompt you) describing what you built and how you validated it.

## Ways to help
- Propose a new report (open a *New report proposal* issue).
- Improve docs, the shared theme, or conventions.
- Report a bug in a model/report or the tooling.

## Issues & security
Use the issue templates. For data-exposure concerns (a value that looks real, a leaked secret),
see [SECURITY.md](SECURITY.md) — please do **not** open a public issue for those.

By contributing, you agree your contributions are licensed under the repo's [LICENSE](LICENSE).
