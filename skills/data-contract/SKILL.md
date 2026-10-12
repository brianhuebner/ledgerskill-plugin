---
name: data-contract
description: Use when describing, writing, reading or checking how a financial source file becomes ledger lines — grain, keys, scope, line rules, mapping, dimensions, offset, control totals or the import layout. Use whenever a branch file in branches/ is created, edited or followed.
---

Every statement about how source rows become output lines uses the terms and slots below, and only those terms.

- Write contracts with the exact slot template below. Every slot is filled; "None" is a valid value.
- Column names go in backticks; values are quoted exactly as they appear in the file.
- Before producing any output from a contract, print the two echo lines defined below.

A data contract states, in fixed slots, exactly how rows of one source file become lines of ledger entries. Use only the terms below, each with only the meaning given. If a concept has no term here, describe it in plain words; do not invent a term.

## Terms

| Term | Definition | Example |
|---|---|---|
| **Source grain** | What one row of the source file represents | One processor balance transaction; one employee's pay line in a payroll register |
| **Row key** | Column(s) that uniquely identify a source row | `id`; `employee_id` + `pay_date` |
| **Scope** | Which source rows this branch uses | `type` in (`charge`, `refund`); `pay_type` in (`regular`, `overtime`) |
| **Exclusion** | Rows in the file but out of scope, each with a reason; still counted | `type` = `payout`: posted from the bank feed; `pay_type` = `reimbursement`: paid through expense reports |
| **Entry key** | Column(s) whose distinct values make one entry | `payout_id`; `pay_date`; `statement_month` |
| **Line rule** | One row of the Lines table: which rows, which amount column, which account, which side, and how its lines group. One source row can feed several rules | `sales`: `charge` rows' `gross` → 4000 Sales, credit, by `location`; `fees`: all rows' `fee` → 6150 Merchant Fees, debit, one per entry |
| **Amount column** | The numeric column a rule totals, signs kept | `gross`; `fee`; `employer_tax` |
| **Group by** | Column(s) whose distinct values make one line of a rule within an entry, or "one per entry" | `location`; `department`; one per entry |
| **Row total** | A source column that states each row's net effect, and how the rules' amount columns make it | `net` = `gross` − `fee`; `net_pay` = `gross_pay` − `employee_tax` |
| **Chart of accounts** | A CSV of the user's accounts, exported from their bookkeeping app, and the column that holds each account's name | `chart-of-accounts.csv`, column `Full name` |
| **Mapping** | A table a rule can name instead of one account: a source column's value → account and side; covers every value among the rule's rows | on `pay_type`: `regular` → 6000 Wages, debit; `overtime` → 6010 Overtime Wages, debit |
| **Dimension** | A source column carried to the ledger as a tag; never changes the account | `location` → QBO Class; `department` → QBO Class |
| **Dimension mapping** | Source value → ledger dimension value | `LOC01` → Class "Downtown" |
| **Blank rule** | What happens when a mapping or dimension value is blank | Class "Unassigned" + flag |
| **Fixed field** | A ledger field set to one value the user chose, on the lines of one account or on all lines | Customer "Card Sales" on every Accounts Receivable (A/R) line; Vendor "ADP" on every payroll liability line |
| **Offset line** | The line that balances each entry: the last row of the Lines table | 1099 Processor Clearing; 2150 Wages Payable |
| **Control total** | The outside number the output must agree with, from an outside document | Deposit on the bank statement; balance on the lender statement; funding total on the payroll provider report |
| **Import layout** | The columns and formats of the file the user's import tool reads | `plain`; `saasant-journal-entry`; the header row of a Xero or NetSuite import template |

A **branch** is one named workflow with one contract (e.g. `stripe-payout-settlement`, `semi-monthly-payroll`). Name it for what distinguishes it from similar workflows (`line-of-credit-draw` vs. `line-of-credit-paydown`).

## Distinctions

1. **Source grain ≠ line.** "Group by pay date" means nothing until the source grain is stated.
2. **Mapping column ≠ dimension.** A mapping's column picks the account. A dimension only tags the line.
3. **A rule that carries a dimension groups by it.** Otherwise its lines merge the dimension's values and the tag is lost. A rule that groups one per entry carries no dimension.
4. **QBO Class ≠ QBO Location.** Two separate QuickBooks Online fields. Ask which one a column maps to.
5. **Excluded ≠ unmapped.** Excluded rows are left out on purpose, with a reason. An unmapped value stops the run and becomes a question. There is no catch-all "other" account.
6. **Names come from a list.** When there is a chart of accounts, every account in the Lines table, its mappings and the fixed fields is spelled exactly as in its account column. When the layout's line names are something else, such as products and services, they are spelled exactly as in that list. A name not in its list is a question, never a new name.
7. **Dimension ≠ fixed field.** A dimension's value comes from a source column and can differ row to row. A fixed field's value is a constant the user stated; it is not read from the source.
8. **One row, several rules.** A row can feed more than one rule, each with its own amount column. A card charge feeds sales at `gross` and fees at `fee`; a pay line feeds wages, employer taxes and withholdings. Each rule groups its own way.

