---
title: "Building a Break-Even Calculator in Google Sheets: Contribution Margin and a What-If Table"
date: 2026-08-07
description: "How to build a break-even calculator in Google Sheets using contribution margin formulas, plus an ARRAYFORMULA sensitivity table for price and cost changes."
tags: ["break-even analysis", "google sheets", "formulas"]
---

Break-even analysis answers one question: how many units before revenue
covers fixed costs. Building it in a spreadsheet instead of running the
math once means you can change any input, price, cost per unit, fixed
overhead, and see the break-even point move immediately. Nothing here is
pricing or business strategy advice. Just the calculator and how to read
it.

## Setting up the inputs

Four inputs, each in its own cell so every formula below references them
instead of hardcoding numbers:

| Cell | Label | Example value |
|------|-------|---------------|
| B2 | Price per unit | 25 |
| B3 | Variable cost per unit | 10 |
| B4 | Fixed costs (monthly) | 3000 |
| B5 | Current unit sales | 250 |

Variable cost per unit is everything that scales with each sale:
materials, per-transaction fees, shipping. Fixed costs don't change with
volume in the period you're measuring: rent, software subscriptions,
salaried labor.

## The core formulas

Contribution margin per unit, what's left after variable costs to put
toward fixed costs:

```
=B2-B3
```

Contribution margin ratio, the same thing as a percentage of price:

```
=(B2-B3)/B2
```

Break-even in units, fixed costs divided by contribution margin per unit:

```
=B4/(B2-B3)
```

With the example numbers, `3000/(25-10)` is 200 units. Below 200, fixed
costs aren't fully covered. Above it, each unit's contribution margin is
surplus.

Break-even in revenue, for when you sell multiple products at different
prices and "units" isn't one clean number:

```
=B4/((B2-B3)/B2)
```

This is the dollar amount of sales required, independent of unit count.

Margin of safety, how far current sales sit above or below break-even:

```
=(B5-B4/(B2-B3))/B5
```

At B5 = 250 and break-even at 200, that returns 20%. Sales could drop 20%
before hitting break-even.

## A what-if table for price changes

One break-even number only holds until someone asks what happens at $22
instead of $25. Rather than editing B2 repeatedly, put candidate prices in
D2:D8 (20, 22, 24, 25, 26, 28, 30), then in E2:

```
=ARRAYFORMULA(B4/(D2:D8-B3))
```

This spills break-even units for every price in that column against the
same fixed and variable cost inputs. A small price cut near a thin margin
moves break-even a lot more than the same cut against a wide margin, and
scanning the column shows that directly.

Do the same for variable cost instead of price to see how a shipping rate
increase moves the break-even point.

## Two-variable sensitivity with SEQUENCE

For break-even units across a full grid of price and fixed-cost
combinations rather than one variable at a time:

```
=ARRAYFORMULA(SEQUENCE(1,6,2000,500)/(20-B3))
```

`SEQUENCE(1,6,2000,500)` generates six fixed-cost values starting at 2000
in steps of 500. Dividing by a fixed contribution margin (`20-B3`) returns
break-even units for each. Nest the row and column ranges together for a
full grid instead of a single strip.

## Putting it together

Three sections: the four raw inputs, the four formulas that reference
them, one what-if table. Shade the input cells so it's obvious which are
meant to be edited and which are calculated. That's what makes it a
calculator someone reuses next month with new numbers instead of a
one-time calculation rebuilt from scratch.
