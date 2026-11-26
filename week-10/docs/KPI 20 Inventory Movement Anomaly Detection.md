# KPI 20 Inventory Movement Anomaly Detection

Project: Heavy Supplier & Warehouse Analytics | Week 10 | Notebook: Week10 KPI 20 Inventory Movement Anomaly Detection

## Summary

KPI 20 finds the rows of the stock ledger that are unusual enough for someone to look at. The ledger has 237,230 rows: 127,611 IN, 107,165 OUT and 2,454 ADJUSTMENT.

- Size cannot make a row unusual in this ledger. Within each movement type every quantity falls in a fixed range, no quantity is more common than chance gives, and the largest movement is at most 1.91 times the average of its type (see Check 6).
- The flag rule therefore looks at whether a row agrees with the order records.
- Group A, ledger rows tied to cancelled orders: 2,817 rows (1,542 IN and 1,275 OUT), 261,790 units.
- Group B, ledger rows where the quantity recorded for an order and product is lower than the order lines: 5,521 rows in 4,947 order and product pairs.
- In total 8,338 rows are flagged, which is 3.51% of the ledger, and they belong to 6,595 orders (1,993 cancelled orders and 4,602 orders with a lower quantity). No row is in both groups.
- Cancelled orders reach the ledger less often than ordinary orders: 46.33% of cancelled purchase orders and 45.5% of cancelled sales orders, against 98.9% of received purchase orders and 99.1% of delivered sales orders.
- The ledger never holds more than the order lines ask for, and every ledger row has an order behind it.
- The flagged share does not differ by more than chance between branches, between products or between purchase rows and sales rows.
- The flagged share is steady between 3.50% and 3.65% in each year from 2020 to 2024, and lower in 2019 (3.24%) and in the partial year 2025 (2.59%). No cause for the lower shares was tested.
- All 8 verification checks were completed (see Verification Log).

## 1 KPI Definition

| Item | Definition |
|---|---|
| Flagged row | A ledger row that is tied to a cancelled order (Group A) or whose quantity for its order and product is lower than the quantity on the order lines (Group B) |
| Flagged rate | Flagged rows divided by all ledger rows: 8,338 / 237,230 = 3.51% |
| Rows looked at | All 237,230 ledger rows; ADJUSTMENT rows are looked at but not flagged (see Methodology) |
| Not part of this KPI | Order and product pairs that never reached the ledger; they are not ledger rows and belong to KPI 21 |

Why this metric: a cancelled order should not move stock, and a received or delivered order should leave its full quantity in the ledger. A row that breaks either rule is worth a look.

## 2 Methodology

### Sources

| Table | Rows | Used for |
|---|---|---|
| stock_ledger | 237,230 | The rows that are flagged (9 columns) |
| purchase_orders_header | 24,000 | Status, branch and received date of each purchase order |
| purchase_orders_lines | 155,495 | Quantity ordered per order and product |
| sales_orders_header | 20,000 | Status, branch and delivery date of each sales order |
| sales_orders_lines | 130,402 | Quantity ordered per order and product |
| inventory_master (180 rows) and products (30 rows) | 180 and 30 | Loaded; not used in the flag rule |

### Reused from earlier KPIs

| From | What was reused |
|---|---|
| KPI 8 | Every ledger row follows from the one before it, and every adjustment has a proven direction; ledger rows by movement type and order status (IN Cancelled 1,542 rows, OUT Cancelled 1,275 rows); units received 22,387,123 and delivered 1,234,939 on the order lines; sales deliveries end on 2025-01-13 and only purchase receipts continue to 2025-01-28 |
| KPI 18 | Order and product combinations on Delivered sales lines (104,418) |

### Steps

- Loaded the ledger and the four order tables and checked their sizes.
- Looked at the quantity of each movement type: spread, range and how evenly the quantities fall inside the range.
- Linked every ledger row to the status of its order.
- Counted how many cancelled orders and how many ordinary orders appear in the ledger.
- Compared the ledger quantity with the order-line quantity for every order and product on received purchase orders and delivered sales orders.
- Checked that each ledger row agrees with its order on date and branch.
- Looked for repeated movement ids and for identical rows.
- Marked Group A and Group B on every ledger row and counted them.
- Ran the 8 verification checks.

### Choices and alternatives

