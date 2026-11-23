# KPI 19 Stockout Risk and Reorder Point

Project: Heavy Supplier & Warehouse Analytics | Week 9 | Notebook: Week9 KPI 19 Stockout Risk and Reorder Point

## Summary

- This KPI answers when each of the 180 product-branch pairs (30 products in 6 branches) should be reordered, and whether any pair is close to running out.
- The reorder point of a pair is the demand during the supplier's real waiting time plus a safety stock for a 95% service level. It runs from 89.9 to 144.5 units per pair (average 116.2).
- Stock per pair, rebuilt from the order lines as in KPI 8, runs from 98,642 to 132,967 units (21,185,215 in total).
- No pair is at or below its reorder point (0 of 180). The closest pair, P019 at KOL001, holds 745.7 times its reorder point; with no new deliveries it would take 74.8 years of average sales to reach it.
- The answer holds at service levels of 90%, 95% and 99%, with every wait at the longest ever seen (35 days), and against the most any pair really sold in any 35 days (383 units, against stock of at least 98,642 units).
- The stored reorder levels in the inventory table are below the calculated reorder points in 175 of 180 pairs. The stored lead times in the products table differ from the real average wait by more than 1 day in 27 of 30 products.
- All 8 verification checks are done. Stock figures come from records, not from a physical count.

## 1 KPI Definition

### Metrics

| Metric | Meaning | How it is calculated |
|---|---|---|
| Average daily demand | Units a pair sells on an average day | Delivered units of the pair from 2019-01-01 to 2024-12-31 divided by the 2,192 days of that period, days without a sale counted as zero |
| Daily swing | How much a pair's daily sales move around its average | Standard deviation of the pair's units per day over the same 2,192 days |
| Real wait | Days between ordering from a supplier and receiving the goods | Received date minus order date of each Received purchase order, averaged over all Received purchase lines of the product; its standard deviation is the swing of the wait |
| Demand during the wait | Units expected to sell while waiting for a new delivery | Average daily demand times the product's average wait |
| Safety stock | Cushion for lumpy sales and uneven waits, at a 95% service level (z = 1.645) | 1.645 x square root of (average wait x daily variance + average daily demand squared x wait variance) |
| Reorder point | Stock level at which a new order should be placed | Demand during the wait plus safety stock |
| Stock | Units the records show for the pair | Opening stock + units on Received purchase orders - units on Delivered sales orders + net adjustments from the stock ledger (the order-line stock of KPI 8) |
| Times the reorder point | How far the stock is above the reorder point | Stock divided by reorder point |
| Years until the reorder point | How long a pair could sell with no new delivery before reaching its reorder point | (Stock - reorder point) / average daily demand / 365 |

### Business question

When should each product be reordered in each branch, and is any pair close to running out of stock?

### Why this metric

A reorder point turns the supplier's waiting time and the pair's sales pattern into one number the warehouse can act on: when stock falls to it, order. Comparing it with the stock held shows how close any pair is to a stockout.

### Scope

- 30 products x 6 branches = 180 pairs.
- Demand comes from Delivered sales orders (18,033 orders, 117,656 lines, 1,234,939 units). Waits come from Received purchase orders (21,630 orders, 140,077 lines, 22,387,123 units). Cancelled orders (1,967 sales orders, 2,370 purchase orders) are left out of both.
- Demand history runs from 2019-01-01 to 2024-12-31. Stock stands at the end of the records: sales deliveries to 2025-01-13, purchase receipts and ledger entries to 2025-01-28.

## 2 Methodology

### Sources and scoping

| Table | Rows | Used for |
|---|---|---|
| sales_orders_header | 20,000 | Delivered status, order date, branch |
| sales_orders_lines | 130,402 | Units sold per product |
| purchase_orders_header | 24,000 | Received status, order date, received date, branch |
| purchase_orders_lines | 155,495 | Units received per product, product of each wait |
| inventory_master | 180 | Opening stock; stored reorder_level and safety_stock for comparison |
| stock_ledger | 237,230 | Net adjustments per pair |
| products | 30 | Stored lead_time_days for comparison |
| suppliers | 8 | Loaded; not used in the calculation |

### Reused and new

