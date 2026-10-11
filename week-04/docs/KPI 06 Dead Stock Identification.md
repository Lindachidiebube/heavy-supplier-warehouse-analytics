# KPI 06: Dead Stock Identification

**Project:** Heavy Supplier & Warehouse Analytics | **Week 4, Phase 2** | **Notebook:** Week4_KPI06_Dead_Stock_Identification
**Data used:** sales_orders_header, sales_orders_lines, purchase_orders_header, purchase_orders_lines, inventory_master, stock_ledger
**Scope:** 180 product-branch pairs (30 products x 6 branches: AHM001, CHN001, DEL001, HYD001, KOL001, PUN001)

---

## 1. KPI Definition

| Metric | Definition | Result |
|---|---|---|
| **Days since last sale** | for each pair: the last Delivered sale date in the data (13 Jan 2025) minus the pair's own last Delivered sale date | All 180 pairs have a Delivered sale. Mean 6.82 days, median 6, min 0, max 27. No pair has gone 30 days without a sale |
| **Units sold in the last 12 months** | units on Delivered sales orders from 15 Jan 2024 to 13 Jan 2025 (365 days), per pair | Total 197,300 units; mean 1,096.11 per pair; min 703; max 1,548; every pair sold |
| **Years of cover (headline)** | order-line stock (opening stock + units on Received purchase orders - units on Delivered sales orders + net adjustments, as built in KPI 8) divided by units sold in the last 12 months | Mean 110.46 years; median 108.24; min 72.04; max 171.92; std 19.63 |
| Years of cover with ledger-based stock (comparison) | ledger-based stock (last running balance of the pair, KPI 8) divided by units sold in the last 12 months | Mean 100.7 years; min 64.8; max 157.9 |
| **Cover bands (ranking aid)** | 1 Highest: above 150 years; 2 High: 120 to 150; 3 Medium: 100 to 120; 4 Lower: up to 100 | Headline stock: 6, 53, 60 and 61 pairs (total 180). Ledger-based stock: 3, 24, 57 and 96 pairs (total 180) |
| **Dead stock by the classic tests** | pairs with no sale for a long period, or with almost no sales in 12 months | **0 pairs**. Longest gap since a sale: 27 days. Lowest 12-month sales: 703 units |

**Business question:** Which stock is sitting at branches with no demand, and where should the business act first?

**Why this KPI:** KPI 1 and KPI 8 showed that stock is far above max_stock in every pair. KPI 6 tests whether any of that stock is truly dead (not selling at all) and, because the answer is no, ranks the 180 pairs by how long their stock would take to sell, so the business knows where to start.

---

## 2. Methodology

### 2.1 Sources and scoping
- **sales_orders_header** (20,000 rows: 18,033 Delivered, 1,967 Cancelled) gives branch, status and delivery_date. **sales_orders_lines** (130,402 rows) gives product and quantity. Lines are joined to their header on so_id.
- Only **Delivered** sales orders count as sales. Cancelled orders were never delivered and are excluded.
- **Stock** is the order-line stock rebuilt in KPI 8: opening_stock (inventory_master) + units on Received purchase orders (purchase_orders_lines joined to purchase_orders_header on po_id) - units on Delivered sales orders + net adjustments from the stock_ledger. The ledger-based stock (last running balance of each pair) is used as a comparison.
- **Window:** the 12-month window is the 365 days before the last delivery date (13 Jan 2025). The filter is delivery_date after 14 Jan 2024, so the days counted are **15 Jan 2024 to 13 Jan 2025 inclusive (365 days)**. 14 Jan 2024 itself (12 order lines, 135 units) is not counted.