- Flagging by size was rejected. Quantities are spread evenly inside fixed ranges, so a bigger movement is not more unusual than a smaller one (see Check 6).
- ADJUSTMENT rows are not flagged. Each one has a proven direction (KPI 8) and no order to compare with.
- Pairs missing from the ledger are not flagged: they are not ledger rows, and KPI 21 measures them.
- Lines for the same product on one order are added together before comparing, because one order can have two lines for the same product (see Check 3).
- The rule has no threshold: a row either agrees with its order or it does not, so there is nothing to tune.

## 3 Verification Log

### Summary of the 8 checks

| # | Check | Result |
|---|---|---|
| 1 | Blank, zero and negative values in every column used | No problem in any column used; the only blank dates are the received dates of the 2,370 cancelled purchase orders |
| 2 | Edge cases: orders and ledger rows that do not line up | Every ledger row has an order; the ledger is never above the order lines; cancelled orders appear in the ledger about 46% of the time and ordinary orders about 99% |
| 3 | Duplicates | No movement id is repeated; all 606 groups of identical rows come from order lines that repeat |
| 4 | Totals and units reconcile | Rows and units match KPI 8 and the order lines |
| 5 | Second method | Two independent methods give the same 8,338 flagged rows |
| 6 | Outliers: size and where the flags sit | No size outlier; no branch, product or movement type stands out; 2019 and 2025 have lower shares |
| 7 | Dates and history | 73 months, none missing; flagged rows in every month |
| 8 | Hand check of 3 items from the raw files | All three match the table |

### Check 1: blank, zero and negative values

| Table | Rows | Result |
|---|---|---|
| Stock ledger | 237,230 | product_id, branch_id, movement_type, movement_date, quantity, reference_type and reference_id have 0 blanks; quantity has 0 zeros and 0 negatives |
| Purchase orders header | 24,000 | po_id, branch_id and po_status have 0 blanks; received_date has 2,370 blanks |
| Sales orders header | 20,000 | so_id, branch_id, order_status and delivery_date have 0 blanks |
| Purchase orders lines | 155,495 | po_id, product_id and quantity have 0 blanks; quantity has 0 zeros and 0 negatives |
| Sales orders lines | 130,402 | so_id, product_id and quantity have 0 blanks; quantity has 0 zeros and 0 negatives |

Received purchase orders have 0 blank received dates and cancelled purchase orders have 2,370, so the only blank dates are those of the cancelled purchase orders, the same 2,370 found in KPI 8.

### Check 2: edge cases

- Purchase orders header: 24,000 rows and 24,000 distinct ids. Sales orders header: 20,000 rows and 20,000 distinct ids. Ledger rows with no order status: 0.

Ledger rows and units by movement type and order status:

| Movement type | Order status | Rows | Units |
|---|---|---|---|
| ADJUSTMENT | ADJ (no order) | 2,454 | 66,990 |
| IN | Cancelled | 1,542 | 248,556 |
| IN | Received | 126,069 | 20,149,239 |
| OUT | Cancelled | 1,275 | 13,234 |
| OUT | Delivered | 105,890 | 1,111,598 |

By hand, the rows tied to cancelled orders are 1,542 + 1,275 = 2,817, which is 1.19% of the ledger, and their units are 261,790. All counts equal KPI 8.

Orders that appear in the ledger:

| Order group | Orders | In the ledger | Share |
|---|---|---|---|
| Received purchase orders | 21,630 | 21,391 | 98.9% |
| Cancelled purchase orders | 2,370 | 1,098 | 46.33% |
| Delivered sales orders | 18,033 | 17,871 | 99.1% |
| Cancelled sales orders | 1,967 | 895 | 45.5% |

By hand, 239 received purchase orders and 162 delivered sales orders have no ledger row at all.

Ledger rows per cancelled order (rows: orders): purchase orders 1: 742, 2: 282, 3: 61, 4: 12, 5: 1 (1,098 orders, 1,542 rows); sales orders 1: 591, 2: 238, 3: 58, 4: 6, 5: 2 (895 orders, 1,275 rows). Cancelled IN rows run from 2019-01-14 to 2025-01-22 and cancelled OUT rows from 2019-01-05 to 2025-01-07.

Ledger against order lines for every order and product (repeated lines for one product on one order added together):

| Side | Pairs on the order lines | Ledger equals the lines | Ledger lower | Ledger higher | Missing from the ledger | In the ledger with no order line |
|---|---|---|---|---|---|---|
| Received purchase orders | 124,417 | 110,594 | 2,685 | 0 | 11,138 | 0 |
| Delivered sales orders | 104,418 | 92,792 | 2,262 | 0 | 9,364 | 0 |

