---
title: "A 13-Week Cash Flow Forecast in Google Sheets: Weekly Buckets and a Running Balance"
date: 2026-08-14
description: "Build a rolling 13-week cash flow forecast in Google Sheets using SUMIFS to bucket transactions by week and a running balance formula."
tags: ["google-sheets", "cash-flow"]
---

A 13-week cash flow forecast is a rolling view of money in and money out, broken into weekly buckets instead of monthly ones. Freelancers and small operators use it because a month is too coarse to catch a week where three invoices are late and a tax payment lands on the same Friday. The spreadsheet mechanics are just date bucketing, SUMIFS, and a running balance. Here's how to build it.

## Sheet structure

Use two tabs. `Transactions` holds every actual and projected cash movement:

| Date | Description | Type | Amount |
|------|-------------|------|--------|
| 2026-08-17 | Client invoice #114 | Income | 2400 |
| 2026-08-18 | Software subscriptions | Expense | 85 |
| 2026-08-21 | Contractor payment | Expense | 600 |

`Type` is either "Income" or "Expense", entered as a positive number regardless of direction. Keeping the sign out of the amount column avoids double-negative bugs later when you sum things.

The `Forecast` tab has 13 columns, one per week, with a date in row 1 marking each week's start:

```
        B1          C1          D1     ...
Week    8/17/2026   8/24/2026   8/31/2026
```

Each subsequent week's header is just the prior one plus 7:

```
=B1+7
```

## Bucketing transactions into weeks

Row 3 pulls total income for each week by matching the transaction date against that week's start and the following week's start:

```
=SUMIFS(Transactions!$D:$D, Transactions!$C:$C, "Income", Transactions!$A:$A, ">="&B$1, Transactions!$A:$A, "<"&B$1+7)
```

Row 4 does the same for expenses, changing only the type:

```
=SUMIFS(Transactions!$D:$D, Transactions!$C:$C, "Expense", Transactions!$A:$A, ">="&B$1, Transactions!$A:$A, "<"&B$1+7)
```

Copy both formulas across all 13 columns. Because the date bounds reference `B$1` relatively (column shifts, row stays fixed) and the ranges reference `Transactions!` absolutely, each column automatically pulls its own week without any manual editing.

Net change for the week is row 3 minus row 4:

```
=B3-B4
```

## The running balance

This is the part that makes the forecast useful instead of just a chart of weekly totals. Put a starting cash balance in a fixed cell, say `A2`, then build the ending balance row so each week's ending balance feeds into the next week's beginning balance.

Beginning balance for week 1:

```
=A2
```

Beginning balance for week 2 onward (in cell C6, referencing week 1's ending balance in B7):

```
=B7
```

Ending balance for each week:

```
=B6+B5
```

Where row 6 is beginning balance and row 5 is net change. Drag both rows across, and the balance rolls forward automatically. If income in week 4 comes in light, every week after it shows the drop without you touching a single cell.

## Finding the tightest week

Once the running balance row is built, `MIN` tells you the lowest point in the 13-week window:

```
=MIN(B7:N7)
```

To find which week that low point falls on, pair `INDEX` with `MATCH` against the header row:

```
=INDEX(B1:N1, MATCH(MIN(B7:N7), B7:N7, 0))
```

That returns the date of the week where the balance bottoms out, which is more useful than the number alone when you're scanning the sheet quickly.

## Adding a forecast-vs-actual check

If you're logging both projected and actual transactions, add a `Status` column to `Transactions` ("Forecast" or "Actual") and split the SUMIFS with an extra criteria pair:

```
=SUMIFS(Transactions!$D:$D, Transactions!$C:$C, "Income", Transactions!$B:$B, "Actual", Transactions!$A:$A, ">="&B$1, Transactions!$A:$A, "<"&B$1+7)
```

Subtracting this from the all-status total for the same week gives you the outstanding forecasted amount still expected to land, which is handy for spotting weeks that are more guess than fact.

## Keeping it rolling

To keep the sheet at 13 weeks as time passes, don't hardcode 13 static columns forever. Once week 1 closes, delete that column and add a new week 13 at the end with a header formula referencing the prior week plus 7. The running balance formulas don't need to change since they reference the row above them, not a fixed starting column. The only manual step is dropping in the new week's header date and copying the SUMIFS formulas one column to the right.
