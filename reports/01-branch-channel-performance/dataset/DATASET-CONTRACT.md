# Dataset contract — Branch & Channel Performance

What `dataset/raw/` must contain. This is the agreement between the data producer and the semantic
model. The model's Power Query partitions read these files relative to the `DataFolder` parameter.
The dataset is the **BankDIAD** sample (monthly partitions, 2023-01 → 2025-12).

> ✅ Status: data is present and the model is bound to it. See [docs/data-profile.md](../docs/data-profile.md).

## Files consumed by THIS report (#1)

### `Transactions/transactions_YYYY-MM.csv`  → fact `Transactions`
| Column | Type | Notes |
|--------|------|-------|
| TransactionID | text | unique |
| Date | date | transaction date |
| CustomerID | text | → DimCustomer |
| ProductID | text | → DimProduct |
| BranchID | text | → DimBranch (B001–B050) |
| ChannelID | text | CH1–CH6 |
| ChannelName | text | ATM, Internet/Mobile/Phone Banking, Quầy giao dịch, Đại lý |
| TransactionTypeID | text | T01–T08 |
| TransactionTypeName | text | Vietnamese type name |
| Direction | text | Credit / Debit |
| Amount_VND | integer | **signed** (debit negative) |
| Fee_VND | integer | ≥ 0 |
| ServiceTimeMinutes | decimal | may be blank |

### `NPSSurveys/nps_YYYY-MM.csv`  → fact `NPS Surveys`
`SurveyID, SurveyDate (date), CustomerID, BranchID, Score (0–10 int), NPSCategory
(Promoter/Passive/Detractor), PrimaryReasonID (R01–R08), PrimaryReason`. No channel column.

### `Targets/BranchTargets.xlsx`, sheet `MonthlyTargets`  → fact `Branch Targets`
Wide format: `BranchID, BranchName, TargetType` + one column per month `2023-01 … 2025-12`.
`TargetType ∈ {Total Revenue (VND), CASA Balance (VND)}`. The model **unpivots** the month columns.

### `Dimensions/BankDIAD_Dimensions.xlsx`  → dimensions
| Sheet | Model table | Key columns |
|-------|-------------|-------------|
| DimBranch | `Branch` | BranchID, BranchName, City, District, Region, BranchType, OpenDate, Manager |
| DimProduct | `Product` | ProductID, ProductName, ProductCategory, ProductSubCategory, BaseRate |
| DimCustomer | `Customer` | CustomerID, FullName, Gender, DateOfBirth, Age, Occupation, City, HomeBranchID, Segment, JoinDate, KYCStatus |
| DimDate | `Date` | Date, Year, Quarter, Month, MonthName, MonthYear, DayOfWeek, IsWeekend |

`Channel` and `Transaction Type` dimensions are **derived** from distinct values in `Transactions`
(no sheet exists for them).

## Files NOT used by this report (reserved for future reports)

| File | Reserved for |
|------|--------------|
| `Dimensions/Accounts.csv` | account dimension (future) |
| `AccountSnapshots/snapshot_YYYY-MM.csv` | **Report #2 — Deposits & Balances** (Balance_VND, NPLGroup) |
| `CustomerProductMonth/cpm_YYYY-MM.csv` | **Report #2/#3** — product holdings / customer 360 |
| `LoanOriginations/origination_YYYY-MM.csv` | **Report #3 — Lending / Credit Risk** (DisbursedAmount, TermMonths, InterestRate, Status) |

## Refresh rules
- Add new monthly files into the matching folder using the same name pattern; the folder-combine
  queries pick them up automatically — no model change needed.
- Keep column names and order stable. If they change, update the matching `*.tmdl` partition and the
  [data dictionary](../dictionary/data-dictionary.md).
- Encoding must be **UTF-8** (Vietnamese text); the queries read with `Encoding = 65001`.