By hand, both sides add up. The ledger is never higher than the order lines and never holds a row with no order line. The 104,418 sales pairs equal the order and product combinations found in KPI 18.

### Check 3: duplicates

- The ledger has 237,230 rows and 237,230 distinct movement ids; 0 rows have an id that appears more than once.
- 1,214 rows are identical to another row in product, branch, movement type, date, quantity and order, apart from movement id and balance. They form 606 groups: OUT 1,114 rows, IN 100 rows, ADJUSTMENT 0. By hand, that is 608 extra copies and about 2 rows per group.
- All 606 groups match an order with two or more lines for the same product and quantity. No group has only one such line, none has no matching line, and none has more ledger rows than order lines. These identical rows come from order lines that repeat; they are not repeated ledger rows.

### Check 4: totals and units

- By hand, IN + OUT + ADJUSTMENT = 127,611 + 107,165 + 2,454 = 237,230 rows.
- The rows and units by movement type and order status equal the KPI 8 figures (see Check 2).
- Units on the order lines against the ledger:

| Side | Units on the order lines | Units in the ledger | Difference | On pairs where the ledger is lower | On pairs missing from the ledger |
|---|---|---|---|---|---|
| Received purchase orders | 22,387,123 | 20,149,239 | 2,237,884 | 435,404 | 1,802,480 |
| Delivered sales orders | 1,234,939 | 1,111,598 | 123,341 | 24,216 | 99,125 |

The two parts add up to each difference. By hand, the units on lower pairs are 19.46% of the purchase difference and 19.63% of the sales difference, so about a fifth of the missing units sit on pairs where the ledger holds something and the rest sit on pairs that are missing.

- Units on rows tied to cancelled orders: 261,790 (equals 248,556 + 13,234 by hand). Ledger units on rows with a quantity lower than the order lines: 508,520. By hand, those order lines asked for 508,520 + 459,620 = 968,140 units, where 459,620 = 435,404 + 24,216 is the shortfall.
- By hand, Group B has 5,521 - 4,947 = 574 more rows than pairs, which means some pairs have more than one ledger row.

### Check 5: second method

The flags were rebuilt with plain loops and lookups, without merging tables. The second method finds 2,817 cancelled-order rows, 5,521 lower rows and 8,338 flagged rows; the first method finds 8,338 flagged rows. Rows flagged by the first method only: 0. Rows flagged by the second method only: 0.

### Check 6: outliers

Size. Quantity of each movement type:

| Movement type | Rows | Mean | Std | Min | 25% | Median | 75% | Max | Largest over mean |
|---|---|---|---|---|---|---|---|---|---|
| ADJUSTMENT | 2,454 | 27.30 | 13.25 | 5 | 16 | 27 | 38 | 50 | 1.83 |
| IN | 127,611 | 159.84 | 81.14 | 20 | 90 | 160 | 230 | 300 | 1.88 |
| OUT | 107,165 | 10.50 | 5.78 | 1 | 5 | 11 | 16 | 20 | 1.91 |

Evenness of the quantities (chi-square against an equal share, strict line 0.05):

| Movement type | Quantity values | Equal share | Smallest share | Largest share | p-value |
|---|---|---|---|---|---|
| IN | 281 (20 to 300) | 0.356% | 0.319% at 226 | 0.397% at 163 | 0.9377 |
| OUT | 20 (1 to 20) | 5.0% | 4.881% at 13 | 5.135% at 12 | 0.2592 |
| ADJUSTMENT | 46 (5 to 50) | 2.174% | 1.467% at 21 | 2.689% at 12 | 0.8916 |

By hand, 300 - 20 + 1 = 281 and 50 - 5 + 1 = 46, so every quantity inside each range appears. All p-values are above 0.05, so no quantity is more common than chance gives. A movement is therefore not unusual because of its size.

Where the flags sit (share of rows flagged; chi-square of flagged against place, strict line 0.05):

| Branch | Rows | Flagged | Share |
|---|---|---|---|
| AHM001 | 39,911 | 1,366 | 3.42% |
| CHN001 | 44,029 | 1,482 | 3.37% |
| DEL001 | 37,967 | 1,351 | 3.56% |
| HYD001 | 36,300 | 1,329 | 3.66% |
| KOL001 | 41,272 | 1,429 | 3.46% |
| PUN001 | 37,751 | 1,381 | 3.66% |

