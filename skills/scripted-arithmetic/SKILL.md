---
name: scripted-arithmetic
description: Use whenever counting, totaling, grouping, rounding, balancing or tying out numbers from a financial file, or when quoting any such number to the user. Use before writing any script that reads transaction data.
---

Never do arithmetic in your head. Every number you state about a financial file comes from the output of a script you ran.

Before writing a script that reads transaction data, read [script-rules.md](script-rules.md) in full and follow every rule. When a script reports a failure, show the failure to the user. Never edit numbers, drop rows or loosen a check to make it pass.
