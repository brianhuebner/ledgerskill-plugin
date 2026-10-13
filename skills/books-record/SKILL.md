---
name: books-record
description: Use whenever a grill or a run starts or finishes, and whenever a contract header, a branch's procedure summary (branches/BRANCH.procedure.md), an archived contract or a run folder is written or read.
---

The plugin leaves a written record of every workflow it sets up and every run it makes. Claude writes these records without being asked. The user reads sentences and tables; never show them a YAML header.

The books folder is a knowledge bundle. Client facts, policies, parties, decision memos, `index.md` and `log.md` belong to `ledgerskill:books-context`; load it with this skill. The contract and procedure summary headers below use its header rules, so they are concepts in the same bundle.

## Books folder

| Path | Record | Written | Changes after writing |
|---|---|---|---|
| `index.md`, `log.md`, `entity.md`, `systems/`, `dimensions/`, `parties/`, `terms/`, `policies/`, `decisions/`, `questions/`, `references/` | The books' context | See `ledgerskill:books-context` | See `ledgerskill:books-context` |
| `branches/<branch>.md` | Contract, with a header | End of a grill | Only when the user changes the contract; `version` rises by one each time |
| `branches/<branch>.py` | Script | When the contract is written or changed | Regenerated when the contract changes |
| `branches/<branch>.procedure.md` | Procedure summary | End of the first grill | Rows appended to its Runs table and values to its Values seen list; nothing else |
| `branches/archive/<branch>.v<N>.md` and `.py` | An earlier contract and its script | Before the contract changes; the copy's header gets `status: deprecated` | Never |
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
| `title` | The branch name in plain words, as a JSON string: `"Stripe payout settlement"` |
| `description` | One sentence, as a JSON string: what file it reads and what it produces |
| `status` | `stable`; `deprecated` in an archived copy |
| `generated` | As `ledgerskill:books-context` defines; `at` is when this version was written |
| `id` | The branch name |
| `version` | `1` when created |
| `supersedes` | The version this one replaced (`version` − 1); empty for version 1 |
| `chart` | The chart of accounts file name; empty when None |
| `layout` | The built-in layout name, or the template file name |
| `facts` | Concept IDs this contract relies on, as a JSON list: `["entity", "systems/stripe", "dimensions/class"]`; `["BOOKS-CONTEXT"]` in a legacy folder; empty when none |
| `policies` | Policy IDs this contract relies on, as a JSON list: `["policies/cap-threshold"]`; empty when none |
| `decisions` | Decision IDs this contract relies on, as a list: `[D-0001, D-0003]`; empty when none |
| `procedure` | `<branch>.procedure.md` |

## Procedure summary

Write `branches/<branch>.procedure.md` at the end of the first grill for a branch, whether or not the user ran it. It explains the workflow to someone who was not in the conversation: a reviewer, an auditor, or the user next year. Start with this header, then use these seven sections, in this order:

```markdown
---
type: procedure
title: "<branch>: procedure"
description: "How to run <branch> each period, and why it is set up as it is."
status: stable
generated: {"by": "ledgerskill/0.4.0", "at": "<now>"}
contract: /branches/<branch>.md
---
# <branch>: procedure

## 1. Source, result and dates
Source: `<file name>`, <rows> rows, <first date> to <last date>, exported from <system>.
Result: <what it is imported as>, one entry per <entry key>.
Set up: <YYYY-MM-DD>. Contract version: <N>.

## 2. Profile facts
<Facts from the profiling script: columns used, row count, distinct values of the scope,
mapping and dimension columns with counts, amount totals. Copied from script output.>

Values seen:
- `<column>`: `<value>`, `<value>`, …   (one line per Group by, dimension and code-like entry-key column)

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

## Client facts and decision memos

These moved to `ledgerskill:books-context` in 0.4.0. Facts that hold across workflows are concepts (`entity.md`, `systems/`, `dimensions/`, `parties/`, `terms/`); rules are `policies/`; choices a reviewer will ask about are `decisions/`. A legacy `BOOKS-CONTEXT.md` is read, never written.
