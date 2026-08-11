---
title: "A Debt Payoff Calculator in Google Sheets: Snowball vs. Avalanche with a Rolling Extra-Payment Formula"
date: 2026-08-11
description: "Build a Google Sheets calculator that ranks debts by snowball or avalanche order and rolls freed-up minimum payments into the next target automatically."
tags: ["google sheets", "formulas", "debt payoff"]
---

Snowball and avalanche are just two different sort orders applied to the same debt list. Once you see them that way, the spreadsheet part is mechanical: rank the debts, pick a target, and write a formula that redirects freed-up minimum payments toward whichever debt is currently first in line. This post builds that calculator step by step, with no opinion on which order you should use.

## Setting Up the Debt Table

Start with a plain list. Four columns is enough:

| Name | Balance | APR | Min Payment |
|---|---|---|---|
| Card A | 4200 | 22% | 120 |
| Card B | 1800 | 18% | 60 |
| Loan C | 9600 | 8% | 210 |

Those numbers are placeholders. Pull your own balance, APR, and minimum payment from your statements before using this for anything real.

## Ranking Debts Two Ways

Avalanche order sorts by APR, highest first:

```
=RANK(C2,$C$2:$C$4,0)
```

Snowball order sorts by balance, lowest first:

```
=RANK(B2,$B$2:$B$4,1)
```

The third argument controls direction. `0` (or omitting it) ranks descending, so the largest APR gets rank 1. `1` ranks ascending, so the smallest balance gets rank 1. Put both formulas in adjacent columns and you can flip strategies just by pointing the schedule at a different rank column.

## Estimating Payoff Time in Isolation

Before building the full rolling schedule, it's worth knowing how long each debt would take on its own, paying only the minimum:

```
=NPER(C2/12,-D2,B2)
```

`NPER` returns the number of monthly periods needed to bring the balance to zero given a monthly rate and a fixed payment. The payment argument is negative because it's money leaving the balance. If a debt's minimum payment doesn't cover its monthly interest, this formula returns an error or a huge number, which is itself useful information about that row.

## Building the Rolling Extra-Payment Schedule

The interesting mechanic is what happens to a debt's minimum payment once it hits zero: that money doesn't disappear, it gets added to whatever extra budget you're throwing at the current target debt. Build a month-by-month table where each row is a month and each column is a debt's remaining balance.

First, figure out which debt is the current target. It's the one with the lowest rank number among debts that still have a balance:

```
=MINIFS(Debts!$F$2:$F$4, B10:D10, ">0")
```

Here `B10:D10` holds last month's ending balances for the three debts, and `Debts!$F$2:$F$4` is whichever rank column you chose. Debts at zero are excluded automatically because their balance fails the `">0"` test.

Next, total up the extra payment pool for this month: your fixed monthly extra, plus the minimum payments of any debts that already hit zero.

```
=ExtraBudget + SUMIF(B10:D10, 0, Debts!$D$2:$D$4)
```

`SUMIF` checks last month's balances for zeros and, wherever it finds one, adds that debt's minimum payment from the debt table into the pool.

Finally, roll each balance forward one month. For Card A:

```
=IF(B10<=0, 0, MAX(0, B10*(1+Debts!$C$2/12) - (Debts!$D$2 + IF(Debts!$F$2=E11, F11, 0))))
```

Read it right to left. Interest accrues on last month's balance at the monthly rate. Then the minimum payment comes off, and if this debt's rank matches this month's target rank (`E11`), the whole extra pool (`F11`) comes off too. `MAX(0, ...)` stops the balance from going negative once it's paid off. Copy the same pattern across for Card B and Loan C, swapping in each debt's own APR and minimum payment cells.

## Reading the Output

Once the table is built, two things fall out for free. `COUNTIF` on each balance column tells you which month it hit zero:

```
=MATCH(0, B11:B60, 0)
```

And summing each debt's interest portion per month (`B10*Debts!$C$2/12`, before the payment is subtracted) across the whole schedule gives you total interest paid under that ranking. Swap the rank column feeding `MINIFS` from avalanche to snowball, recalculate, and compare the two totals side by side. The spreadsheet will show you the difference in months and in dollars. What you do with that comparison is up to you.