### 2.2 Step-by-step record
1. **Recency test.** For each pair, the last Delivered sale date and the days between it and 13 Jan 2025. All 180 pairs have a Delivered sale (0 without). Days since last sale: mean 6.82, std 5.45, min 0, 25% 2.75, median 6, 75% 11, max 27. Pairs with no sale in the last 7 days: 62; 14 days: 13; 30, 60, 90 and 180 days: 0 each. The 5 pairs furthest behind: P023 DEL001 (27 days), P011 HYD001 (26), P012 HYD001 (25), P017 PUN001 (23), P012 DEL001 (22). **A "no recent sale" rule finds no dead stock.**
2. **Low-sales test.** Units sold per pair in the last 12 months: 180 of 180 pairs sold; mean 1,096.11, std 183.11, min 703, 25% 950.75, median 1,082, 75% 1,230, max 1,548. The 5 pairs that sold the fewest units (units, order lines): P015 PUN001 703, 67; P005 HYD001 715, 72; P004 HYD001 729, 67; P019 HYD001 743, 71; P008 HYD001 793, 79. **A "very low sales" rule also finds no dead stock**, because even the slowest pair sold 703 units, and sales are close across pairs (std 183 on a mean of 1,096).
3. **Years of cover with the recorded current_stock (first look).** current_stock divided by units sold in the last 12 months: 180 pairs, 0 missing current_stock; mean 100.68, std 18.19, min 64.75, 25% 86.52, median 98.36, 75% 114.18, max 157.85. Highest 5 (units sold, order lines, current_stock, years): P005 HYD001 715, 72, 112,865, 157.85; P015 PUN001 703, 67, 106,048, 150.85; P004 HYD001 729, 67, 109,943, 150.81; P018 HYD001 809, 82, 112,953, 139.62; P002 HYD001 880, 85, 121,013, 137.51. Lowest 5: P027 CHN001 1,513, 130, 97,973, 64.75; P004 CHN001 1,465, 138, 103,984, 70.98; P002 CHN001 1,485, 146, 105,885, 71.30; P029 CHN001 1,423, 123, 101,572, 71.38; P023 CHN001 1,500, 149, 107,267, 71.51.
4. **Years of cover by branch (current_stock version).** HYD001 mean 119.9 (min 89.2, max 157.9); PUN001 108.7 (84.1 to 150.9); DEL001 105.5 (80.9 to 129.5); AHM001 101.7 (79.2 to 126.8); KOL001 86.4 (72.4 to 111.4); CHN001 81.8 (64.8 to 101.3); 30 pairs in each branch. The highest and lowest branch means are 38.1 years apart. The ranges overlap (HYD001's lowest pair, 89.2, is below CHN001's highest, 101.3), so branch is a strong driver but not the only one.
5. **Would a fixed line work?** Pairs above 1, 2, 5, 10 and 50 years of cover: 180 of 180 each; above 100 years: 84 of 180 (current_stock version). An "above this line" flag therefore flags every pair at any sensible line, so pairs have to be ranked against each other.
6. **Why HYD001 is higher (current_stock version, mean per pair by branch):** HYD001 stock 108,176 and 916 units sold (118.1 years); PUN001 108,542 and 1,013 (107.1); DEL001 109,622 and 1,052 (104.2); AHM001 105,166 and 1,046 (100.5); KOL001 104,127 and 1,218 (85.5); CHN001 107,808 and 1,331 (81.0). Stock per pair is about the same in every branch (104,127 to 109,622, about 5% apart) while units sold range from 916 to 1,331 (CHN001 about 45% higher than HYD001). **HYD001's higher cover comes from lower sales, not from holding more stock**; its stock is not the highest (DEL001 and PUN001 are higher). The ratio of the means (118.1 for HYD001) differs slightly from the mean of the 180 pair ratios (119.9) because they are two different calculations; both are correct.
7. **Order-line stock rebuilt in this notebook.** Ledger chain on all 237,230 rows from opening_stock: IN rows 127,611 (change = +quantity for 127,611); OUT rows 107,165 (change = -quantity for 107,165); ADJUSTMENT rows 2,454 (change = +/- quantity for 2,454). Net adjustments, all pairs: +62. Order-line stock, all pairs: **21,185,215**, equal to the KPI 8 headline.
8. **Years of cover with the headline order-line stock.** 180 pairs, 0 empty values, 0 negative stock. Mean 110.46, std 19.63, min 72.04, 25% 96.29, median 108.24, 75% 124.37, max 171.92. Highest 5 (units sold, order-line stock, years): P005 HYD001 715, 122,924, 171.92; P004 HYD001 729, 120,019, 164.64; P015 PUN001 703, 115,509, 164.31; P018 HYD001 809, 123,601, 152.78; P002 HYD001 880, 132,967, 151.10. Lowest 5: P027 CHN001 1,513, 108,989, 72.04; P023 CHN001 1,500, 116,198, 77.47; P002 CHN001 1,485, 115,872, 78.03; P030 CHN001 1,450, 114,014, 78.63; P029 CHN001 1,423, 112,030, 78.73. Pairs above 100 years: 119 of 180; above 120 years: 59; above 150 years: 6. Values are higher than with current_stock because order-line stock is higher than current_stock in all 180 pairs (KPI 8).
9. **Cover bands (headline stock).** 1 Highest (above 150 years): 6 pairs; 2 High (120 to 150): 53; 3 Medium (100 to 120): 60; 4 Lower (up to 100): 61; total 180. Pairs per branch, in the band order 1, 2, 3, 4: AHM001 0, 8, 16, 6; CHN001 0, 0, 5, 25; DEL001 0, 15, 10, 5; HYD001 5, 17, 7, 1; KOL001 0, 0, 8, 22; PUN001 1, 13, 14, 2. Every branch has 30 pairs. The 6 pairs in the highest band (units sold, order-line stock, years): P005 HYD001 715, 122,924, 171.92; P004 HYD001 729, 120,019, 164.64; P015 PUN001 703, 115,509, 164.31; P018 HYD001 809, 123,601, 152.78; P002 HYD001 880, 132,967, 151.10; P019 HYD001 743, 112,112, 150.89. **Five of the six are at HYD001 and one is at PUN001.**
10. **Ledger-based stock as a comparison.** Ledger-based stock, all pairs: 19,305,994 (KPI 8 figure). Years of cover: mean 100.7, min 64.8, max 157.9. Bands: 3, 24, 57 and 96 pairs. Fewer pairs reach the high bands because the ledger is missing about 10% of the order lines (KPI 8).