| Reused from an earlier KPI | New in this KPI |
|---|---|
| Stock per pair (order-line stock, KPI 8): 21,185,215 units | Average daily demand and daily swing per pair |
| Real waiting time measured from received date minus order date (KPI 13 measured it per supplier; KPI 11 found 7 to 35 days, average 17.9) | Real wait and its swing per product |
| Delivered sales lines and the 2019 to 2024 history (KPI 18) | Safety stock and reorder point per pair |
| | Times the reorder point and years until the reorder point per pair |

### Steps

- Load the tables and check their row counts and columns.
- Join purchase lines to Received purchase headers and count how many suppliers each product and each pair comes from.
- Measure the real wait of every Received order and average it per product, with its swing.
- Build a daily table of Delivered units for each of the 180 pairs over all 2,192 days, then take each pair's average and swing.
- Calculate demand during the wait, safety stock and reorder point for each pair.
- Rebuild the stock of each pair as in KPI 8.
- Compare stock with the reorder point (times the reorder point, years until the reorder point), by pair, branch and product.
- Compare the calculated values with the stored values in the inventory and products tables.
- Run the 8 verification checks.

### Alternatives considered

| Alternative | Decision | Reason and the test behind it |
|---|---|---|
| Use the stored lead_time_days of the products table as the wait | Not used | Stored days run from 4 to 40 (average 17.0); the real average waits run from 17.63 to 18.15 (average 17.9). 27 of 30 products differ by more than 1 day and 19 by more than 5 days (stored lead time test) |
| Use one supplier's wait for each product | Not used | All 30 products and all 180 pairs are supplied by all 8 suppliers (suppliers per product test) |
| Use the stored reorder_level and safety_stock of the inventory table | Not used as inputs; compared with the calculated values | The stored reorder level is below the calculated reorder point in 175 of 180 pairs and at or above it in 5 (stored values test) |
| Service level of 90% or 99% instead of 95% | 95% used; 90% and 99% tested | No pair is at or below its reorder point at any level; smallest stock over the reorder point 840.9 / 745.7 / 615.1 times (Check 6) |
| Use the ledger's last balances (19,305,994 units) instead of the order-line stock (21,185,215 units) | Order-line stock used (the KPI 8 headline) | The ledger is lower by 1,879,221 units, as found in KPI 8 (Check 4). By hand: the smallest ledger-based stock found in KPI 8, 88,853, over the largest reorder point, 144.5, is at least 614 times |
| Judge the risk on the average demand only | Average demand used for the reorder point; the largest real demand tested as well | The most any pair sold in any 35 days is 383 units, and every pair's stock is at least 285.5 times its own largest 35-day demand (Check 6) |

## 3 Verification Log

### Summary of the 8 checks

| Check | What it tested | Result |
|---|---|---|
| Check 1 | Blanks, zeros and negatives in every column used and in the four tables built from them | None found |
| Check 2 | Edge cases: the same 180 pairs in every table, activity per pair, Cancelled orders kept out | All 180 pairs in all five tables; every pair has at least 395 sale days and 671 Received purchase lines; 0 Cancelled lines in the tables used |
| Check 3 | Repeated rows in the order tables, on both the sales and the purchase side | Every order ID used once; no repeated order ID with line number; no accidental copies found; equal-quantity shares at chance |
| Check 4 | Totals reconcile: lines, units, daily table, stock, ledger, reorder point | Everything adds up and ties to KPI 8 |
| Check 5 | Second method with plain loops over the raw rows | Same numbers as the tables for all 180 pairs |
| Check 6 | Outliers and sensitivity: service levels, worst-case wait, real largest demand, single-day sales | No pair at or below its reorder point in any version |
| Check 7 | History and dates | Full history; stock stands at the end of the records |
| Check 8 | 3 pairs rebuilt by hand from the raw files | Identical to the tables for all 3 |

### Check 1: blanks, zeros and negatives

Blanks in the columns used:

| Table | Rows | Columns tested | Blanks |
|---|---|---|---|
| Received purchase headers | 21,630 | po_id, branch_id, order_date, received_date | 0 in each |
| Purchase lines | 155,495 | po_id, product_id, quantity | 0 in each |
| Delivered sales headers | 18,033 | so_id, branch_id, order_date | 0 in each |
| Sales lines | 130,402 | so_id, product_id, quantity | 0 in each |
| Inventory table | 180 | product_id, branch_id, opening_stock | 0 in each |
| Stock ledger | 237,230 | product_id, branch_id, movement_type, running_balance | 0 in each |

