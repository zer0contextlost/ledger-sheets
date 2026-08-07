---
title: "A 50/30/20 Budget Tracker in Google Sheets, Formula by Formula"
date: 2026-08-07
description: "How to build a self-updating 50/30/20 budget tracker in Google Sheets: category tagging, SUMIF rollups, and a progress bar that doesn't need manual updates."
tags: ["budgeting", "google-sheets", "templates"]
---

The 50/30/20 rule splits take-home income into needs, wants, and
savings/debt, roughly in a 50%, 30%, 20% ratio. Whether that split fits
your situation is a separate question. What it gives a spreadsheet is three
categories to tag transactions against and measure against a target, and
that's what this post builds.

## Sheet layout

Two tabs: `Transactions` for raw entries, `Summary` for the rollup. Add a
transaction and nothing else needs to change, because the formulas live on
a different tab.

`Transactions` columns:

| Date | Description | Amount | Category |
|------|-------------|--------|----------|
| 2026-08-01 | Rent | 1400 | Needs |
| 2026-08-02 | Streaming | 15 | Wants |
| 2026-08-03 | 401k transfer | 300 | Savings |

Restrict Category to a dropdown with **Data → Data validation → Dropdown**
(`Needs`, `Wants`, `Savings`). The rollup formulas below match against
these strings exactly, so a typo in a manually-typed category breaks the
totals silently.

## The rollup formulas

On `Summary`, with income in `B1`:

```
=SUMIF(Transactions!D:D, "Needs", Transactions!C:C)
=SUMIF(Transactions!D:D, "Wants", Transactions!C:C)
=SUMIF(Transactions!D:D, "Savings", Transactions!C:C)
```

Each scans the full column regardless of row count, so new transactions
don't require touching the formula.

As a percentage of income:

```
=SUMIF(Transactions!D:D, "Needs", Transactions!C:C) / B1
```

Format the cell as a percentage rather than multiplying by 100 by hand.
That keeps the underlying value a true fraction, which matters if you
chart it later.

## A progress bar without a chart

`SPARKLINE` renders a small horizontal bar inline in a cell:

```
=SPARKLINE(B2/B1, {"charttype","bar";"max",0.5})
```

`B2` is the Needs total, `0.5` is 50% as a fraction. The bar fills relative
to that max, so it visually clips right at the target ratio instead of
scaling to whatever the current value happens to be.

## Flagging over-target categories

Conditional formatting, custom formula:

```
=B2/$B$1 > 0.5
```

Set the highlight color and this flags the Needs percentage the moment it
crosses 50%. No manual checking required.

## Categories that don't fit cleanly

Real spending rarely sorts into three clean buckets. A phone bill is partly
a need and partly discretionary. The formulas above don't care about that
ambiguity; they only need every row to have some value in Category. Pick a
tie-breaking rule once, apply it consistently, and remember the SUMIF
totals are only as meaningful as the categorization behind them.

The same pattern, raw log plus dropdown-constrained categories plus SUMIF
rollups, works past three categories too. A six-category zero-based budget
uses the same structure with more rows on the Summary tab.