### 2.3 Alternatives considered and rejected
- **"No sale in the last N days" as the dead-stock rule.** Rejected: it finds 0 dead pairs at 30 days or more (longest gap is 27 days), so it cannot separate any pair.
- **"Very low sales in 12 months" as the dead-stock rule.** Rejected: the slowest pair still sold 703 units, so no pair has near-zero demand.
- **A fixed years-of-cover line (such as 5 or 10 years) as the flag.** Rejected: every one of the 180 pairs is above 50 years, so the line would flag everything. Only a ranking separates the pairs.
- **Recorded current_stock as the stock figure.** Kept only as a first look. The headline uses the order-line stock from KPI 8, which is the more complete figure (the ledger misses about 10% of order lines and current_stock follows the ledger).
- **Counting Cancelled orders as sales.** Rejected: they were never delivered. Their units are measured in Check 3.
- **Putting a money value on the stock or measuring turnover.** Left to KPI 10 (excess stock value) and KPI 9 (turnover), to keep KPI 6 to one question.

---

## 3. Verification Log

| # | Check | Method | Result |
|---|---|---|---|
| 1 | Empty, zero and negative values in every column used | isna, == 0 and < 0 counts on all columns used in the 6 tables | 0 empty values in every column used. 0 zeros and 0 negatives in every quantity and stock column (sales lines, purchase lines, inventory_master, stock_ledger) |
| 2 | Every pair has sales; no division by zero | Pairs in inventory_master vs pairs in the 12-month sales; zero units; sales pairs not in inventory_master; stock of zero or less | 180 pairs, 0 missing from the sales, 0 with zero units sold, 0 sales pairs outside inventory_master, 0 pairs with stock of zero or less; smallest units sold 703 |
| 3 | Cancelled orders excluded; window edge | Units in the window by order status; Delivered total vs units_sold total; start-day units; window moved by one day each way | Delivered 197,300 units equal the units_sold total 197,300; Cancelled orders carry 24,652 units in the window and are excluded; with a 364-day window 1 pair changes band, with 366 days 0 pairs change |
| 4 | Totals reconcile | Direct recount from the raw tables; stock parts vs the verified KPI 8 figures; pair counts | Units sold in the window 197,300 by the direct route; delivered all time 1,234,939; opening 32,969; received 22,387,123; delivered 1,234,939; net adjustments 62; order-line stock 21,185,215; every figure equals KPI 8; 180 pairs in the cover table and 180 in the bands |
| 5 | Second-method recompute | Years of cover rebuilt with a pivot table straight from the raw order lines; ledger-based stock comparison | 180 cells compared, largest difference 0.0 years; ledger-based stock total 19,305,994 equals KPI 8 |
| 6 | Outliers and band edges | 1.5 x IQR rule; pairs within 2 years of 100, 120 and 150; overlap of the worst 6 pairs under both stock figures | 0 low outliers, 1 high outlier (P005 HYD001, a real value); 23 pairs within 2 years of a band line; the same 6 pairs are the worst under both stock figures |
| 7 | Date coverage | First and last date in the window; order lines per month per branch; last sale per branch | Window 15 Jan 2024 to 13 Jan 2025; all 78 month-branch cells have sales; every branch sold within 6 days of the end of the data |
| 8 | Manual spot-check | 3 pairs rebuilt by hand from the raw order lines, purchase lines, opening stock and ADJUSTMENT rows | By-hand and table figures are identical for all 3 pairs |

