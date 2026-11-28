# Sprint 10 Notes

**Project:** Heavy Supplier & Warehouse Analytics
**Sprint:** Week 10 (Phase 4: Risk, Anomaly and Control)

## Sprint Goal

Detect stock risks and operational anomalies early. This sprint found the unusual rows in the stock ledger (KPI 20), measured the loss that adjustments record (KPI 21) and combined four risk signals into one score per branch (KPI 22). KPI 22 is the last of the 22 KPIs in the roadmap.

## Work Completed

### KPI 20 Inventory Movement Anomaly Detection

- Looked at every one of the 237,230 stock ledger rows (127,611 IN, 107,165 OUT, 2,454 ADJUSTMENT) one at a time, to flag the rows that stand out.
- Found that size cannot flag a row. Each movement type sits inside a hard range (IN 20 to 300, OUT 1 to 20, ADJUSTMENT 5 to 50), the largest quantity is at most 1.91 times the average of its type, and every quantity value is used about equally often (p 0.9377 for IN, 0.2592 for OUT, 0.8916 for ADJUSTMENT).
- Found that ordinary orders reach the ledger about 99% of the time (98.9% of Received purchase orders, 99.1% of Delivered sales orders), while Cancelled orders reach it about 46% of the time (46.33% of Cancelled purchase orders, 45.5% of Cancelled sales orders).
- Fixed the flag rule: flag A is a ledger row tied to a Cancelled order (2,817 rows, 1,542 IN and 1,275 OUT, 261,790 units); flag B is a ledger row whose units are below the units on its order lines for the same order and product (5,521 rows in 4,947 order-and-product pairs). In total 8,338 rows are flagged, which is 3.51% of the ledger.
- Confirmed that every ledger row of a Received or Delivered order carries the same date and branch as its order (0 differences). The ledger is never above the order lines.
- Explained the 1,214 identical rows (606 groups): every group comes from an order with two or more lines of the same product and quantity, and no movement id is repeated, so there are no true repeated rows.
- Tested where the flags sit. No branch difference (p 0.1089), no product difference (p 0.1897), and purchases and sales are flagged at the same rate (3.55% each, p 0.9566). The yearly share is between 3.50% and 3.65% in 2020 to 2024 (p 0.6997); 2019 (3.24%) and the partial year 2025 (2.59%) are lower, and the yearly difference is below the 0.05 line only when those two years are included (p 0.0126).
- Split the order-line gap: units short on the lower pairs are 435,404 (purchases) and 24,216 (sales); the pairs missing from the ledger entirely (1,802,480 purchase units and 99,125 sales units) were passed to KPI 21.
- Passed all 8 verification checks, including a second method that gives the same 8,338 flagged rows and the same set of movement ids, and three rows rebuilt by hand from the raw files.

### KPI 21 Stock Shrinkage Rate

- Measured recorded loss as the units subtracted by adjustments divided by the units handled (IN and OUT units), because the data holds no physical stock count.
- Read the direction of every adjustment from the change in the running balance: 1,227 adjustments added 33,526 units and 1,227 subtracted 33,464 units, a net of +62 units. No adjustment was unreadable.
- Found a shrinkage rate of 0.1555% (33,464 subtracted units over 21,522,627 units handled). By branch it runs from 0.1378% (Pune) to 0.1752% (Kolkata).
- Tested whether the subtractions sit unevenly. No difference was found between branches (p 0.5024), products (p 0.2496) or years 2019 to 2024 (p 0.8518). Every one of the 180 product-branch pairs has adjustments in both directions, and no pair lies outside the normal range of pair loss rates (0.0335% to 0.3092%; pair test p 0.778).
- Found that no adjustment of either kind is recorded after 2024-12-29, while the ledger runs to 2025-01-28. Subtractions appear in every one of the 72 months from 2019-01 to 2024-12. The data does not show why recording stops.
- Kept the records gap apart and did not call it shrinkage: 2,237,884 purchase units (10.0%) and 123,341 sales units (9.99%) on order lines are missing from the ledger, and the ledger holds 248,556 IN units and 13,234 OUT units on Cancelled orders. The net gap is 1,879,221 units, about 67 times the subtracted units (by hand).
- Passed all 8 verification checks, including a second method built with plain loops and three items rebuilt by hand from the raw files.

