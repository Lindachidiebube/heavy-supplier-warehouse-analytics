# KPI 13: Supplier Lead Time and Reliability

**Project: Heavy Supplier & Warehouse Analytics | Week 7 | Notebook: Week7_KPI13_Supplier_Lead_Time_and_Reliability**

---

## Summary

- KPI 13 measures how long each of the 8 suppliers really takes to deliver, and how often it misses the promised delivery date.
- It uses the 21,630 Received purchase orders. The 2,370 Cancelled orders never arrived, so they are left out.
- About 1 order in 7 is late: 3,267 of 21,630 (15.10%). 17,392 orders (80.41%) arrived on the promised day and 971 (4.49%) arrived early.
- A late order is 1 to 7 days late (3.99 days on average). The earliest arrival is 3 days early. Over all orders the average is 0.513 days after the promised day.
- All 8 suppliers keep the promised date equally well: late share between 14.67% and 15.80%. A test shows these differences are within chance (p-value 0.9702).
- The suppliers differ in how long they take: from 10.50 days (Qingdao) to 28.49 days (Tianjin).
- The promised days in the suppliers table are supported by the orders (the real wait is 0.49 to 0.55 days longer). The reliability score in the suppliers table is not supported by the orders.
- All 8 verification checks were run and every check agrees.

---

## 1. KPI Definition

### Metric table

| Metric | Meaning | How it is worked out | Result |
|---|---|---|---|
| Real lead time | How long we really wait for an order | Received date minus order date, averaged per supplier | 10.50 to 28.49 days depending on the supplier |
| Days late | How far from the promised day the order arrived | Received date minus promised date (negative means early) | -3 to 7 days, average 0.513 |
| Late share | How often a supplier misses the promised date | Late orders divided by Received orders, times 100 | 15.10% overall, 14.67% to 15.80% by supplier |
| On time or early share | How often a supplier keeps the promised date | 100 minus the late share | 84.90% overall, 84.20% to 85.33% by supplier |
| Average days late | How late a supplier is on average, counting early orders as below zero | Average of days late per supplier | 0.485 to 0.546 days by supplier |

### Business question

Which suppliers deliver late, how late, and how long do we really wait for each supplier?

### Why this metric

Buyers choose suppliers and decide how early to place orders. To do that they need to know whether a supplier keeps its promised date and how long the real wait is. KPI 13 works these out from the real order dates, not from the numbers stored in the suppliers table, because the stored reliability score turned out not to follow the real deliveries (see Supporting tests).

### Scope

- Received purchase orders only: 21,630 of the 24,000 orders.
- 8 suppliers, orders placed from January 2019 to December 2024 (72 months).
- The 2,370 Cancelled orders have no received date, so there is nothing to measure for them.

---

## 2. Methodology

### Sources and scoping

- `purchase_orders_header.csv`: order date, promised date (`expected_delivery_date`), received date, status and supplier of every order.
- `suppliers.csv`: supplier names, and the stored promised days (`lead_time_days`) and reliability score used in the comparison.
- `purchase_orders_lines.csv` and `products.csv` were loaded and read but are not used in the KPI 13 numbers.
- Only orders with status Received are used.

### Steps

- Load the four tables and look at their columns and row counts.
- Count orders by status and check for missing dates in the Received orders.
- Work out days late (received minus promised) and real lead time (received minus order) for every Received order.
- Count how many orders are early, on the promised day and late, and look at the spread of days late.
- Build the per-supplier table: orders, late orders, late share, average days late, average real lead time.
- Test whether the differences in late share between suppliers are more than chance.
- Compare the suppliers table's own numbers (promised days, reliability score) with what really happened.
- Run the 8 verification checks.

### Alternatives considered

| Alternative | Decision | Reason |
|---|---|---|
| Include the Cancelled orders | Rejected | They have no received date, so lateness cannot be measured |
| Use the stored reliability score as the reliability measure | Rejected | The score is 3, 4 or 5 but the real late share is about 15% for every supplier; the lowest-score group is not the latest |
| Use the stored promised days as the lead time | Not used as the measure | Real lead time is worked out from the dates. The stored days are kept for comparison and are supported by the orders |
| Use `lead_time_days` from the products table | Rejected | A Week 3 finding showed it is not supported by the order history (real average about 17.7 to 18.1 days) |
| Call the supplier with the highest late share the worst | Rejected | The late share test gives a p-value of 0.9702, so the differences are within chance |
| Add the other Week 7 catalogue tasks (Supplier Contribution Analysis, Supplier Dependency Risk Analysis, Critical Supplier Identification, Supplier Diversification Analysis, Warehouse Utilisation Benchmarking) | Not selected | They answer other questions (how much we depend on each supplier; warehouse use). Supplier Performance Comparison is included because it is the per-supplier comparison in this KPI |

