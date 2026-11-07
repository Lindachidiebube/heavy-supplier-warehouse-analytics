# KPI 10: Capital Tied Up in Excess Stock

Project: Heavy Supplier & Warehouse Analytics | Week 6 | Notebook: Week6_KPI10_Capital_Tied_Up_in_Excess_Stock

## Summary

- At the end of the data (end of January 2025) the 180 product-branch pairs hold 21,185,215 units of stock. Taking 3 months of sales as the stock the business needs, 21,135,890 units (99.77%) are excess.
- The money tied up in that excess is 294,576,746,335, valued at the cost paid on received purchases. This is 99.77% of the stock value (295,255,911,357) and 94.41% of the 312,027,429,108 spent on received purchases. It is money held in stock, not a loss.
- Every one of the 180 pairs has excess under every rule tested (1, 3, 6 and 12 months of sales and the two company limits). The excess share stays between 99.07% and 99.92%, so the choice of rule hardly changes the finding.
- The money follows the price of the product. P015 Engine Block (61,841,619,704) and P014 Transmission Assembly (45,331,993,198) hold 36.38% of the excess; the top 5 products hold 58.58%. The six branches are close: 15.94% (KOL001) to 16.95% (CHN001).
- Measured at each year-end, the excess value grew from 46,509,105,934 (2019) to 291,825,795,283 (2024), about 49 billion a year.
- Understock: no pair was below a stock level at any month-end. A daily test found 14 pairs below a level on 42 pair-days, all between 2 and 25 January 2019.
- All 8 verification checks were run on the exact tables KPI 10 uses, and every claim was tested at the point it appeared. The figures tie to KPI 6, KPI 8 and KPI 9.

## 1. KPI Definition

| # | Metric | Plain meaning | Result |
|---|--------|---------------|--------|
| M1 | Excess units | Stock above the stock needed for 3 months of sales, per pair, never below zero, added over the 180 pairs | 21,135,890 units |
| M2 | Excess value (capital tied up) | Excess units multiplied by the received cost per unit of the product | 294,576,746,335 |
| M3 | Excess share of stock | Excess units divided by stock units | 99.77% |
| M4 | Year-end trend | Excess value at each year-end, using that year's own sales for the 3-month need | 46,509,105,934 (2019) to 291,825,795,283 (2024) |

The branch table and the product table are supporting tables of M2, not extra metrics.

**Business question:** How much money is held in stock that the business does not need, and where does it sit?

**Why this metric:** Stock that is far above what sales need ties up money that could be used elsewhere. KPI 6 showed every pair holds 72 to 172 years of cover and KPI 9 showed turnover falling about 11 times; KPI 10 puts a money figure on that.

**What the figure means:** Capital tied up is money held in stock. It is not a loss. The cost basis is the received cost (see Methodology). The stock figure comes from order records (opening stock, received lines, delivered lines and stock corrections), not from a physical count. The data names no currency, so amounts are written as money without a currency name.

**Scope:** One pair = one product in one branch (30 products x 6 branches = 180 pairs). Stock is the order-line stock at the end of the data (end of January 2025), decided in KPI 8 as the headline stock; the ledger stock is shown beside it as a sensitivity. The needed stock is measured from sales in the 12-month window 15 January 2024 to 13 January 2025 (18,889 delivered lines, 197,300 units).

## 2. Methodology

### Sources and scoping

| Table | Rows x columns | Use in KPI 10 |
|-------|----------------|---------------|
| ledger (stock_ledger) | 237,230 x 9 | Adjustments, ledger stock, ledger daily test |
| inv (inventory_master) | 180 x 8 | Opening stock, reorder level, safety stock, maximum stock per pair |
| po_h (purchase_orders_header) | 24,000 x 10 | Order status and received date for purchase lines |
| po_l (purchase_orders_lines) | 155,495 x 9 | Received quantity and cost |
| so_h (sales_orders_header) | 20,000 x 11 | Order status, branch and delivery date for sales lines |
| so_l (sales_orders_lines) | 130,402 x 9 | Delivered quantity |
| products | 30 x 23 | Product names, unit cost (sensitivity), levels |
| branches | 6 x 13 | Branch list |

All eight counts equal the verified Week 3 to Week 5 counts.

- **sd** = Delivered sales lines joined to their order headers: 117,656 rows, 1,234,939 units.
- **pr** = Received purchase lines joined to their order headers: 140,077 rows, 22,387,123 units.
- Cancelled orders are left out of sd and pr (lines of cancelled orders are kept in the line tables, so the filter on Delivered and Received is applied).
- **Stock** = opening stock + received - delivered + stock corrections (adjustments), per pair, at the end of the data. This is the order-line stock of KPI 8: 21,185,215 units.
- **Needed stock** = 3 months of sales per pair = units sold in the window / 12 x 3.
- **Excess units** = stock minus needed stock, never below zero. **Excess value** = excess units x received cost per unit, where the received cost per product = line_total / quantity summed over its Received purchase lines.

### Step 1: Setup

The eight tables were loaded with the counts shown above.

### Step 2: Small data check (what the two tables hold)

Inventory master (inv), 180 rows, every column counted 180:

| Column | Mean | Std | Min | 25% | Median | 75% | Max |
|--------|------|-----|-----|-----|--------|-----|-----|
| opening_stock | 183.2 | 63.4 | 80 | 129 | 178.5 | 234.5 | 300 |
| reorder_level | 64.6 | 24.1 | 24 | 44 | 62 | 82.2 | 119 |
| safety_stock | 45.2 | 16.9 | 16 | 31 | 43.5 | 58 | 85 |
| max_stock | 321.8 | 116.6 | 131 | 230 | 310.5 | 415.5 | 582 |
| current_stock | 107,240.4 | 5,341.1 | 88,853 | 103,487.5 | 107,790.5 | 110,779.5 | 121,013 |

current_stock is the inflated field known since Week 1 (88,853 to 121,013 against an opening stock of 80 to 300); it is not used.

Products table, 30 rows, every column counted 30:

| Column | Mean | Std | Min | 25% | Median | 75% | Max |
|--------|------|-----|-----|-----|--------|-----|-----|
| unit_cost | 14,585 | 21,478.8 | 180 | 957.5 | 7,600 | 18,312.5 | 95,500 |
| reorder_level | 39.7 | 48.4 | 3 | 8.5 | 13.5 | 57.5 | 200 |
| max_stock_level | 104.8 | 124.0 | 8 | 22.8 | 40 | 150 | 500 |
| lead_time_days | 17.0 | 9.3 | 4 | 9 | 18 | 22 | 40 |

unit_price: mean 18,593.7, min 310, median 10,250, max 119,000. safety_stock: mean 20.1, min 2, median 8, max 100. No empty or zero values in any of these columns.

### Step 3: Two sets of maximum levels

The inventory master (per pair) and the products table (per product) each hold a maximum stock level, a reorder level and a safety stock. They are different sets: inventory master means are 321.8 / 64.6 / 45.2, products table means are 104.8 / 39.7 / 20.1. Which set the company treats as official is not in the data, so both are used as sensitivity rules. The two maximum levels are not a scaled copy of each other:

