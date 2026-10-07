---
name: data-contract
description: Use when describing, writing, reading or checking how a financial source file becomes ledger lines — grain, keys, scope, mapping, dimensions, offset or control totals. Use whenever a branch file in branches/ is created, edited or followed.
---

Every statement about how source rows become output lines uses the terms and slots in [data-contract.md](data-contract.md), and only those terms. Read it in full before writing or following a contract.

- Write contracts with the exact slot template in that file. Every slot is filled; "None" is a valid value.
- Column names go in backticks; values are quoted exactly as they appear in the file.
- Before producing any output from a contract, print the two echo lines defined there.
