---
name: demo
description: Write a sample Stripe file and a sample chart of accounts into the current folder so the user can try grill-books without exporting anything.
disable-model-invocation: true
---

1. Check the current folder for `stripe_balance_sep.csv` and `chart-of-accounts.csv`. If either exists, say so and stop. Never overwrite a file.
2. Write the two files below into the current folder, exactly as shown: same lines, same quoting, ending with one newline. Do not reformat, reorder or add anything.
3. Tell the user, in no more than 4 lines:
   - The two files you wrote: a September Stripe balance export (10 transactions, 2 payouts) and an 8-account chart of accounts.
   - Next, run: `/ledgerskill:grill-books`
   - When the questions come, reply `use your recommendations` to accept every recommended answer.

## stripe_balance_sep.csv

```csv
id,created,type,reporting_category,amount,fee,net,currency,payout_id,location
txn_001,2026-09-01 10:14:00,charge,charge,120.00,3.78,116.22,usd,po_A1,LOC01
txn_002,2026-09-01 11:02:00,charge,charge,45.50,1.62,43.88,usd,po_A1,LOC02
txn_003,2026-09-01 15:40:00,charge,charge,"1,250.00",36.55,"1,213.45",usd,po_A1,LOC01
txn_004,2026-09-02 09:05:00,refund,refund,(45.50),0.00,(45.50),usd,po_A1,LOC02
txn_005,2026-09-02 12:30:00,charge,charge,310.00,9.29,300.71,usd,po_A1,
txn_006,2026-09-03 08:00:00,payout,payout,"(1,628.76)",0.00,"(1,628.76)",usd,po_A1,
txn_007,2026-09-03 13:20:00,charge,charge,88.00,2.85,85.15,usd,po_B2,LOC02
txn_008,2026-09-04 10:45:00,charge,charge,640.25,18.87,621.38,usd,po_B2,LOC01
txn_009,2026-09-04 16:10:00,adjustment,adjustment,-15.00,0.00,-15.00,usd,po_B2,LOC01
txn_010,2026-09-05 08:00:00,payout,payout,(691.53),0.00,(691.53),usd,po_B2,
```

## chart-of-accounts.csv

```csv
Account #,Full name,Type,Detail type,Description
1000,Checking,Bank,Checking,Operating account
1099,Stripe Clearing,Other Current Assets,Undeposited Funds,Stripe balance awaiting payout
1200,Accounts Receivable (A/R),Accounts receivable (A/R),Accounts Receivable (A/R),
2000,Accounts Payable (A/P),Accounts payable (A/P),Accounts Payable (A/P),
4000,Sales,Income,Sales of Product Income,
4050,Refunds and Returns,Income,Discounts/Refunds Given,
6150,Merchant Fees,Expenses,Bank Charges,Card processing fees
6900,Payment Processor Adjustments,Expenses,Other Miscellaneous Service Cost,Stripe adjustments and disputes
```
