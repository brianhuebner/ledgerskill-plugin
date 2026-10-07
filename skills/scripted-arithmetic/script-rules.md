# Script rules

Rules for every script that reads a financial file. They make results exact, repeatable and traceable.

## Numbers

- Python 3 standard library only: `csv`, `decimal`, `hashlib`, `sys`, `pathlib`, `collections`.
- Read every amount as text and convert straight to `Decimal`. Never create a `float`, never call `float()`, never let a library infer numbers.
- Accepted amount text: `1234.56`, `-1234.56`, `1,234.56`, `(1,234.56)` (negative), `$1,234.56`, `-$1,234.56`. Anything else is a reported problem with its row number.
- A blank amount is a reported problem, not zero.
- Sum first, then round to cents once per output line: `.quantize(Decimal("0.01"), rounding=ROUND_HALF_UP)`.
- Keep signs through every grouping. Never use `abs()`. A negative group flips to the other side.

## Rows

- Row numbers are 1-based data rows (the header is not counted). Every message about a row names its row number.
- Every source row ends as exactly one of: **used** or **excluded with a reason**. Nothing is skipped silently.

## Contract constants

A script built from a contract starts with a constants block that mirrors the contract slots one-to-one, using the slot names:

```python
# --- Contract: branches/<branch>.md ---
BRANCH = "stripe-payout-settlement"
ROW_KEY = ["id"]
SCOPE = {"column": "type", "values": ["charge", "refund", "stripe_fee"]}
EXCLUSIONS = {"payout": "posted by bank deposit"}   # value -> reason
MEASURE = "net"
ENTRY_KEY = ["payout_id"]
LINE_KEY = ["reporting_category", "location"]
CATEGORY_COLUMN = "reporting_category"
MAPPING = {"charge": ("4000 Sales", "Credit")}       # value -> (account, side when positive)
DIMENSIONS = {"location": {"field": "Class", "map": {"LOC01": "Downtown"}, "blank": "Unassigned"}}
OFFSET_ACCOUNT = "1099 Stripe Clearing"
ENTRY_DATE = ("created", "max")
```

If the contract changes, regenerate the script. Never let the two drift apart.

## Required checks

Every processing script runs all of these and prints one line per check, `PASS` or `FAIL`, with the numbers:

1. **Completeness:** rows in = rows used + rows excluded, and every excluded row has a reason.
2. **Unique row key:** no duplicate row key values.
3. **Mapped:** every category value in scope has a mapping. An unmapped value is a FAIL that lists the value and its rows.
4. **Dimensions in line key:** every dimension column is in `LINE_KEY`.
5. **Amount conservation:** the signed sum of the measure over used rows equals the signed sum of output lines before the offset.
6. **Balance:** every entry's debits equal its credits, to the cent.
7. **Control total** (when given): the output agrees with it per entry key, and any difference is shown.

If any check fails, the script prints every failure and exits with status 1. It still writes `checks.txt` but writes no `import.csv`. The script never "fixes" data to pass.

## Outputs

Write to `runs/<period>/<branch>/`:

- `import.csv`: balanced journal lines with columns `entry`, `date`, `account`, `debit`, `credit`, one column per dimension field, `memo`. Debit and credit are positive and two-decimal; exactly one is filled.
- `detail.csv`: one row per source row, with `row`, the row key, `status` (`used` / `excluded`), `reason`, `account`, `amount`, and `import_line` (the `import.csv` line it feeds, blank if excluded).
- `checks.txt`: the header (source file name, SHA-256, branch, run time) followed by the check lines.

## Profiling scripts

A profiling script reports facts only, with no interpretation:

- file name, SHA-256, row count, columns with inferred type (text / amount / date)
- date range for each date column
- candidate row keys (columns with all-unique non-blank values)
- distinct values and counts for low-cardinality text columns (≤ 20 distinct)
- for each amount column: count positive / negative / zero / blank, and total; also totals by each low-cardinality column
- blank counts per column, and any unparseable amounts or dates with row numbers