### Check 1: empty, zero and negative values
| Table | Rows | Columns tested | Result |
|---|---|---|---|
| sales_orders_lines | 130,402 | so_id, product_id, quantity | empty 0; quantity zeros 0, negatives 0 |
| sales_orders_header | 20,000 | so_id, branch_id, order_status, delivery_date | empty 0 |
| purchase_orders_lines | 155,495 | po_id, product_id, quantity | empty 0; quantity zeros 0, negatives 0 |
| purchase_orders_header | 24,000 | po_id, branch_id, po_status | empty 0 |
| inventory_master | 180 | product_id, branch_id, opening_stock, current_stock | empty 0; opening_stock and current_stock zeros 0, negatives 0 |
| stock_ledger | 237,230 | product_id, branch_id, movement_type, movement_date, quantity, running_balance | empty 0; quantity and running_balance zeros 0; negative quantity 0; negative running_balance 0 |

### Check 2: every pair has sales
- Pairs in inventory_master: 180.
- Pairs missing from the 12-month sales: 0.
- Pairs with zero units sold in the window: 0.
- Sales pairs that are not in inventory_master: 0.
- Pairs with order-line stock of zero or less: 0.
- Smallest units sold in the window: 703.

### Check 3: Cancelled orders and the window edge
- Order statuses in sales_orders_header: Delivered 18,033 and Cancelled 1,967 (total 20,000; no other status).
- In the window, by status:

| Order status | Order lines | Units |
|---|---|---|
| Delivered | 18,889 | 197,300 |
| Cancelled | 2,357 | 24,652 |

- Delivered units in the window (197,300) equal the sum of units_sold across the 180 pairs (197,300). The Cancelled units (24,652) are about 11.1% of the units on all orders in the window and are excluded.
- On the start day, 14 Jan 2024, there were 12 Delivered order lines (135 units); the filter does not count that day, so the window counted is 15 Jan 2024 to 13 Jan 2025.
- Moving the window start by one day each way: a 364-day window changes the band of 1 pair; a 366-day window changes 0 pairs.

### Check 4: totals reconcile
| Item | This notebook | KPI 8 figure |
|---|---|---|
| Units sold in the window, direct route (Delivered order ids first, then delivery dates) | 197,300 | not applicable |
| Delivered units, all time | 1,234,939 | 1,234,939 |
| Opening stock, all pairs | 32,969 | 32,969 |
| Received units, all pairs | 22,387,123 | 22,387,123 |
| Delivered units, all pairs | 1,234,939 | 1,234,939 |
| Net adjustments, all pairs | 62 | 62 |
| Order-line stock, all pairs | 21,185,215 | 21,185,215 |

Pairs in the cover table: 180. Pairs in the bands: 180. The sum 32,969 + 22,387,123 - 1,234,939 + 62 = 21,185,215.

### Check 5: second-method recompute
- Years of cover was rebuilt with a pivot table (product by branch) straight from the raw Delivered order lines in the window, divided into the order-line stock. Cells compared: 180. Largest difference from the main table: 0.0 years.
- Ledger-based stock, all pairs: 19,305,994 (KPI 8 figure 19,305,994). Years of cover with it: mean 100.7, min 64.8, max 157.9.
- Bands with ledger-based stock: 1 Highest 3; 2 High 24; 3 Medium 57; 4 Lower 96 (total 180). With the headline stock: 6, 53, 60, 61. The bands depend on which stock figure is used, because the ledger misses about 10% of order lines and so shows less stock.