Zeros and negatives in the number columns used:

| Column | Zeros | Negatives | Smallest | Largest |
|---|---|---|---|---|
| Purchase line quantity | 0 | 0 | 20 | 300 |
| Sales line quantity | 0 | 0 | 1 | 20 |
| Opening stock | 0 | 0 | 80 | 300 |
| Ledger running balance | 0 | 0 | 14 | 121,013 |

Our own tables:

| Table | Rows | Blanks | Zeros or negatives |
|---|---|---|---|
| Wait per product | 30 | 0 | 0 |
| Daily demand per pair | 180 | 0 | 0 |
| Reorder point table | 180 | 0 | 0 |
| Stock table | 180 | 0 | 0 |

The line tables are the full tables, so they include the Received and Delivered lines that are used.

### Check 2: edge cases

- The inventory table, the stock ledger, the Delivered sales, the Received purchases and the reorder point table each hold 180 pairs, and each set is the same as the inventory table.
- Days with a sale per pair: smallest 395, largest 643, out of 2,192 days. By hand this is 18.0% to 29.3% of days, which matches the no-sale shares of 82.0% and 70.7% in the daily demand table.
- Received purchase lines per pair: smallest 671, largest 842 (by hand, the average is 140,077 / 180 = 778).
- Purchase lines left out because the order was not Received: 15,418 (155,495 - 140,077, by hand). Sales lines left out because the order was not Delivered: 12,746 (130,402 - 117,656, by hand). Both match the cell output.
- Lines of Cancelled purchase orders inside the purchase table used: 0. Lines of Cancelled sales orders inside the sales table used: 0.
- Order statuses: purchase orders Received 21,630 and Cancelled 2,370; sales orders Delivered 18,033 and Cancelled 1,967.

### Check 3: repeated rows

| Test | Sales side | Purchase side |
|---|---|---|
| Order IDs in the header / distinct | 20,000 / 20,000 | 24,000 / 24,000 |
| Lines with a repeated order ID and line number | 0 | 0 |
| Lines in the table used | 117,656 (Delivered) | 140,077 (Received) |
| Lines inside a group of lines identical in every column except the line number | 1,374 | 0 |
| Extra copies if one line of each group is kept | 688 | 0 |
| Order and product combinations with more than one line | 12,184 | 14,408 |
| Combinations by number of lines | 2 lines: 11,202; 3: 914; 4: 65; 5: 2; 6: 1 | 2 lines: 13,219; 3: 1,129; 4: 57; 5: 3 |
| Two-line combinations with equal quantity | 547 of 11,202 = 4.88% | 53 of 13,219 = 0.40% |
| Share expected by chance from the quantity mix | 5.0% | 0.36% |
| p-value of the test against chance | 0.5879 | 0.3806 |

- The combinations by number of lines add up to 12,184 and 14,408 (by hand).
- Both equal-quantity shares are at what chance gives, and both p-values are well above 0.05. This is a finding of no difference from chance; it is not proof that no copies exist.
- The chance test covers combinations with exactly two lines: 11,202 of 12,184 on the sales side (91.9%, by hand) and 13,219 of 14,408 on the purchase side (91.7%, by hand). The combinations with three or more lines (982 on the sales side and 1,189 on the purchase side, by hand) are not tested separately.
- No purchase line is identical to another in every column except the line number. Why the 53 equal-quantity pairs on the purchase side are not identical in every column was not investigated.

### Check 4: totals reconcile

| Total | Value | Compared with |
|---|---|---|
| Received purchase lines + lines left out | 140,077 + 15,418 = 155,495 | All purchase lines 155,495 |
| Delivered sales lines + lines left out | 117,656 + 12,746 = 130,402 | All sales lines 130,402 |
| Received units | 22,387,123 | Expected 22,387,123 |
| Delivered units | 1,234,939 | Expected 1,234,939 |
| Units in the daily demand table | 1,234,939 | Expected 1,234,939 |
| Largest difference per pair, daily table against sales lines | 0 | |
| Average daily demand x 2,192 days, added up | 1,234,939.0 | Expected 1,234,939 |
| Opening stock | 32,969 | |
| Net adjustments | 62 | |
| Opening + Received - Delivered + net adjustments | 21,185,215 | Final stock table 21,185,215 |
| Ledger last balances added up | 19,305,994 | Expected 19,305,994 (KPI 8) |
| Order-line stock minus ledger | 1,879,221 | Expected 1,879,221 (KPI 8) |
| Reorder point minus (demand during the wait + safety stock), largest difference | 1.42e-14 | Computer rounding, zero in practice |

