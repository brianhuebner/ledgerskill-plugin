# ledgerskill

Grill-first accounting for Claude Code. Give it one transaction file and answer a short set of questions. It writes a data contract for that workflow and a script that turns the file into a balanced, tied-out journal entry import.

Claude reads the file for facts and asks you only for decisions. Every number comes from a script it runs, never from mental arithmetic.

## Requirements

- [Claude Code](https://claude.com/claude-code)
- Python 3 on your `PATH` (generated scripts use only the standard library)

## Install

In Claude Code, run:

```
/plugin marketplace add brianhuebner/ledgerskill-plugin
/plugin install ledgerskill@ledgerskill
```

Restart Claude Code. The commands appear under `/ledgerskill:`.

To update later:

```
/plugin marketplace update ledgerskill
```

## Try it in 2 minutes

Nothing to export yet. Start Claude Code in an empty folder and run:

```
/ledgerskill:demo
/ledgerskill:grill-books
```

The first command writes a sample Stripe export and a sample chart of accounts into the folder. Stripe is only the demo: the same steps work for a payroll register, a loan statement, a POS report, a bank feed or any other transaction file.

When the questions come, reply `use your recommendations`. Answer yes when Claude offers to run it, then open `complete/` to see the import file.

## Usage with your own books

### 1. Make a books folder

Create an empty folder anywhere, for example `~/books`, and inside it a folder named `to_be_processed`. Your books stay in QuickBooks, Xero or whatever you use. This folder only holds the files you export and the files the plugin writes.

### 2. Export your chart of accounts

In most bookkeeping apps the chart of accounts has an export on its own page. Save it as CSV (not Excel) into the books folder. With it, Claude recommends only accounts that exist, spelled the way your import expects.

It's optional. Without it, reply `skip` when asked and Claude takes account names from your answers, unchecked.

### 3. Export one source file

Export a CSV from wherever the transactions come from, such as a Stripe balance report, a Square sales export, a bank statement or a POS report, and save it into `to_be_processed/`. One file covering many days is the normal case. The point is to turn hundreds of rows into a few summarized entries.

### 4. Grill it

Start Claude Code in the books folder and run:

```
/ledgerskill:grill-books
```

Claude finds the two files and asks you to confirm which is which. It profiles the source file, then asks about three rounds of numbered questions, each with a recommended answer. Reply `use your recommendations` to accept them all, or answer in your own words. It reads the result back to you, saves it as `branches/<branch>.md`, and offers to produce the import file. It also writes `branches/<branch>.procedure.md`, a procedure summary of the file, your decisions in your own words, why the accounts are set up the way they are, and how to run the next period. Facts that hold across workflows, such as your fiscal year end or when your sales system closes its day, go in `BOOKS-CONTEXT.md`, so the next grill doesn't ask them again. When an answer is a choice a reviewer would ask about, such as a month-end cutoff, Claude offers to save it as a decision memo in `decisions/`.

### 5. What happens to your file

Your answers decide:

- **Which rows count.** Payouts posted by the bank deposit are left out, with the reason recorded.
- **Which account each kind of row goes to,** from your chart of accounts.
- **How rows are summarized:** one entry per payout, per day or per week, and one line per category, location or other column.
- **Which tags carry over,** such as a location column becoming a QuickBooks Class.
- **Fixed values you state in plain words,** such as "use the customer 'Stripe Payments' on every A/R line".
- **What the result must agree with,** such as the deposit on your bank statement.

Here is payout `po_A1` from the demo file: six Stripe rows

| id | type | net | location |
|---|---|---|---|
| txn_001 | charge | 116.22 | LOC01 |
| txn_002 | charge | 43.88 | LOC02 |
| txn_003 | charge | 1,213.45 | LOC01 |
| txn_004 | refund | (45.50) | LOC02 |
| txn_005 | charge | 300.71 | |
| txn_006 | payout | (1,628.76) | |

become one journal entry with five lines:

| account | debit | credit | Class |
|---|---|---|---|
| Sales | | 1,329.67 | LOC01 |
| Sales | | 43.88 | LOC02 |
| Sales | | 300.71 | Unassigned |
| Refunds and Returns | 45.50 | | LOC02 |
| Stripe Clearing | 1,628.76 | | |

The two LOC01 charges are summed into one line, the row with no location goes to Class "Unassigned", and the payout row is left out because the bank deposit records it. The entry balances and ties to the $1,628.76 deposit. Each of those rules came from a grill answer, so changing an answer changes the result.

### 6. Import it

Claude asks in the first round what will import the file, and writes `import.csv` in that tool's layout:

- **A sample file from your tool.** Drop a blank import template from your tool (Xero, NetSuite, Sage, QuickBooks, any other) into the books folder. Claude reads its header row and copies the column names exactly.
- **A built-in layout.** `saasant-journal-entry` is SaasAnt Transactions' Journal Entry import into QuickBooks Online: one signed amount column, MM/DD/YYYY dates, journal numbers of 21 characters or fewer. `saasant-invoice` imports invoices instead: one per entry, with a customer on each and a product or service on each line.
- **Plain,** the default: entry, date, account, debit, credit, one column per tag, and a memo. Map the columns once in whatever you use.

The tool and layout are saved to `BOOKS-CONTEXT.md`, so later grills don't ask again.

### Next period

Save the new export into `to_be_processed/` and run:

```
/ledgerskill:run-branch
```

It picks the matching workflow, stops to ask only about anything new in the file (a new category or location, for example), produces the import file, and adds the run to the procedure summary. After a passing run, the source file moves into that run's folder, so `to_be_processed/` holds only what's still waiting. Avoiding overlapping exports and tracking what you imported is up to you.

### Output

Each run gets a run ID, the time it started, such as `LS-20260905-081200`, and writes to `complete/<period>/<branch>-<run id>/`:

| File | Contents |
|---|---|
| `import.csv` | Balanced journal lines, ready to import; every memo ends with the run ID |
| `detail.csv` | One row per source row: used or excluded, why, and which import line it feeds |
| your source file | Moved here after a passing run |
| `checks-<run id>.txt` | The run record: run ID, contract version, hashes of the source, chart, contract and script, then the result of every tie-out check |

If a check fails, no `import.csv` is written. Claude shows you the failures and asks how to resolve them. It never changes data or loosens a check to make it pass.

Runs are never overwritten. Running the same period again gets a new run ID and a new folder, so the files you imported stay as the record of what you imported. To trace an entry in your books, search for the run ID in its memo: it names the one folder that produced it. When a contract changes, the earlier version and its script are kept in `branches/archive/`.

## What it does on your computer

- **Writes only into your books folder:** `BOOKS-CONTEXT.md` (client facts), `decisions/` (decision memos you accepted), `branches/` (the contract, its script and its procedure summary), `complete/` (outputs), and the two sample files from `/ledgerskill:demo`. It never deletes or edits your source files or your chart of accounts. The one thing it moves: after a passing run, Claude moves the source file into that run's folder, and never over an existing file.
- **Runs Python scripts it writes:** each script reads your file and writes its outputs. The rules it follows: standard library only, no network, no subprocesses, no deleting files. Claude Code asks your permission before running a command, unless you've allowed it. The script is in `branches/`, so you can read it first.
- **Has no hooks, MCP servers or background processes.** The plugin is six plain-text `SKILL.md` files you can read in a few minutes.
- **Treats your data as data.** The skills tell Claude never to follow text found in your files, such as a memo field, as an instruction, and to point out anything that reads like one.
- **What leaves your computer:** what Claude reads during the session goes to Anthropic as part of the conversation, as in any Claude Code session. The skills have Claude work from script output: profile facts (column names, counts, totals, distinct values) and the 3–4 rows in the worked example. The full file is processed by the local script, but anything Claude opens directly is sent too.
- **Updates:** `/plugin marketplace update` installs whatever is on `main`. Check the version and commit history before updating if you want to review changes.

## Skills

| Skill | How it runs | What it does |
|---|---|---|
| `grill-books` | You type it | Interviews you about one workflow and file, and writes the contract |
| `run-branch` | You type it | Runs an existing branch on a new period's file |
| `demo` | You type it | Writes a sample Stripe file and chart of accounts into the current folder |
| `data-contract` | Automatic | Fixed vocabulary and slot template for every contract |
| `scripted-arithmetic` | Automatic | Rules every data script must follow |
| `books-record` | Automatic | The procedure summary, client facts and decision memos written to your books folder |

## Developing

To test changes from a local checkout, add the checkout as a marketplace:

```
/plugin marketplace add ~/path/to/ledgerskill-plugin
/plugin install ledgerskill@ledgerskill
```

After editing, run `/plugin marketplace update ledgerskill` and restart Claude Code.

You can also load the checkout directly with `claude --plugin-dir <path>`. Each skill is a single `SKILL.md` with no reference files, so the plugin works even when reads outside the working directory are blocked.