| Product | products max_stock_level | Mean inv max_stock over its 6 branches | Ratio inv / products |
|---------|--------------------------|----------------------------------------|----------------------|
| P015 | 8 | 305.3 | 38.16 |
| P014 | 12 | 319.7 | 26.64 |
| P010 | 15 | 254.0 | 16.93 |
| P019 | 15 | 308.3 | 20.55 |
| P023 | 18 | 263.8 | 14.66 |
| P021 | 20 | 360.8 | 18.04 |
| P004 | 20 | 308.5 | 15.42 |
| P024 | 22 | 331.2 | 15.05 |
| P009 | 25 | 327.2 | 13.09 |
| P011 | 25 | 329.3 | 13.17 |
| P030 | 25 | 288.2 | 11.53 |
| P018 | 30 | 401.5 | 13.38 |
| P006 | 30 | 285.8 | 9.53 |
| P022 | 30 | 337.0 | 11.23 |
| P025 | 40 | 355.7 | 8.89 |
| P001 | 40 | 391.2 | 9.78 |
| P029 | 60 | 243.7 | 4.06 |
| P016 | 60 | 288.7 | 4.81 |
| P005 | 80 | 351.3 | 4.39 |
| P028 | 100 | 346.2 | 3.46 |
| P012 | 120 | 306.0 | 2.55 |
| P003 | 150 | 366.2 | 2.44 |
| P017 | 150 | 361.3 | 2.41 |
| P002 | 200 | 329.8 | 1.65 |
| P007 | 200 | 421.5 | 2.11 |
| P013 | 200 | 247.3 | 1.24 |
| P020 | 250 | 272.0 | 1.09 |
| P008 | 300 | 392.5 | 1.31 |
| P027 | 400 | 265.3 | 0.66 |
| P026 | 500 | 293.3 | 0.59 |

The ratio runs from 0.59 to 38.16 (median 9.21). The products-table maximum follows the kind of product (expensive P015 Engine Block = 8, P014 Transmission Assembly = 12; cheap P026 Grease Cartridge = 500, P027 O-Ring Set = 400); the inventory-master maximum averages 244 to 422 per product whatever the product.

### Step 4: Column check

Columns were printed before any join to avoid wrong names: po_h (po_id, supplier_id, branch_id, order_date, expected_delivery_date, received_date, po_status, total_cost, total_gst_amount, grand_total); po_l (po_id, line_number, product_id, quantity, unit_cost, gst_rate, line_total, gst_amount, line_grand_total, with no branch column); so_h (so_id, customer_id, branch_id, order_date, delivery_date, order_status, payment_terms, total_order_value, total_gst_amount, grand_total, sales_channel); so_l (so_id, line_number, product_id, quantity, unit_price, gst_rate, line_total, gst_amount, line_grand_total); ledger (movement_id, product_id, branch_id, movement_type, movement_date, quantity, reference_type, reference_id, running_balance). The purchase status column is po_status (value Received) and the sales one is order_status (value Delivered); lines take branch_id from their headers.

### Step 5: Delivered sales lines (sd) and Received purchase lines (pr)

| Table | Rows | Units |
|-------|------|-------|
| sd (Delivered) | 117,656 | 1,234,939 |
| pr (Received) | 140,077 | 22,387,123 |

Both equal the verified Week 3 to Week 5 figures.

### Step 6: Ledger chain re-test and the pair table

| Ledger movement | Rows | Rows where the change equals the quantity (with sign) |
|-----------------|------|------------------------------------------------------|
| IN | 127,611 | 127,611 (+quantity) |
| OUT | 107,165 | 107,165 (-quantity) |
| ADJUSTMENT | 2,454 | 2,454 (+/- quantity) |

The table **pair** has 180 rows (index product_id and branch_id). Opening stock 32,969 + received 22,387,123 - delivered 1,234,939 + adjustments 62 = stock_end 21,185,215 (my arithmetic matches the table). No pair has stock below zero. stock_end per pair: smallest 98,642, median 117,900.5, largest 132,967. Even the smallest pair holds about 170 times the largest inventory-master maximum (582) (my arithmetic: 98,642 / 582 = 169.5).

### Step 7: Sales window and years of cover

Window 15 January 2024 to 13 January 2025 (the latest delivery date in sd is 2025-01-13), as in KPI 6.

| Measure | Value |
|---------|-------|
| sd rows in the window | 18,889 |
| Units in the window | 197,300 |
| Pairs | 180 |
| Pairs with no sales in the window | 0 |
| Units sold per pair (mean / smallest / largest) | 1,096.11 / 703 / 1,548 |
| Years of cover = stock_end / sold_12m (mean / smallest / largest) | 110.46 / 72.0 / 171.9 |

These tie to KPI 6 (mean cover 110.46, range 72 to 172, lowest 12-month sales 703, mean 1,096.11 units per pair; my arithmetic 197,300 / 180 = 1,096.11).

### Step 8: Excess units under six rules for the needed stock

Total stock 21,185,215 units, 180 pairs.

| Rule for the needed stock | Needed units | Excess units | Excess share of stock | Pairs with excess |
|---------------------------|--------------|--------------|-----------------------|-------------------|
| 1 month of sales | 16,442 | 21,168,773 | 99.92% | 180 |
| **3 months of sales (headline)** | 49,325 | 21,135,890 | 99.77% | 180 |
| 6 months of sales | 98,650 | 21,086,565 | 99.53% | 180 |
| 12 months of sales | 197,300 | 20,987,915 | 99.07% | 180 |
| Company limit, inventory master max_stock | 57,916 | 21,127,299 | 99.73% | 180 |
| Company limit, products table max_stock_level (per pair) | 18,870 | 21,166,345 | 99.91% | 180 |

The predicted shares (99.92 / 99.77 / 99.53 / 99.07 / 99.73 / 99.91) all held. Every pair has excess under every rule. Moving from 1 month to 12 months of sales changes the excess by 180,858 units (about 0.85% of stock; my arithmetic).

Why 3 months is the headline: the real supplier lead time is about 18 days (Week 3), so 3 months of sales is a generous allowance and the excess is a conservative (lower) figure. The other five rules are the sensitivity.

### Step 9: Money

| Measure | Received cost (headline) | Products-table cost (sensitivity) |
|---------|--------------------------|-----------------------------------|
| Stock value | 295,255,911,357 | 308,895,127,500 |
| Excess value | 294,576,746,335 | 308,184,841,342 |
| Excess value as a share of stock value | 99.77% | 99.77% |

Products with a received cost: 30. Received cost per unit: smallest 232.45 (P026), largest 89,239.25 (P015), equal to KPI 9. The two cost sets differ by 13,608,095,007 in excess value (about 4.6% of the received-cost figure; my arithmetic).

### Step 10: By branch and by product (received cost, 3-month rule)

| Branch | Stock value | Excess value | Share of excess % |
|--------|-------------|--------------|-------------------|
| AHM001 | 48,651,160,842 | 48,543,863,142 | 16.48 |
| CHN001 | 50,072,886,813 | 49,935,378,833 | 16.95 |
| DEL001 | 49,928,692,929 | 49,823,387,369 | 16.91 |
| HYD001 | 49,509,223,586 | 49,413,003,293 | 16.77 |
| KOL001 | 47,098,343,177 | 46,967,415,811 | 15.94 |
| PUN001 | 49,995,604,011 | 49,893,697,886 | 16.94 |

The branch excess values add up to 294,576,746,335 (the headline). The shares add to 99.99 because of rounding (my arithmetic). The highest branch (CHN001) is about 1.06 times the lowest (KOL001).

Top 5 products by excess value:

| Product | Excess value |
|---------|--------------|
| P015 Engine Block EB-600 | 61,841,619,704 |
| P014 Transmission Assembly TA-800 | 45,331,993,198 |
| P021 Boom Cylinder BC-400 | 24,247,416,699 |
| P004 Track Chain TC-45 | 20,808,955,672 |
| P019 Swing Motor SWM-350 | 20,330,241,141 |

The top 5 hold 58.58% of the excess. Smallest 5 products:

