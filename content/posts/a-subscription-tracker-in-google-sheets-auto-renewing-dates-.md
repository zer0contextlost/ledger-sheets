---
title: "A Subscription Tracker in Google Sheets: Auto-Renewing Dates and a Normalized Monthly Cost Formula"
date: 2026-08-11
description: "Build a subscription and recurring expense tracker in Google Sheets that calculates renewal dates automatically and normalizes annual, quarterly, and monthly charges into one comparable figure."
tags: ["google-sheets", "expense-tracking"]
---

A list of subscriptions with a "monthly cost" column looks fine until you add a yearly plan next to a monthly one. Now the column is lying to you, because $120/year and $12/month aren't the same kind of number even though they end up in the same cell. The fix is a small set of formulas that normalize billing cycles and recalculate renewal dates on their own, so the sheet stays accurate without you touching it every month.

## The column setup

Six columns do the work:

- **Service** (text)
- **Cost** (number, whatever the actual charge is)
- **Billing Cycle** (dropdown: Monthly, Quarterly, Annual)
- **Last Billed** (date)
- **Next Renewal** (formula)
- **Monthly Equivalent** (formula)

Put the billing cycle options in a data validation dropdown so the formulas below can match against exact text. Free-typed entries like "yearly" vs "Annual" will break a lookup.

## Calculating the next renewal date

`EDATE` adds whole months to a date, which handles month-length differences correctly (it won't turn Jan 31 into an invalid Feb 31). Wrap it in an `IF` that reads the billing cycle:

```
=IF(C2="Monthly", EDATE(D2,1),
  IF(C2="Quarterly", EDATE(D2,3),
    IF(C2="Annual", EDATE(D2,12), "")))
```

Here `D2` is Last Billed and `C2` is Billing Cycle. This gives you a single Next Renewal date regardless of cycle length, which means you can sort the whole sheet by that one column and see everything coming up in order.

## Normalizing cost to a monthly figure

This is the formula that actually makes the sheet useful for comparison. Divide or leave alone depending on cycle:

```
=IF(C2="Monthly", B2,
  IF(C2="Quarterly", B2/3,
    IF(C2="Annual", B2/12, "")))
```

Now every row in the Monthly Equivalent column means the same thing: what this subscription costs you per month, regardless of how often it actually bills. Summing that column with `=SUM(E2:E50)` gives a real total. Summing the raw Cost column would mix apples and oranges, since a $200 annual charge would swamp a $10 monthly one in the total even though the annual one costs less per month.

## Flagging what's renewing soon

A simple date comparison against `TODAY()` flags anything renewing in the next 7 days:

```
=IF(F2-TODAY()<=7, "Renews soon", "")
```

`F2` here is the Next Renewal cell. Conditional formatting can use the same logic directly, no helper column needed: **Format > Conditional formatting > Custom formula is**, then:

```
=$F2-TODAY()<=7
```

applied to the whole row range, with a fill color. That turns the sheet into something you can scan in two seconds instead of reading every date.

## Rolling the "Last Billed" date forward automatically

The one manual step left is updating Last Billed after a charge actually happens, since a spreadsheet can't detect that your card was charged. One option is a checkbox column ("Paid this cycle?") that, when checked, is meant to signal you to update Last Billed to today. Sheets can't trigger that update from a checkbox without a script, but you can use `TODAY()` combined with the checkbox as a visual cue:

```
=IF(G2=TRUE, "Update Last Billed to " & TEXT(TODAY(),"yyyy-mm-dd"), "")
```

where `G2` is the checkbox. It's a nudge, not automation, but it keeps the renewal-date formula from silently drifting out of sync with reality.

## Annual cost by category

If you're tagging subscriptions by category (software, streaming, memberships) in a separate column, `SUMIF` rolls up the annualized total per category:

```
=SUMIF(H2:H50, "Software", E2:E50) * 12
```

`H` is the category column, `E` is Monthly Equivalent. Multiplying by 12 converts the normalized monthly figure back into an annual view for that one category, which is a different question than "what does this cost me next month" and worth keeping as a separate cell rather than overloading one total.

## Why this beats a plain list

A flat list of charges answers "what am I paying for." A normalized tracker answers "what does this actually cost me per month" and "what's renewing before I forget about it," which are the two questions that matter when the list grows past ten or fifteen rows. Once the Monthly Equivalent and Next Renewal columns are in place, adding a new subscription is one row and zero new formulas, since both columns reference the row's own cells and copy down automatically when you drag the fill handle.
