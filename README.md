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
/ledgerskill:grill-books stripe_balance_sep.csv
```

The first command copies a sample Stripe export and a sample chart of accounts into the folder.

When the questions come, reply `use your recommendations`. Answer yes when Claude offers to run it, then open `runs/` to see the import file.

## Usage

Start Claude Code in the folder where you keep your books, then:

```
/ledgerskill:grill-books path/to/transactions.csv
```

Claude profiles the file, asks a few rounds of numbered questions (each with a recommended answer), reads the contract back to you, and writes `branches/<branch>.md`. It then offers to generate `branches/<branch>.py` and produce the import file.

Next period, run the same workflow on a new file:

```
/ledgerskill:run-branch path/to/next-month.csv
```

It picks the matching branch, stops to ask about anything new in the file, and runs the script.

### Output

Each run writes to `runs/<period>/<branch>/`:

| File | Contents |
|---|---|
| `import.csv` | Balanced journal lines, ready to import |
| `detail.csv` | One row per source row: used or excluded, why, and which import line it feeds |
| `checks.txt` | Source file hash and the result of every tie-out check |

If a check fails, no `import.csv` is written. Claude shows you the failures and asks how to resolve them. It never changes data or loosens a check to make it pass.

## Skills

| Skill | How it runs | What it does |
|---|---|---|
| `grill-books` | You type it | Interviews you about one workflow and file, and writes the contract |
| `run-branch` | You type it | Runs an existing branch on a new period's file |
| `demo` | You type it | Copies a sample Stripe file and chart of accounts into the current folder |
| `data-contract` | Automatic | Fixed vocabulary and slot template for every contract |
| `scripted-arithmetic` | Automatic | Rules every data script must follow |

## Developing

To test changes from a local checkout, add the checkout as a marketplace:

```
/plugin marketplace add ~/path/to/ledgerskill-plugin
/plugin install ledgerskill@ledgerskill
```

After editing, run `/plugin marketplace update ledgerskill` and restart Claude Code.

You can also load the checkout directly with `claude --plugin-dir <path>`. If your setup blocks reads outside the working directory, add `--add-dir <path>` as well, so Claude can open the skills' reference files.
