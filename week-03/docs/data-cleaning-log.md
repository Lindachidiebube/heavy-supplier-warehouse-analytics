# Data Cleaning Log

**Project:** Heavy Supplier & Warehouse Analytics
**Sprint:** Week 3
**Purpose:** A record of every data problem found in the 12 source tables, how many rows it affects, the decision taken, and which KPIs it touches. Every KPI built from Week 4 onward follows these decisions.

**Rule used throughout:** the transaction tables (orders, order lines, invoices, payments) agree with each other and are the source of truth. Summary columns stored in the master tables (customers, products, inventory_master) do not match the transactions and are not used in any KPI. Where a KPI needs that information, it is worked out from the transactions.

---

## 1. How the data was checked

All checks were run in the notebook `Week3_ID_Uniqueness_Check` (Google Colab) on the 12 raw tables.

1. Key uniqueness on every ID column of all 12 tables
2. Orphan check on 18 links between tables
3. Column health: empty cells, negatives, zeros, minimum and maximum
4. Lead time: supplier table vs product table vs real days from purchase order dates
5. Date columns: conversion from text, range, and order rules
6. Text categories: every value listed, plus a spacing and capital-letter check on every text column
7. Cancelled orders against the stock ledger
8. Maths rules: line totals, GST, header totals, invoice vs sales order, purchase order totals, product margin, customer totals
9. Stock ledger running balance, step by step
10. Order lines against stock ledger entries, by order and by product
11. Cross-table consistency: customer, branch, payment terms
12. Payments against invoice totals and payment status

---

## 2. Problems found and decisions

### A. Duplicate keys

**1. `invoices.invoice_id` is not unique**
- Found: 18,033 rows but 17,836 unique IDs. 393 rows (196 duplicate groups, one of them a triplet) share an ID with another row. They are different, unrelated invoices that were given the same ID.
- Decision: not repaired, because no other field separates them. Rows with a duplicated ID are left out only where invoices are joined to payments. KPI 3 reads the invoice status directly and keeps all 18,033 rows. Bias check from KPI 2: the gap is at most 0.57 percentage points.
- KPIs affected: 2, 3, 4, 15, 16, 17.

**2. `payments.payment_id` is not unique**
- Found: 19,257 rows but 19,055 unique IDs. 403 rows (201 duplicate groups, one of them a triplet) share an ID. In KPI 4, 409 more payments were dropped because they pointed to excluded invoices.
- Decision: same rule as entry 1. The 208 "payment before invoice" records seen in Week 1 are false matches caused by duplicate invoice IDs and disappear after exclusion.
- KPIs affected: 2, 4, 16, 17.

### B. Stock ledger (`stock_ledger`)

**3. ADJUSTMENT movements**
- Found: 2,454 rows (1.03%) with `reference_type` ADJ and `reference_id` like ADJ-1519. They have no purchase or sales order, which is why the orphan check flagged them. Spread evenly over the 6 branches (386 to 429 each). The quantity has no sign, but the running balance shows they go both ways: 1,219 add and 1,222 subtract (of 2,441 testable).
- Decision: left out of KPI 1 (corrections, not buying or selling). Included in KPI 8, with direction read from the change in running balance.
- KPIs affected: 1, 8, 21.

**4. Ledger rows that belong to cancelled orders**
- Found: 1,275 OUT rows tied to cancelled sales orders and 1,542 IN rows tied to cancelled purchase orders, 2,817 rows in total (1.19%). The running balance counts them. They hold 13,234 units OUT and 248,556 units IN, about 1.2% of each.
- Effect on KPI 1: average ratio 18.14 with them and 18.132 without. Largest change for any product: 0.39%. KPI 1 stands as submitted.
- Decision: flagged, not removed. The data cannot show whether goods really moved. KPI 8 reports stock both ways and shows the difference.
- KPIs affected: 1, 8, 9, 21.

**5. About 10% of order lines never reached the ledger**
- Found: the ledger has one row per order line, but about 1 in 10 lines is missing, at random. Delivered sales lines 117,656 vs 105,890 OUT rows (90.0%). Received purchase lines 140,077 vs 126,069 IN rows (90.0%). Units: 1,234,939 on delivered sales lines vs 1,111,598 in the ledger; 22,387,123 on received purchase lines vs 20,149,239 in the ledger (both about 10% lower).
- Detail: 162 delivered sales orders and 239 received purchase orders have no ledger entry at all, almost all one-line orders (about 10% of one-line orders). 9.0% of order-product pairs have no entry. 2.2% have an entry with fewer units, always because the product is on 2 or more lines and only some were recorded. The ledger never contains a movement that is not on an order.
- Decision: the order lines (Delivered and Received orders only) are the complete source for demand and purchase quantities. The ledger is used for balances, with this caveat stated. KPI 1 is not affected, because both sides are about 10% low (ratio 18.13 from order lines and 18.13 from the ledger).
- KPIs affected: 5, 7, 8, 9, 18, 19, 21.

