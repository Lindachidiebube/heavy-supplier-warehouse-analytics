# KPI 21 Stock Shrinkage Rate

Project: Heavy Supplier & Warehouse Analytics | Week 10 | Notebook: Week10 KPI 21 Stock Shrinkage Rate

## Summary

This KPI asks how many units the warehouse writes off, as a share of the units it handles. The data has no physical stock count (by inspection of the table list in KPI 8), so the KPI measures recorded loss only: units taken out of the stock book by adjustment rows that lower the balance.

- Adjustment rows: 2,454. Added: 1,227 rows, 33,526 units. Subtracted: 1,227 rows, 33,464 units. Net: +62 units.
- Units handled (purchases plus sales): 21,522,627.
- Shrinkage rate: 0.1555% (33,464 / 21,522,627).
- Row share: 1,227 of 234,776 purchase and sales rows, 0.5226% (by hand).
- Adjustments put back 62 more units than they removed.
- No difference was found beyond chance between branches (p 0.5024), products (p 0.2496) or product-and-branch pairs (p 0.778), or between years 2019 to 2024 (p 0.8518).
- No adjustment of either kind is recorded after 2024-12-29, while the ledger runs to 2025-01-28.
- Records gap, kept separate and not called shrinkage: 1,879,221 units net. The missing purchase units are about 67 times the subtracted units (by hand: 2,237,884 / 33,464).

## 1 KPI Definition

- Shrinkage rate = units subtracted by adjustments divided by units handled (purchase units plus sales units in the ledger), times 100.
- Net adjustment = units added minus units subtracted, shown beside the rate.
- Records gap = units on Received purchase lines and Delivered sales lines that are not in the ledger, together with the units on ledger rows of cancelled orders. It is shown separately from the rate.
- A physical loss cannot be measured, because none of the twelve tables listed in KPI 8 records a physical stock count (by inspection of that list).

## 2 Methodology

### Sources

| Table | Rows | Columns used |
|---|---|---|
| stock_ledger | 237,230 | product_id, branch_id, movement_type, movement_date, quantity, reference_type, reference_id, running_balance |
| inventory_master | 180 | product_id, branch_id, opening_stock |
| purchase_orders_header | 24,000 | po_id, po_status |
| purchase_orders_lines | 155,495 | po_id, product_id, quantity |
| sales_orders_header | 20,000 | so_id, order_status |
| sales_orders_lines | 130,402 | so_id, product_id, quantity |

### What is reused and what is new

| Reused from KPI 8 | New in KPI 21 |
|---|---|
| Reading the direction of each adjustment from the running balance; the records gap (every number re-run in this notebook) | The shrinkage rate; where the subtractions sit (branch, year, product, pair); adjustments by year and by month; the loss rate of each of the 180 pairs |

### Steps

- Load the six tables and count the adjustment rows and their units (2,454 rows, 66,990 units).
- Read the direction of every adjustment: for each product and branch pair, rows are taken in date and file order, starting from opening stock. An adjustment is added when the balance rose by its quantity and subtracted when the balance fell by its quantity.
- Work out units handled, the shrinkage rate, the net adjustment and the same figures by branch.
- Test where the subtractions sit: subtraction rows against purchase and sales rows by branch, year, product and pair (chi-square, strict line 0.05).
- Show adjustments by year, the last adjustment date and the year test for 2019 to 2024.
- Re-run the records gap from the order lines and the ledger.
- Run the 8 verification checks.

### Choices and alternatives

- Rate base: units handled (purchase units plus sales units), the stock that moved through the warehouse. The stock held at the end of the ledger (a balance at one moment) was not used as the base.
- The tests on where subtractions sit count rows, not units.
- The year test is shown with 2025 and for 2019 to 2024 only, because 2025 has no adjustment rows.
- The records gap is not called shrinkage and is not added to the rate.

## 3 Verification Log

### Summary

| Check | What it checked | Result in plain words |
|---|---|---|
| Check 1 | Blanks, zeros and negatives in every column used | 0 problems |
| Check 2 | Order statuses, ledger rows with no status, pairs with adjustments, pairs missing from the inventory master | Expected statuses only; all 180 pairs covered; 0 missing pairs |
| Check 3 | Repeated ids, repeated pairs, copies among adjustment rows | 0 repeats and 0 copies |
| Check 4 | Totals add up from different angles | All totals agree |
| Check 5 | Second method with plain loops | Same numbers as the table method |
| Check 6 | Outliers in adjustment size and in the loss rate of each pair | No pair outside the normal range; no pair difference found beyond chance |
| Check 7 | Dates and history | Subtractions in every one of 72 months |
| Check 8 | Three items rebuilt by hand from the raw files | All three match the table |