---

## 3. Verification Log

### Summary of the 8 checks

| # | Check | Method | Result |
|---|---|---|---|
| 1 | Missing values | Count blanks in the 5 columns used; match every order's supplier to the suppliers table | 0 blanks in every column; 0 orders with an unknown supplier |
| 2 | Impossible dates | Count orders received or promised on or before the order date; smallest waits | 0 and 0; smallest real wait 7 days, smallest promised wait 10 days |
| 3 | Repeated orders | Compare rows with different order IDs, in the Received orders and in the full table | 21,630 rows and 21,630 different IDs; 0 repeated IDs in both tables |
| 4 | Totals reconcile | Add the per-supplier numbers back up to the totals | All four totals match |
| 5 | Second method | Recompute each supplier's three numbers with a loop and compare | Largest difference 0.0 for all three numbers |
| 6 | Extreme orders | Smallest and largest days late, median against average real wait, longest wait against average | No extreme orders; the averages are not pulled by a few orders |
| 7 | History length | Months with orders per supplier, first and last order month, fewest orders in a month | 72 months for all 8 suppliers; fewest orders in a month 19 to 26 |
| 8 | Raw file check of 3 suppliers | Recalculate Qingdao, Tianjin and Hangzhou from the raw orders file with a plain loop | Identical to the supplier table |

### Check 1: Missing values

Method: count missing values in the Received orders for `po_id`, `supplier_id`, `order_date`, `expected_delivery_date` and `received_date`, and count orders whose supplier is not in the suppliers table.

| Column | Missing values |
|---|---|
| po_id | 0 |
| supplier_id | 0 |
| order_date | 0 |
| expected_delivery_date | 0 |
| received_date | 0 |

Orders whose supplier is not in the suppliers table: 0.

Result: no blanks, and every order matches a supplier, so no order drops out of a supplier's numbers.

### Check 2: Impossible dates

| Test | Result |
|---|---|
| Received on or before the order date | 0 |
| Promised on or before the order date | 0 |
| Smallest real wait | 7 days |
| Smallest promised wait | 10 days |

Result: no impossible dates. The smallest real wait (7 days) agrees with the KPI 11 work. The smallest promised wait (10 days) equals the smallest `lead_time_days` in the suppliers table (Guangzhou and Qingdao), and 10 promised days minus the earliest arrival of 3 days early is 7.

### Check 3: Repeated orders

| Test | Result |
|---|---|
| Rows in the Received orders table | 21,630 |
| Different order IDs in it | 21,630 |
| Order IDs that appear more than once in it | 0 |
| Order IDs that appear more than once in the full 24,000-row table | 0 |

Result: every order appears once.

### Check 4: Totals reconcile

| Test | Left side | Right side |
|---|---|---|
| Orders added up over the 8 suppliers, against all Received orders | 21,630 | 21,630 |
| Late orders added up over the 8 suppliers, against all late Received orders | 3,267 | 3,267 |
| Early + on the promised day + late | 21,630 | 21,630 (all Received orders) |
| Received + Cancelled, against all orders | 24,000 | 24,000 |

Result: all totals match, so no order is lost or counted twice.

### Check 5: Second method

Method: the per-supplier table was first built with a group-by. Each supplier's late share, average days late and average real lead time were then recomputed with a plain loop over the 8 suppliers and compared.

| Number | Largest difference over the 8 suppliers |
|---|---|
| Late share (percent) | 0.0 |
| Average days late | 0.0 |
| Average real lead days | 0.0 |

Result: both methods give identical numbers.

### Check 6: Extreme orders

| Supplier | Smallest days late | Largest days late | Median real wait | Average real wait | Longest real wait | Longest vs average |
|---|---|---|---|---|---|---|
| Shenzhen OEM Supplies | -3 | 7 | 19.0 | 19.50 | 26 | 1.33 |
| Guangzhou Local Vendor Supplies | -3 | 7 | 10.0 | 10.52 | 17 | 1.62 |
| Shanghai Distributor Supplies | -3 | 7 | 16.0 | 16.53 | 23 | 1.39 |
| Ningbo Distributor Supplies | -3 | 7 | 18.0 | 18.50 | 25 | 1.35 |
| Tianjin OEM Supplies | -3 | 7 | 28.0 | 28.49 | 35 | 1.23 |
| Qingdao Distributor Supplies | -3 | 7 | 10.0 | 10.50 | 17 | 1.62 |
| Suzhou Local Vendor Supplies | -3 | 7 | 12.0 | 12.53 | 19 | 1.52 |
| Hangzhou OEM Supplies | -3 | 7 | 26.0 | 26.55 | 33 | 1.24 |

