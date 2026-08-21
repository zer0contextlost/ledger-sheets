---
title: "A Client Retainer Tracker in Google Sheets: Banked Hours, Rollover Caps, and Overage Billing"
date: 2026-08-21
description: "Build a Google Sheets retainer tracker that calculates hours used, rolls over unused hours up to a cap, and bills overage automatically."
tags: ["google sheets", "freelancing", "formulas"]
---

Retainer agreements break down the moment the tracking gets sloppy. A client pays for 15 hours a month, some months you use 11, some months you use 19, and by month four nobody remembers what's banked, what expired, and what should get billed as overage. This is a two-tab spreadsheet that keeps a running answer to all three questions using formulas instead of a memory.

## The two-tab structure

**Log tab** holds one row per work session: date, hours, task description, client (if you run this across multiple retainers). Nothing calculated here, just raw entries.

**Summary tab** has one row per billing month, with columns for the monthly allotment, hours used, rollover in, rollover out, overage hours, and overage amount. This is where all the logic lives.

## Pulling hours used from the log

Each month's row on the Summary tab needs a sum of everything logged in that period. Assuming Log tab columns are A (date), B (hours), C (task):

```
=SUMIFS(Log!B:B, Log!A:A, ">="&B2, Log!A:A, "<"&EDATE(B2,1))
```

Here B2 holds the first day of the billing month, and EDATE(B2,1) gets the first day of the next month. That gives you a clean sum with no off-by-one errors from month lengths.

## Rolling over unused hours, with a cap

Most retainer agreements that allow rollover still cap it, otherwise a slow quarter turns into an unlimited hour bank. The rollover-out formula for a given month needs three inputs: rollover coming in, the monthly allotment, and hours used:

```
=MIN(RolloverCap, MAX(0, [RolloverIn] + [Allotment] - [HoursUsed]))
```

The inner MAX(0, ...) stops the rollover from going negative when someone underuses by a lot in a month that already had banked hours. The outer MIN caps it at whatever ceiling the agreement specifies, say 10 hours. Set RolloverCap as a named range so you can change it in one place if a contract terms change.

Next month's RolloverIn cell just references this month's RolloverOut:

```
=D2
```

where D2 is the prior row's rollover-out cell. This is the same row-referencing-the-row-above pattern you'd use in an amortization schedule, it's what makes the balance carry forward without a script.

## Billing overage automatically

Overage hours are whatever's left after you've drawn down the allotment and any banked rollover:

```
=MAX(0, [HoursUsed] - ([Allotment] + [RolloverIn]))
```

Multiply by the agreed overage rate to get a billable amount:

```
=[OverageHours] * OverageRate
```

Where OverageRate is another named range, since it's common for it to differ from the effective hourly rate baked into the retainer itself.

## Handling rollover expiration

Some agreements let hours roll over for one month only, not indefinitely. To model that, track rollover in two pieces instead of one lump sum: hours rolled from two months ago (which expire this month if unused) and hours rolled from last month.

```
=MAX(0, [RolloverIn_2moAgo] - [HoursUsed])
```

If hours used exceeds what's available from the older rollover, whatever's left of that older batch is dropped rather than added to the current bank. Whether this matters depends entirely on the contract language, so read the agreement before deciding if your sheet needs one rollover bucket or two.

## A minimal summary row

Put it together and one month's row looks like this, with named ranges for Allotment (15) and RolloverCap (10):

| Month | Rollover In | Hours Used | Rollover Out | Overage Hrs | Overage $ |
|---|---|---|---|---|---|
| Jan | 0 | 11 | 4 | 0 | $0 |
| Feb | 4 | 19 | 0 | 0 | $0 |
| Mar | 0 | 22 | 0 | 3 | $225 |

Row three shows the mechanism working as intended: 15 allotted plus 0 banked leaves 15 hours before overage kicks in, and the 22 hours logged trigger a 3-hour overage charge at a $75 illustrative rate. Swap in whatever rate and cap your actual agreement specifies, the formulas don't change, only the named ranges do.