### Check 1: blanks, zeros and negatives

- Stock ledger, 237,230 rows: product_id, branch_id, movement_type, movement_date, quantity, reference_type, reference_id and running_balance have 0 blanks. quantity and running_balance have 0 zeros and 0 negatives.
- Inventory master, 180 rows: product_id, branch_id and opening_stock have 0 blanks; opening_stock has 0 zeros and 0 negatives.
- Purchase orders header (24,000 rows): po_id and po_status have 0 blanks. Sales orders header (20,000 rows): so_id and order_status have 0 blanks.
- Purchase order lines (155,495 rows) and sales order lines (130,402 rows): ids, product_id and quantity have 0 blanks; quantity has 0 zeros and 0 negatives.
- Result: 0 blanks, zeros or negatives in every column used.

### Check 2: edge cases

- Purchase orders are only Received (21,630) or Cancelled (2,370), sales orders are only Delivered (18,033) or Cancelled (1,967), and 0 ledger order rows lack a status (purchase ledger rows with no status 0, sales ledger rows with no status 0).
- All 180 pairs have adjustments in both directions (pairs with at least one adjustment 180 of 180; pairs with one direction only 0; fewest 5 rows in a pair, most 25), and 0 ledger pairs are missing from the inventory master.

### Check 3: duplicates

- No movement id, order id or inventory pair is repeated, and 0 adjustment rows are copies of another. Ledger rows 237,230 with 237,230 distinct movement ids; purchase orders 24,000 with 24,000 distinct ids; sales orders 20,000 with 20,000 distinct ids; repeated inventory pairs 0; adjustment rows identical to another adjustment row 0.

### Check 4: totals reconcile

- Units in 20,397,795; units out 1,124,832; units handled 21,522,627.
- Added plus subtracted equals all adjustment units (66,990), and the final stock total is the same by two routes (19,305,994: last balances added up, and opening stock 32,969 plus in minus out plus the net of 62).
- The branch tables add up to the overall totals (handled 21,522,627, subtracted 33,464, added 33,526).
- Records gap 1,879,221 (missing purchase units 2,237,884, minus missing sales units 123,341, minus ledger units on cancelled purchase orders 248,556, plus ledger units on cancelled sales orders 13,234).

### Check 5: second method

- A plain-loop method, going through the ledger row by row and pair by pair in date order from opening stock, gave added 33,526, subtracted 33,464, units handled 21,522,627, 0 unreadable adjustments and a rate of 0.1555%. It gives the same added, subtracted and handled units, the same rate and the same units by branch as the table method.

### Check 6: outliers

- Adjustment size by direction (count / mean / std / min / 25% / median / 75% / max): added 1,227 / 27.32 / 13.25 / 5 / 16 / 27 / 38.5 / 50; subtracted 1,227 / 27.27 / 13.25 / 5 / 16 / 27 / 38 / 50. The largest adjustment (50) is 1.83 times the subtracted average (by hand).
- Loss rate per product and branch pair (180 pairs): average 0.1556%, std 0.0635, minimum 0.0335%, 25% 0.1064, median 0.1482, 75% 0.2033, maximum 0.3092%. Normal range by the 1.5 x IQR rule: -0.039% to 0.3486%; pairs above the range 0; pairs below 0.
- No pair lies outside the normal range (upper limit 0.3486%).

Five highest pairs:

| Pair | Subtracted units | Units handled | Rate % |
|---|---|---|---|
| P013 KOL001 | 370 | 119,661 | 0.3092 |
| P026 AHM001 | 359 | 116,712 | 0.3076 |
| P025 CHN001 | 386 | 125,550 | 0.3074 |
| P028 CHN001 | 356 | 118,135 | 0.3014 |
| P013 PUN001 | 339 | 114,830 | 0.2952 |

Five lowest pairs:

| Pair | Subtracted units | Units handled | Rate % |
|---|---|---|---|
| P022 AHM001 | 39 | 116,417 | 0.0335 |
| P012 AHM001 | 58 | 120,819 | 0.0480 |
| P006 AHM001 | 61 | 119,167 | 0.0512 |
| P024 KOL001 | 65 | 126,193 | 0.0515 |
| P008 DEL001 | 61 | 118,162 | 0.0516 |

- The highest pair rate is about 9.2 times the lowest (by hand: 0.3092 / 0.0335).
- Pair test: fewest subtraction rows in a pair 2, most 14; pairs expected to have fewer than 5 subtraction rows 0 (the test is usable). No pair-level difference was found beyond chance (p 0.778).

### Check 7: dates and history