Result: every supplier has the same range of days late (-3 to 7). The median real wait equals the promised days for all 8 suppliers, and the average is 0.49 to 0.55 days above it. The longest wait is the promised days plus 7 for all 8. The longest wait is 1.23 to 1.62 times the average, below 2 times as in KPI 1. No few extreme orders move the averages.

### Check 7: History length

| Supplier | Months with orders | First order month | Last order month | Fewest orders in a month |
|---|---|---|---|---|
| SUP0001 | 72 | 2019-01 | 2024-12 | 26 |
| SUP0002 | 72 | 2019-01 | 2024-12 | 23 |
| SUP0003 | 72 | 2019-01 | 2024-12 | 25 |
| SUP0004 | 72 | 2019-01 | 2024-12 | 24 |
| SUP0005 | 72 | 2019-01 | 2024-12 | 25 |
| SUP0006 | 72 | 2019-01 | 2024-12 | 19 |
| SUP0007 | 72 | 2019-01 | 2024-12 | 23 |
| SUP0008 | 72 | 2019-01 | 2024-12 | 25 |

Months with orders, all suppliers together: 72.

Result: every supplier has the full 72-month history and no thin months. The KPI numbers are for the whole period per supplier (about 2,700 orders each), so even the quietest month (19 orders, SUP0006) is not a concern.

### Check 8: Raw file check of 3 suppliers

Method: the raw orders file was read again with a plain loop (no pandas). For three suppliers across the range, all Received orders were counted and the numbers were compared with the supplier table. The three were chosen as Qingdao (fastest wait), Tianjin (slowest wait) and Hangzhou (highest late share).

| Supplier | Source | Orders | Late | Late share (%) | Average days late | Average real lead days |
|---|---|---|---|---|---|---|
| Qingdao Distributor Supplies (SUP0006) | Raw file | 2,752 | 407 | 14.79 | 0.501 | 10.5 |
| | Supplier table | 2,752 | 407 | 14.79 | 0.501 | 10.5 |
| Tianjin OEM Supplies (SUP0005) | Raw file | 2,754 | 404 | 14.67 | 0.485 | 28.49 |
| | Supplier table | 2,754 | 404 | 14.67 | 0.485 | 28.49 |
| Hangzhou OEM Supplies (SUP0008) | Raw file | 2,722 | 430 | 15.8 | 0.546 | 26.55 |
| | Supplier table | 2,722 | 430 | 15.8 | 0.546 | 26.55 |

Result: identical for all three suppliers. (10.5 and 15.8 are 10.50 and 15.80 with the last zero not printed.)

### Supporting tests

These tests were run while building the KPI, before the numbered checks. They are separate from the 8 checks and do not replace any of them.

#### Loading the tables

| Table | Rows | Columns |
|---|---|---|
| Purchase orders header (po_h) | 24,000 | 10 |
| Purchase orders lines (po_l) | 155,495 | 9 |
| Suppliers | 8 | 12 |
| Products | 30 | 23 |

The header columns are `po_id`, `supplier_id`, `branch_id`, `order_date`, `expected_delivery_date`, `received_date`, `po_status`, `total_cost`, `total_gst_amount` and `grand_total`. The suppliers table also holds `lead_time_days` and `reliability_score`, which are compared with the real orders below.

Result: all four row counts are as expected.

#### How late is late

| Test | Result |
|---|---|
| Orders by status | Received 21,630; Cancelled 2,370 (total 24,000) |
| Missing dates in Received orders | 0 in order date, promised date and received date |
| Late (received after the promised day) | 3,267 (15.10%) |
| On the promised day | 17,392 (80.41%) |
| Early | 971 (4.49%) |
| On the promised day or earlier | 18,363 (84.90%) |
| Smallest days late | -3 |
| Largest days late | 7 |
| Average days late | 0.513 (11,090 days over 21,630 orders) |
| Average days late of the late orders only | 3.99 (13,029 days over 3,267 late orders) |

Orders for each number of days late:

| Days late | Orders |
|---|---|
| -3 | 334 |
| -2 | 300 |
| -1 | 337 |
| 0 | 17,392 |
| 1 | 467 |
| 2 | 475 |
| 3 | 461 |
| 4 | 469 |
| 5 | 464 |
| 6 | 484 |
| 7 | 447 |