## Slot template

Every branch file `branches/<branch>.md` has exactly these sections and slots, in this order. It starts with the header that `ledgerskill:books-record` defines; Claude writes it, and it is never shown to the user.

```markdown
---
type: contract
id: <branch>
version: 1
supersedes:
chart: <chart file name, or empty>
layout: <layout name, or the template file name>
facts:
decisions:
procedure: <branch>.procedure.md
---
# <branch>

Basis: <observable fact in the source that selects this branch>

## Data contract

Source grain:  <what one row is>
Row key:       `<col>`
Scope:         `<col>` in (`<value>`, …)
Exclusions:    `<col>` = `<value>` → reason: <reason>   (or None)
Entry key:     `<col>`
Row total:     `<col>` = `<amount col>` − `<amount col>` …   (or None)

## Lines
Chart of accounts: `<file>`, column `<col>`   (or None — account names not checked)
Name list:    `<file>`, column `<col>`   (only when the layout's line names are not accounts; or None — names not checked)
| Rule | Rows | Amount column | Account | Side when positive | Group by |
|---|---|---|---|---|---|
| `<rule>` | all rows, or `<col>` in (`<value>`, …) | `<col>` | <code name>, or mapping on `<col>` | Debit / Credit, or from mapping | `<col>`, …, or one per entry |
| `offset` | every entry | balancing amount | <code name> | | one per entry |
(no `offset` row when the layout is not balanced)

### Mapping on `<col>` (rule `<rule>`)
| Value | Account | Side when positive |
|---|---|---|
| `<value>` | <code name> | Debit / Credit |
Unmapped value → stop and ask.
(one such table per rule that names a mapping; or none)

## Dimensions
| Source column | Ledger field | Mapping | Blank rule | Rules |
|---|---|---|---|---|
| `<col>` | <field> | `<value>` → "<ledger value>", … | <rule> | `<rule>`, … |
(or None)

## Fixed fields
| Applies to | Ledger field | Value |
|---|---|---|
| <account, or "all lines"> | <field> | "<value>" |
(or None)

Control total: <outside number>, from <outside document>, per <entry key, or for the period>; also stated in the file by <rows or column>   (or None)
Entry date:    `<col>` (<which value within the entry: max / min / first>)

## Import layout
Tool:          <what imports the file, and what it imports as>
| Column | Filled with |
|---|---|
| `<column, spelled exactly>` | <entry number / entry date / account / debit / credit / signed amount / memo / a dimension / a fixed field / blank> |
Line names:   <account | product-service | another list>: what the Lines table's Account cells name
Balanced:     <yes: every entry balances, offset line required | no: offset line None>
Required columns: `<column>`, …   (or None): must be non-blank on every line
Amount style:  <debit-credit: two positive columns, one filled | signed: one column, debit positive, credit negative | positive only>
Date format:   <e.g. YYYY-MM-DD, MM/DD/YYYY>
Entry number:  <how it is built>; at most <N> characters (or no limit); `-2`, `-3` appended when two entries would share one
Memo:          <what it holds>, ending `<entry key value> · run <run id>`
Name column:   `<column>` carries the <Customer / Vendor / Employee> fixed field   (or None)
Header quirks: <what was copied exactly from a template, such as stray spaces>   (or None)

## Example
<3–4 source rows, then the output lines they become>
```

## Lines examples

A card processor's balance export, booking sales at gross and fees as their own line:

| Rule | Rows | Amount column | Account | Side when positive | Group by |
|---|---|---|---|---|---|
| `sales` | `type` in (`charge`, `refund`) | `gross` | mapping on `type` | from mapping | `location` |
| `fees` | all rows | `fee` | 6150 Merchant Fees | Debit | one per entry |
| `offset` | every entry | balancing amount | 1099 Processor Clearing | | one per entry |

Row total: `net` = `gross` − `fee`.

A payroll register:

| Rule | Rows | Amount column | Account | Side when positive | Group by |
|---|---|---|---|---|---|
| `wages` | all rows | `gross_pay` | 6000 Wages | Debit | `department` |
| `employer-tax` | all rows | `employer_tax` | 6100 Payroll Taxes | Debit | one per entry |
| `employer-tax-owed` | all rows | `employer_tax` | 2100 Payroll Liabilities | Credit | one per entry |
| `withholding` | all rows | `employee_tax` | 2100 Payroll Liabilities | Credit | one per entry |
| `offset` | every entry | balancing amount | 2150 Wages Payable | | one per entry |

