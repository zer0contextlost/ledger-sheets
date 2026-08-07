---
title: "Building a Loan Amortization Schedule in Google Sheets"
date: 2026-08-07
description: "How to build a full month-by-month amortization schedule in Sheets using PMT, IPMT, and PPMT, plus a running balance that hits exactly zero at term end."
tags: ["loans", "google-sheets", "templates"]
---

An amortization schedule shows, for every payment period, how much of a
fixed payment goes to interest versus principal, and what the remaining
balance is. Sheets has three built-in functions that do the math directly
— this covers how they fit together, not whether a particular loan or rate
is a good deal, which depends on numbers specific to an actual offer.

## The three functions

Given a loan amount (`principal`), an annual rate (`rate`), and a term in
months (`term`):

```
=PMT(rate/12, term, -principal)
```

`PMT` returns the fixed monthly payment. `rate/12` converts an annual rate
to a monthly one; `principal` is negated because Sheets treats it as an
outflow from the lender's perspective by convention — leaving it positive
would return a negative payment.

```
=IPMT(rate/12, period, term, -principal)
=PPMT(rate/12, period, term, -principal)
```

`IPMT` and `PPMT` split that same fixed payment into interest and principal
for a specific `period` (e.g., month 7 of 360). `IPMT + PPMT` for any given
period always equals the `PMT` result — that's a useful sanity check when
setting up the sheet.

## Building the schedule

Columns: `Period`, `Payment`, `Interest`, `Principal`, `Balance`.

Row for period 1:

```
Payment:   =PMT($B$1/12, $B$2, -$B$3)
Interest:  =IPMT($B$1/12, A2, $B$2, -$B$3)
Principal: =PPMT($B$1/12, A2, $B$2, -$B$3)
Balance:   =$B$3 - SUM($D$2:D2)
```

Here `$B$1` is annual rate, `$B$2` is term in months, `$B$3` is principal,
and `A2` is the period number. The `$` locks give an absolute reference for
the loan constants and a relative one for the period, so dragging the
formula down 359 more rows recalculates each period correctly without
retyping anything.

`Balance` sums *all* principal paid so far and subtracts from the original
loan — not just "previous balance minus this period's principal" — because
that formulation is self-correcting if you ever insert or reorder rows,
where a chained previous-row reference would break.

## Verifying it's built correctly

The last row's `Balance` should equal exactly 0 (allowing for floating-point
rounding in the 12th decimal place, which is normal). If it doesn't, the
usual culprits are: `rate` and `term` not both being monthly (mixing an
annual rate with a monthly term or vice versa), or `principal` not
negated consistently across all three functions.

## Extra payments

To model an extra principal payment in a given month, add an
`ExtraPayment` column and change `Balance` to:

```
=$B$3 - SUM($D$2:D2) - SUM($F$2:F2)
```

where `F` is the extra-payment column. This shortens the effective payoff
period but doesn't recompute the fixed `Payment` amount — modeling a
recast (where the payment itself drops after a lump-sum payment) requires
re-running `PMT` with the new remaining balance and remaining term as
inputs to a second schedule starting from that point, rather than a single
continuous formula.

## A recap you can build once and reuse

Once the five-column structure and the four core formulas are in place,
this schedule is a template — plug in a different rate, term, or principal
in the three top-of-sheet input cells and the entire schedule below
recalculates. That reusability is the actual point of building it as
formulas rather than a one-off calculation.