By hand: 32,969 + 22,387,123 - 1,234,939 + 62 = 21,185,215.

### Check 5: second method

Every number was recomputed with plain Python loops over the raw rows (no group-by), then compared with the tables:

| Number | Largest difference over 180 pairs |
|---|---|
| Average daily demand | 0.0 |
| Daily swing | 2.1e-13 |
| Average wait | 0.0 |
| Swing of the wait | 1.5e-14 |
| Demand during the wait | 0.0 |
| Safety stock | 1.2e-12 |
| Reorder point | 1.2e-12 |
| Stock | 0.0 |

The values written in e-12 to e-14 are computer rounding and are zero in practice. In the loop method 0 pairs are at or below their reorder point, and the smallest stock over the reorder point is 745.7 times, the same as in the tables.

### Check 6: outliers and sensitivity

Service level:

| Service level | z | Average reorder point | Largest reorder point | Pairs at or below | Smallest stock over reorder point |
|---|---|---|---|---|---|
| 90% | 1.282 | 102.9 | 128.8 | 0 | 840.9 |
| 95% (used) | 1.645 | 116.2 | 144.5 | 0 | 745.7 |
| 99% | 2.326 | 141.1 | 174.6 | 0 | 615.1 |

By hand, the 99% reorder point of any pair is at most 1.414 x 144.5 = about 204, and the smallest stock over it is then at least about 483; the output is inside both limits.

Worst-case wait (every order takes 35 days, the longest ever seen):

- The longest wait is 1.96 times the average wait of 17.9 days (by hand, 35 / 17.9).
- The largest reorder point would be 221.1; 0 pairs are at or below it; the smallest stock over it is 485.1 times.

Real largest demand, with no model:

- The most units any pair sold in any 35 days is 383.0; the average pair's largest 35-day demand is 256.0.
- Pairs with stock below their own largest 35-day demand: 0. The smallest stock over the largest 35-day demand is 285.5 times (by hand, at least 98,642 / 383 = about 258).

Single-day sales:

- The largest single-day sale of any pair is 92 units; the average pair's biggest day is 57.4 units.
- A pair's biggest day is from 11.8 to 30.5 times its daily average.

### Check 7: history and dates

| Input | First | Last |
|---|---|---|
| Delivered sales, order date | 2019-01-01 | 2024-12-31 |
| Delivered sales, delivery date | 2019-01-03 | 2025-01-13 |
| Received purchases, order date | 2019-01-01 | 2024-12-31 |
| Received purchases, received date | 2019-01-11 | 2025-01-28 |
| Stock ledger, entry date | 2019-01-01 | 2025-01-28 |

Received orders with a wait of zero days or less: 0.

By year:

| Year | Received lines | Average wait (days) | Units sold | Average daily demand per pair |
|---|---|---|---|---|
| 2019 | 23,431 | 18.12 | 207,925 | 3.165 |
| 2020 | 23,773 | 17.96 | 212,007 | 3.218 |
| 2021 | 23,443 | 17.91 | 208,269 | 3.170 |
| 2022 | 23,097 | 17.78 | 202,242 | 3.078 |
| 2023 | 23,377 | 17.79 | 204,638 | 3.115 |
| 2024 | 22,956 | 17.84 | 199,858 | 3.034 |

- Received lines add up to 140,077 and units sold to 1,234,939. The units sold per year equal those of KPI 18.
- By hand: 2019 daily demand per pair = 207,925 / 365 / 180 = 3.165. The yearly average waits lie within 0.34 days of each other (18.12 - 17.78). The 2024 daily demand per pair, 3.034, is 3.1% below the six-year average of 3.130. The yearly numbers are given without any claim about a trend.
- Months with Received lines per product: smallest 72, largest 72 (of 72). Months with sales per pair: smallest 70, largest 72; the 70 is P004 at HYD001, which has 2 months without sales (KPI 18).

### Check 8: three pairs rebuilt by hand from the raw files

The raw csv files were read with plain Python (no pandas, no tables from the notebook).

