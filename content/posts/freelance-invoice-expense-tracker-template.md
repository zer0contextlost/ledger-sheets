---
title: "Freelance Invoice and Expense Tracker: A Two-Tab Spreadsheet That Scales"
date: 2026-08-07
description: "A Google Sheets template for freelancers: an invoice log with automatic overdue flags, and an expense tab that rolls up by category and quarter for tax prep."
tags: ["freelancing", "invoicing", "google-sheets", "templates"]
---

Freelancers generally need two things a generic budget spreadsheet doesn't
give them: a record of what's been invoiced versus actually paid, and
expenses organized in a way that's easy to hand off at tax time. This
covers both as a two-tab template — the tabs and formulas, not what counts
as deductible where you live, which is a question for a tax professional
or your local tax authority's current guidance, not a spreadsheet post.

## Tab 1: Invoice log

Columns: `Invoice #`, `Client`, `Date Sent`, `Due Date`, `Amount`, `Status`
(dropdown: `Sent`, `Paid`, `Overdue`).

Rather than manually updating Status to `Overdue`, compute it:

```
=IF(Status="Paid", "Paid", IF(TODAY() > DueDate, "Overdue", "Sent"))
```

This assumes `Status` only ever gets manually set to `Paid` when payment
actually arrives — everything else derives from the due date automatically,
so nothing goes stale just because you forgot to update a cell.

Total outstanding (unpaid) invoices:

```
=SUMIF(Status_range, "<>Paid", Amount_range)
```

Total overdue specifically, which is the number worth watching:

```
=SUMPRODUCT((Status_range<>"Paid") * (TODAY()>DueDate_range) * Amount_range)
```

`SUMPRODUCT` is used here instead of a second `SUMIF` because the condition
combines two separate ranges (status AND date) — `SUMIF`/`SUMIFS` can do
multiple criteria too, but `SUMPRODUCT` reads more directly once you're
combining a boolean condition with a date comparison in the same
expression.

## Tab 2: Expense log

Columns: `Date`, `Vendor`, `Category`, `Amount`, `Quarter` (computed, not
typed).

Quarter as a formula, so it's never manually wrong:

```
=ROUNDUP(MONTH(Date)/3, 0)
```

This turns any date into 1, 2, 3, or 4. Combined with `YEAR(Date)`, you get
a clean grouping key for a pivot table without maintaining it by hand.

## Rolling expenses up by category and quarter

Build a pivot table (**Insert → Pivot table**) with `Category` as rows,
`Quarter` (computed) as columns, and `Amount` summed as values. Because
Quarter is a formula column rather than manual entry, the pivot table
stays accurate as you add rows all year — there's no year-end reclassification
step.

## Linking the two tabs for a net-income view

On a third `Summary` tab:

```
=SUMIF('Invoice Log'!Status, "Paid", 'Invoice Log'!Amount) - SUM('Expense Log'!Amount)
```

This gives paid income minus total expenses — a rough net figure. It
deliberately uses *paid* invoices, not sent ones, since unpaid invoices
aren't income yet regardless of what's been billed.

## Why two tabs instead of one combined log

A single mixed log of income and expenses works for very light use, but
splitting them means the `Status`/overdue logic (which only applies to
invoices) and the `Quarter` rollup (which is most useful for expenses)
don't have to coexist in one set of columns with half of them unused per
row. Two tabs with a summary formula pulling from both keeps each tab's
formulas simple and specific to what it's tracking.