Result: 4,238 orders (3,267 late and 971 early) have a received date different from the promised date, so the received date is real data and not a copy of the promised date. Each late day (1 to 7) has between 447 and 484 orders, so there is no long tail of very late orders. The average of 0.513 days matches the Week 3 finding of about half a day.

#### Per-supplier results

| Supplier | Orders | Late orders | Late share (%) | On time or early (%) | Average days late | Average real lead days |
|---|---|---|---|---|---|---|
| SUP0001 Shenzhen OEM Supplies | 2,653 | 400 | 15.08 | 84.92 | 0.496 | 19.50 |
| SUP0002 Guangzhou Local Vendor Supplies | 2,635 | 396 | 15.03 | 84.97 | 0.516 | 10.52 |
| SUP0003 Shanghai Distributor Supplies | 2,723 | 408 | 14.98 | 85.02 | 0.531 | 16.53 |
| SUP0004 Ningbo Distributor Supplies | 2,664 | 404 | 15.17 | 84.83 | 0.497 | 18.50 |
| SUP0005 Tianjin OEM Supplies | 2,754 | 404 | 14.67 | 85.33 | 0.485 | 28.49 |
| SUP0006 Qingdao Distributor Supplies | 2,752 | 407 | 14.79 | 85.21 | 0.501 | 10.50 |
| SUP0007 Suzhou Local Vendor Supplies | 2,727 | 418 | 15.33 | 84.67 | 0.528 | 12.53 |
| SUP0008 Hangzhou OEM Supplies | 2,722 | 430 | 15.80 | 84.20 | 0.546 | 26.55 |
| All suppliers | 21,630 | 3,267 | 15.10 | 84.90 | 0.513 | not reported |

#### Late share test: is any supplier really more late than the others?

Method: a chi-square test asks how often chance alone would produce differences in late share as big as these if all 8 suppliers were equally late.

p-value: 0.9702 (above 0.05, so the differences are within normal chance).

| Supplier | Late orders | Late orders if all equal | Difference |
|---|---|---|---|
| Shenzhen OEM Supplies | 400 | 400.7 | -0.7 |
| Guangzhou Local Vendor Supplies | 396 | 398.0 | -2.0 |
| Shanghai Distributor Supplies | 408 | 411.3 | -3.3 |
| Ningbo Distributor Supplies | 404 | 402.4 | 1.6 |
| Tianjin OEM Supplies | 404 | 416.0 | -12.0 |
| Qingdao Distributor Supplies | 407 | 415.7 | -8.7 |
| Suzhou Local Vendor Supplies | 418 | 411.9 | 6.1 |
| Hangzhou OEM Supplies | 430 | 411.1 | 18.9 |

Result: no supplier is really more late than the others. The largest gap (Hangzhou, 18.9 more late orders than expected) is within chance. The expected counts add up to 3,267.1 (rounding) and the differences to about zero.

#### Supplier table against real orders

| Supplier | Promised days (table) | Average real lead days | Real minus promised | Reliability score (table) | Late share (%) |
|---|---|---|---|---|---|
| Shenzhen OEM Supplies | 19 | 19.50 | 0.50 | 5 | 15.08 |
| Guangzhou Local Vendor Supplies | 10 | 10.52 | 0.52 | 5 | 15.03 |
| Shanghai Distributor Supplies | 16 | 16.53 | 0.53 | 4 | 14.98 |
| Ningbo Distributor Supplies | 18 | 18.50 | 0.50 | 3 | 15.17 |
| Tianjin OEM Supplies | 28 | 28.49 | 0.49 | 3 | 14.67 |
| Qingdao Distributor Supplies | 10 | 10.50 | 0.50 | 3 | 14.79 |
| Suzhou Local Vendor Supplies | 12 | 12.53 | 0.53 | 5 | 15.33 |
| Hangzhou OEM Supplies | 26 | 26.55 | 0.55 | 4 | 15.80 |

Reliability score grouped:

| Reliability score | Suppliers | Late orders | Orders | Late share |
|---|---|---|---|---|
| 5 | Shenzhen, Guangzhou, Suzhou | 1,214 | 8,015 | 15.15% |
| 4 | Shanghai, Hangzhou | 838 | 5,445 | 15.39% |
| 3 | Ningbo, Tianjin, Qingdao | 1,215 | 8,170 | 14.87% |

