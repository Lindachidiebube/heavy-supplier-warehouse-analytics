# KPI 22 Operational Risk Score
Project: Heavy Supplier & Warehouse Analytics | Week 10 | Notebook: Week10 KPI 22 Operational Risk Score

## Summary

- KPI 22 puts four risk signals side by side for each of the six branches and combines them into one score. The signals come from KPI 4 (late payment rate), KPI 6 (years of stock cover), KPI 20 (flagged ledger rows) and KPI 21 (recorded loss rate).
- Each signal becomes an index: the branch value divided by the overall value, times 100. The score is the simple average of the four indexes. A score of 100 equals the overall figure, and a higher score means more risk.
- Score and rank (rank 1 is the highest score): Hyderabad (HYD001) 105.1, rank 1; Pune (PUN001) 101.5, rank 2; Chennai (CHN001) 100.0 (99.95 before rounding), rank 3; Delhi (DEL001) 98.7, rank 4; Kolkata (KOL001) 98.6, rank 5; Ahmedabad (AHM001) 97.5, rank 6.
- The highest score minus the lowest score is 7.6 points (by hand).
- The late payment rate differs between branches by more than chance (p below 0.000001).
- Years of cover differs between branches by more than chance (p below 0.000001).
- No difference was found between branches in the flagged share (p 0.1089).
- No difference was found between branches in the loss rate (p 0.4993).
- Chennai has the highest late payment rate in every due year from 2019 to 2025 (7 of 7).
- The difference in years of cover between branches follows the difference in units sold, not the difference in stock.
- The ranking is not stable: it depends on the weights given to the four signals.
- Hyderabad and Pune hold the top two places in the headline score and in both alternative scores, and which of the two is first depends on the method.
- All 8 verification checks are done, and the supporting tests are listed in the Verification Log.

## 1 KPI Definition

| Item | Definition |
|---|---|
| Operational Risk Score | For each of the six branches, the simple average of four indexes. A higher score means more risk. Rank 1 is the highest score. |
| Late payment index | Branch late payment rate divided by the overall late payment rate (41.50%), times 100. A payment is late when its payment date is after the invoice due date. Invoices and payments with a repeated id are left out (the KPI 4 rule). |
| Cover index | Branch years of cover divided by the overall years of cover (107.38), times 100. Years of cover is the order-line stock divided by the units sold from 2024-01-15 to 2025-01-13 (the KPI 6 rule with the KPI 8 stock rebuild). |
| Flagged index | Branch share of ledger rows flagged, divided by the overall share (3.51%), times 100. A ledger row is flagged when it belongs to a Cancelled order or its units are below the units on its order lines (the KPI 20 rule). |
| Loss index | Branch loss rate divided by the overall loss rate (0.1555%), times 100. Loss rate is the units subtracted by adjustments divided by the units handled (IN and OUT units) (the KPI 21 rule). |
| Level | The six branches. |

**Business question:** Which branch carries the most operational risk once the four warning signs are put together, and which warning sign is behind each branch's score?

**Why this metric and not another**

- One score per branch lets the owners see which branch needs attention first.
- From the lowest to the highest branch, the late payment rate varies by 26.2% of the overall figure, years of cover by 37.7%, loss rate by 24.1% and flagged share by 8.4%.
- The risk profile table (see Business Interpretation) shows the four indexes beside the score.

## 2 Methodology

**Sources**

| Table | Rows | Used for |
|---|---|---|
| invoices | 18,033 | Late payment signal (branch, due date) |
| payments | 19,257 | Late payment signal (payment date) |
| stock_ledger | 237,230 | Flagged share, loss rate, stock rebuild |
| inventory_master | 180 | Opening stock per product and branch |
| purchase_orders_header | 24,000 | Order status and branch of purchases |
| purchase_orders_lines | 155,495 | Units received, units on order lines |
| sales_orders_header | 20,000 | Order status, branch, delivery date |
| sales_orders_lines | 130,402 | Units delivered and sold, units on order lines |
| branches | 6 | List of branches |
| products | 30 | Loaded, not used in the score |