### Check 6: outliers and band edges
- Years of cover (headline): Q1 96.3, Q3 124.4; the normal range by the 1.5 x IQR rule is 54.2 to 166.5 years. Pairs below the range: 0. Pairs above the range: 1, which is P005 HYD001 at 171.92 years, the pair with the highest cover. Its value is real (Check 8 rebuilds it by hand).
- Pairs within 2 years of a band line: 12 near 100, 9 near 120 and 2 near 150, so 23 of 180 pairs (about 12.8%). The exact band of these pairs is soft.
- The 6 worst pairs by headline stock are the same 6 pairs as the 6 worst by ledger stock (6 of 6). Worst 6 by ledger stock (years): P005 HYD001 157.9, P015 PUN001 150.9, P004 HYD001 150.8, P018 HYD001 139.6, P002 HYD001 137.5, P019 HYD001 134.6.

### Check 7: date coverage
- First Delivered sale in the window: 2024-01-15. Last: 2025-01-13.
- Delivered order lines per month and branch:

| Month | AHM001 | CHN001 | DEL001 | HYD001 | KOL001 | PUN001 |
|---|---|---|---|---|---|---|
| 2024-01 | 136 | 165 | 206 | 136 | 215 | 110 |
| 2024-02 | 319 | 324 | 306 | 173 | 235 | 216 |
| 2024-03 | 209 | 432 | 246 | 260 | 274 | 244 |
| 2024-04 | 242 | 240 | 187 | 259 | 335 | 222 |
| 2024-05 | 211 | 366 | 251 | 184 | 254 | 278 |
| 2024-06 | 250 | 328 | 196 | 182 | 278 | 230 |
| 2024-07 | 332 | 360 | 211 | 227 | 293 | 266 |
| 2024-08 | 308 | 331 | 292 | 176 | 305 | 247 |
| 2024-09 | 224 | 263 | 241 | 296 | 298 | 273 |
| 2024-10 | 281 | 278 | 288 | 284 | 298 | 361 |
| 2024-11 | 258 | 386 | 197 | 188 | 286 | 243 |
| 2024-12 | 225 | 317 | 299 | 183 | 371 | 164 |
| 2025-01 | 52 | 41 | 45 | 58 | 69 | 75 |

- 13 calendar months x 6 branches = 78 cells; none is empty or zero. The first month (from 15 Jan) and the last month (to 13 Jan) are partial. The lowest full-month value is 164 order lines (PUN001, December 2024).
- Last Delivered sale per branch (all history): AHM001 2025-01-12; CHN001 2025-01-13; DEL001 2025-01-07; HYD001 2025-01-11; KOL001 2025-01-09; PUN001 2025-01-12. Every branch sold within 6 days of the end of the data.
- HYD001 sold in every month of the window, so its lower units sold are not caused by a missing month.

### Check 8: manual spot-check of 3 pairs
Rebuilt by hand from the raw sales lines, purchase lines, opening stock and the pair's ADJUSTMENT rows, then compared with the table:

| | P005 HYD001 (highest cover) | P027 CHN001 (lowest cover) | P015 AHM001 (closest to the median) |
|---|---|---|---|
| Opening stock | 178 | 84 | 85 |
| Units received (Received POs) | 128,466 | 117,598 | 123,103 |
| Units delivered, all time | 5,663 | 8,628 | 7,250 |
| ADJUSTMENT rows | 19 | 8 | 16 |
| Net adjustments | -57 | -65 | -28 |
| Order-line stock by hand | 122,924 | 108,989 | 115,910 |
| Units sold in the window by hand | 715 | 1,513 | 1,074 |
| Years of cover by hand | 171.92 | 72.04 | 107.92 |
| Table: units sold, stock, years | 715, 122,924, 171.92 | 1,513, 108,989, 72.04 | 1,074, 115,910, 107.92 |

All 3 pairs match the table on every line. For example, 178 + 128,466 - 5,663 - 57 = 122,924, and 122,924 / 715 = 171.92. The stock of P015 AHM001 (115,910) also equals its figure in KPI 8.

---

## 4. Data Quality Notes