| Product | Excess value |
|---------|--------------|
| P002 Engine Oil Filter OF-90 | 390,692,855 |
| P008 Bucket Tooth BT-10 | 299,160,095 |
| P017 Fan Belt FB-30 | 280,853,016 |
| P027 O-Ring Set OR-50 | 249,384,614 |
| P026 Grease Cartridge GC-5 | 160,051,742 |

Total spent on received purchases: 312,027,429,108; the excess value is 94.41% of it. The predicted pattern held: each branch about 16 to 17% and P015 and P014 on top. P015 is about 21.0% and P014 about 15.4% of the excess (my arithmetic), together 36.38% (tested in Check 6). The top-5 excess products are the same five products as the top 5 by revenue in KPI 7 (P015, P014, P021, P019, P004; the last two swap order), because units sold are about equal for all products, so money follows price. The top product is about 386 times the smallest (my arithmetic: 61,841,619,704 / 160,051,742).

### Step 11: Year-end trend (M4)

Each year uses its own calendar-year sales for the 3-month need and the same received cost per unit.

| Year-end | Stock units | Excess units | Excess value |
|----------|-------------|--------------|--------------|
| 2019 | 3,384,660 | 3,333,513 | 46,509,105,934 |
| 2020 | 6,960,247 | 6,907,449 | 96,005,272,745 |
| 2021 | 10,529,943 | 10,478,209 | 145,745,513,777 |
| 2022 | 14,009,405 | 13,958,726 | 194,159,992,126 |
| 2023 | 17,557,387 | 17,505,992 | 244,028,606,230 |
| 2024 | 20,993,633 | 20,943,492 | 291,825,795,283 |

All six year-end stock totals equal the KPI 9 totals exactly, and the rebuild gives 21,185,215 at the end of January 2025. My arithmetic on the table: the excess value rises by 49,496,166,811 (2020), 49,740,241,032 (2021), 48,414,478,349 (2022), 49,868,614,104 (2023) and 47,797,189,053 (2024), an average of 49.06 billion a year and no sign of slowing. After the first year it is already 46.5 billion (about 15.8% of the 2024 figure). The excess share of stock units is 98.49% (2019), 99.24% (2020), 99.51% (2021) and 99.76% (2024). The needed units are about 51,000 to 53,000 each year (2019: 204,586 / 4 = 51,146.5). The difference between the 2024 year-end value (291,825,795,283) and the headline (294,576,746,335) is 2,750,951,052 and is fully explained in Check 7.

### Step 12: Month-end stock and understock against the inventory-master levels

Month-end stock was rebuilt for every pair from 2019-01 to 2025-01: 180 pairs, 73 months, 13,140 pair-months (180 x 73). Total stock at the end of January 2025 is 21,185,215 (expected 21,185,215). The smallest month-end stock of any pair is 67 (as in KPI 9 Check 2). Pair-months below the inventory-master reorder_level: 0 (pairs affected 0). Pair-months below the inventory-master safety_stock: 0 (pairs affected 0).

### Step 13: Month-end understock against the products-table levels

The products table holds a second set of levels (reorder_level 3 to 200, safety_stock 2 to 100). Pair-months below the products-table reorder_level: 0 (pairs affected 0). Pair-months below the products-table safety_stock: 0 (pairs affected 0). The closest any pair came to its inventory-master reorder level (stock / reorder_level) is 1.91. The 5 lowest month-end stocks are all in 2019-01: P016 HYD001 67, P004 HYD001 132, P011 AHM001 149, P025 HYD001 190, P023 HYD001 230. Result: no understock at any month-end under either set of levels. Decision agreed on 5 November 2026: understock is a test inside KPI 10 (Steps 12 to 15); KPI 19 keeps the forward-looking reorder point.

### Step 14: Daily understock test

Month-end stock can miss short dips inside a month, so the stock was rebuilt for every day on which a pair had a movement. The month-end test was chosen first because it ties to KPI 9; the daily test is the stricter test of "ever below the level". Dates in pr (received_date) and in the ledger (movement_date) are text and sd (delivery_date) is a real date; all three were converted with pd.to_datetime before the rebuild (see Data Quality Notes, note 4).

| Measure | Result |
|---------|--------|
| Event rows (140,077 received lines + 117,656 delivered lines + 2,454 adjustments) | 260,187 |
| Pair-days | 175,335 |
| Pairs whose last daily balance equals stock_end | 180 of 180 |
| Lowest end-of-day stock of any pair | 0.0 |
| Pairs that dipped below the inventory-master reorder_level / safety_stock | 5 / 1 |
| Pairs that dipped below the products-table reorder_level / safety_stock | 11 / 6 |

Month-end said 0 breaches; the daily test shows short dips inside a month for 5 to 11 pairs, and one pair reached exactly 0.

### Step 15: The dips, pair by pair

Pair-days below at least one of the four levels: 42, in 14 pairs. All dates are between 2019-01-02 and 2019-01-25; none from February 2019 onward. "days" counts movement days; "highest level" is the highest of the four levels.

| Pair | Days | First day | Last day | Lowest stock | Highest level |
|------|------|-----------|----------|--------------|---------------|
| P004 KOL001 | 1 | 01-12 | 01-12 | 37 | 44 |
| P005 CHN001 | 1 | 01-16 | 01-16 | 62 | 68 |
| P008 PUN001 | 3 | 01-03 | 01-12 | 34 | 120 |
| P013 PUN001 | 6 | 01-04 | 01-15 | 19 | 80 |
| P020 DEL001 | 2 | 01-11 | 01-17 | 67 | 90 |
| P020 PUN001 | 1 | 01-07 | 01-07 | 75 | 90 |
| P023 HYD001 | 2 | 01-09 | 01-12 | 22 | 31 |
| P026 CHN001 | 5 | 01-07 | 01-20 | 73 | 200 |
| P026 DEL001 | 2 | 01-06 | 01-12 | 94 | 200 |
| P026 HYD001 | 7 | 01-07 | 01-25 | 148 | 200 |
| P027 AHM001 | 2 | 01-05 | 01-12 | 135 | 150 |
| P027 CHN001 | 6 | 01-03 | 01-17 | 0 | 150 |
| P027 DEL001 | 2 | 01-05 | 01-20 | 62 | 150 |
| P027 HYD001 | 2 | 01-02 | 01-21 | 141 | 150 |

42 of 175,335 pair-days is about 0.02% (my arithmetic). Month-end missed these dips because every pair was back above its levels by the end of January 2019 (month-end test: 0 pairs below any level at any month-end, Steps 12 and 13). The tests that explain the dips (Tests B, C, C2, D) and the repeat on the ledger (Test E) are in Check 2 of the Verification Log.

### Alternatives considered

| Question | Choice made | Alternative tested | Result of the alternative |
|----------|-------------|--------------------|---------------------------|
| What is the needed stock? | 3 months of sales per pair | 1, 6, 12 months; inventory-master max_stock; products-table max_stock_level | Excess 99.07% to 99.92% of stock; every pair has excess under every rule (Step 8) |
| Which cost per unit? | Received cost (line_total / quantity on Received purchase lines) | Products-table unit_cost | Excess value 308,184,841,342, about 4.6% higher; the products-table cost differs from the purchase cost on 155,492 of 155,495 purchase lines (KPI 9) |
| Which stock record? | Order-line stock 21,185,215 (KPI 8 decision) | Ledger stock 19,305,994 | Excess value 268,214,199,098 = 91.05% of the headline (Check 5B) |
| Which sales window for the need? | 12 months 15 Jan 2024 to 13 Jan 2025 (as KPI 6) | Calendar 2024 | Excess changes by 0.004% (Test 7A) |
| Month-end or daily understock test? | Both | Daily test on the ledger running_balance | Same finding on the ledger (Test E) |
| Where does understock belong? | Test inside KPI 10 | Leave all of it to KPI 19 | KPI 19 keeps the forward-looking reorder point |

