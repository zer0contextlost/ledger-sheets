---
title: "A 50/30/20 Budget Tracker in Google Sheets, Formula by Formula"
date: 2026-08-07
description: "How to build a self-updating 50/30/20 budget tracker in Google Sheets: category tagging, SUMIF rollups, and a progress bar that doesn't need manual updates."
tags: ["budgeting", "google-sheets", "templates"]
---

The 50/30/20 rule splits take-home income into three buckets — needs,
wants, savings/debt — in roughly a 50%, 30%, 20% ratio. It's a starting
framework, not a rule everyone's budget should fit exactly; the useful part
for a spreadsheet is that it gives you three categories to tag transactions
against and measure against a target. This post covers the mechanics of
building that tracker, not whether the 50/30/20 split is right for your
situation.

## Sheet layout

Two tabs: `Transactions` (raw entries) and `Summary` (the rollup). Keeping
raw data and rollups separate means you can add a transaction without
touching any formulas.

`Transactions` columns:

| Date | Description | Amount | Category |
|------|-------------|--------|----------|
| 2026-08-01 | Rent | 1400 | Needs |
| 2026-08-02 | Streaming | 15 | Wants |
| 2026-08-03 | 401k transfer | 300 | Savings |

Category is a dropdown restricted to `Needs`, `Wants`, `Savings` via
**Data → Data validation → Dropdown**, so rollup formulas below can trust
the values exactly match.

## The rollup formulas

On `Summary`, with income in `B1`:

```
=SUMIF(Transactions!D:D, "Needs", Transactions!C:C)
=SUMIF(Transactions!D:D, "Wants", Transactions!C:C)
=SUMIF(Transactions!D:D, "Savings", Transactions!C:C)
```

Each pulls the total for one category regardless of row count or order —
`SUMIF` scans the whole range, so new transaction rows don't require
touching the formula.

To get each category as a percentage of income:

```
=SUMIF(Transactions!D:D, "Needs", Transactions!C:C) / B1
```

Format that cell as a percentage (**Format → Number → Percent**) rather
than multiplying by 100 manually — keeps the underlying value a true
fraction, which matters if you chart it later.

## A progress bar without a chart

`SPARKLINE` can render a simple horizontal bar inline in a cell, which
reads faster than a full chart for "am I over or under target":

```
=SPARKLINE(B2/B1, {"charttype","bar";"max",0.5})
```

Here `B2` is the Needs total and `0.5` is 50% expressed as a fraction —
the bar fills relative to that max, so it visually clips right at the
target ratio.

## Flagging over-target categories conditionally

Conditional formatting rule on the percentage cells: **Format → Conditional
formatting → Custom formula is**:

```
=B2/$B$1 > 0.5
```

Set the format to a highlight color. This flags the Needs percentage the
moment it crosses 50% of income, without any manual checking.

## Handling categories that don't cleanly fit

Real spending rarely sorts perfectly into three buckets — a phone bill is
partly "need," a subscription might be partly work-related. The formulas
above don't need a fourth category to handle this; they only care that
every row has *some* value in the Category column. Decide your own
tie-breaking rule once (e.g., "phone goes to Needs") and apply it
consistently, since the SUMIF totals are only as meaningful as the
categorization behind them.

This structure — raw log, dropdown-constrained categories, SUMIF rollups —
extends past three categories too; the same pattern works for a 6-category
zero-based budget, just with more SUMIF rows on the Summary tab.