**Reused and new**

| Reused (verified in earlier KPIs) | New in KPI 22 |
|---|---|
| Late payment rule and left-out ids (KPI 2, KPI 4) | Index scaling (branch value over overall value) |
| Years of cover with order-line stock (KPI 6, KPI 8) | Equal-weight score and branch ranking |
| Flag rule for ledger rows (KPI 20) | Sensitivity of the ranking to the weights |
| Loss rate from adjustment direction (KPI 21) | Cover decomposition, alternative scores, risk profile table |

**Steps**

- Rebuild the late payment rate per branch as in KPI 4 and test it against branch.
- Rebuild the order-line stock and years of cover per branch as in KPI 6 and test the pair values against branch.
- Rebuild the loss rate per branch as in KPI 21 and test it against branch.
- Rebuild the flagged share per branch as in KPI 20 and test it against branch.
- Turn each signal into an index, average the four indexes into the score and rank the branches.
- Test the ranking: drop each signal in turn, try 10,000 random weight sets, remove the outlier pair, try two alternative scores.
- Run the 8 verification checks.

**Choices and alternatives**

- **Level.** The level is the six branches. A late payment rate per product-branch pair would need invoices split across products, because an order holds several products (117,656 Delivered lines on 18,033 Delivered orders, about 6.5 lines per order, by hand from KPI 18).
- **Scaling.** Every signal is divided by its own overall value, and 100 equals the overall figure.
- **Stretched alternative.** A score that stretches each signal from 0 (lowest branch) to 100 (highest branch) gives the same top three places (Hyderabad, Pune, Chennai) and the same last place (Ahmedabad) as the headline score, and only Delhi and Kolkata swap places 4 and 5 (see Supporting tests).
- **Weights.** Equal weights are the headline. The ranking is then tested against dropping each signal and against 10,000 random weight sets (see Supporting tests).
- **Two-signal alternative.** A score from only the two signals that separate branches by more than chance (late payment and cover) puts Pune first (106.8), Hyderabad second (106.3), Chennai third, Ahmedabad fourth, Delhi fifth and Kolkata sixth. The gap between Pune and Hyderabad in that score is 0.5 points (by hand).
- **Operational Bottleneck Detection** is Not selected: supplier waits are measured in KPI 13 and payment delays in KPI 4, and none of the 22 KPIs measures the wait from sales order to delivery (by inspection of the KPI list); the roadmap fixes KPI 22 as the combination of KPI 4, 6, 20 and 21, so no new signal was added.

## 3 Verification Log

| Check | What it checks | Result |
|---|---|---|
| Check 1 | Blanks, zeros and negatives in every column used | 0 problems in every column used |
| Check 2 | Edge cases: branches, zero denominators, invoice branch, order statuses | Six branches in every table, smallest denominators above 0, 0 invoices without an order, 0 branch differences |
| Check 3 | Duplicates | No repeated branch, movement, inventory pair or order id; known repeated invoice and payment ids left out |
| Check 4 | Totals reconcile | Every branch table adds to its overall figure; each index averages 100 with its own branch weights |
| Check 5 | Second method with plain loops | Same signals, score and ranking |
| Check 6 | Outliers and spread | One pair outside the normal range; removing it changes no rank |
| Check 7 | Dates and history | Periods of every signal shown; Chennai highest in all 7 due years |
| Check 8 | Three branches rebuilt by hand from the raw csv files | All 24 numbers and 3 scores equal the table |

**Check 1: blanks, zeros and negatives**

