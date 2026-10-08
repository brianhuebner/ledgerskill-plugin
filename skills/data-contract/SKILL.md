---
name: data-contract
description: Use when describing, writing, reading or checking how a financial source file becomes ledger lines — grain, keys, scope, mapping, dimensions, offset or control totals. Use whenever a branch file in branches/ is created, edited or followed.
---

Every statement about how source rows become output lines uses the terms and slots below, and only those terms.

- Write contracts with the exact slot template below. Every slot is filled; "None" is a valid value.
- Column names go in backticks; values are quoted exactly as they appear in the file.
- Before producing any output from a contract, print the two echo lines defined below.

A data contract states, in fixed slots, exactly how rows of one source file become lines of ledger entries. Use only the terms below, each with only the meaning given. If a concept has no term here, describe it in plain words; do not invent a term.

## Terms

| Term | Definition | Example |
|---|---|---|
| **Source grain** | What one row of the source file represents | One Stripe balance transaction |
| **Row key** | Column(s) that uniquely identify a source row | `id` |
| **Scope** | Which source rows this branch uses | `type` in (`charge`, `refund`, `stripe_fee`) |
| **Exclusion** | Rows in the file but out of scope, each with a reason; still counted | `type` = `payout`: posted by bank deposit |
| **Measure** | The numeric column that is totaled, and its sign convention | `net`; positive increases Stripe balance |
| **Entry key** | Column(s) whose distinct values make one entry | `payout_id` |
| **Line key** | Column(s) whose distinct values make one line within an entry | `reporting_category`, `location` |
| **Output grain** | What one output line represents; always entry key + line key | One line per payout × category × location |
| **Rollup** | How the measure combines within a line, and when rounding happens | Sum; round to cents after summing |
| **Chart of accounts** | A CSV of the user's accounts, exported from their bookkeeping app, and the column that holds each account's name | `chart-of-accounts.csv`, column `Full name` |
| **Category column** | The source column that decides the account | `reporting_category` |
| **Mapping** | Category value → account and side; covers every value in scope | `charge` → 4000 Sales, credit |
| **Dimension** | A source column carried to the ledger as a tag; never changes the account | `location` → QBO Class |
| **Dimension mapping** | Source value → ledger dimension value | `LOC01` → Class "Downtown" |
| **Blank rule** | What happens when a category or dimension value is blank | Class "Unassigned" + flag |
| **Fixed field** | A ledger field set to one value the user chose, on the lines of one account or on all lines | Customer "Stripe Payments" on every Accounts Receivable (A/R) line |
| **Offset line** | The line that balances each entry | 1099 Stripe Clearing |
| **Control total** | The external number the output must agree with | Payout amount on the bank statement |

A **branch** is one named workflow with one contract (e.g. `stripe-payout-settlement`). Name it for what distinguishes it from similar workflows (`line-of-credit-draw` vs. `line-of-credit-paydown`).

## Distinctions

1. **Source grain ≠ output grain.** "Group by payout" means nothing until the source grain is stated.
2. **Category ≠ dimension.** The category column picks the account. A dimension only tags the line.
3. **Every dimension is in the line key.** Otherwise the rollup merges dimension values and the tag is lost.
4. **QBO Class ≠ QBO Location.** Two separate QuickBooks Online fields. Ask which one a column maps to.
5. **Excluded ≠ unmapped.** Excluded rows are left out on purpose, with a reason. An unmapped value stops the run and becomes a question. There is no catch-all "other" account.
6. **Accounts come from the chart.** When there is a chart of accounts, every account in the mapping, the offset line and the fixed fields is spelled exactly as in its account column. An account not in the chart is a question, never a new account.
7. **Dimension ≠ fixed field.** A dimension's value comes from a source column and can differ row to row. A fixed field's value is a constant the user stated; it is not read from the source.

## Slot template

Every branch file `branches/<branch>.md` has exactly these sections and slots, in this order:

```markdown
# <branch>

Basis: <observable fact in the source that selects this branch>

## Data contract

Source grain:  <what one row is>
Row key:       `<col>`
Scope:         `<col>` in (`<value>`, …)
Exclusions:    `<col>` = `<value>` → reason: <reason>   (or None)
Measure:       `<col>`; <what positive means>
Entry key:     `<col>`
Line key:      `<col>`, …
Output grain:  one line per <entry key> × <line key>
Rollup:        sum `<col>`; round to cents after summing

## Mapping (category column: `<col>`)
Chart of accounts: `<file>`, column `<col>`   (or None — account names not checked)
| Value | Account | Side when positive |
|---|---|---|
| `<value>` | <code name> | Debit / Credit |
Unmapped value → stop and ask.

## Dimensions
| Source column | Ledger field | Mapping | Blank rule |
|---|---|---|---|
| `<col>` | <field> | `<value>` → "<ledger value>", … | <rule> |
(or None)

## Fixed fields
| Applies to | Ledger field | Value |
|---|---|---|
| <account, or "all lines"> | <field> | "<value>" |
(or None)

Offset line:   <account>, one per entry
Control total: <external number>, per <entry key>   (or None)
Entry date:    `<col>` (<which value within the entry: max / min / first>)

## Example
<3–4 source rows, then the output lines they become>
```

## Writing rules

- Column names in backticks; values quoted exactly as in the file. Never "the type column."
- Every slot is filled. "None" is valid; a missing slot is an error.
- One worked example. It encodes grain better than any sentence.

## Echo lines

Before producing any output from a contract, print:

```
Branch: <branch>. Basis: <fact observed in this file>.
Grain: <source grain> → <output grain>. Rows: <N> in, <M> excluded.
```

Row counts come from a script, never from estimation.
