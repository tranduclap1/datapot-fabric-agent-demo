# Changelog

All notable changes to this repository are documented here.
Format based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
This project is pre-1.0 and research-grade — expect breaking changes.

## [Unreleased]
### Added
- **Report 01 — VanArsdel Sales Analytics** (manufacturing &amp; retail, 100% synthetic VanArsdel sample):
  full Part 1 deliverables (business glossary, data dictionary + CSV, data gaps &amp; questions) and Part 2
  deliverables (BRD, report spec, wireframe), plus data profile, model design, dataset contract, and build log.
- TMDL semantic model — 2 facts (`Sales`, `Budget`), 6 dimensions (incl. a calculated 2015–2021 calendar,
  a conformed `Product Group`, and a snowflaked `Geo`), 8 relationships, 20 explicit measures.
- PBIR report — 5 pages, 43 visuals, Datapot theme applied; Budget-vs-Actual page-filtered to the
  Jan–Jun 2020 actuals∕budget overlap.
- Engine-level validation (Power BI TOM parse + a Power Query refresh replay) in place of the absent CLIs.
- Build recap for the VanArsdel build at `docs/session-recap.html` (served via GitHub Pages, with the
  `docs/index.html` landing redirect).
- Community-health files: LICENSE (MIT), CONTRIBUTING, CODE_OF_CONDUCT, SECURITY, issue/PR templates,
  CHANGELOG, and `.gitattributes` (LF enforcement for PBIP text).

### Changed
- Repo-wide docs (`README.md`, `CONVENTIONS.md`, `docs/report-lifecycle.md`, templates) made
  domain-neutral / VanArsdel-focused.
- `docs/session-recap.html` rebuilt for the VanArsdel build, in the Datapot Design System.

### Removed
- The previous **Branch &amp; Channel Performance** (banking) report and all of its artifacts.
- Banking-specific repo-wide docs: the shared business glossary (`docs/business-glossary.md`) and the
  banking star-schema diagram (`docs/assets/star-schema.svg`).

## [0.1.0] - 2026-06-16
### Added
- Initial public release: multi-report PBIP monorepo scaffold.
- Repo-wide docs, conventions, shared Datapot theme and templates, and a session-recap framework.