| | P019 at KOL001 | P001 at CHN001 | P004 at HYD001 |
|---|---|---|---|
| Why chosen | Closest to its reorder point; smallest stock | Third closest; highest-demand branch | Two months without sales |
| Days with a sale | 560 | 614 | 404 |
| Units sold | 7,904 | 8,681 | 5,148 |
| Average daily demand | 3.6058 | 3.9603 | 2.3485 |
| Daily swing | 7.9247 | 8.1704 | 6.0275 |
| Average wait / swing of the wait | 17.868 / 6.673 | 17.707 / 6.616 | 17.954 / 6.641 |
| Opening / received / adjustments | 92 / 106,282 / 172 | 187 / 119,011 / -115 | 193 / 125,091 / -117 |
| Stock | 98,642 | 110,402 | 120,019 |
| Safety stock | 67.85 | 71.11 | 49.23 |
| Reorder point | 132.28 | 141.24 | 91.39 |
| Times the reorder point | 745.7 | 781.7 | 1,313.2 |
| Same as the tables | Yes | Yes | Yes |

By hand: 7,904 / 2,192 = 3.6058; 8,681 / 2,192 = 3.9603; 5,148 / 2,192 = 2.3485. Stock: 92 + 106,282 - 7,904 + 172 = 98,642; 187 + 119,011 - 8,681 - 115 = 110,402; 193 + 125,091 - 5,148 - 117 = 120,019. Reorder points: 3.6058 x 17.868 + 67.85 = 132.28; 3.9603 x 17.707 + 71.11 = 141.23 (141.24 before rounding); 2.3485 x 17.954 + 49.23 = 91.39.

### Supporting tests

#### Loading the tables

| Table | Rows x columns |
|---|---|
| Sales orders header | 20,000 x 11 |
| Sales orders lines | 130,402 x 9 |
| Purchase orders header | 24,000 x 10 |
| Purchase orders lines | 155,495 x 9 |
| Products | 30 x 23 |
| Suppliers | 8 x 12 |
| Inventory master | 180 x 8 |
| Stock ledger | 237,230 x 9 |

All shapes match the earlier KPIs. The inventory table holds reorder_level, safety_stock and max_stock per pair; the products table holds reorder_level, safety_stock, max_stock_level and lead_time_days per product. The purchase lines have no supplier column; the supplier of a line comes from joining to the purchase header on po_id.

#### Suppliers per product

- Received purchase lines 140,077 and units 22,387,123 (both as expected).
- All 30 products come from all 8 suppliers. All 180 product-branch pairs come from all 8 suppliers.

#### Waiting time per product

The real wait of a product is its average over all its Received purchase lines. Over all lines the smallest wait is 7 days, the average 17.9 and the largest 35. Every product has a smallest wait of 7 and a largest of 35 days.

| Product | Lines | Average wait | Swing | Product | Lines | Average wait | Swing |
|---|---|---|---|---|---|---|---|
| P001 | 4,635 | 17.71 | 6.62 | P016 | 4,581 | 17.85 | 6.71 |
| P002 | 4,752 | 17.96 | 6.72 | P017 | 4,726 | 17.93 | 6.65 |
| P003 | 4,739 | 17.78 | 6.71 | P018 | 4,730 | 17.87 | 6.74 |
| P004 | 4,613 | 17.95 | 6.64 | P019 | 4,539 | 17.87 | 6.67 |
| P005 | 4,774 | 18.11 | 6.62 | P020 | 4,510 | 17.84 | 6.63 |
| P006 | 4,685 | 17.93 | 6.63 | P021 | 4,638 | 17.93 | 6.56 |
| P007 | 4,718 | 17.82 | 6.74 | P022 | 4,667 | 17.96 | 6.62 |
| P008 | 4,621 | 17.63 | 6.60 | P023 | 4,679 | 18.15 | 6.64 |
| P009 | 4,614 | 17.69 | 6.62 | P024 | 4,755 | 18.03 | 6.80 |
| P010 | 4,751 | 18.03 | 6.72 | P025 | 4,700 | 17.77 | 6.65 |
| P011 | 4,730 | 18.01 | 6.65 | P026 | 4,624 | 17.93 | 6.72 |
| P012 | 4,753 | 17.95 | 6.73 | P027 | 4,702 | 17.85 | 6.66 |
| P013 | 4,655 | 17.81 | 6.61 | P028 | 4,538 | 17.75 | 6.69 |
| P014 | 4,796 | 17.93 | 6.65 | P029 | 4,665 | 18.03 | 6.69 |
| P015 | 4,605 | 18.04 | 6.73 | P030 | 4,582 | 17.94 | 6.69 |

