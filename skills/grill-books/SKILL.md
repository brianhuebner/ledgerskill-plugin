---
name: grill-books
description: Interview the user about one accounting workflow and one source file until every decision is settled, then write the data contract and offer to produce the import file.
disable-model-invocation: true
---

Use the `data-contract` and `scripted-arithmetic` skills throughout. Facts come from the file through scripts. Only decisions come from the user.

## 0. Find the files
You need one source file and, ideally, a chart of accounts.

- If no source file was given, list the CSV files in the current folder. Propose which is the source file and which is the chart of accounts (a chart has one row per account and columns such as account name and type), and ask the user to confirm. If there are no CSV files, ask for a source file and stop.
- If there is no chart of accounts, ask once: *"Do you have a chart of accounts export? In most bookkeeping apps it's on the Chart of Accounts page; save it as CSV into this folder. Or reply `skip` and I'll take account names from your answers, unchecked."* If the user skips, the contract's chart is None. Do not ask again.

## 1. Profile
Write and run a profiling script that follows [script-rules.md](../scripted-arithmetic/script-rules.md) on the source file and, if there is one, the chart. Report the facts to the user in 8 lines or fewer. Do not interpret them yet.

If there is a chart, ask in Round 1 which column holds the account names to import. Recommend the column with all-unique, non-blank values that the user's import tool matches on, usually the full name.

## 2. Round 1: purpose
Ask at most 4 numbered questions, each with a recommended answer based on the profile:
1. What is this file?
2. What period does it cover, and is it complete?
3. What will it be imported as? (Journal Entry by default)
4. What should the result agree with? (a bank deposit, lender statement, payout report…)

Avoid the words "branch", "grain", "contract" and "provenance" in this round. A reply of "use your recommendations" accepts every recommended answer.

## 3. Rounds 2–3: fill the contract
Ask only the questions whose prerequisites are settled, numbered, each with a recommended answer. Fill the slots in this order:

1. Branch name and basis (the observable fact that identifies this kind of file). If `branches/` already has a branch whose basis matches, propose using it and ask about differences only.
2. Source grain and row key.
3. Scope and exclusions (every distinct value of the scope column is either in scope or excluded with a reason).
4. Measure and its sign.
5. Entry key, line key, rollup, entry date.
6. Mapping: every category value in scope gets an account and a side. When there is a chart, recommend only accounts from it, spelled exactly. If the user names an account that is not in the chart, say so and offer the closest accounts from it.
7. Dimensions: for each low-cardinality text column the profile found, ask whether it goes to the ledger, which field, the value mapping and the blank rule.
8. Fixed fields: ask whether any field should always have the same value, on one account's lines or on all lines. Recommend one when the mapping needs it: an Accounts Receivable or Accounts Payable line needs a Customer or Vendor name in QuickBooks. Accept fixed fields stated in plain words in any round, e.g. *"Use the customer 'Stripe Payments' for every A/R line."*
9. Offset line and control total.
10. Edge cases the profile found: negative groups, blanks, unparseable values, duplicate keys.

When an answer conflicts with the data, say so using numbers from a script, e.g. *"2 rows have no `location`, totaling $305.80. Where do they go?"* Run a script to get those numbers.

## 4. Read-back
Read the filled slots back to the user, slot by slot, including the chart of accounts and fixed fields, and get explicit confirmation. Change anything they correct.

## 5. Write
Write `branches/<branch>.md` in the user's working folder using the slot template in [data-contract.md](../data-contract/data-contract.md) exactly, including one worked example built by running a script on 3–4 real rows.

## 6. Offer to run
Ask: "Want me to run it now and produce the import file?" If yes:
1. Write `branches/<branch>.py` from the contract, following [script-rules.md](../scripted-arithmetic/script-rules.md) (constants block, all required checks, all three outputs).
2. Print the echo lines.
3. Run the script on the source file, with the period taken from the answer to Round 1.
4. Report the result of every check. If any check failed, show the failures and ask how to resolve them. Do not change data or checks to make them pass.
