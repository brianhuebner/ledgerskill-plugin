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

The first command writes a sample Stripe export and a sample chart of accounts into the folder.

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

Runs are never overwritten. Running the same period again writes to `runs/<period>/<branch>-2/`, then `-3`, so the files you imported stay as the record of what you imported.

## What it does on your computer

- **Writes only into your books folder:** `branches/` (the contract and its script), `runs/` (outputs), and the two sample files from `/ledgerskill:demo`. It never deletes, renames or edits your source files or your chart of accounts.
- **Runs Python scripts it writes:** each script reads your file and writes its outputs. The rules it follows: standard library only, no network, no subprocesses, no deleting files. Claude Code asks your permission before running a command, unless you've allowed it. The script is in `branches/`, so you can read it first.
- **Has no hooks, MCP servers or background processes.** The plugin is five plain-text `SKILL.md` files you can read in a few minutes.
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

## Developing

To test changes from a local checkout, add the checkout as a marketplace:

```
/plugin marketplace add ~/path/to/ledgerskill-plugin
/plugin install ledgerskill@ledgerskill
```

After editing, run `/plugin marketplace update ledgerskill` and restart Claude Code.

You can also load the checkout directly with `claude --plugin-dir <path>`. Each skill is a single `SKILL.md` with no reference files, so the plugin works even when reads outside the working directory are blocked.