The product averages run from 17.63 (P008) to 18.15 (P023), a spread of 0.52 days (by hand). The swing is about 6.6 days for every product (6.56 to 6.80).

#### Daily demand per pair

Delivered lines 117,656, units 1,234,939, days 2,192, daily table 180 pairs x 2,192 days, units in the table 1,234,939 (all as expected).

| Per pair | Average | Std | Smallest | 25% | Median | 75% | Largest |
|---|---|---|---|---|---|---|---|
| Average daily demand | 3.130 | 0.478 | 2.318 | 2.728 | 3.062 | 3.527 | 4.042 |
| Daily swing | 7.088 | 0.572 | 5.858 | 6.595 | 6.976 | 7.585 | 8.263 |
| Share of days without a sale | 0.767 | 0.030 | 0.707 | 0.743 | 0.768 | 0.794 | 0.820 |

The average daily demand matches 1,234,939 / 394,560 = 3.130 by hand. A pair sells nothing on 76.7% of days on average (70.7% to 82.0%), and the daily swing (7.088) is 2.26 times the daily average (3.130) (by hand). Sales come in lumps.

#### Reorder point per pair

| Per pair | Average | Std | Smallest | 25% | Median | 75% | Largest |
|---|---|---|---|---|---|---|---|
| Demand during the wait | 56.0 | 8.6 | 41.8 | 48.7 | 54.9 | 63.2 | 73.4 |
| Safety stock | 60.1 | 6.2 | 48.1 | 54.7 | 58.8 | 65.2 | 72.7 |
| Reorder point | 116.2 | 14.7 | 89.9 | 103.5 | 112.9 | 128.1 | 144.5 |

180 pairs, 0 empty values. The average demand during the wait matches 3.13 x 17.9 = 56 (by hand). First three rows, checked by hand:

| Pair | Average daily demand | Daily swing | Wait | Swing of the wait | Demand during the wait | Safety stock | Reorder point |
|---|---|---|---|---|---|---|---|
| P001 AHM001 | 3.20 | 7.26 | 17.71 | 6.62 | 56.60 | 61.13 | 117.72 |
| P001 CHN001 | 3.96 | 8.17 | 17.71 | 6.62 | 70.13 | 71.11 | 141.24 |
| P001 DEL001 | 2.94 | 6.95 | 17.71 | 6.62 | 52.05 | 57.76 | 109.81 |

By hand, for P001 AHM001: the square root of (17.71 x 7.26 squared + 3.2 squared x 6.62 squared) is about 37.2, times 1.645 is 61.2, and 56.6 + 61.13 = 117.73.

#### Stock rebuild

180 pairs, 0 empty values. Received units 22,387,123, Delivered units 1,234,939, net adjustments 62, total stock 21,185,215, smallest pair 98,642, largest pair 132,967: all equal to KPI 8.

#### Stock against the reorder point

| Per pair | Average | Std | Smallest | 25% | Median | 75% | Largest |
|---|---|---|---|---|---|---|---|
| Stock | 117,695.6 | 5,367.5 | 98,642 | 113,562.2 | 117,900.5 | 121,422.5 | 132,967 |
| Times the reorder point | 1,030.5 | 146.7 | 745.7 | 907.0 | 1,020.0 | 1,148.9 | 1,377.2 |
| Days of demand (stock / average daily demand) | 38,529.6 | 6,422.0 | 27,356.2 | 33,159.6 | 38,473.1 | 43,772.3 | 53,845.1 |

- Pairs with stock at or below the calculated reorder point: 0. Pairs with stock at or below the stored reorder level: 0.
- By hand, the days of demand are 74.9 years at the smallest (27,356.2 / 365), 105.6 at the average and 147.5 at the largest. KPI 6 found 72 to 172 years of cover on the last year's sales; the windows differ, so the numbers are not identical.
- The smallest times the reorder point, 745.7, is above the lower limit of about 683 worked out by hand (98,642 / 144.5).

The three pairs closest to their reorder point:

| Pair | Stock | Reorder point | Times the reorder point | Days of demand |
|---|---|---|---|---|
| P019 KOL001 | 98,642 | 132.3 | 745.7 | 27,356.2 |
| P027 CHN001 | 108,989 | 141.2 | 771.6 | 27,689.4 |
| P001 CHN001 | 110,402 | 141.2 | 781.7 | 27,877.1 |

