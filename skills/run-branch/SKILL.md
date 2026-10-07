---
name: run-branch
description: Run an existing branch from branches/ on a new period's source file and produce the import file, detail and checks.
disable-model-invocation: true
---

Use the `data-contract` and `scripted-arithmetic` skills throughout.

If no source file was given, ask for one and stop. If `branches/` has no branch files, tell the user to run `/ledgerskill:grill-books` first and stop.

1. **Profile** the file with a script that follows [script-rules.md](../scripted-arithmetic/script-rules.md).
2. **Select the branch.** Compare the profile with the `Basis:` line of each `branches/*.md`. Pick the branch whose basis the file satisfies. If none or more than one matches, ask the user which one applies, and name the fact that is ambiguous.
3. **Check the shape against the contract.** Using the profile, confirm that every contract column exists and that every distinct value of the scope and category columns is in scope, excluded, or mapped. If anything is new or missing, stop. Ask about the new or missing items only, update `branches/<branch>.md`, and regenerate `branches/<branch>.py`.
4. **Echo.** Print the two echo lines from [data-contract.md](../data-contract/data-contract.md).
5. **Run** `branches/<branch>.py` on the file, writing to `runs/<period>/<branch>/`.
6. **Report** every check line. If any check failed, show the failures and ask how to resolve them. Never change data or checks to make them pass.