- invoices (18,033 rows): invoice_id, branch_id, due_date blank 0.
- payments (19,257): payment_id, invoice_id, payment_date blank 0.
- stock_ledger (237,230): movement_id, product_id, branch_id, movement_type, movement_date, reference_id, quantity, running_balance blank 0; quantity zero 0, negative 0; running_balance zero 0, negative 0.
- inventory_master (180): product_id, branch_id, opening_stock blank 0; opening_stock zero 0, negative 0.
- purchase_orders_header (24,000): po_id, branch_id, po_status blank 0.
- purchase_orders_lines (155,495): po_id, product_id, quantity blank 0; quantity zero 0, negative 0.
- sales_orders_header (20,000): so_id, branch_id, order_status, delivery_date blank 0.
- sales_orders_lines (130,402): so_id, product_id, quantity blank 0; quantity zero 0, negative 0.
- Result: 0 problems in every column used.

**Check 2: edge cases**

- Branches in the branch table: 6. Invoices, joined payments, stock ledger, inventory pairs, purchase orders and sales orders each hold 6 branches.
- Every table holds the same six branches as the branch table, and every invoice carries the same branch as its sales order.
- Smallest payments in a branch 2,576; smallest units sold in a branch 27,484; smallest units handled in a branch 3,538,563; smallest ledger rows in a branch 36,300; pairs with zero units sold in the window 0.
- Invoices with no sales order 0; invoices whose branch differs from the order branch 0.
- Clean payments with no clean invoice 409 (18,854 clean payments minus 18,445 joined, by hand).
- Purchase order statuses: Received 21,630, Cancelled 2,370. Sales order statuses: Delivered 18,033, Cancelled 1,967.

**Check 3: duplicates**

- Branch ids 6, distinct 6. Ledger rows 237,230, distinct movement ids 237,230. Inventory rows 180, distinct product and branch pairs 180. Purchase orders 24,000, distinct ids 24,000. Sales orders 20,000, distinct ids 20,000.
- Invoice rows in repeated id groups 393; payment rows in repeated id groups 403.
- The repeated invoice and payment ids are the known KPI 2 numbers (393 and 403 rows), and none remains after they are left out.
- Invoice ids still repeated after leaving them out 0; payment ids still repeated 0; payment rows identical to another payment row 0.

**Check 4: totals reconcile**

- Late payment: branches add to 18,445 payments and 7,655 late; overall late rate 41.50%.
- Stock: opening 32,969; received 22,387,123; delivered 1,234,939; net adjustments 62; opening plus received minus delivered plus net adjustments 21,185,215 (by hand 32,969 + 22,387,123 - 1,234,939 + 62 = 21,185,215), the same as the sum over the branches. Units sold add to 197,300; overall years of cover 107.38.
- Flagged: ledger rows add to 237,230 and flagged rows to 8,338; overall share 3.51%.
- Loss: units handled add to 21,522,627 and units subtracted to 33,464; overall loss rate 0.1555%.
- Each index averaged with its own branch weights (payments, units sold, ledger rows, units handled): late 100.0, cover 100.0, flagged 100.0, loss 100.0.
- Every branch table adds to its overall figure, and the overall figures equal those of KPI 4, KPI 6, KPI 8, KPI 20 and KPI 21.
- Each index averaged with its own branch weights equals 100, which shows the indexes are built correctly.

**Check 5: second method with plain loops**

- Built from the raw tables with dictionaries and loops (no groupby, no merge). Totals from the loops: payments 18,445, late 7,655; order-line stock 21,185,215, units sold 197,300; ledger rows 237,230, flagged 8,338; units handled 21,522,627, subtracted 33,464.
- Largest difference between the two methods: late rate 0.0, years of cover 0.0, flagged share 0.0, loss rate 0.0, risk score 0.0. Same ranking: yes.
- The loop method gives the same signals, score and ranking as the table method.

**Check 6: outliers and spread**