- First and last ledger date: 2019-01-01 and 2025-01-28. First and last adjustment date: 2019-01-01 and 2024-12-29.
- Months from the first to the last adjustment: 72. Months with no subtraction row: 0. Fewest subtraction rows in a month 8, most 28 (about 17.0 a month by hand: 1,227 / 72).
- Subtractions are recorded in every one of the 72 months from 2019-01 to 2024-12 (8 to 28 rows a month).

### Check 8: three items by hand from the raw files

| Item | By hand from the raw csv files | Table |
|---|---|---|
| MOV-ADJ-1519, P001 AHM001, quantity 38 | Balance before 4,245, after 4,207: subtracted | Subtracted; agree |
| MOV-ADJ-524, P001 AHM001, quantity 33 | Balance before 13,946, after 13,979: added | Added; agree |
| Pair P019 KOL001 end to end | Opening 92, in 95,724, out 7,135, added 253, subtracted 81, net +172; 92 + 95,724 - 7,135 + 172 = 88,853 = last balance 88,853 | Added 253, subtracted 81, last balance 88,853; agree |

- In all three items the by-hand result from the raw files equals the table.

### Supporting tests

**Direction of every adjustment.** Purchase rows 127,611, balance rose by the quantity for 127,611. Sales rows 107,165, balance fell by the quantity for 107,165. Adjustments added 1,227 rows (33,526 units); subtracted 1,227 rows (33,464 units); none unreadable.

**Rate by branch.**

| Branch | Units handled | Subtracted | Added | Net | Rate % |
|---|---|---|---|---|---|
| AHM001 | 3,539,666 | 5,190 | 4,992 | -198 | 0.1466 |
| CHN001 | 3,692,804 | 5,980 | 5,334 | -646 | 0.1619 |
| DEL001 | 3,622,464 | 5,431 | 5,303 | -128 | 0.1499 |
| HYD001 | 3,548,234 | 5,730 | 5,920 | +190 | 0.1615 |
| KOL001 | 3,538,563 | 6,200 | 5,632 | -568 | 0.1752 |
| PUN001 | 3,580,896 | 4,933 | 6,345 | +1,412 | 0.1378 |

- The net adjustment is negative in four branches (AHM001, CHN001, DEL001, KOL001) and positive in two (HYD001, PUN001) (by hand, from the table).

**Where the subtractions sit (rows, chi-square, strict line 0.05).**

| Branch | Handled rows | Subtraction rows | Share % |
|---|---|---|---|
| AHM001 | 39,525 | 196 | 0.4959 |
| CHN001 | 43,600 | 226 | 0.5183 |
| DEL001 | 37,575 | 202 | 0.5376 |
| HYD001 | 35,877 | 207 | 0.5770 |
| KOL001 | 40,848 | 218 | 0.5337 |
| PUN001 | 37,351 | 178 | 0.4766 |

| Year | Handled rows | Subtraction rows | Share % |
|---|---|---|---|
| 2019 | 37,989 | 197 | 0.5186 |
| 2020 | 39,857 | 198 | 0.4968 |
| 2021 | 39,294 | 207 | 0.5268 |
| 2022 | 38,571 | 219 | 0.5678 |
| 2023 | 39,352 | 207 | 0.5260 |
| 2024 | 38,247 | 199 | 0.5203 |
| 2025 | 1,466 | 0 | 0.0000 |

- By product the share runs from 0.3698% (P018) to 0.7133% (P015).
- No difference was found between branches (p 0.5024).
- No difference was found between products (p 0.2496).
- No difference was found between years when 2025 is included (p 0.1377), and none for 2019 to 2024 (p 0.8518).
- Handled rows add to 234,776 in each table and subtraction rows to 1,227 (0.5226% overall, by hand).

**Adjustments by year and direction.**

| Year | Added rows | Subtracted rows |
|---|---|---|
| 2019 | 187 | 197 |
| 2020 | 193 | 198 |
| 2021 | 213 | 207 |
| 2022 | 221 | 219 |
| 2023 | 214 | 207 |
| 2024 | 199 | 199 |
| 2025 | 0 | 0 |

**Records gap.**

| Item | Units |
|---|---|
| Units on Received purchase lines | 22,387,123 |
| Ledger units from Received purchase orders | 20,149,239 |
| Purchase units missing from the ledger | 2,237,884 (10.0%) |
| Units on Delivered sales lines | 1,234,939 |
| Ledger units from Delivered sales orders | 1,111,598 |
| Sales units missing from the ledger | 123,341 (9.99%) |
| Ledger units on cancelled purchase orders | 248,556 |
| Ledger units on cancelled sales orders | 13,234 |
| Records gap (missing purchases, minus missing sales, minus cancelled purchases, plus cancelled sales) | 1,879,221 |

