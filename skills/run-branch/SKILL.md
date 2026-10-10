---
name: run-branch
description: Run an existing branch from branches/ on a new period's source file and produce the import file, detail and checks.
disable-model-invocation: true
---

Load the `ledgerskill:data-contract`, `ledgerskill:scripted-arithmetic` and `ledgerskill:books-record` skills first, and follow them throughout.

Facts come from the file through scripts; decisions come from the user; outside facts, numbers the user reads from an outside document, are never recommended and never answered by `use your recommendations` (see `scripted-arithmetic`).

If `BOOKS-CONTEXT.md` exists, read it first and use its facts for every recommended answer.

If no source file was given, look in `to_be_processed/`, then the books folder, as `books-record` describes. Propose the file and ask the user to confirm. If there is none, ask for one and stop. If `branches/` has no branch files, tell the user to run `/ledgerskill:grill-books` first and stop.

1. **Profile** the file with a script that follows the `scripted-arithmetic` rules.
2. **Select the branch.** Compare the profile with the `Basis:` line of each `branches/*.md`. Pick the branch whose basis the file satisfies. If none or more than one matches, ask the user which one applies, and name the fact that is ambiguous.
   If the contract has no `## Lines` section (written before line rules), read it as one rule plus the offset: Rows = Scope, Amount column = Measure, Account = mapping on the category column, Group by = Line key, minus the entry key. Rewrite it in the current `data-contract` template, read the new Lines table back to the user for confirmation, archive the old version and raise the version as step 4 describes, and regenerate the script.
3. **Check the chart of accounts.** If the contract names a chart and that file is missing, tell the user and ask for a fresh export, or for permission to run with the chart set to None for this run only. Check 8 catches renamed or deleted accounts.
4. **Check the shape against the contract.** Using the profile, confirm that every contract column exists and that every distinct value of the scope and mapping columns is in scope, excluded, or mapped. Then compare the values of every Group by and dimension column, and of an entry-key column that holds a short list of codes rather than IDs or dates, with the values seen so far listed in section 2 of `branches/<branch>.procedure.md`. A new value, such as a new location code, is a question even when the contract would accept it. If anything is new or missing, stop and ask about the new or missing items only. Before changing the contract, copy `branches/<branch>.md` and `branches/<branch>.py` to `branches/archive/<branch>.v<N>.md` and `.py`, where N is the current `version`; if either archive file exists, stop and tell the user. Then update the contract, set `supersedes` to N and `version` to N + 1, and regenerate the script. If an answer passes the `books-record` decision test, offer a memo in one line.
5. **Echo.** Print the two echo lines from the `data-contract` skill.
6. **Ask for the outside number** with the outside-fact question from `scripted-arithmetic`, phrased from the contract's Control total. Skip it when the contract has none.
7. **Run** `branches/<branch>.py` on the file, with the outside amounts as arguments. It writes a new folder, `complete/<period>/<branch>-<run id>/`.
8. **Report** the run ID, the folder and every check line. If every check passed, move the source file into the run folder (or copy it, on a rerun) as `books-record` describes. If any check failed, show the failures and ask how to resolve them. Never change data or checks to make them pass. If check 7 is a WARN because the user skipped, repeat it in one line of the report.
9. **Record.** Append one row to the Runs table of `branches/<branch>.procedure.md`: the run ID, the period, what changed in the contract in step 4 (or "no change"), and the outcome. Append any newly accepted values to the Values seen list in section 2. Append only; change nothing else in the file. Add any durable fact learned in this run to `BOOKS-CONTEXT.md` as `books-record` describes.