## 3. Verification Log

Every check was run on the exact table KPI 10 uses (sd = Delivered lines only, 117,656 rows; pr = Received lines only, 140,077 rows; pair = 180 rows), with the table name and row count printed. Every claim, reason or "why" that needed a test was tested inside the check where it appeared. Extra tests are listed under the check they belong to; the 8 numbered checks stay 8.

### Summary of the 8 checks

| # | Check | Method | Result in plain words |
|---|-------|--------|-----------------------|
| 1 | Missing, zero and negative values | Counted empty, zero and negative values in every column KPI 10 uses in sd, pr, inv, products, ledger and pair | No empty, zero or negative value in any column KPI 10 uses |
| 2 | Edge cases | Every pair has sales in the window, needed stock never above stock held, no negative stock, every product has a received cost; daily understock tests A to E as extras | No edge case found; understock exists only as short dips in January 2019 under both stock records |
| 3 | Repeated lines | Duplicate keys, lines sharing an order and product, exact copies, chance test (Test F), pair unique | No duplicate keys; no exact copies in pr; the exact copies in sd are slightly fewer than chance alone gives |
| 4 | Totals reconcile | Units tie to stock, needed + excess = stock, excess value three ways, stock value, spend | Every total equals its verified number and the parts add up to the whole |
| 5 | Second method | 5A: stock, excess units and excess value rebuilt from the raw tables with pivot_table; 5B: ledger stock as a sensitivity | 5A identical for all 180 pairs; 5B: finding holds on the ledger, money 91.05% of the headline |
| 6 | Outliers | 1.5 x IQR on pair excess units and excess value; 5 largest and 5 smallest pairs; Tests 6A, 6B, 6C | 1 low pair in units (least stock); 12 high pairs in money = every pair of P015 and P014; no data error |
| 7 | History | 73 months, year-end totals against KPI 9, first and last dates, calendar year against 12-month window; Tests 7A, 7B | History complete and tied; the whole gap to the 2024 year-end figure is explained |
| 8 | Manual spot-check | 3 pairs rebuilt by hand from raw rows (largest excess value, smallest, a middle pair) | All three equal the pair table |

### Check 1: Missing, zero and negative values

Counts of empty / zero / negative values:

| Table (rows) | Columns checked | Empty | Zeros | Negatives |
|--------------|-----------------|-------|-------|-----------|
| sd (117,656) | product_id, branch_id, delivery_date | 0 | not applicable | not applicable |
| sd (117,656) | quantity | 0 | 0 | 0 |
| pr (140,077) | po ids, product_id, branch_id, received_date | 0 | not applicable | not applicable |
| pr (140,077) | quantity, line_total | 0 | 0 | 0 |
| inv (180) | opening_stock, reorder_level, safety_stock, max_stock | 0 | 0 | 0 |
| products (30) | unit_cost, reorder_level, safety_stock, max_stock_level | 0 | 0 | 0 |
| ledger (237,230) | product_id, branch_id, movement_type, movement_date | 0 | not applicable | not applicable |
| ledger (237,230) | quantity, running_balance | 0 | 0 | 0 |
| pair (180) | opening_stock, received, delivered, stock_end, sold_12m, cover_years, inv_max, prod_max, monthly_sales, cost_received, cost_table, needed_3m, excess_3m, excess_value, stock_value | 0 | 0 | 0 |
| pair (180) | adjust | 0 | not tested | not tested |

adjust (stock corrections) can be zero or negative by design, so only empty values were counted. Result: no empty, zero or negative value in any column KPI 10 uses.

### Check 2: Edge cases

| Test | Result |
|------|--------|
| sd rows in the 12-month window | 18,889 rows, 197,300 units; sum of sold_12m 197,300 (ties to KPI 6) |
| Pairs with at least one delivered line in the window | 180 of 180 |
| Fewest delivered lines in one pair | 67 |
| Smallest needed_3m | 175.75 |
| Pairs where needed_3m is above stock_end (before cutting at zero) | 0 |
| Pairs with negative stock_end | 0 |
| Lowest end-of-day stock of any pair | 0.0 (the single zero of P027 CHN001, Step 15) |
| Products with received lines | 30 of 30 |
| Smallest cost_received | 232.45 |
| Pairs with cost_received of 0 or less | 0 |

Result: every pair has sales in the window, needed stock never exceeds the stock held so the excess never had to be cut to zero, no stock is negative, and every product has a received cost.

**Extra tests of this check: the daily understock tests (Steps 14 and 15).** Each test was run when its claim appeared.

**Test A: why the first daily run counted 200,149 pair-days and the clean run 175,335.** Claim: the same day was counted twice, once as text and once as a real date. Distinct text pair-days (receipts and adjustments) 108,505; distinct sales pair-days 91,644; text + sales = 200,149 (equal to the first run); correct pair-days 175,335; days that were in both 24,814 (my arithmetic: 200,149 - 175,335). Result: confirmed. The 24,814 days that had both a sales day and a receipt or adjustment day were counted twice in the first run; the correct count is 175,335.

**Test B: the January 2019 dips happen while pairs still hold opening stock, before the first receipt.** Per pair (opening stock, first dip, last dip, lowest, first receipt, receipts up to the last dip):

| Pair | Opening stock | First dip | Last dip | Lowest | First receipt | Receipts up to last dip |
|------|---------------|-----------|----------|--------|---------------|-------------------------|
| P004 KOL001 | 115 | 01-12 | 01-12 | 37 | 01-15 | 0 |
| P005 CHN001 | 172 | 01-16 | 01-16 | 62 | 01-19 | 0 |
| P008 PUN001 | 86 | 01-03 | 01-12 | 34 | 01-12 | 1 |
| P013 PUN001 | 80 | 01-04 | 01-15 | 19 | 01-12 | 1 |
| P020 DEL001 | 91 | 01-11 | 01-17 | 67 | 01-22 | 0 |
| P020 PUN001 | 80 | 01-07 | 01-07 | 75 | 01-13 | 0 |
| P023 HYD001 | 82 | 01-09 | 01-12 | 22 | 01-17 | 0 |
| P026 CHN001 | 113 | 01-07 | 01-20 | 73 | 01-21 | 0 |
| P026 DEL001 | 96 | 01-06 | 01-12 | 94 | 01-12 | 1 |
| P026 HYD001 | 203 | 01-07 | 01-25 | 148 | 01-28 | 0 |
| P027 AHM001 | 151 | 01-05 | 01-12 | 135 | 01-13 | 0 |
| P027 CHN001 | 84 | 01-03 | 01-17 | 0 | 01-19 | 0 |
| P027 DEL001 | 98 | 01-05 | 01-20 | 62 | 01-22 | 0 |
| P027 HYD001 | 123 | 01-02 | 01-21 | 141 | 01-25 | 0 |

For 11 of the 14 pairs the first receipt comes after the last dip day. Three pairs (P008 PUN001, P013 PUN001, P026 DEL001) had one receipt by the last dip day: the first receipt falls on the last dip day for P008 PUN001 and P026 DEL001, and for P013 PUN001 the first receipt was on 01-12 while the dip lasted until 01-15. One pair needed a follow-up: P027 HYD001 has a lowest stock (141) above its opening stock (123) with no receipt, so a positive movement came before the first receipt (Tests C and C2).

