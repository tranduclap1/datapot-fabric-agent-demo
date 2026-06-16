# Data profile — Branch & Channel Performance

Profiling of the data pasted into `dataset/raw/` (BankDIAD sample). One representative file per
table was profiled; monthly partitions span **2023-01 → 2025-12** (36 months).

## Files found

| Folder / file | Partitions | Grain | Feeds |
|---------------|-----------|-------|-------|
| `Transactions/transactions_YYYY-MM.csv` | 36 (~8,850 rows/mo) | one transaction | **Report #1** — `Transactions` fact |
| `NPSSurveys/nps_YYYY-MM.csv` | 36 (~580 rows/mo) | one survey response | **Report #1** — `NPS Surveys` fact |
| `Targets/BranchTargets.xlsx` (MonthlyTargets) | 1 sheet (100 rows, wide) | branch × target type × 36 months | **Report #1** — `Branch Targets` fact (unpivoted) |
| `Dimensions/BankDIAD_Dimensions.xlsx` | sheets: DimCustomer, DimBranch, DimProduct, DimDate | dimension rows | **Report #1** — dimensions |
| `Dimensions/Accounts.csv` | 1 (15,200 rows) | one account | future reports (account dim) |
| `AccountSnapshots/snapshot_YYYY-MM.csv` | 36 (10–20k rows/mo) | account balance @ month-end | **future #2** (Deposits & Balances) |
| `CustomerProductMonth/cpm_YYYY-MM.csv` | 36 (~16k rows/mo) | customer × product @ month-end | **future #2/#3** |
| `LoanOriginations/origination_YYYY-MM.csv` | 36 (tens of rows/mo) | one new loan | **future #3** (Lending / Credit Risk) |

## Key column facts (from profiling)

- **Transactions** — `TransactionID, Date, CustomerID, ProductID, BranchID, ChannelID, ChannelName,
  TransactionTypeID, TransactionTypeName, Direction, Amount_VND, Fee_VND, ServiceTimeMinutes`.
  - 50 branches (`B001`–`B050`), **6 channels** `CH1`–`CH6` = ATM, Internet Banking, Mobile Banking,
    Phone Banking, Quầy giao dịch (counter), Đại lý (agent).
  - **8 transaction types** `T01`–`T08` (cash deposit/withdrawal, internal/interbank transfer,
    bill payment, card purchase, salary, loan repayment).
  - `Direction` ∈ {Credit, Debit}. **`Amount_VND` is signed** (debits negative). `Fee_VND` ≥ 0.
  - `ServiceTimeMinutes` is a decimal and is **frequently blank** (only relevant for some channels).
- **NPSSurveys** — `SurveyID, SurveyDate, CustomerID, BranchID, Score (0–10), NPSCategory
  (Promoter/Passive/Detractor), PrimaryReasonID (R01–R08), PrimaryReason`. **No channel** in NPS data.
- **DimBranch** (50) — `BranchID, BranchName, City, District, Region (Bắc/Trung/Nam),
  BranchType (Digital/Hub/Spoke), OpenDate, Manager`.
- **DimProduct** (24) — `ProductID, ProductName, ProductCategory (Card/Deposit/Loan/Wealth),
  ProductSubCategory, BaseRate`.
- **DimCustomer** (5,000) — `CustomerID, FullName, Gender, DateOfBirth, Age, Occupation, City,
  HomeBranchID, Segment (Mass/Affluent/Priority/Private), JoinDate, KYCStatus`.
- **DimDate** (1,096) — `Date, Year, Quarter, Month, MonthName, MonthYear, DayOfWeek, IsWeekend`.
- **BranchTargets** — wide: `BranchID, BranchName, TargetType ∈ {Total Revenue (VND),
  CASA Balance (VND)}`, then one column per month `2023-01 … 2025-12`.

## Data-quality findings (handled / to watch)

| Finding | Where | Action |
|---------|-------|--------|
| Trailing spaces in city names (`"Hải Phòng "`, `"TP. Hồ Chí Minh "`) | DimBranch | `Text.Trim` applied in the Branch query |
| `Amount_VND` is signed | Transactions | Use `Total Inflow` / `Total Outflow` measures, not a raw SUM |
| `ServiceTimeMinutes` blank for many rows | Transactions | `Avg Service Time` uses AVERAGE (ignores blanks); interpret per channel |
| No `DimChannel` / `DimTransactionType` sheet | — | Derived as dimensions from distinct values in Transactions |
| Targets are monthly & wide | BranchTargets | Unpivoted; related to Date on the 1st of each month (use at month grain+) |
| "Total Revenue (VND)" target ≫ transaction fees | BranchTargets vs Transactions | Revenue actual is **not** in the transaction data (fees only). Target exposed as context; actual-vs-target left as an open question — see `model-design.md`. |
