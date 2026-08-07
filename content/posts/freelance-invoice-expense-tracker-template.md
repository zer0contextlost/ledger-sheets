---
title: "Freelance Invoice and Expense Tracker: A Two-Tab Spreadsheet That Scales"
date: 2026-08-07
description: "A Google Sheets template for freelancers: an invoice log with automatic overdue flags, and an expense tab that rolls up by category and quarter for tax prep."
tags: ["freelancing", "invoicing", "google-sheets", "templates"]
---

Freelancers need two things a generic budget spreadsheet doesn't give
them: a record of what's been invoiced versus what's actually been paid,
and expenses sorted in a way that's easy to hand off at tax time. What
counts as deductible where you live is a question for a tax professional,
not a spreadsheet post. This covers the tabs and the formulas.

## Tab 1: Invoice log

Columns: `Invoice #`, `Client`, `Date Sent`, `Due Date`, `Amount`, `Status`
(dropdown: `Sent`, `Paid`, `Overdue`).

Compute Status instead of updating it by hand:

```
=IF(Status="Paid", "Paid", IF(TODAY() > DueDate, "Overdue", "Sent"))
```

Status only ever gets manually set to `Paid`, and only when payment
actually arrives. Everything else derives from the due date, so nothing
goes stale because you forgot to update a cell.

Total outstanding:

```
=SUMIF(Status_range, "<>Paid", Amount_range)
```

Total overdue, which is the number worth watching:

```
=SUMPRODUCT((Status_range<>"Paid") * (TODAY()>DueDate_range) * Amount_range)
```

`SUMPRODUCT` handles this better than a second `SUMIF` because the
condition spans two ranges at once, status and date. `SUMIFS` could do it
too, but `SUMPRODUCT` reads more directly once a boolean condition and a
date comparison sit in the same expression.

## Tab 2: Expense log

Columns: `Date`, `Vendor`, `Category`, `Amount`, `Quarter` (computed, not
typed).

Quarter as a formula:

```
=ROUNDUP(MONTH(Date)/3, 0)
```

Any date turns into 1, 2, 3, or 4. Combined with `YEAR(Date)`, that's a
clean grouping key for a pivot table, and it's never manually wrong.

## Rolling expenses up by category and quarter

Build a pivot table with `Category` as rows, `Quarter` as columns, `Amount`
summed as values. Because Quarter is a formula column, the pivot stays
accurate all year. There's no year-end reclassification step to remember.

## Linking the two tabs

On a `Summary` tab:

```
=SUMIF('Invoice Log'!Status, "Paid", 'Invoice Log'!Amount) - SUM('Expense Log'!Amount)
```

Paid income minus total expenses. This uses *paid* invoices, not sent
ones. An unpaid invoice isn't income yet, no matter what's been billed.

## Why two tabs, not one

A single mixed log of income and expenses works for very light use. But
the overdue logic only applies to invoices, and the quarter rollup is
mostly useful for expenses. Splitting them keeps each tab's formulas
specific to what it tracks, instead of half the columns sitting unused on
every row.