**Test C: P027 HYD001, first days.** Movements: 2019-01-02 +20, 01-21 -2, 01-25 +233, 01-27 +92, 01-28 +159, 01-29 +261, 02-01 +205, 02-06 -8. End-of-day stock: 143, 141, 374, 466, 625, 886, 1,091, 1,148. Result: stock went from 123 to 143 on 2 January with a +20 movement before the first receipt (25 January), which explains why the lowest stock (141) is above the opening stock. The pair was only 9 units below its highest level (150) on 2 and 21 January, and the first receipt (+233) lifted it to 374.

**Test C2: ledger rows of P027 HYD001 (to confirm what the +20 is).** 2019-01-02 ADJUSTMENT 20, balance 143; 01-21 OUT 2, balance 141; 01-27 IN 92, balance 233; 01-28 IN 159, balance 392; 01-29 IN 261, balance 653; 02-01 IN 205, balance 858. Result: the +20 is an ADJUSTMENT in the ledger (confirmed). The ledger balances agree with the order-line rebuild on 2 January (143) and 21 January (141). The ledger has no row for the 233 receipt of 25 January that the purchase lines show, so from 25 January the ledger balance is 233 lower than the order-line rebuild (233 on 27 January against 466; 858 on 1 February against 1,091). This is consistent with the verified Week 3 and KPI 8 finding that about 10% of order lines never reached the ledger. On the ledger this pair stays at 141 until 27 January.

**Test D: the 3 pairs with a receipt inside the dip window.** P008 PUN001: highest level 120, receipt 2019-01-12 of 84, end-of-day stock 118 on 01-12. P013 PUN001: level 80, receipt 01-12 of 39, stock 58 on 01-12 and 62 on 01-15. P026 DEL001: level 200, receipt 01-12 of 85, stock 172 on 01-12. Result: in all three pairs the receipt was too small to lift the stock above the highest level, so the pair stayed below it. The January 2019 claim is closed: all 14 pairs dipped before a receipt large enough to lift them above their levels arrived (11 pairs had no receipt until after the last dip day, 3 pairs had one small receipt inside the window).

**Test E: the daily test repeated on the ledger running_balance (sensitivity).**

| Measure | Order-line stock (Steps 14 and 15) | Ledger running_balance |
|---------|------------------------------------|------------------------|
| Pairs below inventory-master reorder_level | 5 | 3 |
| Pairs below inventory-master safety_stock | 1 | 1 |
| Pairs below products-table reorder_level | 11 | 11 |
| Pairs below products-table safety_stock | 6 | 6 |
| Pair-days below at least one level | 42 | 37 |
| Pairs with a dip | 14 | 13 |
| Dip dates | 2 Jan to 25 Jan 2019 | 2 Jan to 28 Jan 2019 |
| Lowest end-of-day stock | 0 | 14 |
| Pair-days in the ledger rebuild | not applicable | 165,246 |

Result: the finding does not change with the record used. Understock exists only as short dips in January 2019, none from February 2019 onward, under both order-line stock and the ledger.

### Check 3: Repeated lines

| Table | Rows | Duplicate (order id, line_number) | Lines sharing an order and a product | Exact copies (all columns the same except line_number) |
|-------|------|-----------------------------------|--------------------------------------|---------------------------------------------------------|
| sd | 117,656 | 0 | 25,422 | 688 rows that repeat an earlier line |
| pr | 140,077 | 0 | 30,068 | 0 |
| pair | 180 | pair unique: True | not applicable | not applicable |

**Test F: are the exact copies in sd copy-paste errors or chance?** Idea: sales lines for the same product in the same order often have the same quantity by chance, so the observed number of exact copies must be compared with the number chance alone gives (the sum over quantities of the squared share of each quantity for that product). Order-product groups with 2 or more lines: 12,184; line pairs inside them: 14,369; pairs that are exact copies (observed): 690; expected by chance: 721.7. That is 4.80% observed against 5.02% expected (my arithmetic; KPI 5 Check 3 found 4.88% against 5.0%). Check 3 counts the rows that repeat an earlier line (688); Test F counts pairs of identical lines (690). Result: the exact copies in the sales lines are slightly fewer than chance alone gives, so nothing points to copied lines being counted twice. No line was removed.

### Check 4: Totals reconcile

| Test | Result | Expected |
|------|--------|----------|
| opening 32,969 + received 22,387,123 - delivered 1,234,939 + adjust 62 | 21,185,215 = stock_end total | 21,185,215 |
| Largest gap between needed_3m + excess_3m and stock_end in any pair | 0.0 | 0 |
| needed_3m total | 49,325.0 | 49,325 (my arithmetic: 3 x 197,300 / 12) |
| excess_3m total | 21,135,890 | 21,135,890 |
| excess_value column total | 294,576,746,335 | 294,576,746,335 |
| Units x cost_received recomputed | 294,576,746,335 | 294,576,746,335 |
| Sum of the six branch totals | 294,576,746,335 | 294,576,746,335 |
| Sum of the 30 product totals | 294,576,746,335 | 294,576,746,335 |
| Stock value | 295,255,911,357 | 295,255,911,357 |
| Spend on received purchases (pr line_total) | 312,027,429,108 | 312,027,429,108 |

Result: every total equals its verified number and the parts add up to the whole.

### Check 5: Second method

**5A: rebuilt from the raw tables with pivot_table instead of the pair table** (opening stock from inv, received from pr, delivered from sd, adjustments from the ledger, sales in the 12-month window, cost = pr line_total / quantity per product).

| Measure | Result | Expected |
|---------|--------|----------|
| Pairs | 180 | 180 |
| Stock total | 21,185,215 | 21,185,215 |
| Excess units | 21,135,890 | 21,135,890 |
| Excess value | 294,576,746,335 | 294,576,746,335 |
| Largest difference to the pair table, stock | 0.0 | 0 |
| Largest difference to the pair table, excess units | 0.0 | 0 |

Result: the independent rebuild gives exactly the same stock, excess units and excess value, and every pair agrees.

**5B: ledger-based stock as a sensitivity** (stock = last running_balance per pair; excess = ledger stock minus needed_3m, cut at zero; value = excess x cost_received).

| Measure | Ledger stock | Order-line stock (headline) |
|---------|--------------|-----------------------------|
| Stock total | 19,305,994 | 21,185,215 |
| Pairs where needed_3m is above stock | 0 | 0 |
| Excess units | 19,256,669 (99.74% of stock) | 21,135,890 (99.77%) |
| Excess value | 268,214,199,098 | 294,576,746,335 |

The ledger excess matches my arithmetic (19,305,994 - 49,325 = 19,256,669). The units gap to the headline, 21,135,890 - 19,256,669 = 1,879,221, equals the KPI 8 stock gap exactly (my arithmetic). The ledger excess value is 91.05% of the headline, about 26,362,547,237 lower (my arithmetic), about 8.95%. Result: the finding (more than 99% of stock is excess, no pair short of the needed stock) holds on either record; the headline keeps the order-line stock (KPI 8 decision) and the ledger figure is reported as the sensitivity.

### Check 6: Outliers (1.5 x IQR rule)

The IQR rule: sort the 180 pairs; q1 is a quarter of the way up and q3 three quarters of the way up; IQR = q3 - q1; a pair is an outlier if it lies below q1 - 1.5 x IQR or above q3 + 1.5 x IQR. An outlier is unusual, not necessarily an error. (The same rule was used in KPI 8 Check 6.)

| Column | q1 | q3 | Normal range | Pairs below | Pairs above |
|--------|----|----|--------------|-------------|-------------|
| excess_3m | 113,295 | 121,150 | 101,513 to 132,932 | 1 | 0 |
| excess_value | 126,004,441 | 2,140,896,367 | -2,896,333,448 to 5,163,234,256 | 0 | 12 |

The lower limit of the excess_value range is negative and excess value cannot be negative, so no pair can fall below it.

