# KPI 08: Reconstructed Stock Balance

**Project:** Heavy Supplier & Warehouse Analytics | **Week 4, Phase 2** | **Notebook:** Week4_KPI08_Reconstructed_Stock_Balance
**Data used:** stock_ledger, inventory_master, purchase_orders_header, purchase_orders_lines, sales_orders_header, sales_orders_lines
**Scope:** 180 product-branch pairs (30 products x 6 branches: AHM001, CHN001, DEL001, HYD001, KOL001, PUN001)

---

## 1. KPI Definition

| Metric | Definition | Result |
|---|---|---|
| **Reconstructed stock (headline): order-line stock** | opening_stock + units on Received purchase orders - units on Delivered sales orders + net adjustments, per product-branch pair | Total **21,185,215** units; mean **117,695.64** per pair; min 98,642; max 132,967 |
| Ledger-based stock (comparison) | opening_stock + every signed ledger movement (IN +, OUT -, ADJUSTMENT by its proven direction) = last running_balance of the pair | Total **19,305,994**; mean **107,255.52**; min 88,853; max 121,013 |
| Ledger-based stock without cancelled-order rows | the ledger figure after removing the rows that belong to Cancelled orders | Total **19,070,672**; mean **105,948.18** |
| Recorded current_stock vs ledger-based stock | pairs where inventory_master.current_stock equals the rebuilt ledger balance | Equal for **166 of 180** pairs; 14 differ by -515 to +16 (sum -2,728) |
| Gap: order-line stock minus ledger-based stock | per pair | Total **1,879,221** (about 9.7% of ledger-based stock); mean 10,440.12; min 6,226; max 15,127; **all 180 gaps are positive** |
| Stock as a multiple of max_stock | stock divided by the pair's max_stock | Ledger-based: min 177.4, median 349.4, max 858.1. Order-line: min 193.2, median 385.6, max 929.2. **0 pairs at or below max_stock** |

**Business question:** What is the real stock level of each product at each branch, and can the recorded current_stock be trusted?

**Why this KPI:** Week 1 found that current_stock (88,853 to 121,013 units per pair) is 100 to 850 times larger than opening_stock (80 to 300 units per pair). KPI 8 rebuilds the stock from the movements and the orders to test whether current_stock is right, and to give the later stock KPIs (6, 9, 10, 19, 21) one verified stock figure to start from.

---

## 2. Methodology

### 2.1 Sources and scoping
- **stock_ledger:** 237,230 rows, 9 columns (movement_id, product_id, branch_id, movement_type, movement_date, quantity, reference_type, reference_id, running_balance). IN 127,611 rows (reference PO), OUT 107,165 rows (reference SO), ADJUSTMENT 2,454 rows (reference ADJ-n).
- **inventory_master:** 180 rows, 8 columns (product_id, branch_id, opening_stock, reorder_level, safety_stock, max_stock, current_stock, warehouse_bin).
- **Order tables:** purchase_orders_header 24,000 rows (21,630 Received + 2,370 Cancelled); purchase_orders_lines 155,495; sales_orders_header 20,000; sales_orders_lines 130,402. Branch and status sit only in the headers, product and quantity only in the lines, so lines are joined to their header on po_id / so_id.
- Only **Received** purchase orders and **Delivered** sales orders count as stock movements in the order-line method.