- Range across the six branches: late index 90.9 to 117.1; cover index 82.8 to 120.5; flagged index 95.8 to 104.2; loss index 88.6 to 112.7; risk score 97.5 to 105.1.
- Normal range of years of cover per pair (1.5 times the middle spread): 54.2 to 166.5. Pairs outside the range: 1 (P005 HYD001: stock 122,924, units sold 715, cover 171.92).
- Without that pair (cover index, score, rank): Ahmedabad 103.3, 97.6, rank 6; Chennai 83.0, 100.0, rank 3; Delhi 106.4, 98.7, rank 4; Hyderabad 119.7, 104.9, rank 1; Kolkata 87.8, 98.7, rank 5; Pune 109.8, 101.6, rank 2. The original ranks are 6, 3, 4, 1, 5, 2.
- Leaving the outlier pair out lowers Hyderabad's cover index from 120.5 to 119.7 and its score from 105.1 to 104.9 and changes no rank, so no single pair drives the ranking.
- Delhi and Kolkata print the same score (98.7) without that pair, so ranks 4 and 5 are a near tie.

**Check 7: dates and history**

- First and last due date 2019-01-20 to 2025-03-14; payment date 2019-01-09 to 2025-03-20; ledger date 2019-01-01 to 2025-01-28; delivery date in the units sold window 2024-01-15 to 2025-01-13.
- Late rate % by branch and due year:

| Due year | AHM001 | CHN001 | DEL001 | HYD001 | KOL001 | PUN001 | Payments |
|---|---|---|---|---|---|---|---|
| 2019 | 41.8 | 48.1 | 39.6 | 39.7 | 42.7 | 43.9 | 2,717 |
| 2020 | 40.3 | 48.2 | 42.5 | 39.6 | 36.9 | 42.4 | 3,060 |
| 2021 | 40.7 | 49.3 | 36.5 | 35.2 | 37.6 | 39.7 | 3,089 |
| 2022 | 40.6 | 48.7 | 38.7 | 36.6 | 44.0 | 46.1 | 3,102 |
| 2023 | 37.2 | 49.0 | 35.1 | 41.4 | 38.2 | 45.2 | 3,101 |
| 2024 | 38.6 | 49.1 | 36.7 | 38.1 | 41.9 | 42.6 | 3,005 |
| 2025 | 31.7 | 40.7 | 20.0 | 28.8 | 27.8 | 34.5 | 371 |

- The payments per due year add to 18,445 (by hand). The smallest number of payments in a branch and due year is 50.
- Chennai has the highest late rate in every due year from 2019 to 2025 (7 of 7). Its rate lies between 48.1% and 49.3% in each of 2019 to 2024 (read from the table).
- Every branch shows a lower late rate in 2025 than in 2024 (2025 is a partial year with 371 payments).
- Years of cover uses the last 12 months of sales, while the other three signals use the full history.

**Check 8: three branches rebuilt by hand from the raw csv files (csv module, no pandas)**

| Number | HYD001 by hand = table | CHN001 by hand = table | AHM001 by hand = table |
|---|---|---|---|
| Payments | 2,576 | 3,732 | 3,196 |
| Late payments | 984 | 1,814 | 1,266 |
| Order-line stock | 3,555,196 | 3,548,773 | 3,472,113 |
| Units sold in the window | 27,484 | 39,930 | 31,376 |
| Ledger rows | 36,300 | 44,029 | 39,911 |
| Flagged rows | 1,329 | 1,482 | 1,366 |
| Units handled | 3,548,234 | 3,692,804 | 3,539,666 |
| Units subtracted | 5,730 | 5,980 | 5,190 |
| Score by hand and in the table | 105.13 | 99.95 | 97.55 |

- The by-hand counts and scores of the three branches equal the table. The scores by hand use the four overall values from the table.

**Supporting tests**

