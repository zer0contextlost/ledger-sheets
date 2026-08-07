---
title: "Building a Break-Even Calculator in Google Sheets: Contribution Margin and a What-If Table"
date: 2026-08-07
description: "How to build a break-even calculator in Google Sheets using contribution margin formulas, plus an ARRAYFORMULA sensitivity table for price and cost changes."
tags: ["break-even analysis", "google sheets", "formulas"]
---

Break-even analysis answers one narrow question: how many units do you need to sell before revenue covers fixed costs? It's a small piece of math, but doing it in a spreadsheet instead of a one-off calculation means you can change any input — price, cost per unit, fixed overhead — and immediately see how the break-even point shifts. This post walks through the formulas, then builds a sensitivity table so you can see break-even across a range of prices at once.

This is a mechanics post, not advice on pricing or business strategy — just how to build the calculator and read what it outputs.

## Setting up the inputs

Put these four inputs in their own cells so every formula below can reference them instead of hardcoding numbers:

| Cell | Label | Example value |
|------|-------|---------------|
| B2 | Price per unit | 25 |
| B3 | Variable cost per unit | 10 |
| B4 | Fixed costs (monthly) | 3000 |
| B5 | Current unit sales | 250 |

"Variable cost per unit" is everything that scales with each sale — materials, per-transaction fees, shipping. "Fixed costs" is everything that doesn't change with volume in the period you're measuring — rent, software subscriptions, salaried labor.

## The core formulas

**Contribution margin per unit** — how much of each sale is left after variable costs, to put toward fixed costs:

```
=B2-B3
```

**Contribution margin ratio** — the same thing expressed as a percentage of price, useful for the revenue-based break-even below:

```
=(B2-B3)/B2
```

**Break-even in units** — fixed costs divided by contribution margin per unit:

```
=B4/(B2-B3)
```

With the example numbers above, that's `3000/(25-10)` = 200 units. Below 200 units sold, fixed costs aren't fully covered; above it, each additional unit's contribution margin becomes surplus.

**Break-even in revenue** — useful when you sell multiple products at different prices and "units" isn't a clean single number:

```
=B4/((B2-B3)/B2)
```

This gives the dollar amount of sales required, independent of how many individual units that took.

**Margin of safety** — how far current sales are above (or below) the break-even point, as a percentage:

```
=(B5-B4/(B2-B3))/B5
```

With B5 at 250 units and break-even at 200, that returns 20% — current sales could drop 20% before hitting break-even.

## A what-if table for price changes

A single break-even number is only useful until you ask "what if price were $22 instead of $25?" Rather than editing B2 repeatedly, build a small table that recalculates break-even for a range of prices in one shot.

Put a column of candidate prices in D2:D8 (e.g., 20, 22, 24, 25, 26, 28, 30), then in E2:

```
=ARRAYFORMULA(B4/(D2:D8-B3))
```

This spills break-even units for every price in that column, using the same fixed cost and variable cost inputs. Scanning the column shows how sensitive the break-even point is to price — small price cuts near a thin contribution margin move break-even a lot more than the same cut does when margins are wide.

You can do the same thing for variable cost instead of price — put candidate variable costs in a column and reuse the break-even formula against them — to see how a per-unit cost increase (say, a shipping rate change) affects the units needed.

## Two-variable sensitivity with SEQUENCE

If you want break-even units across a full grid of price *and* fixed-cost combinations rather than one variable at a time, `SEQUENCE` combined with array math avoids typing the formula into every cell manually:

```
=ARRAYFORMULA(SEQUENCE(1,6,2000,500)/(20-B3))
```

Here `SEQUENCE(1,6,2000,500)` generates six fixed-cost values starting at 2000 in steps of 500 (2000, 2500, 3000, ...), and dividing by a fixed contribution margin (`20-B3`) returns break-even units for each. Swap which side varies depending on which variable you want to test, and pair it with the price column from the previous section by nesting the row and column ranges if you want a full grid rather than a single strip.

## Putting it together

A minimal break-even sheet needs three sections: the four raw inputs, the four core formulas referencing them, and one what-if table. Keep the inputs visually separate (a shaded block at the top works fine) so it's obvious which cells are meant to be edited versus which are calculated — that's the difference between a calculator someone can actually reuse next month with new numbers, and a one-time calculation that gets rebuilt from scratch every time an assumption changes.