5 largest pairs by excess value (all P015, received cost 89,239.25): CHN001 125,269 units, 11.18 billion; HYD001 120,540.5, 10.76 billion; AHM001 115,641.5, 10.32 billion; PUN001 115,333.25, 10.29 billion; DEL001 109,952.25, 9.81 billion. 5 smallest (all P026, received cost 232.45): DEL001 112,178.75 units, 26.08 million; KOL001 113,039, 26.28 million; PUN001 113,309.25, 26.34 million; AHM001 114,587.5, 26.64 million; CHN001 116,420.25, 27.06 million. Units are similar for the largest and smallest pairs; the money differs because the cost per unit differs.

**Test 6A: the 1 pair below the normal range in units.** P019 KOL001: stock_end 98,642, sold_12m 1,009, needed_3m 252.25, excess_3m 98,389.75 (below the 101,513 limit). stock_end 98,642 is the smallest stock of the 180 pairs (equal to Step 6) and sold_12m 1,009 is below the mean (1,096.11), so it is flagged because it holds the least stock, not because it sells more. It is the same pair as the low outlier of KPI 8 Check 6 (there found on ledger stock). Result: an unusual pair, not an error.

**Test 6B: the 12 pairs above the normal range in money.** All 6 pairs of P015 (excess 9.48 to 11.18 billion, cost 89,239.25) and all 6 pairs of P014 (7.27 to 7.74 billion, cost 62,712.84): The pairs as printed: P015 CHN001 125,269 units, 11.18 billion; HYD001 120,540.5, 10.76 billion; AHM001 115,641.5, 10.32 billion; PUN001 115,333.25, 10.29 billion; DEL001 109,952.25, 9.81 billion; KOL001 106,250.25, 9.48 billion; P014 CHN001 123,398.5, 7.74 billion; PUN001 122,746, 7.70 billion; DEL001 122,678, 7.69 billion; HYD001 121,328, 7.61 billion; KOL001 116,716.75, 7.32 billion; AHM001 115,983, 7.27 billion. Their excess units (106,250 to 125,269) are inside the normal units range. The 12 pairs add up to 107,173,612,901, which is 36.38% of the headline.

**Test 6C: is it the price?** The three highest received costs per unit are P015 89,239, P014 62,713 and P021 34,558, so the two products flagged for money are the two most expensive. The largest unflagged pair is P021 DEL001 at 4,374,068,428 against a cut-off of 5,163,234,256 (789,165,828 below the cut-off, my arithmetic). The exact excess values are P015 61,841,619,703.55 and P014 45,331,993,197.63, adding to 107,173,612,901.18. The difference of 1 to the sum of the rounded Step 10 totals (902) is rounding: 0.45 + 0.37 = 0.82 (my arithmetic). Result: the money outliers are exactly the two most expensive products in every branch, their units are normal, and nothing points to a data error.

### Check 7: History

| Test | Result |
|------|--------|
| Month columns in the month-end table | 73 (2019-01 to 2025-01); 72 + 1 months (my arithmetic) |
| Pairs | 180 |
| Year-end stock 2019 / 2020 / 2021 | 3,384,660 / 6,960,247 / 10,529,943 |
| Year-end stock 2022 / 2023 / 2024 | 14,009,405 / 17,557,387 / 20,993,633 |
| Stock at the end of January 2025 | 21,185,215 |
| Receipts (pr) first and last date | 2019-01-11 and 2025-01-28 |
| Deliveries (sd) first and last date | 2019-01-03 and 2025-01-13 |

All six year-end totals equal KPI 9 Step 4 and the last month equals the headline stock. Deliveries stop on 13 January 2025 and receipts on 28 January 2025, so the last month does not hold a full month of both.

**Test 7A: does the choice of sales window matter?** The excess at the end of January 2025 was recomputed with the need taken from calendar 2024 sales.

| Measure | Calendar 2024 need | Headline (12-month window) |
|---------|--------------------|----------------------------|
| Pairs without calendar 2024 sales | 0 | not applicable |
| Units sold | 200,562 | 197,300 |
| Excess units | 21,135,074 | 21,135,890 |
| Excess value | 294,565,059,230 | 294,576,746,335 |

Calendar 2024 sold 3,262 more units (1.65%; my arithmetic). The excess is 816 units lower (my arithmetic: 3 x 3,262 / 12 = 815.5) and the value is 11,687,105 lower, which is 0.004% of the headline. Result: the window choice does not matter.

**Test 7B: where does the gap between the 2024 year-end figure and the headline come from?** The gap is 2,750,951,052 (294,576,746,335 - 291,825,795,283). Units added to stock in January 2025: 191,582 (21,185,215 - 20,993,633). Value of that added stock at received cost: 2,739,263,947. Window effect (Test 7A) 11,687,105 + January stock value 2,739,263,947 = 2,750,951,052, equal to the gap. Result: the whole gap is explained (the choice of window plus the stock added in January 2025); no data problem.

### Check 8: Manual spot-check of 3 pairs

Each pair was rebuilt from raw rows: sales in the window from sd, stock from the month-end table, need = sold / 12 x 3, excess = stock - need, value = excess x cost.

| Pair | Why chosen | Sold | Stock | Need | Excess units | Excess value | Equal to the pair table |
|------|-----------|------|-------|------|--------------|--------------|-------------------------|
| P015 CHN001 | Largest excess value | 1,192 | 125,567 | 298.00 | 125,269.00 | 11,178,911,947 | sold, excess units and value all equal |
| P026 DEL001 | Smallest excess value | 1,013 | 112,432 | 253.25 | 112,178.75 | 26,076,298 | sold, excess units and value all equal |
| P018 KOL001 | Position 90 of 180 by excess value (middle) | 1,312 | 106,960 | 328.00 | 106,632.00 | 904,392,890 | sold, excess units and value all equal |

My arithmetic agrees: 1,192 / 4 = 298 and 125,567 - 298 = 125,269; 1,013 / 4 = 253.25 and 112,432 - 253.25 = 112,178.75; 1,312 / 4 = 328 and 106,960 - 328 = 106,632. Stock and cost per unit were not rebuilt inside this check; both were rebuilt from the raw tables in Check 5A, where stock, excess units and the excess value total were equal for all 180 pairs. Result: for the largest, the smallest and a middle pair the sales, excess units and money are equal to the pair table.

### Test labels glossary

| Label | Plain name | Question | Result | Sits in |
|-------|------------|----------|--------|---------|
| Test A | Pair-day double count | Why did the first daily run give 200,149 pair-days? | Same day counted twice (text and date); correct 175,335 | Check 2 |
| Test B | Dips before the first receipt | Do the January 2019 dips happen while pairs hold opening stock? | 11 of 14 pairs had no receipt before the last dip day; 3 had one small receipt | Check 2 |
| Test C | P027 HYD001 first days | Why is its lowest stock above its opening stock? | A +20 movement on 2 January | Check 2 |
| Test C2 | P027 HYD001 in the ledger | Is the +20 an adjustment? Does the ledger match? | Adjustment confirmed; ledger lacks the 233 receipt of 25 January | Check 2 |
| Test D | Small receipts | Did the 3 receipts inside the dip window lift the stock above the levels? | No, each receipt was too small | Check 2 |
| Test E | Ledger daily test | Does the understock finding change on the ledger? | No: 13 pairs, 37 pair-days, 2 to 28 January 2019 | Check 2 |
| Test F | Chance test for exact copies | Are the exact copies in sd more than chance? | 690 observed against 721.7 expected | Check 3 |
| 5A | Raw-table rebuild | Does an independent rebuild give the same stock, excess units and value? | Identical for all 180 pairs | Check 5 |
| 5B | Ledger sensitivity | What does the ledger stock give? | 19,256,669 excess units, 268,214,199,098 money | Check 5 |
| Test 6A | Low pair in units | Why is P019 KOL001 below the normal range? | Least stock of the 180 pairs | Check 6 |
| Test 6B | High pairs in money | Which 12 pairs are above the normal range? | Every pair of P015 and P014 | Check 6 |
| Test 6C | Price test | Are P015 and P014 the most expensive products, and what is next? | Yes; the largest unflagged pair is P021 DEL001 | Check 6 |
| Test 7A | Window test | Does calendar 2024 instead of the 12-month window change the excess? | 0.004% | Check 7 |
| Test 7B | Gap test | Where does the 2,750,951,052 gap come from? | 11,687,105 window + 2,739,263,947 January stock | Check 7 |

