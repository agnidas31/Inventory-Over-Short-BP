Field-Tested Innovation · US Convenience Retail · bp

# Creative Solutions: Inventory Over & Short

How store inventory audit results were turned from a pile of adjustments into a clear view of where stock was being lost, how fast, and which stores needed attention first.

Agnivesh Das · Data Analyst · bp

> **Note:** the structure and metrics reflect how the report worked. All stores, values and examples are illustrative, not real production figures.

---

## My Role

As Data Analyst, I owned the data behind the report end to end:

- **Data preparation.** Brought in inventory audit results, item and department hierarchies, store attributes and sales data across two convenience brands (Thorntons and ampm).
- **Cleaning and transformation.** Standardised audit categories across brands, aligned audits to store and period, and built the loss-per-day logic that makes stores with different audit timing comparable.
- **Reporting.** Built the Power BI report: monthly analysis, item-level inventory, department, category and account time series, outlier stores, latest audits, and loss per day.

---

## The Problem

### Shrink was measured, but not managed

Every convenience store loses some stock to theft, damage, spoilage, receiving errors and admin mistakes. This is known as **shrink**.

Periodic inventory audits count what's actually on the shelf and compare it with what the system says should be there. The difference is the **over/short**: a negative number means stock is missing.

The audit results existed, but they arrived as thousands of adjustments by store and category. It was hard to answer:

- Which stores are losing the most?
- Is it getting better or worse?
- Is the loss concentrated in specific categories, such as tobacco, beer or fresh food?
- Which stores should loss-prevention teams visit first?

---

## The Constraint Nobody Was Naming

### Audit totals weren't comparable between stores

**1. Audits happen at different intervals.** One store is audited after 30 days, another after 90. A $3,000 loss over 90 days is a smaller problem than a $2,000 loss over 30 days, but the raw totals say the opposite.

**2. Two brands, two ways of categorising.** Thorntons and ampm used different category names and structures, so a cigarette loss in one brand didn't line up with the other.

**3. Big stores lose more in dollars.** A high-volume store will naturally have bigger adjustments. Ranking by raw dollars sends teams to the busiest stores, not necessarily the worst-run ones.

---

## The Fix

### Normalise by time, standardise by category, rank by exception

1. **Loss per day.** Every audit adjustment is divided by the number of days since the store's previous audit, so stores audited at different intervals become comparable.
2. **A common category hierarchy.** Audit categories (beer, candy, carbonated and packaged beverages, cigarettes, dispensed beverages, fresh food, grocery and more) were mapped across both brands into one department and category structure.
3. **Outlier stores.** Stores were ranked by monthly loss, so the stores furthest from normal stood out immediately.
4. **Trend views.** Department, category and account time series show whether a store's or category's loss is improving or getting worse.

### How it works

1. **Ingest:** Audit results, item master and hierarchy, store attributes (brand, region, director, status), and sales.
2. **Standardise:** Map both brands' audit categories into one hierarchy.
3. **Align:** Link each audit to the previous audit at the same store to get the audit period length.
4. **Calculate:** Over/short in $, loss per day, and loss as a share of sales.
5. **Rank:** Identify outlier stores by loss per day and by trend.
6. **Report:** Monthly analysis, item level, time series by department, category and account, outlier stores, latest audits, and loss per day.

---

## Worked Example

### Why loss per day changes the ranking

> **Illustrative values only.**

| Store | Audit adjustment | Days since last audit | Loss per day |
| ----- | ---------------- | --------------------- | ------------ |
| A | −$3,000 | 90 | −$33 |
| B | −$2,000 | 30 | **−$67** |
| C | −$1,200 | 45 | −$27 |

Ranked by raw adjustment, Store A looks worst. Ranked by loss per day, **Store B is losing stock twice as fast** and should be visited first.

Drilling into Store B by category shows most of the loss is in cigarettes and beer. That points to a targeted response, such as tighter receiving checks or secure display, rather than a general store review.

---

## Why This Beat The Old Approach

|                          | Before                                   | With the Over & Short report                   |
| ------------------------ | ---------------------------------------- | ---------------------------------------------- |
| View of loss             | Raw audit adjustments                    | Loss per day, comparable across stores         |
| Categories               | Different per brand                      | One hierarchy across Thorntons and ampm        |
| Prioritisation           | Biggest dollar figure                    | Outlier stores and worsening trends            |
| Root cause               | Store-level total only                   | Drill-down to department, category and item    |
| Timing                   | Whenever someone compiled the numbers    | Refreshed with every audit                     |

---

## Impact

**22% better inventory recovery:** loss-prevention effort went where stock was actually being lost.

**4× faster prioritization:** the stores needing attention first surfaced in a fraction of the time.

**2 brands, 1 view:** Thorntons and ampm audit results in one consistent hierarchy.

**Comparable stores:** loss per day removed the distortion of different audit intervals.

**Targeted action:** outlier stores and problem categories surfaced immediately for loss-prevention teams.

The bigger shift was moving from "here are this month's audit adjustments" to "here is where we are losing stock fastest, and in what."

---
---

# Technical Deep Dive: How the Report Is Built

## 1. Source data

| Source | Grain | Used for |
| ------ | ----- | -------- |
| Inventory audit results | Store × audit date × audit category | Over/short $ |
| Item master & hierarchy | Item | Department, category, account |
| Store master | Store | Brand, region, director, open/closed status |
| Sales | Store × day × category | Loss as a share of sales |

## 2. Map categories across brands

A mapping table translates each brand's audit categories into one hierarchy:

```
brand      | source_category        | department      | category
-----------+------------------------+-----------------+-------------------
Thorntons  | Cigarettes Audit Adj   | Tobacco         | Cigarettes
ampm       | CIGS                   | Tobacco         | Cigarettes
Thorntons  | Beer Audit Adjust      | Alcohol         | Beer
...
```

Unmapped categories are flagged rather than dropped, so nothing silently disappears.

## 3. Audit period length

```sql
SELECT store_id,
       audit_date,
       category,
       adjustment_usd,
       DATEDIFF(day,
                LAG(audit_date) OVER (PARTITION BY store_id ORDER BY audit_date),
                audit_date)                                   AS days_in_period
FROM   inventory_audits;
```

## 4. Loss per day and share of sales

```
loss_per_day   = adjustment_usd ÷ days_in_period
loss_pct_sales = adjustment_usd ÷ sales_usd in the same period
```

## 5. Outlier detection

Two simple, explainable rules:

- **Rank:** stores ranked by monthly loss and by loss per day.
- **Peer comparison:** flag stores whose loss per day is far above the average for similar stores, for example more than two standard deviations above their region's mean.

Closed and non-operating stores are filtered out so they don't distort rankings.

## 6. Report pages (Power BI)

1. Monthly analysis
2. Item-level inventory
3. Department time series
4. Category time series
5. Account time series
6. Outlier stores
7. Latest store audits
8. Inventory loss per day
9. Loss per day by audit

Filters: brand, director/region, store, department and audit category.

---

Created by Agnivesh Das · bp US Convenience · Illustrative values throughout. Shrink = stock lost to theft, damage, spoilage or error.
