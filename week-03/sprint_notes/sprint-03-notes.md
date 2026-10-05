# Sprint 3 Notes (Week 3)

**Project:** Heavy Supplier & Warehouse Analytics
**Role:** Data Analyst (solo)
**Phase:** Phase 1, Data Foundation and Preparation (close-out), plus KPI 5

## Sprint Goal

- Close out Phase 1 properly: propose the full KPI roadmap with business questions, test every ID column of all 12 tables, record every cleaning decision, map how the tables join, and document every column and every created field.
- Build and fully verify KPI 5 (Product Sales Velocity), the first KPI that uses sales order lines and purchase order lines together.
- Give every later sprint a trusted base: one place that says which columns can be used, which cannot, and why.

## Work Completed

### Step 1: KPI proposal and business questions (`week-03/docs/KPI-proposal.md`)
- Finalised a roadmap of 22 KPIs across Phases 1 to 4 (KPIs 1-13 from the earlier plan, new KPIs 14-22 for customers, forecasting and risk).
- Week 8 Customer Analytics: KPI 14 RFM Segmentation, KPI 15 Historical Customer Value, KPI 16 Customer Payment Risk, KPI 17 Churn by Payment Behavior.
- Week 9 Forecasting: KPI 18 Product Demand Forecast, KPI 19 Stockout Risk / Reorder Point.
- Week 10 Risk, Anomaly and Control: KPI 20 Inventory Movement Anomaly Detection, KPI 21 Stock Shrinkage Rate, KPI 22 Operational Risk Score. Dashboards moved to Week 11 so all 22 KPIs exist before they are built.
- Applied the Week 3 findings to the proposal: demand comes from Delivered sales lines, not the stock ledger; KPI 5 filters Delivered orders; KPI 13 uses Received orders only; customer KPIs are built from transactions; margin is recomputed.

### Step 2: Key-uniqueness and link checks on all 12 tables (notebook `Week3_ID_Uniqueness_Check`)
- Every real primary key is unique with no empty values. The only duplicate keys are `invoices.invoice_id` (18,033 rows, 17,836 unique, 393 rows in repeated groups) and `payments.payment_id` (19,257 rows, 19,055 unique, 403 rows in repeated groups).
- `inventory_master` has 180 rows = 30 products x 6 branches. `(product_id, branch_id)`, `(so_id, line_number)` and `(po_id, line_number)` are unique.
- 18 cross-table links tested: 0 real orphans. `stock_ledger.reference_id` has 2,454 rows (1.03%) matching no purchase or sales order; these are exactly the ADJUSTMENT rows (ADJ-n references), by design.
- Every Delivered order has an invoice and no invoice belongs to a non-Delivered order.

### Step 3: Data cleaning log (`week-03/docs/data-cleaning-log.md`)
- Duplicate invoice and payment IDs are not repaired; affected rows are excluded only where invoices are joined to payments (KPI 4, 15, 16, 17). KPI 3 keeps all rows. Bias check maximum gap 0.57 percentage points.
- Stock ledger (237,230 rows): ADJUSTMENT rows (2,454) stay out of KPI 1 and are used with their direction in KPI 8 (1,219 add, 1,222 subtract of 2,441 testable).
- 2,817 ledger rows belong to cancelled orders (1,275 OUT, 1,542 IN, about 1.2% of units) and are counted in running_balance. KPI 1 ratio is 18.14 with them and 18.132 without, so KPI 1 stands. Flagged, not removed; KPI 8 will report both ways.
- About 10% of order lines never reached the ledger, at random (delivered lines 117,656 vs 105,890 OUT rows; received lines 140,077 vs 126,069 IN rows). Order lines are therefore the complete source for demand and purchases.
- Cancelled sales orders (1,967) and cancelled purchase orders (2,370, exactly the empty `received_date` rows) keep their lines in the lines tables: filter to Delivered or Received.
- Master-table summary columns that do not match the transactions and are not used: `customers.customer_since`, `customers.last_purchase_date`, `customers.total_purchase_value`, `products.last_purchase_date`, `products.lead_time_days`. `products.margin_percentage` mixes definitions and is recomputed as (unit_price - unit_cost) / unit_price. `current_stock` is inflated but equals the final ledger running_balance.
- `branches.warehouse_capacity` is text such as "45230 sqft" (strip " sqft"). All 12 date columns are text and convert with 0 failures. `products.uom` mixes piece (24), set (5) and cartridge (1), so quantities are never added across products.
- Clean results: no negatives or zeros in any numeric column; date-order rules 0 violations; maths rules (line totals, GST, headers vs lines, invoices vs orders, PO totals) 0 errors; payment_status agrees with payment sums for all 17,640 clean invoices.

### Step 4: Table join map (`week-03/docs/table-join-map.md`)
- Tables and keys for all links plus a Mermaid diagram that renders on GitHub, compared with the Week 1 ERD.
- Differences from the ERD recorded: the ERD draws `stock_ledger.reference_id` as two lines (PO and SO) and does not draw `customers.branch_id`; its customers table shows a `customer_name` column that is not in `customers.csv`.
- Verified in Colab: all 17 direct links including `customers.branch_id` have 0 empty values and 0 orphans.

