# Model design — Branch & Channel Performance

Semantic model: `pbip/BranchChannelPerformance.SemanticModel` (TMDL, Import). Reconciled to the
profiled BankDIAD data — see [data-profile.md](data-profile.md).

## Star schema

```
                 ┌──────────┐   ┌──────────┐   ┌──────────────────┐
                 │  Branch  │   │ Customer │   │ Transaction Type │
                 └────┬─────┘   └────┬─────┘   └────────┬─────────┘
   ┌──────────┐       │              │                  │       ┌──────────┐
   │   Date   │───────┼──────────────┼──────────────────┼───────│ Channel  │
   └────┬─────┘       │              │                  │       └────┬─────┘
        │        ┌────┴──────────────┴──────────────────┴────────────┴───┐
        ├────────│                    Transactions  (fact)                │
        │        └────────────────────────────────────────────────────────┘
        │        ┌────────────────────────┐     ┌──────────┐
        ├────────│   NPS Surveys (fact)    │─────│ Customer │ (also Branch)
        │        └────────────────────────┘     └──────────┘
        │        ┌────────────────────────┐
        └────────│  Branch Targets (fact)  │───── Branch
                 └────────────────────────┘
            ┌────────────┐   ┌──────────┐
            │ Key Measures│   │ Product  │── Transactions
            │(disconnected)│  └──────────┘
            └────────────┘
```

- **Facts:** `Transactions` (transaction grain), `NPS Surveys` (survey grain), `Branch Targets`
  (branch × month × target type, unpivoted).
- **Dimensions:** `Date` (marked Date table), `Branch`, `Channel`, `Transaction Type`, `Product`,
  `Customer`. `Channel` and `Transaction Type` are **derived from `Transactions`** (no source sheet).
- **`Key Measures`** is a disconnected calculated table that holds all 22 measures.

## Relationships (all single-direction, many-to-one)

| From (many) | To (one) |
|---|---|
| Transactions[Date] | Date[Date] |
| Transactions[Branch ID] | Branch[Branch ID] |
| Transactions[Channel ID] | Channel[Channel ID] |
| Transactions[Transaction Type ID] | Transaction Type[Transaction Type ID] |
| Transactions[Product ID] | Product[Product ID] |
| Transactions[Customer ID] | Customer[Customer ID] |
| NPS Surveys[Survey Date] | Date[Date] |
| NPS Surveys[Branch ID] | Branch[Branch ID] |
| NPS Surveys[Customer ID] | Customer[Customer ID] |
| Branch Targets[Branch ID] | Branch[Branch ID] |
| Branch Targets[Month] | Date[Date] |

**Customer is *not* related to Branch** even though `HomeBranchID` exists — that would create a second
filter path Branch → Customer → Transactions alongside Branch → Transactions (ambiguity). `HomeBranchID`
stays a (hidden) attribute.

## Design decisions

- **Import + folder combine.** Monthly CSVs are combined with `Folder.Files` + `PromoteHeaders`, so new
  months are picked up automatically. UTF-8 (`Encoding = 65001`) preserves Vietnamese text.
- **Signed amounts.** `Amount_VND` is negative for debits. Raw `SUM` would net to ~0 and mislead, so the
  model exposes `Total Inflow`, `Total Outflow` (sign-flipped to positive), `Net Flow`, and
  `Gross Transaction Value` instead of a raw amount measure.
- **Derived Channel/Transaction Type dims.** Built from `Table.Distinct` over `Transactions` so the
  CHx↔name and Tx↔name pairings come from the data; `Channel Type`/`Is Digital`/`Transaction Category`
  are mapped from the (known) names.
- **Targets unpivoted & dated to month start.** The wide target sheet is unpivoted to a long fact and
  joined to `Date` on the 1st of each month. Aggregations at **month grain or coarser** are correct
  (each month contributes one target row); a single-day filter will show no target — expected for
  monthly targets.
- **Date table from source.** Uses the supplied `DimDate` sheet (authoritative, matches the fact
  date range) and is marked `dataCategory: Time` for time intelligence.

## Open questions (resolve with the business)

1. **Revenue definition.** The "Total Revenue (VND)" target (~1B VND/branch/month) is far larger than
   transaction `Fee_VND`. Fee revenue alone is not comparable to the target — true revenue (incl.
   interest margin) is not in the transaction data. `Revenue Target` and `Fee Revenue` are exposed
   separately; a meaningful *actual-vs-target* needs a revenue source (or a definition) first. No
   misleading ratio is published.
2. **NPS vs channel.** NPS surveys carry `BranchID` but **no channel**, so NPS can be cut by branch /
   region / customer but not by channel. Channel service quality is proxied by `Avg Service Time`.
3. **Service time coverage.** `ServiceTimeMinutes` is blank for many rows (channel-dependent);
   `Avg Service Time` should be read per channel.

## Validation
`tmdl-validate` passes on all 14 files; `validate_pbip.py --no-pbir-cli` → 0 errors; a cross-reference
check confirms every relationship, measure DAX reference, and report binding resolves.
