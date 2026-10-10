---
name: books-record
description: Use whenever a grill or a run starts or finishes, and whenever a record in the books folder is written or read — the contract header, a branch's procedure summary (branches/<branch>.procedure.md) or the client facts (BOOKS-CONTEXT.md).
---

The plugin leaves a written record of every workflow it sets up and every run it makes. Claude writes these records without being asked. The user reads sentences and tables; never show them a YAML header.

## Books folder

| Path | Record | Written | Changes after writing |
|---|---|---|---|
| `BOOKS-CONTEXT.md` | Client facts | When the first durable fact is learned | New lines appended as facts are learned |
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
| `facts` | `BOOKS-CONTEXT.md` once it exists; empty before |
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

## Client facts

`BOOKS-CONTEXT.md` holds what is true about this client across workflows, so no grill asks it twice. `ledgerskill:grill-books` and `ledgerskill:run-branch` read it first and use it to set recommended answers. A question it already answers is not asked.

Write a fact only if all three hold:
1. It will still be true next month.
2. It applies to more than one branch, or would to the next one set up.
3. It is a fact or a definition, not a choice about how one branch books something. Choices go in the contract.

Examples: "Fiscal year ends June 30." "Payroll runs semi-monthly, on the 15th and the last business day." "The POS business day closes at 6 PM local time, so a UTC export splits evenings across two dates." "`LOC01` is the Downtown store."

Harvest facts from the user's answers and the files. Never interview the user for them. Before writing, name the new facts in the read-back's closing line: "I'll remember: <facts>." Write those the user doesn't correct.

Create the file with this template on the first fact. After that, add new facts as new lines under their section; never change or remove a line. When a fact changes, add the new one with its date and the words "replaces: <old fact>".

```markdown
# Books context

Facts about these books that hold across workflows. Written by ledgerskill.

## Entity
- <name, fiscal year end, base currency, time zone>

## Systems that produce files
- <system>: <what it exports; its time zone or cutoff>

## Import tool and layout
- <tool>; layout <name>

## Glossary
- <term or code>: <what it means here>

## Open questions
- <YYYY-MM-DD> <a question raised and not yet settled>
```
