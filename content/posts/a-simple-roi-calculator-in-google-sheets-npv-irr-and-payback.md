---
title: "A Simple ROI Calculator in Google Sheets: NPV, IRR, and Payback Period"
date: 2026-08-07
description: "How to build a small ROI calculator in Google Sheets using NPV, IRR, XIRR, and a payback period formula, with the mechanics of each explained."
tags: ["google-sheets", "roi-calculator"]
---

Most "ROI calculator" templates floating around are a single cell dividing gain by cost. That works for a one-time, single-period return, but it falls apart the moment you have cash flows spread across multiple months or years, because it ignores when the money actually shows up. This post builds a small calculator that handles multi-period cash flows properly using NPV, IRR, and a payback period formula, and explains what each one is actually computing.

## Setting up the cash flow table

Every formula here depends on having your cash flows laid out in a single column, one row per period, starting with the initial outlay as a negative number.

| Period | Cash Flow |
|---|---|
| 0 | -10000 |
| 1 | 2500 |
| 2 | 3000 |
| 3 | 3200 |
| 4 | 3400 |
| 5 | 3600 |

Put periods in `A2:A7` and cash flows in `B2:B7`. Everything downstream references `B2:B7`.

## NPV: discounting future cash to today's terms

Net present value answers one question: given a discount rate, is this stream of cash flows worth more or less than doing nothing? Google Sheets' `NPV` function only discounts the flows you pass it, it doesn't automatically treat the first one as period 0, so you need to separate the initial outlay and add it back manually.

```
=NPV(discount_rate, B3:B7) + B2
```

Here `B3:B7` is periods 1 through 5, and `B2` (the period-0 outlay, already negative) gets added outside the discount function since it's not being discounted at all. The `discount_rate` cell holds an illustrative rate, say 8%, entered as `0.08` in a labeled input cell, not hardcoded into the formula. If your NPV comes out positive, the discounted future cash flows exceed the initial outlay at that rate. If it's negative, they don't.

A common mistake is feeding `B2:B7` straight into `NPV` and expecting it to know row 2 is "now." It doesn't. `NPV` discounts every value you give it by at least one period, so the outlay needs to sit outside the function.

## IRR: the rate that makes NPV zero

Internal rate of return is the discount rate at which NPV equals zero. Instead of you supplying a rate and getting a dollar answer, IRR flips the question: what rate would make this exact string of cash flows break even in present-value terms?

```
=IRR(B2:B7)
```

This one does include the period-0 outlay in the range, unlike `NPV`. `IRR` also accepts an optional second argument, a guess, which matters if the function fails to converge on unusual cash flow patterns (for example, flows that switch sign more than once):

```
=IRR(B2:B7, 0.1)
```

If your cash flows aren't evenly spaced (say, they land on irregular dates rather than clean annual or monthly intervals), use `XIRR` instead, which takes an actual date column:

```
=XIRR(B2:B7, C2:C7)
```

where `C2:C7` holds the actual calendar date of each cash flow.

## Payback period with partial-year interpolation

Payback period is simpler conceptually, how long until cumulative cash flow turns positive, but a clean formula for it requires a helper column. Add a cumulative cash flow column in `C`:

```
=SUM($B$2:B2)
```

filled down `C2:C7`. Then find the first period where the cumulative total crosses zero, and interpolate within that period rather than just rounding to the nearest whole period:

```
=MATCH(TRUE, C2:C7>=0, 0) - 1 + (-INDEX(C2:C7, MATCH(TRUE, C2:C7>=0, 0)-1) / INDEX(B2:B7, MATCH(TRUE, C2:C7>=0, 0)))
```

Enter this as an array formula (Ctrl+Shift+Enter in Excel, or wrap in `ARRAYFORMULA` in Sheets, since `C2:C7>=0` needs to evaluate across the whole range). The first term finds the last full period still in the red, and the fraction estimates how far into the next period the crossover happens, assuming cash arrives evenly within that period.

## Building a rate sensitivity table

Because NPV depends entirely on the discount rate you feed it, it's worth showing how the result shifts across a range of rates rather than committing to one. Set up a column of candidate rates in `E2:E10` (say 4% to 12% in 1% steps) and next to each one:

```
=NPV($E2, $B$3:$B$7) + $B$2
```

Filled down, this gives a quick table showing where NPV crosses from negative to positive as the rate increases, which is a visual way to see how sensitive the result is to the discount rate assumption without recalculating IRR by hand.

One thing worth checking before trusting any of this output: `IRR` can return multiple mathematically valid answers, or fail entirely, when cash flows change sign more than once (negative, then positive, then negative again). If your project has a cash outlay partway through, like a mid-project reinvestment, sanity-check the IRR result against the NPV table rather than taking it at face value.