## 4. Data Quality Notes

1. **Two sets of levels.** The inventory master (per pair) and the products table (per product) hold different maximum, reorder and safety levels (maximum ratio 0.59 to 38.16). Which set is official is not in the data, so both were tested (Steps 3, 8, 12, 13).
2. **current_stock is inflated.** The inventory-master current_stock (88,853 to 121,013 against an opening stock of 80 to 300) is not used; stock is rebuilt from opening stock, received lines, delivered lines and corrections.
3. **Order-line stock and ledger stock differ.** Order-line stock is 21,185,215 and ledger stock 19,305,994, a gap of 1,879,221 units (the KPI 8 gap). About 10% of order lines never reached the ledger (Week 3 and KPI 8). Example: P027 HYD001 has a 233-unit receipt on 25 January 2019 in the purchase lines but not in the ledger (Test C2). The headline uses order-line stock; the ledger is the sensitivity (Check 5B).
4. **Two date columns are text.** pr received_date and ledger movement_date are text in the session; sd delivery_date is a real date. The first daily run mixed the types, put the days out of order, and gave a lowest stock of -8,712 (P023 CHN001, 2025-01-11) with all 180 pairs below all four levels. That result was wrong and was not used. All three columns were converted with pd.to_datetime and the run repeated (Step 14, Test A).
5. **January 2025 is a partial month.** Deliveries stop on 13 January 2025 and receipts on 28 January 2025 (Check 7).
6. **No currency name.** The data does not name a currency, so amounts are written as money.
7. **Exact copies in sd.** 688 rows in sd repeat an earlier line (all columns equal except line_number). The number is slightly below what chance alone gives (Test F), so no line was removed.
8. **Cost basis.** The received cost (line_total / quantity per product on Received purchase lines) is used. The products-table unit_cost differs from the purchase cost on 155,492 of 155,495 purchase lines (KPI 9) and gives a figure about 4.6% higher.
9. **Stock corrections.** 2,454 ADJUSTMENT rows in the ledger net to +62 units; they are included in the stock.
10. **Cancelled orders.** Lines of cancelled orders are left out of sd and pr. In the ledger, rows of cancelled orders are counted in running_balance (Week 3 and KPI 8), which is one reason the ledger sensitivity differs from the headline.
11. **Records, not a count.** Stock and excess come from order records and the ledger, not from a physical stock count.
12. **Year-end and headline differ.** The 2024 year-end excess value (291,825,795,283) and the headline (294,576,746,335) differ by 2,750,951,052; the difference is the sales window (11,687,105) plus the stock added in January 2025 (2,739,263,947) (Test 7B).

## 5. Business Interpretation

**Business question:** How much money is held in stock that the business does not need, and where does it sit?

**Answer:** 294,576,746,335 (valued at the received cost) is tied up in stock above 3 months of sales. That is 99.77% of the stock and 94.41% of everything spent on received purchases. No pair is short of stock; every pair holds 72 to 172 years of sales.

Findings:

1. 21,135,890 of 21,185,215 units (99.77%) are excess. Every pair has excess under all six rules tested (99.07% to 99.92%), so the finding does not depend on the rule.
2. The money follows price, not volume. Units sold per product are about equal (KPI 7), so the two most expensive products, P015 Engine Block (61,841,619,704) and P014 Transmission Assembly (45,331,993,198), hold 36.38% of the excess; the top 5 products hold 58.58%.
3. The six branches hold similar amounts (15.94% to 16.95% of the excess), so this is a company-wide issue, not one branch.
4. The excess grew by about 49 billion a year, from 46,509,105,934 (2019) to 291,825,795,283 (2024), with no sign of slowing. About 18 units were received for every 1 delivered (my arithmetic: 22,387,123 / 1,234,939 = 18.13).
5. Understock is not a problem: no pair was below a stock level at any month-end, and the only daily dips were 14 pairs on 42 pair-days between 2 and 25 January 2019, before the first large receipts.
6. On the ledger record the money is 268,214,199,098 (91.05% of the headline), so the finding holds on either record.
7. The figure is money held in stock, not a loss. It shows how much money could be released by buying less.

### Recommended actions for the business owners
1. Procurement Manager: place no new purchase orders for any pair until its stock falls to about 3 months of sales; start with P015 and P014.
2. Inventory Manager: review P015 and P014 in all six branches first (36.38% of the excess), then the other products in the top 5 (58.58% together).
3. Finance Controller: report 294,576,746,335 as money tied up in stock (received-cost basis) and review how this stock is valued, since 94.41% of purchase spending is still on the shelf.
4. Data Owner (the team that keeps the stock records): make the ledger complete (1,879,221 units are missing against the order records), store received_date and movement_date as real dates, and decide which set of reorder, safety and maximum levels is official.
5. Inventory Manager: keep the reorder levels in force while purchases are cut; no pair has been below a level since January 2019.

## Glossary

Short names are written as they appear in the notebook. A few names are used for two different things in different steps; both meanings are given.

### Tables and lines

| Short name | Plain meaning |
|------------|---------------|
| ledger | Stock ledger table (237,230 rows) |
| inv | Inventory master table (180 rows, one per pair) |
| po_h, po_l | Purchase order headers and purchase order lines |
| so_h, so_l | Sales order headers and sales order lines |
| products | Products table (30 rows) |
| branches | Branches table (6 rows) |
| sd | Delivered sales lines joined to their order headers (117,656 rows) |
| pr | Received purchase lines joined to their order headers (140,077 rows) |
| L | The ledger sorted by product, branch and date, with the two columns below added |
| prev | In L: the previous running_balance of the pair (for its first row, the opening stock) |
| change | In L: running_balance minus prev, the real change of that movement |
| op | Opening stock per pair (used to fill prev for the first row of each pair) |
| adj | Net stock corrections (ADJUSTMENT rows) per pair, taken from L |
| pair | The table of the 180 product-branch pairs, one row per pair |
| pair-day | One pair on one day on which it had a movement |

### Columns of the pair table

| Short name | Plain meaning |
|------------|---------------|
| opening_stock | Stock of the pair at the start (inventory master) |
| received | Units received by the pair (Received purchase lines) |
| delivered | Units delivered by the pair (Delivered sales lines) |
| adjust | Net stock corrections of the pair |
| stock_end | Order-line stock of the pair at the end of the data: opening + received - delivered + adjust |
| sold_12m | Units delivered by the pair from 15 Jan 2024 to 13 Jan 2025 |
| cover_years | stock_end / sold_12m (years of stock at the current selling rate) |
| inv_max | In the pair table: maximum stock level of the pair from the inventory master. In Step 3: the average inventory-master max_stock per product |
| prod_max | Maximum stock level of the pair's product from the products table |
| monthly_sales | sold_12m / 12 |
| cost_received | Received cost per unit of the product: line_total / quantity on Received purchase lines |
| cost_table | Unit cost from the products table |
| needed_3m | Stock needed for 3 months of sales = 3 x monthly_sales |
| excess_3m | stock_end minus needed_3m, cut at zero (excess units) |
| excess_value | excess_3m x cost_received |
| stock_value | stock_end x cost_received |