Branch p-value 0.1089. By product, the lowest share is P023 (7,873 rows, 247 flagged, 3.14%) and the highest is P029 (7,904 rows, 332 flagged, 4.20%); product p-value 0.1897. Both p-values are above 0.05, so no branch and no product has a flagged share that differs by more than chance.

| Year | Rows | Flagged | Share |
|---|---|---|---|
| 2019 | 38,373 | 1,245 | 3.24% |
| 2020 | 40,248 | 1,466 | 3.64% |
| 2021 | 39,714 | 1,390 | 3.50% |
| 2022 | 39,011 | 1,381 | 3.54% |
| 2023 | 39,773 | 1,406 | 3.54% |
| 2024 | 38,645 | 1,412 | 3.65% |
| 2025 | 1,466 | 38 | 2.59% |

The year p-value is 0.0126, below 0.05. The test was repeated three ways:

| Years included | Lowest yearly share | Highest yearly share | p-value |
|---|---|---|---|
| All years | 2.59% in 2025 | 3.65% in 2024 | 0.0126 |
| Without 2025 | 3.24% in 2019 | 3.65% in 2024 | 0.0285 |
| 2020 to 2024 only | 3.50% in 2021 | 3.65% in 2024 | 0.6997 |

The yearly shares are steady from 2020 to 2024 and lower in 2019 and in the partial year 2025. No cause was tested for the lower shares.

Purchase rows and sales rows (chi-square, strict line 0.05): IN 127,611 rows, 4,535 flagged, 3.55%; OUT 107,165 rows, 3,803 flagged, 3.55%; p-value 0.9566. Purchase and sales rows do not differ by more than chance.

Orders with at least one flagged row: 1,993 with a row tied to a cancelled order (equals 1,098 + 895 by hand), 4,602 with a row lower than the order lines, and 6,595 in total (equals 1,993 + 4,602 by hand).

### Check 7: dates and history