1. **No stock is dead by the classic tests.** The longest gap since a sale is 27 days and the lowest 12-month sales are 703 units (methodology steps 1 and 2). The KPI therefore reports this plainly and ranks pairs by years of cover instead of inventing a cut-off to produce a dead list.
2. **Every pair is far overstocked.** At last year's selling speed the stock of the typical pair would last about 110 years (min 72.04, max 171.92). Any normal stock level would be a few months. This matches KPI 1 and KPI 8, which also found stock far above max_stock in every pair.
3. **Years of cover assumes last year's selling speed continues.** It is an indicator for ranking, not a forecast. It is built from records, not from a physical count.
4. **Window wording.** The 12-month window counted is 15 Jan 2024 to 13 Jan 2025 (365 days); the start day 14 Jan 2024 is not counted. Moving the window by one day changes the band of at most 1 of the 180 pairs (Check 3).
5. **Cancelled orders.** 1,967 of 20,000 sales orders are Cancelled. In the window their lines carry 24,652 units (about 11.1% of the units on all orders in the window). They are excluded from sales.
6. **The bands are a ranking aid, not a business standard.** The lines at 100, 120 and 150 years were chosen from the counts. 23 of 180 pairs are within 2 years of a line, so their exact band is soft. The top of the list is firm: the same 6 pairs are the worst under both stock figures.
7. **The bands depend on the stock figure.** With the headline order-line stock the bands hold 6, 53, 60 and 61 pairs; with the ledger-based stock they hold 3, 24, 57 and 96 pairs, because the ledger is missing about 10% of order lines (KPI 8). Both are shown; the headline is the order-line stock.
8. **Branch pattern.** HYD001 has the highest cover (5 of the 6 highest-band pairs; 17 more in the High band; only 1 pair in the Lower band). CHN001 and KOL001 have no pair above 120 years (25 of CHN001's 30 pairs and 22 of KOL001's 30 pairs are in the Lower band). The recorded stock per pair is about the same in every branch (104,127 to 109,622), while units sold per pair per year range from 916 (HYD001) to 1,331 (CHN001). Stock levels therefore do not follow branch demand.
9. **Why HYD001 sells less is not investigated.** It is outside the question of this KPI and the data used here cannot explain it. It is listed as a follow-up for the demand KPIs (KPI 18) and branch comparisons later in the plan.
10. **The first look used the recorded current_stock.** Those figures (steps 3 to 6 of the methodology) were replaced by the headline order-line stock once it was rebuilt and matched to KPI 8. They are kept here because current_stock equals the ledger for 166 of 180 pairs and the picture is the same.
11. **Units.** products.uom is piece for 24 products, set for 5 and cartridge for 1 (KPI 8). Years of cover is a ratio of stock to sales of the same product in the same unit, so it is not affected by the mix.

---

## 5. Business Interpretation

**Is any stock dead?** No. Every one of the 180 pairs sold within the last 27 days of the data and at least 703 units in the last 12 months. By the usual tests the business has no dead stock.

**The real problem is overstock.** Every pair holds far more than it sells: on average about 110 years of last year's sales, and no pair holds less than 72 years. The six worst pairs hold more than 150 years: P005 HYD001 (171.92), P004 HYD001 (164.64), P015 PUN001 (164.31), P018 HYD001 (152.78), P002 HYD001 (151.10) and P019 HYD001 (150.89). The next 53 pairs hold between 120 and 150 years. The same six pairs are the worst whichever stock figure is used.

**Where it is worst.** HYD001 is the heaviest branch. It holds about the same stock per pair as the others but sells the least (916 units per pair per year, against 1,331 at CHN001), so its stock lasts the longest. Each branch holds about the same stock whatever it sells, so stock levels are not set from each branch's own demand.

**What this means for the next KPIs.** KPI 10 (capital tied up in excess stock) will put a money value on this overstock and can start from the bands here. KPI 9 (inventory turnover) will measure the same problem as a rate. KPI 18 (demand forecast) and the branch comparisons are the place to look at why HYD001 sells less.

**Recommended actions for the business owners**
1. **Purchasing manager:** pause new purchase orders for the 6 highest-band pairs and review them for the 53 High-band pairs. At last year's selling speed they already hold more than 120 years of stock.
2. **Inventory planning team:** set stock targets per branch from each branch's own selling rate. Today every branch holds about 105,000 to 110,000 units per pair whatever it sells.
3. **HYD001 branch manager with sales:** review why HYD001 sells the least per pair; this is a follow-up for the demand KPIs, not something KPI 6 explains.
4. **Warehouse manager:** run a physical count on the six highest-band pairs to confirm the stock figure before any action, as recommended in KPI 8.
