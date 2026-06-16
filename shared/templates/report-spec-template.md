# Report spec — <Report Name>

## Purpose & audience
- **Audience:** <roles>
- **Decisions supported:** <what actions this report drives>
- **Refresh / latency:** <cadence>

## Key questions
1. <question the report must answer>
2. …

## KPIs
| KPI | Measure | Target / benchmark |
|-----|---------|--------------------|
| <kpi> | `[Measure]` | <target> |

## Grain & dimensions
- **Fact grain:** <day × … >
- **Dimensions / slicers:** <Date, …>

## Pages
### Page 1 — <name>
- Purpose: <…>
- Visuals: <KPI cards: …; trend: …; breakdown: …; detail: …>

### Page 2 — <name>
- …

## Filters
- Report-level: <…>
- Page-level: <…>

## Design notes
- Theme: `shared/themes/datapot-theme.json`. Page 1280×720. Format via theme, not per-visual.
