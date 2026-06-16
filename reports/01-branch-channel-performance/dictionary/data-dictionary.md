# Data dictionary — Branch & Channel Performance

Lock-step with the TMDL model in `../pbip/BranchChannelPerformance.SemanticModel`. Currency = **VND**.
Hidden columns are not shown to report users (keys, raw fact measures). `★` = table key.

## Dimensions

### `Date`  (DimDate sheet · marked as Date table)
| Model column | Source | Type | Hidden | Description |
|---|---|---|---|---|
| Date ★ | Date | dateTime | no | Calendar date (relationship key) |
| Year | Year | int64 | no | Calendar year |
| Quarter | Quarter | string | no | Q1–Q4 |
| Month | Month | int64 | yes | Month number (sort key) |
| Month Name | MonthName | string | no | Sorted by Month |
| Month Year | MonthYear | string | no | `YYYY-MM` (chronological text — trend axis) |
| Day Of Week | DayOfWeek | string | no | Monday…Sunday |
| Is Weekend | IsWeekend | boolean | no | Weekend flag |

### `Branch`  (DimBranch sheet · 50 rows)
| Model column | Source | Type | Hidden | Description |
|---|---|---|---|---|
| Branch ID ★ | BranchID | string | yes | B001–B050 |
| Branch Name | BranchName | string | no | e.g. "CN Long Biên" |
| City | City | string | no | Trimmed (source had trailing spaces) |
| District | District | string | no | |
| Region | Region | string | no | Bắc / Trung / Nam |
| Branch Type | BranchType | string | no | Digital / Hub / Spoke |
| Open Date | OpenDate | dateTime | no | Branch opening date |
| Manager | Manager | string | no | Branch manager |

### `Product`  (DimProduct sheet · 24 rows)
| Model column | Source | Type | Hidden | Description |
|---|---|---|---|---|
| Product ID ★ | ProductID | string | yes | |
| Product Name | ProductName | string | no | |
| Product Category | ProductCategory | string | no | Card / Deposit / Loan / Wealth |
| Product Sub Category | ProductSubCategory | string | no | e.g. CA, SA, TD |
| Base Rate | BaseRate | double (%) | no | Product base rate |

### `Customer`  (DimCustomer sheet · 5,000 rows)
| Model column | Source | Type | Hidden | Description |
|---|---|---|---|---|
| Customer ID ★ | CustomerID | string | yes | |
| Full Name | FullName | string | no | |
| Gender | Gender | string | no | Nam / Nữ |
| Date Of Birth | DateOfBirth | dateTime | yes | |
| Age | Age | int64 | no | |
| Occupation | Occupation | string | no | |
| City | City | string | no | |
| Home Branch ID | HomeBranchID | string | yes | Not related to Branch (avoids ambiguity) |
| Segment | Segment | string | no | Mass / Affluent / Priority / Private |
| Join Date | JoinDate | dateTime | no | |
| KYC Status | KYCStatus | string | no | Verified / Pending / Re-KYC Required |

### `Channel`  (derived from Transactions · 6 rows)
| Model column | Source | Type | Hidden | Description |
|---|---|---|---|---|
| Channel ID ★ | ChannelID | string | yes | CH1–CH6 |
| Channel Name | ChannelName | string | no | ATM, Internet/Mobile/Phone Banking, Quầy giao dịch, Đại lý |
| Channel Type | ChannelType (derived) | string | no | Digital / Self-Service / Assisted / Branch |
| Is Digital | IsDigital (derived) | boolean | no | TRUE for Internet & Mobile Banking |

### `Transaction Type`  (derived from Transactions · 8 rows)
| Model column | Source | Type | Hidden | Description |
|---|---|---|---|---|
| Transaction Type ID ★ | TransactionTypeID | string | yes | T01–T08 |
| Transaction Type Name | TransactionTypeName | string | no | Vietnamese type name |
| Transaction Category | TransactionCategory (derived) | string | no | Cash / Transfer / Payment / Income / Loan Repayment |

## Facts

### `Transactions`  (Transactions/*.csv · 1 row per transaction)
| Model column | Source | Type | Hidden | Description |
|---|---|---|---|---|
| Transaction ID | TransactionID | string | yes | |
| Date | Date | dateTime | yes | → Date |
| Customer ID | CustomerID | string | yes | → Customer |
| Product ID | ProductID | string | yes | → Product |
| Branch ID | BranchID | string | yes | → Branch |
| Channel ID | ChannelID | string | yes | → Channel |
| Channel Name | ChannelName | string | yes | denormalized (use Channel dim) |
| Transaction Type ID | TransactionTypeID | string | yes | → Transaction Type |
| Transaction Type Name | TransactionTypeName | string | yes | denormalized |
| Direction | Direction | string | no | Credit / Debit |
| Amount (VND) | Amount_VND | int64 | yes | signed; use Inflow/Outflow measures |
| Fee (VND) | Fee_VND | int64 | yes | fee charged |
| Service Time (min) | ServiceTimeMinutes | double | yes | may be blank |