### 2.2 Step-by-step record
1. **Look at the two tables.** Ledger and inventory_master row and column counts match the Week 3 notes (237,230 / 180 rows; 9 / 8 columns). First 5 ledger rows (P001 at AHM001): OUT 8 (balance 228), OUT 12 (216), IN 174 (390), OUT 12 (378), OUT 11 (367), dated 2019-01-03 to 2019-01-25; each balance equals the previous one +/- the quantity. First 5 inventory_master rows (P001 at DEL001, PUN001, CHN001, HYD001, KOL001): opening_stock 196, 198, 187, 226, 263; current_stock 115,928, 109,389, 99,373, 108,772, 103,808.
2. **Does the ledger start from opening_stock?** For the first ledger row of each pair, balance before the row = running_balance -/+ quantity. Pairs found in both tables: 180; only in one table: 0. First-row type: OUT 146, IN 21, ADJUSTMENT 13. Of the 167 testable pairs (OUT or IN first), **167 match opening_stock and 0 do not.**
3. **Direction of the 13 ADJUSTMENT-first pairs.** change = running_balance - opening_stock, compared with quantity: 8 add and 5 subtract, 0 unreadable (table in Check 3).
4. **Full chain test on all 237,230 ledger rows.** change = running_balance - previous balance of the same pair (opening_stock for each pair's first row). IN rows: change = +quantity for 127,611 of 127,611. OUT rows: change = -quantity for 107,165 of 107,165. ADJUSTMENT rows: 2,454, of which 1,227 subtract and 1,227 add, 0 unreadable. 127,611 + 107,165 + 2,454 = 237,230. The ledger is an unbroken chain from opening_stock for all 180 pairs.
5. **Ledger-based rebuild per pair.** reconstructed_stock = opening_stock + sum of signed movements. 180 pairs; rows with signed quantity 0: 0; rebuilt stock = last ledger running_balance for 180 of 180; pairs that ever went below zero: 0; rebuilt stock equals current_stock for 166 of 180. Rebuilt stock: mean 107,255.52, std 5,348.57, min 88,853, 25% 103,487.5, median 107,790.5, 75% 110,779.5, max 121,013.
6. **The 14 pairs where current_stock differs, and the totals** (table in Data Quality Note 3). Total units by type: ADJUSTMENT 66,990, IN 20,397,795, OUT 1,124,832 (IN 18.13 times OUT). Net effect of adjustments +62. Total opening stock 32,969. 32,969 + 20,397,795 - 1,124,832 + 62 = **19,305,994** = total rebuilt stock.
7. **Why the 14 pairs differ.** current_stock appears as a running_balance in the ledger of each of the 14 pairs (14 of 14), on the same date as the pair's last ledger date, 1 or 2 movements before its final row.
8. **The 17 ledger rows after that match.** Together they equal the gap of every one of the 14 pairs exactly (table in Data Quality Note 3).
9. **The order tables.** 12 csv files in the data folder; row counts and columns as in 2.1.
10. **The orders behind the 17 late rows.** All 16 purchase rows belong to POs with status Received, in the right branch, with movement_date = received_date; the 1 sales row is SO-617518, Delivered, PUN001, movement_date = delivery_date = 2025-01-09.
11. **Second rebuild from the order lines.** Units received (Received POs) 22,387,123; units delivered (Delivered SOs) 1,234,939; order-line stock total 21,185,215; ledger-based total 19,305,994; gap 1,879,221; pairs with order-line stock below zero: 0.
12. **Where the 1,879,221 gap comes from** (reconciliation in Data Quality Note 6).

### 2.3 Alternatives considered and rejected
- **Treating current_stock as wrong.** It equals the ledger balance for 166 of 180 pairs and is a real ledger balance (a few movements early) for the other 14. The problem is not a wrong copy; the ledger itself is incomplete (Note 5).
- **Using the ledger alone as the headline.** About 10% of order lines never reached the ledger, and the ledger also holds rows of cancelled orders. Both effects are measured exactly in Note 6, so the order-line method is used as the headline and the ledger method is kept for comparison.
- **Dropping the ADJUSTMENT rows** (as KPI 1 did for a ratio). Rejected here: their direction is proven for all 2,454, and they change the final balance (net +62 overall; up to +172 or -129 for a single pair).
- **Reading adjustment direction from movement_type.** Not possible: all adjustments are marked ADJUSTMENT and quantity is always positive; direction is read from the change in running_balance.

---

## 3. Verification Log

| # | Check | Method | Result |
|---|---|---|---|
| 1 | Missing, zero and negative values in every column used | isna, == 0 and < 0 counts on all columns used in the 6 tables | 0 empty, 0 zero, 0 negative in every column tested; the only empties in the tables are received_date (2,370, Cancelled POs), tested in Check 7. |
| 2 | Every pair has movements and order lines | For all 180 pairs: IN rows, OUT rows, received units, delivered units | 180 pairs; 0 pairs with no IN rows, 0 with no OUT rows, 0 with no received units, 0 with no delivered units. |
| 3 | All 2,454 adjustments readable | Direction from running_balance change (Steps 3 and 4) | 2,454 of 2,454 readable (1,227 add, 1,227 subtract), 0 unreadable. |
| 4 | Totals reconcile | Opening + IN - OUT + adjustments vs rebuilt stock; ledger rows by type and order status vs the gap | 32,969 + 20,397,795 - 1,124,832 + 62 = 19,305,994 = rebuilt total; 127,611 + 107,165 + 2,454 = 237,230; explained gap 1,879,221 = actual gap 1,879,221 (exact). |
| 5 | Second-method recompute | Ledger-based rebuild vs last running_balance; order-line rebuild; reconciliation of the two | Rebuilt = last running_balance for 180 of 180 pairs; the two methods differ by 1,879,221, and every unit of that is explained. |
| 6 | Outliers | Ratio to max_stock; 1.5 x IQR rule; 5 largest, 5 smallest and 5 highest-ratio pairs | 0 pairs at or below max_stock; 0 high outliers; 1 low outlier (P019 KOL001) under both methods, a real value. |
| 7 | Date coverage | Date ranges of the ledger and both order tables; empties in received_date, delivery_date, reference_type; ledger rows after the last sales date | All order dates fall inside the ledger period; received_date empty only for Cancelled POs; delivery_date and reference_type 0 empty; the gap between the last sale and the last ledger date is explained. |
| 8 | Manual spot-check | 3 pairs (lowest, highest, highest ratio) rebuilt by hand from raw ledger rows, order lines and ADJUSTMENT rows | By-hand and table figures are identical for all 3 pairs. |

### Check 1: empty, zero and negative values
| Table | Rows | Columns tested | Result |
|---|---|---|---|
| stock_ledger | 237,230 | product_id, branch_id, movement_type, movement_date, quantity, running_balance, reference_id | empty 0 everywhere; quantity and running_balance also zeros 0, negatives 0 |
| inventory_master | 180 | product_id, branch_id, opening_stock, max_stock, current_stock | empty 0; opening_stock, max_stock, current_stock zeros 0, negatives 0 |
| purchase_orders_lines | 155,495 | po_id, product_id, quantity | empty 0; quantity zeros 0, negatives 0 |
| sales_orders_lines | 130,402 | so_id, product_id, quantity | empty 0; quantity zeros 0, negatives 0 |
| purchase_orders_header | 24,000 | po_id, branch_id, po_status | empty 0 |
| sales_orders_header | 20,000 | so_id, branch_id, order_status | empty 0 |

The columns reference_type, received_date and delivery_date were tested in Check 7.

### Check 2: every pair is covered
Pairs counted: 180. Pairs with no IN rows: 0. Pairs with no OUT rows: 0. Pairs with no received units: 0. Pairs with no delivered units: 0.

### Check 3: the 13 pairs whose ledger starts with an ADJUSTMENT
| Pair | Quantity | Opening | Balance after | Direction |
|---|---|---|---|---|
| P003 KOL001 | 19 | 234 | 253 | adds |
| P008 PUN001 | 43 | 86 | 43 | subtracts |
| P009 HYD001 | 40 | 282 | 322 | adds |
| P013 AHM001 | 28 | 84 | 112 | adds |
| P013 HYD001 | 37 | 233 | 270 | adds |
| P015 DEL001 | 37 | 223 | 186 | subtracts |
| P016 CHN001 | 42 | 180 | 222 | adds |
| P016 KOL001 | 33 | 148 | 181 | adds |
| P026 AHM001 | 38 | 263 | 301 | adds |
| P027 CHN001 | 29 | 84 | 55 | subtracts |
| P027 DEL001 | 30 | 98 | 68 | subtracts |
| P027 HYD001 | 20 | 123 | 143 | adds |
| P029 KOL001 | 25 | 150 | 125 | subtracts |

8 add and 5 subtract; no result is negative. Together with the Week 3 reading (1,219 add, 1,222 subtract), this gives 1,227 adds and 1,227 subtracts = 2,454, which the full chain test (Step 4) confirmed on all 2,454 rows.

### Check 4: totals reconcile
- Units by movement type: ADJUSTMENT 66,990; IN 20,397,795; OUT 1,124,832 (IN and OUT equal the Week 3 notes exactly). Net effect of adjustments +62. Total opening stock 32,969. Average max_stock 321.76; average ledger-based stock 107,255.52, about 333 times the average max_stock.
- Ledger rows and units by movement type and order status:

| Movement / order status | Rows | Units |
|---|---|---|
| ADJUSTMENT / ADJ | 2,454 | 66,990 |
| IN / Cancelled | 1,542 | 248,556 |
| IN / Received | 126,069 | 20,149,239 |
| OUT / Cancelled | 1,275 | 13,234 |
| OUT / Delivered | 105,890 | 1,111,598 |

Rows add to 237,230; no row has an unknown order status.

### Check 5: second-method recompute
| Version | Total stock | Average per pair |
|---|---|---|
| Ledger as recorded | 19,305,994 | 107,255.52 |
| Ledger without cancelled-order rows (32,969 + 20,149,239 - 1,111,598 + 62) | 19,070,672 | 105,948.18 |
| Order lines (Received POs, Delivered SOs, plus adjustments) | 21,185,215 | 117,695.64 |

order_line_stock per pair: mean 117,695.64, std 5,367.49, min 98,642, 25% 113,562.25, median 117,900.5, 75% 121,422.5, max 132,967. Gap (order-line minus ledger) per pair: mean 10,440.12, std 1,533.93, min 6,226, 25% 9,361.25, median 10,458, 75% 11,409, max 15,127.

### Check 6: outliers
**Stock as a multiple of max_stock** (180 pairs):

| | Mean | Std | Min | 25% | Median | 75% | Max |
|---|---|---|---|---|---|---|---|
| Ledger-based | 385.1 | 155.8 | 177.4 | 259.2 | 349.4 | 477.5 | 858.1 |
| Order-line | 422.6 | 170.6 | 193.2 | 280.9 | 385.6 | 525.3 | 929.2 |

Pairs at or below max_stock: 0 (ledger-based) and 0 (order-line). The mean of the 180 ratios (385.1) and the earlier ratio of averages (107,255.52 / 321.76 = about 333) are two different calculations; both are correct.

**1.5 x IQR rule:**

| Stock | Q1 | Q3 | Normal range | Below | Above |
|---|---|---|---|---|---|
| Ledger-based | 103,488 | 110,780 | 92,550 to 121,718 | 1 | 0 |
| Order-line | 113,562 | 121,422 | 101,772 to 133,213 | 1 | 0 |

**5 smallest pairs** (by order-line stock):

| Pair | max_stock | Ledger | Order-line | Times max_stock |
|---|---|---|---|---|
| P019 KOL001 | 141 | 88,853 | 98,642 | 699.6 |
| P015 KOL001 | 508 | 96,547 | 106,571 | 209.8 |
| P018 KOL001 | 284 | 96,607 | 106,960 | 376.6 |
| P013 AHM001 | 150 | 98,687 | 107,342 | 715.6 |
| P016 KOL001 | 277 | 97,829 | 107,407 | 387.8 |

**5 largest pairs:**

| Pair | max_stock | Ledger | Order-line | Times max_stock |
|---|---|---|---|---|
| P002 HYD001 | 383 | 121,013 | 132,967 | 347.2 |
| P012 DEL001 | 489 | 119,923 | 129,402 | 264.6 |
| P003 PUN001 | 359 | 118,139 | 129,186 | 359.8 |
| P017 HYD001 | 172 | 117,058 | 128,477 | 747.0 |
| P007 DEL001 | 383 | 119,439 | 127,827 | 333.8 |

**5 highest ratios to max_stock:**

| Pair | max_stock | Ledger | Order-line | Times max_stock |
|---|---|---|---|---|
| P025 HYD001 | 133 | 114,128 | 123,586 | 929.2 |
| P011 PUN001 | 138 | 108,691 | 122,817 | 890.0 |
| P015 AHM001 | 131 | 105,135 | 115,910 | 884.8 |
| P023 HYD001 | 134 | 105,981 | 114,637 | 855.5 |
| P013 PUN001 | 133 | 104,221 | 113,527 | 853.6 |

max_stock across the 180 pairs: min 131, max 582. The wide ratio spread (177 to 929) comes from max_stock differing between pairs while stock stays tight (88,853 to 121,013); the highest-ratio pairs all have max_stock of 131 to 138. The one low outlier, **P019 KOL001**, is the same pair under both methods (ledger 88,853 below 92,550; order-line 98,642 below 101,772; the next smallest pair is inside the range). It still holds 699.6 times its max_stock, and its value is real: the ledger chain is unbroken for all 180 pairs. The largest pair, P002 HYD001 (132,967), is inside the normal range (upper limit 133,213).

### Check 7: date coverage
| Source | First date | Last date |
|---|---|---|
| Ledger movement_date | 2019-01-01 | 2025-01-28 |
| Purchase orders received_date | 2019-01-11 | 2025-01-28 |
| Sales orders delivery_date | 2019-01-02 | 2025-01-13 |

- received_date empty: 2,370, all po_status Cancelled (none on a Received order). delivery_date empty: 0. reference_type empty: 0; counts PO 127,611, SO 107,165, ADJ 2,454 (sum 237,230 = ledger rows).
- **Why the ledger runs 15 days past the last sale:** 347 ledger rows are dated after 13 Jan 2025, and all 347 are PO / IN (0 sales, 0 adjustments). Last ledger date per source: ADJ 2024-12-29, PO 2025-01-28, SO 2025-01-13, equal to the last sales delivery_date. Sales stop on 13 Jan 2025 while purchases keep arriving until 28 Jan 2025.

### Check 8: manual spot-check of 3 pairs
Rebuilt by hand from the raw ledger rows, the order lines and the pair's ADJUSTMENT rows, then compared with the table:

| | P019 KOL001 | P002 HYD001 | P025 HYD001 |
|---|---|---|---|
| Opening stock | 92 | 219 | 83 |
| IN (PO rows) | 95,724 | 125,818 | 119,686 |
| OUT (SO rows) | 7,135 | 4,958 | 5,512 |
| Opening + IN - OUT | 88,681 | 121,079 | 114,257 |
| Number of ADJUSTMENT rows | 11 | 11 | 11 |
| Total balance change from the ADJUSTMENT rows | +172 | -66 | -129 |
| Opening + IN - OUT + adjustments | 88,853 | 121,013 | 114,128 |
| Last running_balance | 88,853 | 121,013 | 114,128 |
| Table ledger-based stock | 88,853 | 121,013 | 114,128 |
| Table net adjustments | 172 | -66 | -129 |
| Units received (Received POs) | 106,282 | 138,227 | 129,609 |
| Units delivered (Delivered SOs) | 7,904 | 5,413 | 5,977 |
| Order-line stock by hand (opening + received - delivered + adjustments) | 98,642 | 132,967 | 123,586 |
| Table order-line stock | 98,642 | 132,967 | 123,586 |

On every purchase and sale row of the 3 pairs, the balance moved by exactly the quantity written on the row (0 exceptions). All 3 pairs match the table on every line. Adjustments are marked ADJUSTMENT in movement_type (2,454 rows; PO rows are marked IN, 127,611; SO rows OUT, 107,165) with a positive quantity, so their direction is read from the change in running_balance; a first version of the spot-check that looked for adjustments marked IN/OUT found 0, and the corrected version above gives the figures in the table.

---

## 4. Data Quality Notes

1. **The ledger is an unbroken chain from opening_stock.** For all 180 pairs the first row starts from opening_stock (167 of 167 testable pairs match, 13 ADJUSTMENT-first pairs read through Step 3) and every later row moves the balance by exactly its quantity. Opening stock is small (80 to 300 per pair, 32,969 in total); the ledger builds the stock to about 107,000 units per pair almost entirely through IN movements.
2. **Adjustments.** 2,454 ADJUSTMENT rows (66,990 units): 1,227 add and 1,227 subtract, net +62. Their reference_id (ADJ-n) matches no purchase or sales order by design (1.03% of ledger rows).
3. **current_stock vs the ledger: 14 pairs differ (timing difference).** current_stock equals the ledger-based stock for 166 of 180 pairs. For the other 14, current_stock is a real ledger balance from the pair's last day, 1 or 2 movements before its final row. The rows after it add up to the gap exactly:

| Pair | Opening | Ledger-based stock | current_stock | Difference | Ledger rows after the match (movement, date, quantity, order) |
|---|---|---|---|---|---|
| P002 CHN001 | 267 | 106,182 | 105,885 | -297 | MOV-IN-114717, 2025-01-20, IN 297, PO-148144 |
| P004 PUN001 | 171 | 105,024 | 104,859 | -165 | MOV-IN-63458, 2025-01-18, IN 165, PO-541972 |
| P008 AHM001 | 259 | 109,317 | 109,281 | -36 | MOV-IN-135756, 2025-01-24, IN 36, PO-625015 |
| P010 AHM001 | 128 | 115,549 | 115,149 | -400 | MOV-IN-99729, 2025-01-24, IN 142, PO-327036; MOV-IN-135755, 2025-01-24, IN 258, PO-625015 |
| P021 DEL001 | 152 | 116,107 | 115,902 | -205 | MOV-IN-114633, 2025-01-17, IN 205, PO-167534 |
| P023 HYD001 | 82 | 105,981 | 105,798 | -183 | MOV-IN-105362, 2025-01-15, IN 183, PO-817261 |
| P025 HYD001 | 83 | 114,128 | 113,613 | -515 | MOV-IN-31858, 2025-01-13, IN 237, PO-236466; MOV-IN-42728, 2025-01-13, IN 278, PO-879075 |
| P026 HYD001 | 203 | 108,546 | 108,348 | -198 | MOV-IN-146578, 2025-01-18, IN 198, PO-912346 |
| P026 PUN001 | 112 | 105,335 | 105,190 | -145 | MOV-IN-70912, 2025-01-14, IN 145, PO-317583 |
| P028 DEL001 | 207 | 106,967 | 106,744 | -223 | MOV-IN-56490, 2025-01-23, IN 167; MOV-IN-56494, 2025-01-23, IN 56 (both PO-685650) |
| P028 PUN001 | 279 | 105,385 | 105,401 | +16 | MOV-OUT-193298, 2025-01-09, OUT 16, SO-617518 |
| P029 DEL001 | 98 | 108,702 | 108,622 | -80 | MOV-IN-115298, 2025-01-17, IN 80, PO-735274 |
| P029 KOL001 | 150 | 105,249 | 105,010 | -239 | MOV-IN-76546, 2025-01-09, IN 239, PO-724111 |
| P029 PUN001 | 115 | 109,487 | 109,429 | -58 | MOV-IN-118118, 2025-01-18, IN 58, PO-655518 |

   13 of the 14 are lower than the ledger, 1 is higher; differences run from -515 to +16 and sum to -2,728 (about 0.014% of 19,305,994); each is under 0.5% of its pair's stock. The date of the matching ledger row for each pair is: P002 CHN001 2025-01-20; P004 PUN001 01-18; P008 AHM001 01-24; P010 AHM001 01-24; P021 DEL001 01-17; P023 HYD001 01-15; P025 HYD001 01-13; P026 HYD001 01-18; P026 PUN001 01-14; P028 DEL001 01-23; P028 PUN001 01-09; P029 DEL001 01-17; P029 KOL001 01-09; P029 PUN001 01-18; the matching row is on the pair's last ledger date for all 14. 11 pairs have 1 ledger row after the match and 3 pairs (P010 AHM001, P025 HYD001, P028 DEL001) have 2: 17 rows in all (16 IN with a PO reference, 1 OUT with an SO reference), dated 9 to 24 January 2025.
   - **The 17 orders are ordinary:** all 16 purchase orders are Received, in the same branch as the ledger row, with movement_date = received_date; SO-617518 is Delivered, PUN001, movement_date = delivery_date = 2025-01-09. PO-625015 (24 Jan) feeds both P008 and P010 at AHM001; PO-685650 (23 Jan) has two lines for P028 at DEL001.
   - **Ledger quantity vs the order's total for that product:** equal in 6 rows (P008 PO-625015 36; P023 PO-817261 183; P025 PO-879075 278; P026 PUN001 PO-317583 145; P029 DEL001 PO-735274 80; P029 KOL001 PO-724111 239) and lower in 10 rows (P002 PO-148144 297 vs 378; P004 PO-541972 165 vs 359; P010 PO-327036 142 vs 189; P010 PO-625015 258 vs 466; P021 PO-167534 205 vs 306; P025 PO-236466 237 vs 328; P026 HYD001 PO-912346 198 vs 410; P028 PO-685650 167 and 56, 223 in total, vs 346; P029 PUN001 PO-655518 58 vs 130).
   - **Cause:** the data does not show why current_stock stops before these 17 movements; the gaps are explained exactly, the reason is not. The 14 pairs are logged as a timing difference.
4. **Cancelled orders sit in the ledger.** 2,817 ledger rows belong to Cancelled orders and are counted in running_balance: 1,275 OUT rows (13,234 units) and 1,542 IN rows (248,556 units). They are flagged, not removed; the stock is reported with and without them (Check 5).
5. **Order lines that never reached the ledger.** About 10% of order lines (random) are missing from the ledger: 2,237,884 units on Received purchase orders and 123,341 units on Delivered sales orders. Order lines are the complete source; ledger IN is about 18 times OUT (20,397,795 vs 1,124,832 units), a business finding and not a copy error.
6. **The gap is fully explained.** Missing received units 2,237,884 - missing delivered units 123,341 - ledger IN from Cancelled POs 248,556 + ledger OUT from Cancelled SOs 13,234 = **1,879,221**, equal to the actual gap between the two methods. Missing order lines push the ledger down by 2,114,543 (2,237,884 - 123,341); cancelled-order rows push it up by 235,322 (248,556 - 13,234); net 1,879,221.
7. **Dates.** Orders run to 2024-12-31 (Week 3); the ledger and purchase receipts run to 2025-01-28; sales deliveries end 2025-01-13. The 15-day difference is purchase receipts only (347 ledger rows, all PO / IN; Check 7). The last adjustment is dated 2024-12-29.
8. **Empty dates.** purchase_orders_header.received_date has 2,370 empties, all Cancelled orders. delivery_date has 0 empties.
9. **KOL001 observation.** 4 of the 5 smallest pairs (P019, P015, P018, P016) are at KOL001. Not investigated; no conclusion is drawn.
10. **Mixed units.** products.uom is piece for 24 products, set for 5 and cartridge for 1, so totals across products add different kinds of units; per-pair figures are the safer unit of comparison.
11. **These are records, not a physical count.** The order-line method assumes every Received purchase order was received in full and every Delivered sales order shipped in full, as the status says.

---

## 5. Business Interpretation

**Is the recorded current_stock trustworthy?** As a copy of the ledger, yes: it equals the ledger balance for 166 of 180 pairs, and for the other 14 it is a genuine ledger balance only 1 or 2 movements (about 0.014% of units) before the end. **But the ledger itself is incomplete**: it misses about 10% of the order lines and includes cancelled orders. Rebuilding from the orders gives **21,185,215 units against the ledger's 19,305,994**, and the order-line figure is higher in all 180 pairs (6,226 to 15,127 units each). So the stock figure in the system is a good copy of a ledger that understates received goods, and KPI 8 shows the size and the cause of that difference exactly.

**What the stock level says about the business.** Under either method every pair holds far more than its own limit: at least 177 times max_stock on the ledger and 193 times on the order lines, with 0 pairs at or below it. The average pair holds about 333 times the average max_stock, and 18 times more units came in than went out. This confirms the excess-stock finding of KPI 1 on a second, independent route: stock is not close to a shortage anywhere, it is piled up everywhere.

**What this means for the next KPIs.**
- KPI 6 (dead stock), KPI 9 (turnover) and KPI 10 (overstock) use the stock figure from this KPI; the headline (order-line) figure and the ledger-based figure should both be quoted until a physical count settles which is right.
- KPI 19 (reorder point) and KPI 21 (shrinkage) are built on KPI 8. The 1,879,221-unit gap between the methods is explained by missing ledger rows and cancelled-order rows, not by stock leaving the warehouse; it should **not** be called shrinkage unless the data supports it.
- A small number of pairs (the 14) show a timing difference, not a stock problem; they do not need action.

**Recommended actions for the business owners**
1. **Inventory planning team:** use current_stock only as a copy of the ledger; for stock decisions, use the order-line stock and show the ledger figure beside it.
2. **Data / system owner (the team that maintains the stock ledger):** fix how order lines are posted to the ledger (about 10% never arrive), and review why cancelled orders still move the balance.
3. **Warehouse manager:** run a physical stock count on a few pairs (for example P019 KOL001, P002 HYD001 and P025 HYD001, already rebuilt by hand here) to confirm which method is closer to the shelf.
4. **Purchasing manager with inventory planning:** review max_stock and purchase quantities together; with stock hundreds of times above max_stock in every pair, either max_stock is far too low or purchasing is far too high.
