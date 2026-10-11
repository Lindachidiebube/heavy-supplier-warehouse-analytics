# Sprint 04 Notes

**Project:** Heavy Supplier & Warehouse Analytics
**Sprint:** Week 4 (Phase 2: Inventory Health and Stock Analysis)

---

## Sprint Goal

- Find out how much stock each product really holds in each branch (KPI 8 Reconstructed Stock Balance), because the recorded `current_stock` column could not be trusted without testing.
- Use that stock figure to test whether any product-branch pair holds dead stock, meaning stock that is sitting with no demand (KPI 6 Dead Stock Identification).
- Run the 8 verification checks on every KPI before it is written up, and upload each KPI as soon as it is finished.

---

## Work Completed

### KPI 8: Reconstructed Stock Balance

- **Ledger start point.** Checked where `stock_ledger` begins for each of the 180 product-branch pairs. For the 167 pairs that start with an IN or OUT row, the balance before the first row equals `opening_stock` exactly, with 0 mismatches.
- **ADJUSTMENT direction.** The ADJUSTMENT rows carry no sign in the data. The direction was read from the change in `running_balance`: 1,227 add stock and 1,227 subtract stock, with 0 unreadable. This confirmed the Week 3 pattern.
- **Full chain test.** Tested all 237,230 ledger rows: every IN row adds its quantity (127,611 rows), every OUT row subtracts it (107,165 rows), and every ADJUSTMENT row moves the balance by its quantity in the proven direction (2,454 rows). The ledger is an unbroken chain from `opening_stock` for all 180 pairs.
- **Ledger-based rebuild.** Opening stock plus all signed movements gives 19,305,994 units in total (average 107,255.52 per pair, range 88,853 to 121,013). No pair goes below zero. This equals `current_stock` for 166 of 180 pairs.
- **The 14 pairs that differ.** For each of the 14 pairs, `current_stock` is a real ledger balance from that pair's last day, but it leaves out the last 1 or 2 movements (17 rows in total, 16 purchase receipts and 1 sale, dated 9 to 24 January 2025). The 17 rows add up exactly to each pair's gap. The orders behind them are ordinary Received or Delivered orders in the right branch on the right date, so this is logged as a timing difference. The total gap is about 2,728 units (about 0.014% of stock). The data does not show why the snapshot stops early.
- **Second rebuild from order lines.** Opening stock plus units on Received purchase orders, minus units on Delivered sales orders, plus net adjustments gives 21,185,215 units (average 117,695.64 per pair). This is 1,879,221 units (about 9.7%) higher than the ledger-based total, and higher in all 180 pairs.
- **The 1,879,221 gap is fully explained.** Units on Received purchase orders missing from the ledger: 2,237,884. Units on Delivered sales orders missing from the ledger: 123,341. Ledger IN rows from Cancelled purchase orders: 248,556. Ledger OUT rows from Cancelled sales orders: 13,234. Combined: 2,237,884 - 123,341 - 248,556 + 13,234 = 1,879,221, equal to the actual gap.
- **Headline figure.** Reconstructed stock is the order-line stock (21,185,215). The ledger-based figure and `current_stock` are shown beside it for comparison. The caveat stated in the document: these are records, not a physical count, and the figure assumes every Received purchase order was fully received and every Delivered sales order was fully shipped.
- **Excess stock finding.** Under both methods, every one of the 180 pairs holds at least 177 times its `max_stock` (ledger-based) or 193 times (order-line). Average stock is about 333 times average `max_stock` (321.76). The excess-stock finding holds under every stock figure.
- **8 verification checks completed:** empty, zero and negative values in every column used (0 problems); every pair has IN, OUT, received and delivered units; all 2,454 adjustments readable; totals reconcile; second-method recompute; outliers (one low outlier, P019 KOL001, and no high outlier); date coverage (sales deliveries end on 13 January 2025 while purchase receipts continue to 28 January 2025, explained by 347 purchase-receipt rows after 13 January); and a manual spot-check of three pairs (P019 KOL001, P002 HYD001, P025 HYD001) rebuilt by hand from raw rows.

### KPI 6: Dead Stock Identification

