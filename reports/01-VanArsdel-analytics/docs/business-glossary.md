# Business glossary — VanArsdel Sales Analytics

Business concepts and terminology for this report. **Field-level** (table / column / measure) definitions
live in [`../dictionary/data-dictionary.md`](../dictionary/data-dictionary.md) and
[`../dictionary/measures-list.md`](../dictionary/measures-list.md); this file defines the **business
meaning** behind the report's terms. Currency = **USD**. All data is synthetic (the classic VanArsdel
sales sample).

## Products & catalogue

| Term | Definition |
|------|------------|
| **Product** | A single VanArsdel item offered for sale (e.g. *VanArsdel Maximus UE-44*), identified by a Product ID. 212 products across 173 distinct names. |
| **Manufacturer** | The maker of a product. In this dataset it is **VanArsdel only** — there are no competitors, so no market-share analysis is possible. |
| **Category** | The top-level product grouping — Urban, Mix, Accessory, Youth, Rural. |
| **Segment** | The positioning sub-grouping — Convenience, Moderation, Extreme, Productivity, Accessory, All Season, Youth, Regular, Select. |
| **Category-Segment** | A Category + Segment combination (e.g. *Urban-Convenience*) — the conformed key shared by products and the budget plan. |
| **Product Group** | The conformed dimension of Category-Segment combinations, so a single Category/Segment slicer filters **both** actual sales and the budget plan. Products span 10 combos; the plan covers 9 (*Rural-Productivity* has no budget). |
| **Unit Price** | The product's list selling price per unit (USD). Drives Revenue. |
| **Unit Cost** | The product's list cost per unit (USD). Drives Cost. |

## Sales & volume

| Term | Definition |
|------|------------|
| **Sale** | One unit-level transaction line — a product bought by a customer on a date through a campaign. The Sales fact holds ~675K rows. |
| **Units / Units Sold** | The quantity sold. Here `Units` is **always 1**, so units sold = the number of sales rows; true order quantities greater than 1 are not represented. |
| **Distinct Customers** | The number of unique customers who purchased in the current context. |
| **Distinct Products Sold** | The number of unique products with at least one sale. |
| **Average Selling Price (ASP)** | Revenue ÷ units sold — the average realized price per unit. |

## Revenue & profit

| Term | Definition |
|------|------------|
| **Revenue** | **List-price (gross) sales** — units × the product's list Unit Price. There are **no** discounts, returns, or tax in the data, so this is gross, not net, revenue. |
| **Cost (COGS)** | Cost of goods sold — units × the product's list Unit Cost. |
| **Gross Margin** | Revenue − Cost; profit before operating expenses. |
| **Gross Margin %** | Gross Margin ÷ Revenue. In this sample it sits at a near-constant **~27%** because Unit Cost is a fixed ratio of Unit Price — a data artifact, not a real pricing pattern. |
| **Revenue per Customer** | Revenue ÷ distinct customers — average spend per purchasing customer. |

## Budget & forecast

| Term | Definition |
|------|------------|
| **Budget** | The planned revenue target for a period (plan rows of type *Budget*). Covers 2020 and 2021. |
| **Forecast** | The latest expected revenue (plan rows of type *Forecast*). Covers 2021. |
| **Plan** | Collective term for the Budget + Forecast figures (the `Budget` fact). |
| **Revenue vs Budget** | Actual revenue − Budget (the variance amount); the % form expresses it as a share of budget. |
| **Budget Attainment %** | Actual revenue ÷ Budget; 100% = exactly on plan. |
| **Like-for-like window** | The only period where actuals and budget overlap — **Jan–Jun 2020**. Budget and attainment comparisons are valid only here; over the full range they mislead (≈6 years of actuals vs ~6 months of budget). |

## Marketing & channel

| Term | Definition |
|------|------------|
| **Campaign** | The marketing source of a sale, defined as a **Traffic Channel × Device** combination (22 campaigns). |
| **Traffic Channel** | The acquisition route — Organic Search, SEO, Banner, Affiliate, SEM, Email, SMO, Mail. |
| **Device** | The device used — Mobile, Tablet, Desktop, or Paper. (The raw typo `Deskop` is corrected to Desktop in Power Query.) |
| **Channel Type** | A derived classification: **Digital** if the device is Desktop / Mobile / Tablet, otherwise **Offline** (Paper). |
| **Digital Revenue** | Revenue attributed to digital devices (Desktop / Mobile / Tablet); the % form is its share of total revenue. |

## Customer & geography

| Term | Definition |
|------|------------|
| **Customer** | An individual buyer, identified by a zero-padded Customer ID (~282K customers). The dimension is **thin** — name, synthetic email, and location only; no age, gender, segment, or tenure. |
| **Region** | The top geographic grouping — East, Central, West (USA only). |
| **State / District / City** | The geographic hierarchy below Region (49 states, 39 districts; City is a `City, ST, USA` label). |
| **ZIP** | Postal code; the key linking a customer to geography (snowflake: Sales → Customer → Geo). |
| **Currency** | All monetary amounts are **USD**, shown as whole dollars (ASP to cents). Values are unitless synthetic sample figures. |

## Time

| Term | Definition |
|------|------------|
| **Grain** | The level of detail of one fact row — *unit × product × customer × campaign × date* for Sales, and *type × month × Category-Segment* for the plan. |
| **PY (Prior Year)** | The same period one year earlier — the basis for year-over-year comparison. |
| **YoY** | Year over year — change vs the same period last year (amount or %). |
| **YTD** | Year to date — accumulated from January 1 to the period in context. |
| **MoM** | Month over month — change vs the previous month. |
