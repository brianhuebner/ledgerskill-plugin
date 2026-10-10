---
name: books-record
description: Use whenever a grill or a run starts or finishes, and whenever a record in the books folder is written or read — the contract header, a branch's procedure summary (branches/<branch>.procedure.md), the client facts (BOOKS-CONTEXT.md) or a decision memo (decisions/D-NNNN-<slug>.md).
---

The plugin leaves a written record of every workflow it sets up and every run it makes. Claude writes these records without being asked. The user reads sentences and tables; never show them a YAML header.

## Books folder

| Path | Record | Written | Changes after writing |
|---|---|---|---|
| `BOOKS-CONTEXT.md` | Client facts | When the first durable fact is learned | New lines appended as facts are learned |
| `decisions/D-NNNN-<slug>.md` | Decision memo | When the user accepts the offer | Never, except `status` and `replaced_by` when a new memo replaces it |
| `branches/<branch>.md` | Contract, with a header | End of a grill | Only when the user changes the contract; `version` rises by one each time |
| `branches/<branch>.py` | Script | When the contract is written or changed | Regenerated when the contract changes |
| `branches/<branch>.procedure.md` | Procedure summary | End of the first grill | Rows appended to its Runs table; nothing else |
| `branches/archive/<branch>.v<N>.md` and `.py` | An earlier contract and its script | Before the contract changes | Never |
| `to_be_processed/` | The user's inbox: source files waiting for a run | By the user | Claude moves a file out after a passing run |
| `complete/<period>/<branch>-<run id>/` | One run: `import.csv`, `detail.csv`, `checks-<run id>.txt`, and the source file after a passing run | Each run, by the script; the source file by Claude | Never |

## Rules

- Before writing a record, check whether the file exists. If it does, never rewrite it; only append what this skill allows.
- Every number in a record comes from script output, as `ledgerskill:scripted-arithmetic` requires.
- Header keys are fixed. A key with no value is written empty, never left out.
- Write dates as `YYYY-MM-DD`.

## Source files

Keep the folder tidy; trust the user with the rest. Avoiding overlapping exports, and tracking what reached the books, is up to the user.

- Look for source files in `to_be_processed/` first, then in the books folder itself. A file the user names is always accepted, wherever it is, including inside `complete/`.
- After a **passing** run, Claude (never the script) moves the source file into that run's folder. Check first that no file with that name is there; never overwrite. If the move isn't possible, leave the file where it is and say so in one line of the final report.
- After a failed run, leave the source file where it is.
- When the source file is already inside an earlier run's folder (a rerun), **copy** it into the new run's folder. Never move a file out of an earlier run.
- The chart of accounts and any import template stay where they are.

## Contract header

The header at the top of `branches/<branch>.md` (template in `ledgerskill:data-contract`):

| Key | Value |
|---|---|
| `type` | `contract` |
| `id` | The branch name |
| `version` | `1` when created |
| `supersedes` | The version this one replaced (`version` − 1); empty for version 1 |
| `chart` | The chart of accounts file name; empty when None |
| `layout` | The import layout name; empty until one is set |
| `facts` | `BOOKS-CONTEXT.md` once it exists; empty before |
| `decisions` | Decision IDs this contract relies on, as a list: `[D-0001, D-0003]`; empty when none |
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
| <run ID> | <period> | <"first run", or the contract change> | <pass / fail, entries, total> |

## 6. Before importing
- Open the run's `checks-<run id>.txt` and confirm every check passed; read each WARN.
- Import only one run per period. Every imported line's memo carries its run ID, which names the run folder.
- Compare the result with <the outside document named by the control total>.
- Import `import.csv` with <the import tool>.

## 7. Next period
Save the new export into `to_be_processed/`, then run:
`/ledgerskill:run-branch`
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

## Decision memos

A memo records one choice a reviewer will ask about, in the user's own words. Offer one only when all three hold:
1. It is hard to reverse once the books close on it.
2. A reviewer would ask why.
3. There was a real alternative.

Examples: a month-end cutoff for transactions that straddle two periods; booking card sales at gross with fees as their own line instead of net; accruing payroll by pay period instead of by pay date.

Offer it in one line, after the user has answered: "Save this as a decision? (yes)". Offer at most two memos per grill. Write a memo only on yes.

Number memos `D-0001`, `D-0002`, … in the order written: list `decisions/` and take the next number after the highest. A number is never reused, even when a memo is replaced. The slug is 2–4 words from the question, lowercase with hyphens. After writing, add the ID to the `decisions:` header key of each contract that relies on it.

```markdown
---
type: decision
id: D-NNNN
date: <YYYY-MM-DD>
branches: [<branch>, …]
status: active
replaced_by:
replaces:
---
# D-NNNN: <the question, in a few words>

**Question.** <what had to be decided, and why it came up>

**Options.**
1. <option>
2. <option>

**Choice.** <the option chosen>

**Reason.** "<the user's words, quoted>"

**What would change it.** <the fact or event that would reopen this>
```

A memo is never rewritten. To change a decision, write a new memo whose `replaces:` names the old ID. The only edit ever made to the old memo is setting `status: replaced` and `replaced_by: <new ID>` in its header.