P019 at KOL001 is also the pair with the smallest stock in KPI 8.

#### Branch and product view

Years until each pair would reach its reorder point with no new deliveries, by branch:

| Branch | Pairs | Stock | Reorder points added up | Shortest (years) | Median (years) |
|---|---|---|---|---|---|
| AHM001 | 30 | 3,472,113 | 3,609.1 | 88.0 | 97.4 |
| CHN001 | 30 | 3,548,773 | 4,154.1 | 75.8 | 84.4 |
| DEL001 | 30 | 3,597,678 | 3,199.8 | 102.1 | 116.6 |
| HYD001 | 30 | 3,555,196 | 2,967.4 | 109.2 | 124.8 |
| KOL001 | 30 | 3,435,814 | 3,836.1 | 74.8 | 90.8 |
| PUN001 | 30 | 3,575,641 | 3,145.0 | 107.0 | 118.2 |
| All | 180 | 21,185,215 | | | |

Pairs add up to 180 and stock to 21,185,215. The five products closest to their reorder point (shortest time over their branches): P019 74.8 years, P027 75.8, P001 76.3, P030 77.8, P023 78.7. By hand, for P019 at KOL001: 74.9 years of demand minus about 0.1 year for the 132 units of reorder point is 74.8.

#### Stored values compared

| Stored in the inventory table | Average | Std | Smallest | 25% | Median | 75% | Largest |
|---|---|---|---|---|---|---|---|
| reorder_level | 64.6 | 24.1 | 24 | 44 | 62 | 82.2 | 119 |
| safety_stock | 45.2 | 16.9 | 16 | 31 | 43.5 | 58 | 85 |
| max_stock | 321.8 | 116.6 | 131 | 230 | 310.5 | 415.5 | 582 |

- The stored reorder level is below the calculated reorder point in 175 of 180 pairs and at or above it in 5 (175 + 5 = 180, by hand).
- The calculated reorder point averages 116.2 against a stored reorder level of 64.6, about 1.8 times (by hand). The calculated safety stock averages 60.1 against a stored 45.2, about 1.3 times (by hand). No cause was investigated.

#### Stored lead time

| Product | Stored days | Real average wait | Real minus stored |
|---|---|---|---|
| P015 | 40 | 18.04 | -21.96 |
| P014 | 35 | 17.93 | -17.07 |
| P026 | 4 | 17.93 | +13.93 |
| P017 | 5 | 17.93 | +12.93 |
| P027 | 5 | 17.85 | +12.85 |

These are the five largest differences of 30 products compared. Stored days run from 4 to 40 (average 17.0); real average waits run from 17.63 to 18.15 (average 17.9). 27 products differ by more than 1 day and 19 by more than 5 days. No cause was investigated.

## 4 Data Quality Notes

1. All stock figures come from records, not from a physical count. They assume that every Received purchase order was fully received and every Delivered sales order was fully shipped.
2. The stock stands at the end of the records (28 January 2025); the demand history ends on 31 December 2024.
3. The stock ledger is lower than the order-line stock by 1,879,221 units (KPI 8 found that about 10% of order lines never reached the ledger). The order-line stock is used, as in KPI 8.
4. The reorder_level and safety_stock stored in the inventory table, and the lead_time_days stored in the products table, are not used as inputs: they differ from the calculated and the real values (175 of 180 pairs; 27 of 30 products).
5. The reorder points use six-year averages of demand and of the wait. Yearly figures are given in Check 7 without a claim about a trend; the 2024 daily demand per pair is 3.1% below the six-year average (by hand).
6. Sales come in lumps (76.7% of days without a sale on average). The safety stock formula treats days as independent; the real largest 35-day demand test (Check 6) does not depend on that assumption.
7. No purchase line is identical to another in every column except the line number, although 53 two-line combinations have equal quantities; the reason was not investigated.
8. The chance test for repeated lines covers combinations with exactly two lines (91.9% of repeated sales combinations and 91.7% of repeated purchase combinations, by hand). Combinations with three or more lines are not tested separately.
9. The first run of the empty-values check stopped with an error because one table name had been reused in the notebook; the check was run again in full with a new name and its result is the one reported.

## 5 Business Interpretation