### Step 5: KPI 5 Product Sales Velocity (`week-03/docs/KPI-05-Product-Sales-Velocity.md`, notebook `Week3_KPI05_Product_Sales_Velocity`)
- Five metrics: M1 average units sold per month (headline), M2 trend (2022-24 vs 2019-21), M3 steadiness (bumpiness %), M4 bought per 1 sold, M5 Slow / Medium / Fast label.
- Scope: 18,033 Delivered orders, 117,656 lines, 1,234,939 units sold; 21,630 Received purchase orders, 140,077 lines, 22,387,123 units bought; 30 products, 6 branches, 72 months (2019-01 to 2024-12).
- Products sell almost alike: 548-593 units a month, each 3.20%-3.46% of units. Sales fell 3.4% (628,201 in 2019-21 vs 606,738 in 2022-24): 6 products up, 24 down, from Alternator ALT-500 (+4.3%) to Light Assembly LA-18 (-9.5%).
- Steadiness: 13.9% (Pressure Sensor PS-90) to 19.1% (Fuel Injector FI-220), median 17.06%.
- Overall 18.1 units bought per 1 sold; products 17.4 to 19.2, branches 15.0 (Chennai) to 22.0 (Hyderabad).
- Buying does not follow selling product by product: monthly correlation 0.28, product-level correlation -0.36 over 30 products. Buying runs near a steady 311,000 units a month and 3.66M-3.80M units per branch whatever the sales.
- Velocity depends on the branch, not the product: 65 of 180 product-branch pairs have a ratio above 20, all in Hyderabad, Delhi and Pune, the three lowest-selling branches. Labels by branch (Slow / Medium / Fast): Ahmedabad 0/24/6, Chennai 0/0/30, Delhi 13/17/0, Hyderabad 29/1/0, Kolkata 0/6/24, Pune 18/12/0. Idler Wheel IW-55 is both the fastest pair (Chennai, 8,861 units) and the slowest (Hyderabad, 5,081 units).
- Verification: all 8 checks run and passed (2 and 7 with notes). The coverage statement in the write-up lists exactly what each check covered, including that Check 3 tested combinations with exactly 2 lines and that Check 8 rebuilt 3 product-branch pairs plus trend for 2 products and steadiness for 2 products from the raw files.

### Step 6: Feature list (`week-03/docs/feature-list.md`)
- Feature engineering record built from the real code of the KPI 1-5 notebooks: every created field with its meaning and formula (for example `is_late`, `days_late`, stock ratio, `units_sold`, `avg_units_per_month`, `bought_per_1_sold`, bumpiness, `velocity_group`), the check-only columns, and planned features not yet built (recomputed margin, numeric warehouse capacity, reconstructed stock balance). KPI 2 and KPI 3 create no new columns.

### Step 7: Data dictionary (`week-03/docs/data-dictionary.md`)
- Built from the data itself because the Fields Documentation file was not available to download. A Colab profile cell read all 12 CSV files and wrote one profile of 134 columns (branches 13, customers 15, inventory_master 8, invoices 10, payments 5, products 23, purchase_orders_header 10, purchase_orders_lines 9, sales_orders_header 11, sales_orders_lines 9, stock_ledger 9, suppliers 12).
- Each column has type, empty count, distinct count, range, example values and a meaning marked as inferred from the data. Checked by code: 134 of 134 columns have a meaning. The only empty column in all 12 files is `purchase_orders_header.received_date` (2,370 rows, all Cancelled).
- Includes a "columns that need care" section and the rules checked in Week 3.

## Blockers & Resolutions

- **Fields Documentation not downloadable:** CadetX confirmed by email that it cannot be downloaded. Resolved by building the data dictionary from the data itself and marking meanings as inferred.
- **Colab found 0 files:** Drive was not mounted in that session. Resolved by mounting Drive first, then the profile found 12 of 12 files.
- **KeyError `branch_id` in KPI 5:** the sales lines table has no branch column. Resolved by merging branch from `sales_orders_header` on `so_id` and checking the row count stayed 117,656.
- **Pair-level gaps found in Check 2:** 4 sales pairs and 3 purchase pairs have one or two empty months. Resolved by testing that the branch was active in each empty month (quiet month, not lost data) and by counting an empty month as 0.
- **Label rounding tie:** P008 Kolkata sat exactly on a cut-off after rounding to 1 decimal and was labelled Medium. Resolved by labelling from unrounded averages (groups became exactly 60 / 60 / 60; one label changed to Fast).
- **Earlier conclusion corrected:** "over-buying is evenly spread" held across products but not across branches. Corrected after the branch split.
- **Verification coverage gap:** Checks 3 and 6 had only been run on sales lines. Closed by running them on purchase lines (largest line about 1.9x the average; repeated-line test 0.40% vs 0.36% chance).
- **Team removed for inactivity:** work continues solo; CadetX confirmed requirements stay the same.

## Next Sprint Focus (Week 4)

- KPI 8 Reconstructed Stock Balance: rebuild stock from opening stock, received and delivered order lines, and ADJUSTMENT rows with their direction; compare with the ledger-based balance, with and without cancelled-order rows.
- KPI 6 Dead Stock Identification, using the Week 3 rule that demand comes from Delivered sales lines.
- Apply the same 8 checks, each run on both sides (sales and purchases) when a metric uses both.