### KPI 22 Operational Risk Score

- Combined four signals per branch: late payment rate (KPI 4), years of stock cover (KPI 6), flagged ledger rows (KPI 20) and recorded loss rate (KPI 21). The level is the six branches, because every invoice carries its branch and an order holds several products.
- Turned each signal into an index (the branch value divided by the overall value, times 100) and took the simple average of the four. Scores: Hyderabad 105.1 (rank 1), Pune 101.5, Chennai 100.0 (99.95), Delhi 98.7, Kolkata 98.6, Ahmedabad 97.5. The spread is only 7.6 points.
- Tested each signal against branch. The late payment rate differs by more than chance (p below 0.000001; Chennai 48.61%, Delhi 37.72%) and so does years of cover (p below 0.000001; Hyderabad 129.36, Chennai 88.87). No difference was found for the flagged share (p 0.1089) or the loss rate (p 0.4993).
- Found that Chennai has the highest late payment rate in every due year from 2019 to 2025 (7 of 7).
- Found that the gap in years of cover follows the difference in units sold and not in stock: with equal stock the highest-over-lowest ratio stays at 1.453 (actual 1.455), and with equal sales it falls to 1.047.
- Tested the ranking and found it is not stable. Hyderabad ranks first in 59.8% of 10,000 random weight sets, Chennai in 24.9%, Kolkata in 8.5% and Pune in 6.9%. Leaving out one signal changes the top branch (without cover Chennai is first, without loss Pune is first). Removing the one outlier pair (P005 Hyderabad) changes no rank.
- Measured two alternative scores. A stretched 0-to-100 score keeps the top three and the last place and only swaps Delhi and Kolkata. A score from only the two signals that separate branches (late payment and cover) puts Pune first (106.8) and Hyderabad second (106.3). Hyderabad and Pune hold the top two places in all three scores.
- Added a branch risk profile table (the four indexes beside the score), so owners can see which signal sits behind each branch's score.
- Decided that Operational Bottleneck Detection is Not selected: supplier waits are measured in KPI 13 and payment delays in KPI 4, none of the 22 KPIs measures the wait from sales order to delivery, and the roadmap fixes KPI 22 as the combination of KPI 4, 6, 20 and 21.
- Passed all 8 verification checks, including a second method with plain loops and three branches rebuilt by hand from the raw files (all 24 numbers and the 3 scores equal the table).

## Blockers & Resolutions

- **Size could not flag unusual ledger rows (KPI 20).** Quantities are evenly spread inside hard ranges. Resolved by flagging rows that break the link to the order records instead.
- **Identical ledger rows looked like duplicates (KPI 20).** Resolved by matching all 606 groups to order lines: every group has two or more order lines with the same product and quantity.
- **Flagged share differed by year (KPI 20).** Resolved by testing the years three ways: all years, without 2025, and 2020 to 2024 only. The difference comes from 2019 and the partial year 2025.
- **No physical stock count in the data (KPI 21).** Resolved by measuring only the recorded adjustments and keeping the records gap separate, never calling it shrinkage.
- **No adjustments recorded after 2024-12-29 (KPI 21).** Resolved by reporting the dates and testing the years 2019 to 2024 on their own (p 0.8518). The reason recording stops is not shown by the data.
- **The four signals pull in different directions (KPI 22).** Chennai is worst on late payments but lowest on cover, and Hyderabad is the reverse on cover. Resolved by testing the ranking against other weights and two alternative scores, and by adding the risk profile table.

## Next Sprint Focus

- Week 11: build the dashboards for all 22 KPIs.
- Week 12: turn the findings into recommendations (inventory, warehouse, supplier, customers, cost and efficiency) and prepare the final presentation.
