---
title: "An Invoice Aging and Late Fee Calculator in Google Sheets"
date: 2026-08-18
description: "Build an aging bucket system and automatic late fee formula for overdue invoices in Google Sheets, using DATEDIF, nested IFS, and SUMIFS."
tags: ["google-sheets", "invoicing"]
---

An invoice list with a "Paid" column tells you what's outstanding. It doesn't tell you how outstanding, and it definitely doesn't calculate what a client owes you in late fees without you doing the math by hand every month. This template adds two things to a standard invoice tracker: an aging bucket column that sorts unpaid invoices by how many days overdue they are, and a late fee formula that compounds (or doesn't) based on your own terms.

## The base columns

Start with a normal invoice log:

| Invoice # | Client | Amount | Issue Date | Due Date | Date Paid |
|---|---|---|---|---|---|

`Due Date` should already reflect your payment terms (net 15, net 30, whatever you use). Don't calculate it inline from `Issue Date` on every row unless your terms never change. Better to just enter the due date directly, or compute it once with `=IssueDate + 30` in its own column so you can override individual rows for clients on different terms.

## Days overdue

The core calculation is a simple subtraction, but it needs to handle two cases: invoices that are already paid, and invoices that aren't due yet.

```
=IF(F2<>"", MAX(0, F2-E2-1),
  IF(TODAY()>E2, TODAY()-E2, 0))
```

Where `E2` is Due Date and `F2` is Date Paid. If the invoice was paid, this measures how many days late the payment actually was (subtracting 1 so a same-day payment on the due date shows 0, not 1). If it's unpaid, it checks whether today is past the due date and counts the gap. Either way, a fully current invoice returns 0.

## Aging buckets

Once you have days overdue as a number, bucket it. This is the same logic collections and AR software uses, just written as nested `IFS`:

```
=IFS(
  H2=0, "Current",
  H2<=30, "1-30 days",
  H2<=60, "31-60 days",
  H2<=90, "61-90 days",
  TRUE, "90+ days"
)
```

`H2` here is the Days Overdue cell from the formula above. `IFS` evaluates top to bottom and stops at the first true condition, so ordering matters. Put `TRUE` last as a catch-all rather than trying to write an explicit upper bound.

To see totals by bucket, use `SUMIFS` against the bucket column:

```
=SUMIFS(C:C, I:I, "31-60 days", F:F, "")
```

That sums the Amount column where the aging bucket matches and Date Paid is blank, so paid invoices that happened to sit in that range historically don't get counted. Repeat for each bucket to build a small summary table at the top of the sheet.

## Late fees

Late fee terms vary a lot by contract, so the formula below models a common structure: a flat percentage charged once an invoice crosses the due date, with no additional accrual after that. If your terms compound monthly, you'd multiply by the number of 30-day periods elapsed instead of using a flat trigger.

Flat one-time fee, applied once the invoice is at least 1 day overdue:

```
=IF(H2>0, C2*0.015, 0)
```

Here `0.015` is a stand-in for a 1.5% late fee, an illustrative number, not a legal or typical rate. Whatever percentage you actually charge should come from your invoice terms or contract, and in many places there are legal caps on what you can charge, so check your local rules before setting this for real.

If your terms charge the fee per 30-day period instead of a single flat amount:

```
=IF(H2>0, C2*0.015*ROUNDUP(H2/30,0), 0)
```

`ROUNDUP(H2/30,0)` turns "47 days overdue" into 2 periods, so a fee that started accruing on day 1 also applies once you cross into the second 30-day window. Adjust the period length to match your actual terms.

## Conditional formatting for the aging column

Formulas do the sorting, but color makes the aging table scannable at a glance. Select the Aging Bucket column, then add a custom formula rule per bucket:

- `=$I2="Current"` → default/no fill
- `=$I2="1-30 days"` → light yellow
- `=$I2="31-60 days"` → orange
- `=$I2="61-90 days"` → light red
- `=$I2="90+ days"` → dark red, bold text

Set these as four separate rules rather than one rule with a range, since Google Sheets conditional formatting doesn't support multiple output colors from a single condition.

One thing worth double-checking before you rely on this sheet: the `TODAY()` function recalculates every time the sheet opens, which means Days Overdue and the aging buckets shift automatically without you touching anything. That's useful for a live dashboard, but if you ever need to freeze a snapshot (end-of-quarter AR report, for instance), copy the values and paste them as plain numbers before the next recalculation moves the goalposts.