## 4 Data Quality Notes

1. The last adjustment row is dated 2024-12-29, and no adjustment of either kind exists in 2025 although 1,466 purchase and sales rows are dated 2025. Why recording stops is not shown in the data.
2. ADJUSTMENT quantities are always positive and the direction is read from the running balance; all 2,454 adjustments are readable (0 unreadable).
3. The records gap is made of 2,237,884 purchase units and 123,341 sales units on order lines that are not in the ledger, together with 248,556 purchase units and 13,234 sales units on cancelled-order ledger rows. Why these units are missing or present is not shown in the data.
4. The unit rate (0.1555%) is lower than the row share (0.5226%) because an average subtraction row carries 27.3 units against 91.7 for an average purchase or sales row (by hand: 33,464 / 1,227 and 21,522,627 / 234,776).
5. The normal range of the pair outlier rule has a negative lower limit (-0.039%), so the rule can only flag pairs that are too high (by hand, a rate cannot be negative).
6. January 2025 holds 1,466 purchase and sales rows and no adjustments; the year test is therefore shown with and without 2025.

## 5 Business Interpretation

- Over the whole ledger, adjustments put back 62 more units than they removed. The recorded loss is 33,464 units out of 21,522,627 handled, 0.1555 units for every 100 handled.
- Branch rates run from 0.1378% (PUN001) to 0.1752% (KOL001). No branch, product, year (2019 to 2024) or product-and-branch pair stands out beyond chance.
- The records gap of 1,879,221 units is separate from the rate. The missing purchase units are about 67 times the subtracted units (by hand: 2,237,884 / 33,464).
- The data cannot show whether units were physically lost or only not recorded, because no table holds a physical count (by inspection of the table list in KPI 8).
- KPI 22 (Operational Risk Score) takes the recorded loss rate and the records gap from this KPI as two separate inputs.

## Recommended actions for the business owners

1. Inventory control manager: count a sample of products at every branch against the stock book and give the differences to the data owner.
2. Data and system owner: find out why adjustments stop after 2024-12-29 and why units on order lines are missing from the ledger.
3. Warehouse managers: record a reason (loss, damage or correction) on every adjustment from now on.
4. Inventory planning team with finance: report the recorded loss rate and the records gap as two separate figures each quarter.

## Glossary

| Name | Meaning |
|---|---|
| ledger, inv, po_h, po_l, so_h, so_l | The six loaded tables: stock ledger, inventory master, purchase orders header and lines, sales orders header and lines |
| adj | Ledger rows of type ADJUSTMENT |
| led | The ledger sorted by product, branch, date and file order, with opening stock, previous balance and balance change |
| in_rows, out_rows, adj_rows | Purchase rows, sales rows and adjustment rows of led; adj_rows has the column direction (added or subtracted) |
| handled, added_units, subtracted_units | Units handled (purchase plus sales), units added and units subtracted by adjustments |
| by_branch | Table of units handled, subtracted, added, net and rate by branch |
| handled_rows, subtracted_rows | Purchase and sales rows; subtraction rows (with a year column) |
| by_branch_rows, by_year_rows, by_product_rows | Row counts and shares by branch, year and product |
| purchase_ledger, sales_ledger | Ledger rows tied to purchase orders and to sales orders, with order status |
| purchase_lines_status, sales_lines_status | Order lines with order status |
| units_on_received_lines, units_on_delivered_lines | Units on Received purchase lines and Delivered sales lines |
| ledger_units_received, ledger_units_delivered | Ledger units from Received purchase orders and Delivered sales orders |
| ledger_units_cancelled_in, ledger_units_cancelled_out | Ledger units on cancelled purchase orders and cancelled sales orders |
| records_gap | The records gap (1,879,221 units) |
| tables_used | The tables and columns tested in Check 1 |
| adjustments_per_pair, directions_per_pair | Adjustment rows per pair and number of directions per pair (Check 2) |
| opening_by_pair, rows_by_pair, loop_added, loop_subtracted, loop_handled, loop_unreadable, loop_subtracted_by_branch | Objects of the plain-loop method (Check 5) |
| pair_losses | Subtracted units, handled units and rate of each of the 180 pairs (Check 6) |
| pair_rows | Handled rows and subtraction rows of each pair (pair test in Check 6); the name is used again for a list of raw rows in the by-hand cell of Check 8 |
| subtraction_rows_per_month | Subtraction rows in each month (Check 7) |
| raw_ledger, raw_inventory | The raw csv files read with the csv module (Check 8) |