### `NPS Surveys`  (NPSSurveys/*.csv · 1 row per survey)
| Model column | Source | Type | Hidden | Description |
|---|---|---|---|---|
| Survey ID | SurveyID | string | yes | |
| Survey Date | SurveyDate | dateTime | yes | → Date |
| Customer ID | CustomerID | string | yes | → Customer |
| Branch ID | BranchID | string | yes | → Branch |
| Score | Score | int64 | yes | 0–10 |
| NPS Category | NPSCategory | string | no | Promoter / Passive / Detractor |
| Primary Reason ID | PrimaryReasonID | string | yes | R01–R08 |
| Primary Reason | PrimaryReason | string | no | reason text |

### `Branch Targets`  (BranchTargets.xlsx, unpivoted · 1 row per branch × month × target type)
| Model column | Source | Type | Hidden | Description |
|---|---|---|---|---|
| Branch ID | BranchID | string | yes | → Branch |
| Target Type | TargetType | string | no | Total Revenue (VND) / CASA Balance (VND) |
| Month | Month (from `YYYY-MM`) | dateTime | yes | → Date (1st of month) |
| Target Value | TargetValue | double | yes | monthly target amount |

## Measures  (all on `Key Measures`)

| Measure | Folder | Format | DAX |
|---|---|---|---|
| Total Transactions | 01 Volume | #,##0 | `COUNTROWS(Transactions)` |
| Active Customers | 01 Volume | #,##0 | `DISTINCTCOUNT(Transactions[Customer ID])` |
| Total Inflow | 01 Volume | #,##0 | `CALCULATE(SUM(Transactions[Amount (VND)]), Transactions[Direction]="Credit")` |
| Total Outflow | 01 Volume | #,##0 | `CALCULATE(-1*SUM(Transactions[Amount (VND)]), Transactions[Direction]="Debit")` |
| Net Flow | 01 Volume | #,##0 | `[Total Inflow] - [Total Outflow]` |
| Gross Transaction Value | 01 Volume | #,##0 | `[Total Inflow] + [Total Outflow]` |
| Avg Transaction Size | 01 Volume | #,##0 | `DIVIDE([Gross Transaction Value],[Total Transactions])` |
| Fee Revenue | 02 Revenue & Targets | #,##0 | `SUM(Transactions[Fee (VND)])` |
| Revenue Target | 02 Revenue & Targets | #,##0 | `CALCULATE(SUM('Branch Targets'[Target Value]),'Branch Targets'[Target Type]="Total Revenue (VND)")` |
| CASA Balance Target | 02 Revenue & Targets | #,##0 | `CALCULATE(SUM('Branch Targets'[Target Value]),'Branch Targets'[Target Type]="CASA Balance (VND)")` |
| Avg Service Time (min) | 03 Service & Quality | #,##0.0 | `AVERAGE(Transactions[Service Time (min)])` |
| Survey Responses | 03 Service & Quality | #,##0 | `COUNTROWS('NPS Surveys')` |
| Promoters *(hidden)* | 03 Service & Quality | #,##0 | `CALCULATE(COUNTROWS('NPS Surveys'),'NPS Surveys'[NPS Category]="Promoter")` |
| Detractors *(hidden)* | 03 Service & Quality | #,##0 | `CALCULATE(COUNTROWS('NPS Surveys'),'NPS Surveys'[NPS Category]="Detractor")` |
| NPS | 03 Service & Quality | #,##0 | `DIVIDE([Promoters]-[Detractors],[Survey Responses])*100` |
| Avg NPS Score | 03 Service & Quality | 0.00 | `AVERAGE('NPS Surveys'[Score])` |
| Digital Transactions | 04 Digital | #,##0 | `CALCULATE([Total Transactions], Channel[Is Digital]=TRUE)` |
| Digital Adoption % | 04 Digital | 0.0% | `DIVIDE([Digital Transactions],[Total Transactions])` |
| Transactions PM | 05 Time Intelligence | #,##0 | `CALCULATE([Total Transactions], DATEADD('Date'[Date],-1,MONTH))` |
| Transactions MoM % | 05 Time Intelligence | 0.0% | `DIVIDE([Total Transactions]-[Transactions PM],[Transactions PM])` |
| Transactions YTD | 05 Time Intelligence | #,##0 | `TOTALYTD([Total Transactions],'Date'[Date])` |
| Fee Revenue YTD | 05 Time Intelligence | #,##0 | `TOTALYTD([Fee Revenue],'Date'[Date])` |