| Test | Method | Result |
|---|---|---|
| Late payment rate against branch | Chi-square, strict line 0.05 | Branches differ by more than chance, p below 0.000001; smallest expected count 1,069.1 |
| Years of cover against branch | Kruskal-Wallis on the 180 pair values, 30 per branch | Branches differ by more than chance, p below 0.000001; medians AHM001 113.19, CHN001 89.47, DEL001 117.92, HYD001 131.74, KOL001 92.13, PUN001 114.83 |
| Flagged share against branch | Chi-square | No difference found, p 0.1089 |
| Loss rate against branch | Chi-square of subtraction rows against handled rows | No difference found, p 0.4993 |
| Cover decomposition | Cover with equal stock and with equal sales | Highest over lowest: actual 1.455, equal stock 1.453, equal sales 1.047 |
| Drop one signal | Rank without each signal | See table below |
| Random weights | 10,000 random weight sets (seed 2026) | See table below |
| Outlier pair | Score without P005 HYD001 | See Check 6 |
| Alternative scores | Stretched 0-to-100 score and two-signal score | See table below |
| Spread of each signal | Highest minus lowest branch value over the overall figure | Late rate 26.2%, years of cover 37.7%, flagged share 8.4%, loss rate 24.1% |

- Branch values behind the tests: late rate % AHM001 39.61, CHN001 48.61, DEL001 37.72, HYD001 38.20, KOL001 39.74, PUN001 43.15; years of cover 110.66, 88.87, 113.98, 129.36, 94.02, 117.61; flagged share % 3.42, 3.37, 3.56, 3.66, 3.46, 3.66; loss rate % 0.1466, 0.1619, 0.1499, 0.1615, 0.1752, 0.1378. Overall: 41.50, 107.38, 3.51, 0.1555.
- Stock and units sold by branch (order-line stock, units sold): AHM001 3,472,113 and 31,376; CHN001 3,548,773 and 39,930; DEL001 3,597,678 and 31,565; HYD001 3,555,196 and 27,484; KOL001 3,435,814 and 36,543; PUN001 3,575,641 and 30,402.
- Loss table by branch (handled units, subtracted units, handled rows, subtracted rows): AHM001 3,539,666, 5,190, 39,525, 196; CHN001 3,692,804, 5,980, 43,600, 226; DEL001 3,622,464, 5,431, 37,575, 202; HYD001 3,548,234, 5,730, 35,877, 207; KOL001 3,538,563, 6,200, 40,848, 218; PUN001 3,580,896, 4,933, 37,351, 178.
- Flagged rows by branch (ledger rows, flagged): AHM001 39,911, 1,366; CHN001 44,029, 1,482; DEL001 37,967, 1,351; HYD001 36,300, 1,329; KOL001 41,272, 1,429; PUN001 37,751, 1,381. Flagged in total 8,338 of 237,230 (rows tied to cancelled orders 2,817; rows below the order-line units 5,521).
- Late payments by branch (payments not late, late): AHM001 1,930 and 1,266; CHN001 1,918 and 1,814; DEL001 1,696 and 1,027; HYD001 1,592 and 984; KOL001 2,102 and 1,386; PUN001 1,552 and 1,178.
- The gap in years of cover follows the difference in units sold: with equal stock the highest-over-lowest ratio stays at 1.453 (actual 1.455), and with equal sales it falls to 1.047.

Cover with the same stock or the same sales at every branch:

| Branch | Actual cover | Cover if equal stock | Cover if equal sales |
|---|---|---|---|
| AHM001 | 110.66 | 112.53 | 105.59 |
| CHN001 | 88.87 | 88.43 | 107.92 |
| DEL001 | 113.98 | 111.86 | 109.41 |
| HYD001 | 129.36 | 128.47 | 108.12 |
| KOL001 | 94.02 | 96.62 | 104.48 |
| PUN001 | 117.61 | 116.14 | 108.74 |

Rank when one signal is left out (1 is the highest score):

| Branch | Without late | Without cover | Without flagged | Without loss |
|---|---|---|---|---|
| AHM001 | 5 | 6 | 6 | 4 |
| CHN001 | 6 | 1 | 2 | 5 |
| DEL001 | 2 | 5 | 5 | 3 |
| HYD001 | 1 | 3 | 1 | 2 |
| KOL001 | 4 | 2 | 4 | 6 |
| PUN001 | 3 | 4 | 3 | 1 |