Row total: `net_pay` = `gross_pay` − `employee_tax`.

A file with one amount column has one rule plus the offset.

## Built-in layouts

Fill the `## Import layout` section in this order of preference:

1. **A template file in the books folder** from the user's import tool (Xero, NetSuite, Sage, QuickBooks native, SaasAnt, any other). It always wins. Read its header row and copy every column name exactly, including stray spaces and capitals, in its order. Fill the other slots from the user's answers.
2. **A built-in layout**, when the user names its tool. Copy its block below into the contract.
3. **`plain`**, when neither applies.

A layout's properties (Line names, Balanced, Required columns, Amount style) decide which questions the grill asks and which checks apply. Under positive only, a group that sums negative becomes a question.

Add a new built-in, such as a bill or sales receipt import, by writing another filled block here in the same form. It needs no other change to the skills.

### `plain`
```markdown
## Import layout
Tool:          Any journal entry import that maps columns once
| Column | Filled with |
|---|---|
| `entry` | entry number |
| `date` | entry date |
| `account` | account |
| `debit` | debit |
| `credit` | credit |
| one column per dimension field, named for the field | that dimension |
| one column per fixed field, named for the field | that fixed field, blank on lines it does not apply to |
| `memo` | memo |
Line names:   account
Balanced:     yes
Required columns: None
Amount style:  debit-credit
Date format:   YYYY-MM-DD
Entry number:  the entry key value (columns joined with `-`); no limit
Memo:          `<entry key value> · run <run id>`
Name column:   None
Header quirks: None
```

### `saasant-journal-entry`
SaasAnt Transactions importing into QuickBooks Online as Journal Entries.
```markdown
## Import layout
Tool:          SaasAnt Transactions → QuickBooks Online, Journal Entry
| Column | Filled with |
|---|---|
| `Journal No` | entry number |
| `Journal Date` | entry date |
| `Memo` | memo |
| `Account` | account |
| `Amount` | signed amount |
| `Description` | blank |
| `Name` | the Customer, Vendor or Employee fixed field; blank on other lines |
| `Location` | the dimension mapped to Location; else blank |
| `Class` | the dimension mapped to Class; else blank |
| `Currency Code` | blank |
| `Exchange Rate` | blank |
| `Is Adjustment` | blank |
Line names:   account
Balanced:     yes
Required columns: None
Amount style:  signed
Date format:   MM/DD/YYYY
Entry number:  the entry key value if it fits, else a short prefix the user chooses plus the entry date as YYYYMMDD; at most 21 characters; `-2`, `-3` appended when two entries would share one
Memo:          `<entry key value> · run <run id>` on every line; when too long, shorten the entry key text and keep the run ID
Name column:   `Name`
Header quirks: None
```

### `saasant-invoice`
SaasAnt Transactions importing into QuickBooks Online as Invoices. Each entry is one invoice; each line names a product or service.
```markdown
## Import layout
Tool:          SaasAnt Transactions → QuickBooks Online, Invoice
| Column | Filled with |
|---|---|
| `Invoice No` | entry number |
| `Customer` | the Customer fixed field |
| `Invoice Date` | entry date |
| `Due Date` | blank |
| `Terms` | blank |
| `Location` | the dimension mapped to Location; else blank |
| `Memo` | memo |
| `Product/Service` | line name |
| `Product/Service Description` | blank |
| `Product/Service Quantity` | blank |
| `Product/Service Rate` | blank |
| `Product/Service Amount` | amount |
| `Product/Service Class` | the dimension mapped to Class; else blank |
| `Currency Code` | blank |
Line names:   product-service
Balanced:     no
Required columns: `Customer`
Amount style:  positive only
Date format:   MM/DD/YYYY
Entry number:  the entry key value if it fits, else a short prefix the user chooses plus the entry date as YYYYMMDD; at most 21 characters; `-2`, `-3` appended when two entries would share one
Memo:          `<entry key value> · run <run id>`; when too long, shorten the entry key text and keep the run ID
Name column:   `Customer`
Header quirks: None
```

## Writing rules

- Column names in backticks; values quoted exactly as in the file. Never "the type column."
- Every slot is filled. "None" is valid; a missing slot is an error.
- Every header key is present. A key with no value is written empty.
- One worked example. It encodes grain better than any sentence.

## Echo lines

Before producing any output from a contract, print:

```
Branch: <branch>. Basis: <fact observed in this file>.
Grain: <source grain> → one entry per <entry key>; rules: <rule> by <group by>, …. Rows: <N> in, <M> excluded.
```

Row counts come from a script, never from estimation.