### Money, rules and summary tables (Steps 3 to 11)

| Short name | Plain meaning |
|------------|---------------|
| cmp | Step 3 table comparing the two maximum levels per product |
| rules | The six rules for the needed stock (1, 3, 6, 12 months of sales and the two company limits) |
| total_stock | Total stock of all pairs (21,185,215) |
| cw | Received cost per unit for each product (line_total / quantity on Received lines) |
| ct | Products-table unit_cost for each product |
| pid | The product id of each pair (used to give each pair its cost) |
| stock_value_rc, excess_value_rc | Stock value and excess value at the received cost |
| stock_value_ct, excess_value_ct | Stock value and excess value at the products-table cost |
| by_branch | Stock value, excess value and share of the excess per branch |
| show | Step 10: a rounded copy of by_branch for printing. Check 6: the list of columns printed for the largest and smallest pairs |
| names | Product name for each product id |
| by_prod | Excess value per product, largest first |
| top | The 5 products with the largest excess value |
| low | Step 10: the 5 products with the smallest excess value. From Step 14: the lowest end-of-day stock of each pair |
| spend | Total spent on received purchases (sum of pr line_total) |
| last12 | The sd lines delivered in the 12-month window 15 Jan 2024 to 13 Jan 2025 |
| idx | The pair index (product and branch) |
| cols | Step 11: the list of years 2019 to 2025. From Step 14: the two key columns product_id and branch_id |
| rec, dele, adjy | Units received, delivered and net corrections per pair for each year 2019 to 2025 |
| a | The ADJUSTMENT rows of L with their year |
| stock_y | Stock per pair at the end of each year (opening stock plus the running total of rec - dele + adjy) |
| trend | The year-end table: stock units, excess units and excess value for 2019 to 2024 (M4) |
| M1, M2, M3, M4 | The four KPI 10 metrics: excess units, excess value, excess share of stock, year-end trend |

### Month-end and daily stock (Steps 12 to 15)

| Short name | Plain meaning |
|------------|---------------|
| months | The 73 months 2019-01 to 2025-01 |
| monthly() | Function that totals a column per pair per month |
| rec_m, del_m, adj_m | Units received, delivered and net corrections per pair per month |
| stock_m | Month-end stock per pair, 180 pairs x 73 months |
| reorder, safety | Inventory-master reorder level and safety stock per pair |
| below_reorder, below_safety | Pair-months with stock below the inventory-master reorder level, safety stock |
| pt_reorder, pt_safety | Products-table reorder level and safety stock per pair |
| below_pt_reorder, below_pt_safety | Pair-months with stock below the products-table reorder level, safety stock |
| ratio | Month-end stock divided by the inventory-master reorder level |
| ev | All stock movements (receipts, deliveries, adjustments) as one table, 260,187 rows |
| chg | Change in stock caused by one movement: plus for a receipt, minus for a delivery, plus or minus for an adjustment |
| d | Date of a movement as a real date (in Check 8: the delivery dates of sd as real dates) |
| daily | Total change per pair per day |
| bal | End-of-day stock per pair-day |
| last | Last end-of-day balance of each pair (equals stock_end) |
| lv | The four levels per pair (two reorder levels, two safety stocks) and the highest of them |
| highest_level | The highest of the four levels of a pair |
| b | Step 15: end-of-day stock per pair-day joined to the four levels. In Tests C and D and Check 8: the branch id in a loop |
| hit | The pair-days with stock below at least one level (42 rows) |
| p | The product id in a loop (Tests C and D, Check 8) |
| g | Test B: one row per pair that dipped, with first_dip, last_dip, lowest stock, opening stock, first receipt and receipts up to the last dip. Test F: the order-product groups with 2 or more lines and their number of line pairs |
| first_dip, last_dip | First and last day on which the pair was below a level |
| first_receipt | Date of the first receipt of the pair |
| receipts_up_to_last_dip | Number of receipts of the pair up to its last dip day |
| rd | Received purchase lines with the date converted to a real date |
| three | The three pairs that had a receipt inside the dip window (Test D) |
| txt | Receipt and adjustment rows with their text dates (Test A) |
| n_text, n_sd | Number of distinct pair-days in txt and in sd (Test A) |
| ledger running_balance | The balance column of the stock ledger |
| order-line stock | Stock rebuilt from order lines (opening + received - delivered + adjustments) |
| lgd | End-of-day ledger running_balance per pair-day (Test E) |
| lowL | Lowest end-of-day ledger balance per pair (Test E) |
| bL | lgd joined to the four levels (Test E) |
| hitL | The ledger pair-days below at least one level (Test E) |

### Checks and tests (Checks 1 to 8)

| Short name | Plain meaning |
|------------|---------------|
| scan_text, scan_num | Check 1 helpers: count empty text values; count empty, zero and negative numbers |
| pr_ | The pair table as a plain table (Check 1) |
| win | The sd lines in the 12-month window (Check 2) |
| lines_win | Number of window delivered lines per pair (Check 2) |
| rp | Units received per product (Check 2, to count products with received lines) |
| other_cols | All columns of the table except line_number (Check 3 and Test F) |
| s | Test F: for each product, the chance that two lines have the same quantity (sum of the squared share of each quantity). Check 6: the pair column being tested |
| expected, observed | Exact-copy line pairs chance alone gives, and the number found (Test F) |
| tot | Function that gives the whole-number total of a pair column (Check 4) |
| excess_check | excess_3m x cost_received per pair, recomputed (Check 4) |
| rec2, del2, adj2 | Received, delivered and net correction units per pair rebuilt with pivot_table (Check 5A) |
| in_window | True for sd lines delivered from 15 Jan 2024 to 13 Jan 2025 (Check 5A) |
| sold2 | Units sold in the window per pair, rebuilt with pivot_table (Check 5A) |
| cost | Received cost per product rebuilt from pr (Check 5A) |
| x | Test C2: the ledger rows of P027 HYD001. Check 5A: the second-method table, one row per pair |
| led_stock | Last ledger running_balance per pair (Check 5B) |
| exL, valL | Excess units per pair and total excess value on the ledger stock (Check 5B) |
| IQR, q1, q3, iqr | Width of the middle half of the pairs, the quarter and three-quarter points, and q3 - q1 |
| lo, hi | Lower and upper limit of the normal range |
| normal range | q1 - 1.5 x IQR to q3 + 1.5 x IQR |
| outlier | A pair outside the normal range: unusual, not necessarily an error |
| hiv | Upper limit of the normal range of excess_value |
| out | The 12 pairs above that limit |
| rest | The pairs not flagged |
| cost_rank | Products sorted by received cost per unit |
| tot12 | Excess value of the 12 flagged pairs |
| p15, p14 | Exact excess value of P015 and of P014 |
| cs | List of month labels (year-month) of stock_m |
| cal | Units sold per pair in calendar 2024 |
| need_c, exc_c, val_c | Needed stock, excess units and excess value using calendar 2024 sales (Test 7A) |
| dec, jan | Stock per pair at the end of December 2024 and at the end of January 2025 |
| jan_value | Value of the stock added in January 2025 (units added x cost_received) |
| picks | The three pairs of the manual check (Check 8) |
| Test A to Test F, 5A, 5B, 6A to 6C, 7A, 7B | The supporting tests, described in the test labels table of the Verification Log |
| Check 1 to Check 8 | The 8 verification checks of this KPI |