- Leaving out one signal changes the top branch: Chennai is first without the cover signal, Pune is first without the loss signal, and Hyderabad stays first without the late payment signal or the flagged signal.

10,000 random weight sets:

| Branch | Rank 1 in % of sets | Best rank | Worst rank |
|---|---|---|---|
| AHM001 | 0.0 | 3 | 6 |
| CHN001 | 24.9 | 1 | 6 |
| DEL001 | 0.0 | 2 | 6 |
| HYD001 | 59.8 | 1 | 5 |
| KOL001 | 8.5 | 1 | 6 |
| PUN001 | 6.9 | 1 | 6 |

- Hyderabad ranks first in 59.8% of 10,000 random weight sets, Chennai in 24.9%, Kolkata in 8.5% and Pune in 6.9%, and Ahmedabad and Delhi never rank first.
- Five of the six branches rank last under some weights (Hyderabad's worst rank is 5).
- Hyderabad's cover signal lifts it to first place and Chennai's cover signal holds it down: without the cover signal Hyderabad ranks third and Chennai ranks first.

Alternative scores (score and rank):

| Branch | Headline score | Rank | Stretched 0-to-100 score | Rank | Two-signal score | Rank |
|---|---|---|---|---|---|---|
| AHM001 | 97.5 | 6 | 28.5 | 6 | 99.3 | 4 |
| CHN001 | 100.0 | 3 | 41.1 | 3 | 99.9 | 3 |
| DEL001 | 98.7 | 4 | 39.9 | 5 | 98.5 | 5 |
| HYD001 | 105.1 | 1 | 66.9 | 1 | 106.3 | 2 |
| KOL001 | 98.6 | 5 | 41.0 | 4 | 91.7 | 6 |
| PUN001 | 101.5 | 2 | 55.0 | 2 | 106.8 | 1 |

## 4 Data Quality Notes

1. Invoices with a repeated id (393 rows) and payments with a repeated id (403 rows) are left out of the late payment signal, as in KPI 4.
2. 409 clean payments have no clean invoice, the same count as KPI 4.
3. Years of cover uses the last 12 months of sales (2024-01-15 to 2025-01-13); see Check 7 for the dates of every signal.
4. 2025 is a partial year for late payments (371 payments, due dates up to 2025-03-14, payments up to 2025-03-20); no cause is stated or tested for the 2025 rates.
5. The data holds no physical count (by inspection of the tables, see KPI 21), so the loss rate measures recorded adjustments only.
6. The records gap of 1,879,221 net units (KPI 21) is not part of the score and is not called shrinkage.
7. No adjustment of either kind is recorded after 2024-12-29 while the ledger runs to 2025-01-28 (KPI 21).
8. The p-values printed as 0.0 in the notebook are rounded to six places and mean below 0.000001.
9. The loss rate test here gives p 0.4993 and KPI 21 reported 0.5024; both are far above 0.05, so the conclusion is the same.
10. Years of cover uses the order-line stock (opening stock plus received units minus delivered units plus net adjustments), as in KPI 6, and not the ledger stock.
11. Chennai's score is 99.95 and prints as 100.0 at one decimal.
12. Stock per branch lies within about 4.7% (3,597,678 against 3,435,814, by hand), while units sold differ by about 45% (39,930 against 27,484, by hand).
13. P005 HYD001 is the only pair outside the normal range of years of cover (54.2 to 166.5).
14. Money values are not part of KPI 22; see KPI 3, KPI 4 and KPI 10.

## 5 Business Interpretation

Branch risk profile (indexes: 100 equals the overall figure; the overall figures are late rate 41.50%, years of cover 107.38, flagged share 3.51%, loss rate 0.1555%):

| Branch | Late rate % | Years of cover | Flagged share % | Loss rate % | Late index | Cover index | Flagged index | Loss index | Score | Rank |
|---|---|---|---|---|---|---|---|---|---|---|
| Hyderabad (HYD001) | 38.20 | 129.36 | 3.66 | 0.1615 | 92.0 | 120.5 | 104.2 | 103.9 | 105.1 | 1 |
| Pune (PUN001) | 43.15 | 117.61 | 3.66 | 0.1378 | 104.0 | 109.5 | 104.1 | 88.6 | 101.5 | 2 |
| Chennai (CHN001) | 48.61 | 88.87 | 3.37 | 0.1619 | 117.1 | 82.8 | 95.8 | 104.2 | 100.0 | 3 |
| Delhi (DEL001) | 37.72 | 113.98 | 3.56 | 0.1499 | 90.9 | 106.1 | 101.2 | 96.4 | 98.7 | 4 |
| Kolkata (KOL001) | 39.74 | 94.02 | 3.46 | 0.1752 | 95.7 | 87.6 | 98.5 | 112.7 | 98.6 | 5 |
| Ahmedabad (AHM001) | 39.61 | 110.66 | 3.42 | 0.1466 | 95.4 | 103.1 | 97.4 | 94.3 | 97.5 | 6 |

- Chennai has the highest late payment rate (48.61%, against 41.50% overall) and holds that place in every due year from 2019 to 2025. Of its 3,732 payments, 1,814 were late.
- Hyderabad has the most years of cover (129.36, against 107.38 overall) and the fewest units sold (27,484 in the window).
- The difference in cover between branches follows the difference in units sold and not the difference in stock.
- Pune has the second most years of cover (117.61).
- Kolkata has the highest loss rate (0.1752%) and Pune the lowest (0.1378%), and the difference is not shown to be more than chance (p 0.4993).
- The flagged share runs from 3.37% to 3.66%, and no difference between branches was found (p 0.1089).
- The score is a summary and not a verdict: the headline ranking changes with the weights, and the two alternative scores put Pune first or keep Hyderabad first.
- Without the cover signal Chennai ranks first and Hyderabad ranks third, so cover is what lifts Hyderabad and holds Chennai down.
- Read from the profile table (by hand), the highest index of each branch is cover for Hyderabad (120.5), Pune (109.5), Delhi (106.1) and Ahmedabad (103.1), late payment for Chennai (117.1) and loss for Kolkata (112.7).
- Money values are in KPI 3, KPI 4 and KPI 10 and are not repeated here.
- Limits: the score covers six branches only; equal weights are the headline; the loss signal counts recorded adjustments and the data holds no physical count; years of cover is based on the last 12 months of sales.

## Recommended actions for the business owners

1. Credit controller: start follow-up of late payers at Chennai (CHN001), where 1,814 of 3,732 payments (48.61%) were late and the late payment rate was the highest in every due year from 2019 to 2025.
2. Purchasing manager: slow replenishment at Hyderabad (129.36 years of cover) and Pune (117.61 years) until sales justify more stock; their cover is high because they sell less (27,484 and 30,402 units in the window, the two lowest of the six branches) and not because they hold more.
3. Branch managers and the operations head: read the branch risk profile table and not the rank alone, because the rank changes with the weights.
4. Inventory controller: start physical stock counts at the six branches and record each count against the book; the data holds no physical count (by inspection of the tables).

## Glossary

| Name | Meaning |
|---|---|
| index | A signal value for a branch divided by the overall value, times 100. |
| risk score | The simple average of the four indexes of a branch. |
| rank | Position of a branch by score; 1 is the highest score. |
| years of cover | Order-line stock divided by units sold in the last 12 months of sales. |
| flagged row | A ledger row tied to a Cancelled order or with units below the units on its order lines. |
| loss rate | Units subtracted by adjustments divided by units handled (IN and OUT units). |
| handled units | Units on IN and OUT ledger rows. |
| subtracted units | Units on ADJUSTMENT rows where the balance went down. |
| loaded | Dictionary of the ten tables loaded in the setup cell. |
| ledger, inv, po_h, po_l, so_h, so_l, invoices, payments, products, branches | The tables of the dataset: stock ledger, inventory master, purchase orders header and lines, sales orders header and lines, invoices, payments, products, branches. |
| invoice_counts, payment_counts | How many times each invoice id and payment id appears. |
| clean_invoices, clean_payments | Invoices and payments whose id appears once. |
| paid | Clean payments joined to clean invoices, with the late flag and the due year. |
| late_by_branch | Payments, late payments and late rate per branch. |
| late_table | Counts of not late and late payments per branch for the chi-square test. |
| pair | List of the two columns product_id and branch_id. |
| led | Ledger sorted by product, branch and date, with opening stock, previous balance and balance change. |
| net_adjustments, received_lines, received, delivered_lines, delivered, sold_last_year | Net adjustments per pair, purchase lines with order status, units received, Delivered sales lines, units delivered and units sold from 2024-01-15 to 2025-01-13. |
| risk_stock | Per pair: opening stock, received, delivered, net adjustments, units sold, order-line stock, years of cover. |
| cover_by_branch | Order-line stock, units sold and years of cover per branch. |
| cover_groups, median_cover | Pair values of years of cover grouped by branch and their medians (Kruskal-Wallis test). |
| moves, adjustments, subtracted_part | IN and OUT ledger rows, ADJUSTMENT rows, ADJUSTMENT rows where the balance went down. |
| handled_units, handled_rows, subtracted_units, subtracted_rows | Units and rows handled and subtracted per branch. |
| loss_by_branch | Handled and subtracted units and rows and loss rate per branch. |
| mark_side | Function that marks ledger rows as tied to a Cancelled order or below the order-line units. |
| flagged_rows, flag_by_branch, flag_table | Flag marks per ledger row, flagged rows and share per branch, counts for the chi-square test. |
| late_value, cover_value, flag_value, loss_value | Unrounded signal values per branch. |
| overall | The four overall values: late rate, years of cover, flagged share, loss rate. |
| score_table, index_columns | Signal values, indexes, score and rank per branch; names of the four index columns. |
| index_values, drop_table, random_generator, weight_sets, random_ranks, weight_summary | Sensitivity of the ranking: indexes, rank without one signal, 10,000 random weight sets and their ranks and summary. |
| columns_used, branch_list, sources | Columns checked for blanks, set of branch ids, branches found in each table. |
| pair_cover, first_quarter, third_quarter, low_limit, high_limit, outside, kept, kept_cover, kept_overall, trial | Outlier rule on years of cover per pair and the score rebuilt without the outlier pair. |
| late_by_year, payments_by_year, highest_each_year | Late rate by branch and due year, payments per due year, branch with the highest late rate each year. |
| loop_payments, loop_late, loop_received, loop_delivered, loop_sold, loop_net, loop_handled, loop_subtracted, loop_ledger_rows, loop_flagged, loop_stock, loop_units_sold | Totals built with plain loops (second method). |
| second, overall_second, second_score | Signals, overall values and score from the second method. |
| raw_invoices, raw_payments, raw_ledger_rows, raw_inventory, raw_po_header, raw_po_lines, raw_so_header, raw_so_lines | Raw csv rows read with the csv module for the by-hand check. |
| by_hand, three_branches, table_numbers | By-hand counts for HYD001, CHN001 and AHM001 and the table numbers they are compared with. |
| decomposition, average_stock, average_sold | Years of cover with equal stock or equal sales at every branch. |
| signals, min_max, min_max_score, two_signal_score, compare, spread | Stretched 0-to-100 values and score, two-signal score, side-by-side ranks, spread of each signal over its overall figure. |
