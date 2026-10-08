---
name: demo
description: Copy a sample Stripe file and a sample chart of accounts into the current folder so the user can try grill-books without exporting anything.
disable-model-invocation: true
---

1. Check the current folder for `stripe_balance_sep.csv` and `chart-of-accounts.csv`. If either exists, say so and stop. Never overwrite a file.
2. Copy [stripe_balance_sep.csv](stripe_balance_sep.csv) and [chart-of-accounts.csv](chart-of-accounts.csv) from this skill's directory into the current folder, byte for byte, with `cp`.
3. Tell the user, in no more than 4 lines:
   - The two files you copied: a September Stripe balance export (10 transactions, 2 payouts) and an 8-account chart of accounts.
   - Next, run: `/ledgerskill:grill-books stripe_balance_sep.csv`
   - When the questions come, reply `use your recommendations` to accept every recommended answer.