### Business question and answer

When should each product be reordered in each branch, and is any pair close to running out of stock?

Answer: no pair needs to be reordered now and none is close to a stockout. The reorder points run from 89.9 to 144.5 units per pair (average 116.2), while every pair holds from 98,642 to 132,967 units. The closest pair, P019 at KOL001, holds 745.7 times its reorder point and would need 74.8 years of average sales, with no new deliveries, to reach it.

### Findings

- The answer holds at service levels of 90% (smallest stock 840.9 times the reorder point), 95% (745.7) and 99% (615.1), with every wait at 35 days (485.1 times), and against the most any pair really sold in any 35 days (383 units; every pair's stock is at least 285.5 times its own largest 35-day demand).
- Stock is 745.7 to 1,377.2 times the reorder point (average 1,030.5) and covers 74.9 to 147.5 years of average sales (by hand).
- By branch, KOL001 (shortest 74.8 years) and CHN001 (75.8) have the pairs closest to their reorder points; HYD001 is furthest (shortest 109.2, median 124.8 years). The products closest to their reorder points are P019, P027, P001, P030 and P023.
- Units keep arriving: 22,387,123 units were received and 1,234,939 delivered over the period, about 18.1 to 1 (by hand).
- The stored reorder levels are below the calculated reorder points in 175 of 180 pairs, and the stored lead times differ from the real average wait by more than 1 day in 27 of 30 products.
- Sales come in lumps: on average a pair sells nothing on 76.7% of days, and a pair's biggest day is 11.8 to 30.5 times its daily average (the largest single-day sale of any pair is 92 units).
- The real wait of a product averages 17.63 to 18.15 days (a spread of 0.52 days, by hand), and all 8 suppliers supply every product.

### Recommended actions for the business owners

1. Inventory planning team: use the calculated reorder points (89.9 to 144.5 units per pair, average 116.2) as the level at which a new order is triggered, and recalculate them when average daily demand or the real wait changes.
2. Purchasing manager, with inventory planning: review why purchase orders continue to be placed while every pair holds at least 745.7 times its reorder point (22,387,123 units received against 1,234,939 delivered).
3. Warehouse manager: confirm the recorded stock by physical count, starting with the pairs closest to their reorder points (P019 at KOL001, P027 at CHN001, P001 at CHN001), because every figure here comes from records.
4. Data / system owner: correct the stored reorder_level and safety_stock of the inventory table (below the calculated reorder points in 175 of 180 pairs) and the lead_time_days of the products table (differs from the real average wait by more than 1 day in 27 of 30 products).

## Glossary

| Term | Meaning |
|---|---|
| Pair | One product in one branch (30 products x 6 branches = 180 pairs) |
| Reorder point | Stock level at which a new order should be placed: demand during the wait plus safety stock |
| Safety stock | Extra stock that covers lumpy sales and uneven waits at the chosen service level |
| Service level | Share of reorder cycles in which no stockout is wanted; 95% is used, 90% and 99% are tested |
| z | The factor that belongs to a service level: 1.282 (90%), 1.645 (95%), 2.326 (99%) |
| Real wait | Received date minus order date of a Received purchase order, in days |
| Swing (std) | Standard deviation: how far values typically move around their average |
| Order-line stock | Opening stock + Received units - Delivered units + net adjustments (the KPI 8 headline stock) |
| Net adjustments | Sum of the changes in running balance on the stock ledger's ADJUSTMENT rows |
| Times the reorder point | Stock divided by reorder point |
| Days of demand | Stock divided by average daily demand |
| Years until the reorder point | (Stock - reorder point) / average daily demand / 365 |
| Largest 35-day demand | The most units a pair sold in any run of 35 days (35 days is the longest wait ever seen) |
| By hand | A number worked out with a calculator from printed results, not printed by the notebook |
| pl | Notebook table of Received purchase lines with the supplier, branch and wait of their order |
| sales | Notebook table of Delivered sales lines with order date and branch |
| daily | Notebook table of units sold per day for each of the 180 pairs |
| pair_stats | Notebook table with average daily demand, daily swing and share of days without a sale per pair |
| wait_table | Notebook table with the average wait and its swing per product |
| rp | Notebook table with demand during the wait, safety stock and reorder point per pair |
| stock | Notebook table with opening stock, Received, Delivered, net adjustments and order-line stock per pair |
