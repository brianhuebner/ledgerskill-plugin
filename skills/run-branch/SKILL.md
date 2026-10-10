---
name: run-branch
description: Run an existing branch from branches/ on a new period's source file and produce the import file, detail and checks.
disable-model-invocation: true
---

Load the `ledgerskill:data-contract`, `ledgerskill:scripted-arithmetic` and `ledgerskill:books-record` skills first, and follow them throughout.

If `BOOKS-CONTEXT.md` exists, read it first and use its facts for every recommended answer.

If no source file was given, ask for one and stop. If `branches/` has no branch files, tell the user to run `/ledgerskill:grill-books` first and stop.

1. **Profile** the file with a script that follows the `scripted-arithmetic` rules.
2. **Select the branch.** Compare the profile with the `Basis:` line of each `branches/*.md`. Pick the branch whose basis the file satisfies. If none or more than one matches, ask the user which one applies, and name the fact that is ambiguous.
3. **Check the chart of accounts.** If the contract names a chart and that file is missing, tell the user and ask for a fresh export, or for permission to run with the chart set to None for this run only. Check 8 catches renamed or deleted accounts.
4. **Check the shape against the contract.** Using the profile, confirm that every contract column exists and that every distinct value of the scope and category columns is in scope, excluded, or mapped. If anything is new or missing, stop and ask about the new or missing items only. Before changing the contract, copy `branches/<branch>.md` and `branches/<branch>.py` to `branches/archive/<branch>.v<N>.md` and `.py`, where N is the current `version`; if either archive file exists, stop and tell the user. Then update the contract, set `supersedes` to N and `version` to N + 1, and regenerate the script. If an answer passes the `books-record` decision test, offer a memo in one line.
5. **Echo.** Print the two echo lines from the `data-contract` skill.
6. **Run** `branches/<branch>.py` on the file. It writes a new folder, `runs/<period>/<branch>-<run id>/`.
7. **Report** the run ID, the folder and every check line. If any check failed, show the failures and ask how to resolve them. Never change data or checks to make them pass.
8. **Record.** Append one row to the Runs table of `branches/<branch>.procedure.md`: the run ID, the period, what changed in the contract in step 4 (or "no change"), and the outcome. Append only; change nothing else in the file. Add any durable fact learned in this run to `BOOKS-CONTEXT.md` as `books-record` describes.