**6. `inventory_master.current_stock`**
- Found: 88,853 to 121,013 per product and branch, against opening stock of 80 to 300. It equals the final running balance in the ledger, so it is not a copy error. The reason is the volume of stock coming in: 20,397,795 units IN against 1,124,832 units OUT, about 18 times more.
- Decision: not used as a KPI input. KPI 8 rebuilds stock from the ledger and from the order lines and compares them. The large surplus is treated as a business finding (excess stock).
- KPIs affected: 6, 8, 9, 10, 12, 21.

### C. Cancelled orders

**7. Cancelled sales orders keep their lines**
- Found: 1,967 of 20,000 sales orders are Cancelled. They have no invoices, but all 12,746 of their lines remain in `sales_orders_lines`.
- Decision: every sales KPI filters to `order_status = Delivered`.
- KPIs affected: 5, 7, 9, 14, 15, 18.

**8. Cancelled purchase orders**
- Found: 2,370 of 24,000 purchase orders are Cancelled. They are exactly the orders with an empty `received_date` (the only empty cells in the dataset). Their 15,418 lines remain in `purchase_orders_lines`.
- Decision: not an error. Purchase KPIs filter to `po_status = Received`.
- KPIs affected: 13, 19.

### D. Summary columns that do not match the transactions

**9. `customers.customer_since`** is later than the customer's real first order for 293 of 500 customers. Not used. The first order comes from `sales_orders_header`.

**10. `customers.last_purchase_date`** is earlier than the customer's real last order for 467 of 500 customers (the earliest value is in 2015, while every customer has orders from 2019). Not used. Recency comes from `sales_orders_header`.

**11. `customers.total_purchase_value`** differs by more than 1% from the real delivered order total for all 500 customers. Not used. Customer value comes from the invoices and delivered orders.
- KPIs affected by 9 to 11: 14, 15, 16, 17.

**12. `products.last_purchase_date`** runs to 2025-04-27, after the transaction data ends. Not used.

**13. `products.lead_time_days`** (4 to 40 days) is not supported by purchase history: the real average is 17.7 to 18.1 days for every product. Not used.
- The supplier table agrees with history. `suppliers.lead_time_days` (10 to 28 days) equals the promised days (expected delivery date minus order date) for all 8 suppliers, and real deliveries take about 0.5 day longer. Across 21,630 received orders the real average is 17.9 days (minimum 7, maximum 35), against 17.4 promised.
- Decision: lead time comes from the supplier table or from the real purchase order dates.
- KPIs affected: 13, 19.

**14. `products.margin_percentage`** mixes two definitions. For 29 of 30 products it is a markup on cost, (price - cost) / cost. For P002 Engine Oil Filter OF-90 (cost 450, price 720, column 37.3) it is close to a margin on price (37.5%).
- Decision: not used. Margin is recomputed for every product as (unit_price - unit_cost) / unit_price.
- KPIs affected: 7, 10.

### E. Format

**15. `branches.warehouse_capacity` is text** such as "45230 sqft" in all 6 branches, same unit, no gaps. Decision: remove " sqft" and convert to a number. KPIs affected: 12.

**16. All date columns are stored as text** (12 columns in 7 tables). All convert to real dates with 0 failures. The data runs from 2019-01-01 to 2025-03-20. Decision: convert before any date arithmetic.

**17. `products.uom` mixes units** (24 piece, 5 set, 1 cartridge). Fine within one product. Decision: do not add quantities across different products as if they were the same unit.

---

## 3. Checked and found clean

- Every other primary key is unique with no empty IDs: branch_id, customer_id, product_id, po_id (header), so_id (header), movement_id, supplier_id. `inventory_master` has 180 rows (30 products x 6 branches).
- All 18 links between tables have 0 real orphans.
- No negative or zero values in any numeric column. Quantities, prices and rates are in sensible ranges.
- Date order rules: 0 violations (due date before invoice date, delivery or receipt before order date, invoice before its sales order or its delivery).
- Text columns: no spacing or capital-letter problems anywhere. Order, payment and movement categories each have a small, clear set of values.
- Maths: line totals, GST, grand totals, header totals vs lines, invoice vs sales order totals, and purchase order totals all agree with 0 errors.
- Consistency: each invoice has the same customer and branch as its sales order; each order has the same payment terms and branch as the customer record; stock movements are logged at the branch of their order.
- Ledger running balance: every entry changes the balance by exactly its quantity, and the file is in date order.
- Invoice status: for the 17,640 invoices with a clean ID, every Paid invoice is fully paid, every Partially Paid invoice is part paid, and every Unpaid invoice has no payment. No overpayments.