- **Rule 1, no recent sale.** Using Delivered sales order lines only, every one of the 180 pairs has a sale. The longest gap since a pair's last sale is 27 days (to 13 January 2025). No pair has gone 30 days or more without a sale, so a "no recent sale" rule finds no dead stock.
- **Rule 2, very low sales.** Over the last 12 months (15 January 2024 to 13 January 2025, 365 days), the lowest-selling pair is P015 PUN001 with 703 units. No pair has zero or near-zero demand (average 1,096 units, std 183). A low-sales rule also finds no dead stock.
- **Years of cover instead.** Because both classic tests find nothing, KPI 6 reports no dead stock by the classic tests and then ranks the pairs by years of cover (order-line stock divided by units sold in the last 12 months). An absolute flag cannot separate the pairs: every pair is above 50 years of cover.
- **Results.** Years of cover per pair: mean 110.46, minimum 72.04 (P027 CHN001), maximum 171.92 (P005 HYD001). Pairs above 100 years: 119 of 180. Above 120 years: 59. Above 150 years: 6.
- **Cover bands (a ranking aid chosen for this analysis, not a business standard).** Above 150 years: 6 pairs. 120 to 150 years: 53. 100 to 120 years: 60. Up to 100 years: 61. The 6 pairs in the highest band are 5 at HYD001 and 1 at PUN001.
- **Branch pattern.** HYD001 has the highest cover because it sells the least (about 916 units per pair per year, against about 1,331 at CHN001), not because it holds more stock. Average stock per pair is about the same in every branch (roughly 104,000 to 110,000 units using recorded `current_stock`). Stock levels do not follow branch demand.
- **Ledger-based comparison.** With ledger-based stock, the bands are 3, 24, 57 and 96 pairs. The ledger understates stock by about 10%, so fewer pairs reach the high bands. The worst 6 pairs are the same under both stock figures (6 of 6), so the top of the ranking does not depend on the stock method.
- **8 verification checks completed:** empty, zero and negative values (0 problems); every pair has sales in the window, so no divide-by-zero is possible; Cancelled orders are excluded (24,652 units, about 11.1% of units on all orders in the window) and moving the window by one day changes at most 1 of 180 bands; totals reconcile with the raw tables and with KPI 8 (order-line stock 21,185,215, net adjustments +62); years of cover is identical by two routes; one high outlier (P005 HYD001) and no low outlier, with 23 of 180 pairs within 2 years of a band line (so band placement for those pairs is fragile); every branch sells in every month of the window (78 of 78 branch-months have sales); and a manual spot-check of three pairs (P005 HYD001, P027 CHN001, P015 AHM001) rebuilt by hand from raw rows.

---

## Blockers & Resolutions

- **Is `current_stock` wrong?** Early on it looked as if `current_stock` was the broken column. The tests showed it matches the ledger for 166 of 180 pairs, and that the real issue is that the ledger itself is about 10% short of the order lines. The framing was corrected and the headline stock was moved to the order-line figure.
- **Adjustment direction in the KPI 8 spot-check.** The first spot-check cell assumed ADJUSTMENT rows were marked IN or OUT and returned 0 adjustments. A follow-up cell showed that adjustment rows carry no IN or OUT label, and that the direction shows only in the balance change. The spot-check then reconciled exactly (172, -66 and -129 units for the three pairs).
- **Dead stock rules found nothing.** The classic dead-stock tests flagged no pair. Resolution: report that result plainly and rank pairs by years of cover.
- **Window wording in KPI 6.** The filter excludes 14 January 2024 itself (12 lines, 135 units), so the window counted is 15 January 2024 to 13 January 2025 (365 days). The document uses this wording.
- **Two code runs repeated.** One KPI 6 check output was cut off at the last line and was re-run for that line only. One check failed because output text was pasted into a code cell, and ran cleanly when re-run.

---

## Next Sprint Focus

- **Week 5 (Phase 2):** KPI 7 ABC/Pareto Classification (which products drive revenue) and KPI 9 Inventory Turnover Rate (how fast stock turns over). KPI 9 will use the KPI 8 order-line stock.
- Decide in Week 5 whether Inventory Aging becomes a section inside an existing KPI or is recorded as not selected.
- Follow-up carried forward: the reason HYD001 sells less than the other branches was not investigated in KPI 6, and is a candidate for the branch view in the Week 9 demand forecast (KPI 18).
