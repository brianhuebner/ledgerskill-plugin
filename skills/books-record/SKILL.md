---
name: books-record
description: Use whenever a grill or a run finishes, and whenever a record in the books folder is written or read — the contract header or a branch's procedure summary (branches/<branch>.procedure.md).
---

The plugin leaves a written record of every workflow it sets up and every run it makes. Claude writes these records without being asked. The user reads sentences and tables; never show them a YAML header.

## Books folder

| Path | Record | Written | Changes after writing |
|---|---|---|---|
| `branches/<branch>.md` | Contract, with a header | End of a grill | Only when the user changes the contract |
| `branches/<branch>.py` | Script | When the contract is written or changed | Regenerated when the contract changes |
| `branches/<branch>.procedure.md` | Procedure summary | End of the first grill | Rows appended to its Runs table; nothing else |
| `runs/<period>/<branch>/` | One run's outputs | Each run | Never |

## Rules

- Before writing a record, check whether the file exists. If it does, never rewrite it; only append what this skill allows.
- Every number in a record comes from script output, as `ledgerskill:scripted-arithmetic` requires.
- Header keys are fixed. A key with no value is written empty, never left out.
- Write dates as `YYYY-MM-DD`.

## Contract header

The header at the top of `branches/<branch>.md` (template in `ledgerskill:data-contract`):

| Key | Value |
|---|---|
| `type` | `contract` |
| `id` | The branch name |
| `version` | `1` when created |
| `supersedes` | The version this one replaced; empty for version 1 |
| `chart` | The chart of accounts file name; empty when None |
| `layout` | The import layout name; empty until one is set |
| `facts` | The client facts file name; empty until one exists |
| `decisions` | Decision IDs this contract relies on; empty when none |
| `procedure` | `<branch>.procedure.md` |

## Procedure summary

Write `branches/<branch>.procedure.md` at the end of the first grill for a branch, whether or not the user ran it. It explains the workflow to someone who was not in the conversation: a reviewer, an auditor, or the user next year. Use these seven sections, in this order:

```markdown
# <branch>: procedure

## 1. Source, result and dates
Source: `<file name>`, <rows> rows, <first date> to <last date>, exported from <system>.
Result: <what it is imported as>, one entry per <entry key>.
Set up: <YYYY-MM-DD>. Contract version: <N>.

## 2. Profile facts
<Facts from the profiling script: columns used, row count, distinct values of the scope,
category and dimension columns with counts, amount totals. Copied from script output.>

## 3. Decisions
| Question | Answer | User's words |
|---|---|---|
| <question> | <answer> | "<the user's words, quoted>" or "accepted recommendation" |

## 4. Why the accounts are set up this way
<Plain words: what each account in the contract holds, and why the offset line is the
account it is. For a clearing or suspense account, say what clears it and that it should
return to zero when the outside record arrives.>

## 5. Runs
| Run | Period | What changed | Outcome |
|---|---|---|---|
| <YYYY-MM-DD HH:MM> | <period> | <"first run", or the contract change> | <pass / fail, entries, total> |

## 6. Before importing
- Open the run's checks file and confirm every check passed; read each WARN.
- Compare the result with <the outside document named by the control total>.
- Import `import.csv` with <the import tool>.

## 7. Next period
Save the new export into this folder, then run:
`/ledgerskill:run-branch <file name>`
```

Section 7 always gives the `/ledgerskill:run-branch` command, never a direct `python3` call: running the script directly skips the check for values the contract has not seen.

After a grill that ran, section 5 has one row for that run. After each later run, `ledgerskill:run-branch` appends one row. If the summary is missing when a run finishes (a branch set up before summaries existed), create it from the contract, the profile and this run, and write "Not recorded" in section 3.
