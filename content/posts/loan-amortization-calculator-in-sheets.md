---
title: "Building a Loan Amortization Schedule in Google Sheets"
date: 2026-08-07
description: "How to build a full month-by-month amortization schedule in Sheets using PMT, IPMT, and PPMT, plus a running balance that hits exactly zero at term end."
tags: ["loans", "google-sheets", "templates"]
---

An amortization schedule shows, for every payment period, how much of a
fixed payment goes to interest versus principal, and what the balance is
afterward. Sheets has three built-in functions for this. Whether a
particular rate or term is a good deal depends on numbers specific to an
actual offer, which this post doesn't touch.

## The three functions

Given a loan amount (`principal`), an annual rate (`rate`), and a term in
months (`term`):

```
=PMT(rate/12, term, -principal)
```

`PMT` returns the fixed monthly payment. `rate/12` converts the annual rate
to a monthly one. `principal` is negated because Sheets treats it as an
outflow from the lender's side by convention; leave it positive and the
payment comes back negative.

```
=IPMT(rate/12, period, term, -principal)
=PPMT(rate/12, period, term, -principal)
```

`IPMT` and `PPMT` split that same payment into interest and principal for
a specific `period`, say month 7 of 360. Add them together for any given
period and you get the `PMT` result back. That's a useful check once the
sheet is built.

## Building the schedule

Columns: `Period`, `Payment`, `Interest`, `Principal`, `Balance`.

Row for period 1:

```
Payment:   =PMT($B$1/12, $B$2, -$B$3)
Interest:  =IPMT($B$1/12, A2, $B$2, -$B$3)
Principal: =PPMT($B$1/12, A2, $B$2, -$B$3)
Balance:   =$B$3 - SUM($D$2:D2)
```

`$B$1` is annual rate, `$B$2` is term in months, `$B$3` is principal, `A2`
is the period number. The dollar signs lock the loan constants while
leaving the period reference relative, so dragging the formula down 359
more rows recalculates each period without retyping anything.

`Balance` sums all principal paid so far and subtracts it from the
original loan, rather than chaining off the previous row's balance. That
matters if you ever insert or reorder rows: a chained reference breaks,
this doesn't.

## Verifying it

The last row's Balance should land on exactly 0, give or take
floating-point rounding in the twelfth decimal place. If it doesn't, check
that rate and term are both monthly or both annual, not mixed, and that
principal is negated consistently across all three functions.

## Extra payments

Add an `ExtraPayment` column and change Balance to:

```
=$B$3 - SUM($D$2:D2) - SUM($F$2:F2)
```

where `F` is the extra-payment column. This shortens the payoff period but
doesn't recompute the fixed payment amount. Modeling a recast, where the
payment itself drops after a lump sum, means running `PMT` again with the
new remaining balance and remaining term as inputs to a second schedule
starting from that point.

## Reusing it

Once the five columns and four formulas are in place, this is a template.
Change the rate, term, or principal in the three input cells and the whole
schedule below recalculates.