Result: the promised days in the suppliers table are supported by the orders for all 8 suppliers. The reliability score is not: the scores differ (3, 4, 5) but the real late share does not, and the group with the lowest score is not the latest. The orders add up to 21,630 and the late orders to 3,267.

---

## 4. Data Quality Notes

1. The 2,370 Cancelled orders have no received date and are left out. All 21,630 Received orders have all three dates.
2. The promised days in the suppliers table (`lead_time_days`) are supported: the real average wait is 0.49 to 0.55 days above them for every supplier.
3. The reliability score in the suppliers table is not supported by the orders and is not used as a measure. This KPI measures only delivery against the promised date. It does not say what the stored score is based on.
4. The median real wait equals the promised days for all 8 suppliers, and 80.41% of orders arrive exactly on the promised day. The average sits about half a day higher because of the late orders.
5. Late orders are spread evenly over 1 to 7 days late (447 to 484 orders per day) and no order is more than 7 days late. The earliest arrival is 3 days early.
6. The longest real wait is the promised days plus 7 for every supplier.
7. The `lead_time_days` column in the products table is not used (Week 3 finding: not supported by the order history).
8. Orders were placed from January 2019 to December 2024 (72 months, every supplier in every month). Received dates run to 28 January 2025 (KPI 11 work).
9. The per-supplier late share is based on about 2,700 orders each, so small gaps between suppliers are not evidence of a real difference (see the late share test).
10. Table values that are identical to two decimals are shown with the trailing zero in the document tables (for example 10.50); the raw file check prints them without it (10.5).

---

## 5. Business Interpretation

### Business question and answer

**Question:** which suppliers deliver late, how late, and how long do we really wait for each supplier?

**Answer:** no supplier is really later than the others. All 8 are late on about 15% of orders (1 in 7), and a late order is 1 to 7 days late. What differs between suppliers is how long they take: from about 10.5 days (Qingdao and Guangzhou) to about 28.5 days (Tianjin).

### Findings

- 84.90% of orders arrive on the promised day or earlier; 15.10% are late.
- The average order arrives 0.513 days after the promised day.
- The late share runs from 14.67% (Tianjin) to 15.80% (Hangzhou), and the late share test shows this gap is within chance (p-value 0.9702).
- Real lead time groups the suppliers: about 10 to 13 days (Qingdao, Guangzhou, Suzhou), about 17 to 20 days (Shanghai, Ningbo, Shenzhen), and about 27 to 28 days (Hangzhou, Tianjin).
- The longest wait for any supplier is its promised days plus 7.
- The reliability score stored in the suppliers table does not match the real deliveries.

### Recommended actions for the business owners

1. Procurement Manager: plan every order with the supplier's real lead time (promised days plus about half a day), not the promised days alone.
2. Procurement Manager: choose between suppliers by their real lead time and real late share, not by the stored reliability score.
3. Warehouse Manager: keep a safety margin of 7 days over the promised days for the slowest suppliers, Tianjin (28.49 days) and Hangzhou (26.55 days), because 7 days is the largest delay seen.
4. Supplier Relations Lead: raise the late-delivery rate (about 1 order in 7) with all 8 suppliers, because none is really worse than the others.
5. Operations Director: review this KPI every quarter and set a target for the late share, which is 15.10% now.

---

## Glossary

| Term | Meaning |
|---|---|
| Received order | A purchase order with status Received, which has a received date |
| Promised date / promised days | The `expected_delivery_date`, and the days between the order date and that date |
| Real lead time (real wait) | Received date minus order date, in days |
| Days late | Received date minus promised date; negative means the order arrived early |
| Late order | An order with days late above 0 |
| Late share | Late orders divided by Received orders, times 100 |
| On time or early | An order with days late of 0 or below; its share is 100 minus the late share |
| Median | The middle value: half the orders are at or below it |
| p-value | The chance of seeing differences this big if all suppliers were equally late; above 0.05 means the differences are within chance |
| Late orders if all equal | Each supplier's orders multiplied by the overall late share (15.10%) |
| Fast, medium and slow group | Reading labels used in this document for suppliers at about 10 to 13, 17 to 20 and 27 to 28 days; they are not tested limits |
| Second method | Check 5: the same numbers recomputed with a loop instead of a group-by |
| Raw file check | Check 8: three suppliers recalculated from the raw orders file |
| Supporting tests | Tests run while building the KPI; they are separate from the 8 numbered checks |
| po_h, po_l | Notebook names for the purchase orders header table and the purchase orders lines table |
| rec | Notebook name for the Received orders table |
| by_sup | Notebook name for the per-supplier table |