- First and last ledger date: 2019-01-01 and 2025-01-28.
- Months in the ledger: 73, and every month from 2019-01 to 2025-01 is present.
- Fewest rows in a month: 1,466 in 2025-01. Most rows in a month: 3,846 in 2020-08.
- Months with no flagged row: 0. Fewest flagged rows in a month: 38 in 2025-01. Most: 144 in 2020-08 (3.74% of that month's rows by hand).
- First and last date of flagged rows: 2019-01-04 and 2025-01-28.
- January 2025 is a partial month: sales deliveries end on 2025-01-13 and only purchase receipts continue to 2025-01-28 (KPI 8, 347 purchase rows after 13 January and none for sales or adjustments).

### Check 8: hand check of 3 items from the raw files

Each item was rebuilt from the raw csv files with plain loops, without the tables used in the notebook.

| Item | Order and product | Raw ledger rows | Ledger units | Order status | Units on the order lines | By hand | In the table |
|---|---|---|---|---|---|---|---|
| Cancelled order row | PO-171706, P028 | 1 | 89 | Cancelled | 89 | Flagged: cancelled order | Flagged: cancelled order |
| Lower quantity row | SO-508292, P008 | 1 | 14 | Delivered | 18 | Flagged: ledger lower than the order lines | Flagged: ledger lower than the order lines |
| Ordinary row | PO-141186, P004 | 1 | 234 | Received | 234 | Not flagged | Not flagged |

The by-hand answer from the raw files equals the flag in the table for all three items.

### Supporting tests

| Test | Question | Result | Where |
|---|---|---|---|
| Quantity summary and evenness | Can size alone mark a row as unusual? | No: fixed ranges, even spread, largest at most 1.91 times the average | Check 6 |
| Orders in the ledger by status | Do cancelled orders reach the ledger as often as ordinary orders? | No: about 46% against about 99% | Check 2 |
| Ledger against order lines | Does the ledger hold the quantity the order lines ask for? | 88.89% of purchase pairs and 88.87% of sales pairs match (by hand); never higher | Check 2 |
| Row against order header | Do ledger rows agree with their order on date and branch? | Yes: 126,069 purchase rows and 105,890 sales rows, 0 differences in date and 0 in branch | Check 2 |
| Identical rows | Are identical rows repeated entries? | No: all 606 groups come from repeated order lines | Check 3 |
| Flag list | How many rows and units are flagged? | 8,338 rows; 261,790 units in Group A; 508,520 ledger units in Group B | Check 4 |
| Where the flags sit | Does the flagged share differ by branch, product or movement type? | No difference found in any of the three | Check 6 |
| Year test three ways | Is the yearly difference caused by one year? | Steady from 2020 to 2024; 2019 and 2025 lower; cause not tested | Check 6 |

## 4 Data Quality Notes

1. The only blank dates in the tables used are the received dates of the 2,370 cancelled purchase orders.
2. Part of the cancelled orders appear in the ledger (1,098 of 2,370 purchase orders and 895 of 1,967 sales orders). The data does not show why.
3. The ledger holds less than the order lines on 2,685 purchase pairs and 2,262 sales pairs, and nothing at all for 11,138 purchase pairs and 9,364 sales pairs. The data does not show why.
4. Identical ledger rows exist (1,214 rows in 606 groups); they come from orders with repeated lines and are not duplicates.
5. January 2025 is a partial month: sales end on 2025-01-13 and purchases on 2025-01-28.
6. The lower flagged shares in 2019 and in 2025 were not explained.
7. The ledger and the order tables are records, not a physical stock count. A flagged row is a row to review, not proof of a mistake.

## 5 Business Interpretation

- About 3.5 of every 100 ledger rows are flagged: 8,338 of 237,230, in 6,595 orders.
- Group A: the ledger records stock for orders that the order tables show as cancelled, which is 2,817 rows and 261,790 units in 1,993 orders.
- Only part of the cancelled orders appear in the ledger (1,098 of 2,370 purchase orders and 895 of 1,967 sales orders). The data does not show why.
- Group B shows the ledger holding less than the order lines asked for: 459,620 units short in total (435,404 on purchases and 24,216 on sales, by hand), in 4,602 orders.
- Flagged rows are not concentrated in one branch, one product or one movement type, so the review is not a task for one place.
- A large movement is no more suspicious than a small one, because quantities are spread evenly inside fixed ranges.
- Units on pairs that are missing from the ledger (1,802,480 on purchases and 99,125 on sales) are not flagged here and are measured in KPI 21.
- The reasons for the cancelled-order rows, for the lower quantities and for the lower shares in 2019 and 2025 were not tested.

## Recommended actions for the business owners

1. The inventory control team reviews the 2,817 ledger rows tied to cancelled orders (1,993 orders) and decides whether the stock they moved should stand.
2. The purchasing manager and the warehouse manager review the 5,521 rows where the ledger holds less than the order lines (4,602 orders) and confirm what was really received or delivered.
3. The data or system owner finds out why part of the cancelled orders appear in the ledger and why some received and delivered orders are recorded with less than the order lines.
4. The inventory planning team uses the flag list each month and watches the flagged rate, which was 3.50% to 3.65% a year from 2020 to 2024.

## Glossary

| Label | Meaning |
|---|---|
| Group A | Ledger rows tied to cancelled orders |
| Group B | Ledger rows whose quantity for an order and product is lower than the order lines |
| Flagged row | A row in Group A or Group B |
| Pair | One order and one product (repeated lines for the same product on one order are added together) |
| ADJ | The order status label given to ADJUSTMENT rows, which have no order |
| ledger, inv, products | The stock ledger, inventory master and products tables loaded in the notebook |
| po_h, po_l, so_h, so_l | Purchase orders header and lines, sales orders header and lines |
| size_summary | Quantity summary per movement type |
| led_status | The ledger with the order status of each row added |
| canc_rows | Ledger rows tied to cancelled orders |
| po_compare, so_compare | Ledger quantity against order-line quantity for every order and product, purchases and sales |
| po_rows, so_rows | Ledger rows of received purchase orders and delivered sales orders joined to their order header |
| identical, same_columns | The rows identical to another row, and the columns compared |
| line_groups, dup_groups, matched_groups | Order-line groups with the same order, product and quantity; groups of identical ledger rows; the two matched |
| lower_pairs | Order and product pairs where the ledger is lower than the order lines |
| flagged | The ledger with the columns cancelled_order, lower_than_order, flagged_any, year and month added |
| used | The list of tables and columns checked for blanks, zeros and negatives |
| months | Rows and flagged rows per month |
| first method, second method | Flags built with table merges; flags rebuilt with plain loops and lookups |
