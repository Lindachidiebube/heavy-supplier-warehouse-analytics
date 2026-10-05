# Feature List

**Project:** Heavy Supplier & Warehouse Analytics (CadetX) | **Week:** 3 | **Step:** 6 (feature engineering record)

---

## 1. Purpose

A feature is a column or number that is not in the raw CSV files and that we created ourselves. This list records every one of them: what it means, how it is calculated, which table it lives in and which KPI uses it.

**How this list was built.** It was built by reading the code of the five notebooks (KPI 1, KPI 2, KPI 3, KPI 4, KPI 5). Column names and formulas are copied from that code. Result numbers come from the verified KPI write-ups.

**Notebooks read**
- `week2_KPI_NO1_verification_ipynb_.ipynb` (KPI 1)
- `week2_KPI_2_data_integrity.ipynb` (KPI 2)
- `week2_KPI_NO3_unpaid_invoice_rate.ipynb` (KPI 3)
- `week2_KPI_NO4_late_payment_rate.ipynb` (KPI 4)
- `Week3_KPI_N05_Product_Sales_Velocity.ipynb` (KPI 5)

**Summary**

| KPI | Source tables | New columns stored in a table | Headline metrics |
|-----|---------------|-------------------------------|------------------|
| KPI 1 Stock Accumulation Ratio | stock_ledger | 1 (`month`) plus per-product summary tables | Stock accumulation ratio per product |
| KPI 2 Data Integrity Rate | invoices, payments | none (counts and flags only) | Invoice ID Integrity Rate, Payment ID Integrity Rate |
| KPI 3 Unpaid Invoice Rate | invoices | none (rates only) | Unpaid, Partially Paid, Total Outstanding |
| KPI 4 Late Payment Rate | payments, invoices | 6 (`is_late`, `days_late`, `branch_id`, `customer_id`, `due_year`, `invoice_date`) plus 2 converted date columns | Late Payment Rate, median days late |
| KPI 5 Product Sales Velocity | sales lines and header, purchase lines and header, products | many (sections 6.1 to 6.9) | Average units sold per month, trend, steadiness, bought per 1 sold, velocity label |

---

## 2. KPI 1 Stock Accumulation Ratio (stock_ledger.csv)

| Feature | Where it lives | Meaning | How it is calculated | Used for |
|---------|----------------|---------|----------------------|----------|
| `movement_date` (converted) | `stock_ledger` | The movement date as a real date instead of text | `pd.to_datetime(stock_ledger['movement_date'])` | Month and history-length checks |
| `month` | `stock_ledger` | The calendar month of each ledger movement | `movement_date.dt.to_period('M')` | Zero-OUT and zero-IN edge-case check (months with IN only, OUT only, both) |
| `clean` | table | Ledger rows that are real supply or demand: IN and OUT only. ADJUSTMENT rows are left out because they are corrections | `stock_ledger[movement_type in ['IN', 'OUT']]` | All ratio calculations |
| `m1_in`, `m1_out` | per-product series | Total units received (IN) and total units sent out (OUT) for each product | Sum of `quantity` per `product_id`, separately for IN and OUT, on `clean` | Stock accumulation ratio |
| `ratio_m1` (Stock Accumulation Ratio, headline) | per-product series | Units that came IN for every 1 unit that went OUT. A value of 18 means 18 units in for every 1 out | `m1_in / m1_out` per product | KPI 1 headline |
| `ratio_m2` | per-product series | The same ratio built a second way, to confirm the first | `pivot_table(index='product_id', columns='movement_type', values='quantity', aggfunc='sum')`, then `IN / OUT` | Check 5 (second method). Largest difference between `ratio_m1` and `ratio_m2` was tested against 0.0001 |
| `stats['max_to_mean_ratio']` | per-product table | How big the largest incoming delivery is compared with the average delivery. Shows whether one huge delivery distorts a product | `max / mean` of IN `quantity` per product | Check 6 (outliers) |
| `span['days']` | per-product table | How many days of history each product has in the ledger | `max(movement_date) - min(movement_date)` in days, per product | Check 7 (history length) |

---

## 3. KPI 2 Data Integrity Rate (invoices.csv, payments.csv)

No new column was added to any table. The KPI is built from duplicate flags and counts.

| Feature | Meaning | How it is calculated | Used for |
|---------|---------|----------------------|----------|
| `dup_rows_A` (invoices) | Every invoice row whose `invoice_id` appears more than once | `invoices[invoices.duplicated(subset='invoice_id', keep=False)]` | Counting affected invoice rows (393) |
| `dup_rows_A_pay` (payments) | Every payment row whose `payment_id` appears more than once | `payments[payments.duplicated(subset='payment_id', keep=False)]` | Counting affected payment rows (403) |
| `group_sizes`, `dup_groups` (invoices) | How many rows share each `invoice_id`, and the IDs that are repeated | `invoices.groupby('invoice_id').size()`, then keep sizes above 1 | Second counting method (196 groups) |
| `group_sizes_pay`, `dup_groups_pay` (payments) | Same for `payment_id` | `payments.groupby('payment_id').size()`, then keep sizes above 1 | Second counting method (201 groups) |
| Invoice ID Integrity Rate (headline) | Share of invoice rows whose ID is not unique | Affected rows / all invoice rows = 393 / 18,033 = 2.18% | KPI 2 headline |
| Payment ID Integrity Rate (headline) | Share of payment rows whose ID is not unique | Affected rows / all payment rows = 403 / 19,257 = 2.09% | KPI 2 headline |

---

## 4. KPI 3 Unpaid Invoice Rate (invoices.csv)

No new column was added to any table. Each rate is calculated three ways in the notebook and all three agree.

| Feature | Meaning | How it is calculated | Result |
|---------|---------|----------------------|--------|
| `unpaid_rate` (headline) | Share of invoices with payment status Unpaid | `(payment_status == 'Unpaid').sum() / len(invoices)`. Checked with a `groupby` version and a `value_counts(normalize=True)` version | 10.19% (of 18,033 invoices) |
| `partial_rate` | Share of invoices with payment status Partially Paid | Same three methods with `'Partially Paid'` | 20.30% |
| Total Outstanding | Unpaid plus Partially Paid together | Unpaid rate + partially paid rate | 30.49% |
| `unpaid`, `partial` | The Unpaid rows and the Partially Paid rows, kept as separate tables for the branch, customer and year splits | `invoices[invoices['payment_status'] == ...]` | Breakdown by branch, top 10 customers and year |

---

## 5. KPI 4 Late Payment Rate (payments.csv joined to invoices.csv)

**Tables built**

| Table | Meaning | How it is built |
|-------|---------|-----------------|
| `affected_invoice_ids`, `affected_payment_ids` | The IDs that appear more than once (from KPI 2) | `value_counts() > 1` on `invoice_id` and on `payment_id` |
| `invoices_clean` | Invoices without any repeated `invoice_id` | Raw invoices minus `affected_invoice_ids` (17,640 rows) |
| `payments_clean` | Payments without any repeated `payment_id` | Raw payments minus `affected_payment_ids` (18,854 rows) |
| `merged` | One row per clean payment with its invoice dates | `payments_clean.merge(invoices_clean[['invoice_id','invoice_date','due_date','grand_total']], on='invoice_id', how='inner', validate='m:1')` (18,445 rows) |

**Columns added to `merged`**

| Feature | Meaning | How it is calculated | Used for |
|---------|---------|----------------------|----------|
| `payment_date` (converted) | Payment date as a real date | `pd.to_datetime(..., errors='coerce')`. Nulls after conversion were counted | `is_late`, `days_late` |
| `due_date` (converted) | Invoice due date as a real date | Same conversion | `is_late`, `days_late` |
| `is_late` | True if the payment came after the due date, False if on time or early | `payment_date > due_date` | Late Payment Rate (headline) |
| `days_late` | Days between the due date and the payment date. Positive = late, 0 = paid on the due date, negative = paid early | `(payment_date - due_date).dt.days` | Median days late, late and early groups |
| `branch_id` | Branch of the invoice, brought into the merged table | `merged['invoice_id'].map(invoices_clean.set_index('invoice_id')['branch_id'])` (safe because `invoice_id` is unique in `invoices_clean`) | Late rate by branch |
| `customer_id` | Customer of the invoice | Same `map` from `invoices_clean` | Customer concentration check |
| `due_year` | Year the invoice was due | `due_date.dt.year` | Late rate by year, 2025 check |
| `invoice_date` | Invoice date as a real date | `pd.to_datetime`, then `map` from `invoices_clean` | Payment-before-invoice check |

**Metrics**

| Metric | How it is calculated | Result |
|--------|----------------------|--------|
| Late Payment Rate (headline) | `is_late.mean()`. Confirmed by `query('payment_date > due_date')` and by `(days_late > 0).sum() / len`. All three agree | 41.50% (7,655 of 18,445) |
| Median days late | Median of `days_late` among late payments only | 17 days (middle half 9 to 28) |
| Mean days late | Mean of `days_late` among late payments, three ways | Supporting number |
| `by_branch` | Per branch: `payments` (count), `late_rate` (mean of `is_late`), `median_days_late` | Chennai highest at 48.61% |
| `by_year` | Per due year: `payments`, `late_rate`, `median_days_late` | Year-by-year view |
| Customer share of payments | `customer_id.value_counts(normalize=True)` | Concentration check |
| Check flags | `pay_before_inv` (payment_date before invoice_date), due date before invoice date, late rate without those rows | Resolved the 208 payment-before-invoice records |

---

## 6. KPI 5 Product Sales Velocity (sales, purchase and product files)

### 6.1 Working tables

| Table | Meaning | How it is built |
|-------|---------|-----------------|
| `sales` | Delivered sales lines with date and product name | `sales_orders_lines` joined to Delivered headers on `so_id`, product name added from `products` (117,656 lines) |
| `bought_lines` | Received purchase lines | `purchase_orders_lines` joined to Received orders on `po_id` (140,077 lines) |
| `s` | Delivered sales lines with branch | `sales` joined to the sales header on `so_id` with `validate='many_to_one'` |
| `b` | Received purchase lines with branch and month | Purchase lines joined to Received headers. Header branch is renamed `hdr_branch` |
| `bought_d`, `sales_b`, `bought_b` | Purchase lines with dates, sales lines with branch, purchase lines with branch | Joins to the headers on `po_id` or `so_id` |

### 6.2 Columns added to the line tables

| Feature | Table | Meaning | How it is calculated |
|---------|-------|---------|----------------------|
| `order_date` (converted) | `sales`, `bought_d` | Order date as a real date | `pd.to_datetime` |
| `month` | `sales`, `bought_d` | First day of the order's month | `order_date.dt.to_period('M').dt.to_timestamp()`. In `s` and `b` it is the monthly period (`to_period('M')`) |
| `year` | `sales`, `bought_d` | Calendar year of the order | `order_date.dt.year` |
| `product_name` | `sales` | Product name | Left join from `products` on `product_id` |
| `n_lines` | `ln`, `bb` (checks) | How many lines the same product has inside one order | `groupby(['so_id','product_id'])['line_number'].transform('size')` (purchases: `groupby(['po_id','product_id'])['quantity'].transform('size')`) |

### 6.3 Monthly table `monthly` (2,160 rows = 30 products x 72 months)

| Feature | Meaning | How it is calculated |
|---------|---------|----------------------|
| `units_sold` | Units sold for one product in one month | Sum of `quantity` per `product_id`, `product_name`, `month` |

### 6.4 Per-product tables

| Feature | Table | Meaning | How it is calculated |
|---------|-------|---------|----------------------|
| `total_units` | `velocity` | Total units sold for the product over 72 months | Sum of `units_sold` |
| `avg_units_per_month` (M1) | `velocity` | Average units sold per month | Mean of the 72 `units_sold` values, rounded to 1 decimal (equals total / 72 because every product sold in every month) |
| `share_pct` | `velocity` | Product's share of all units sold | `total_units / sum(total_units) * 100`, rounded to 2 decimals |
| `2019` to `2024` | `yearly` | Units sold per product per year | `pivot_table` of `quantity` by year |
| `change_2019_to_2024_pct` | `yearly` | Change from 2019 to 2024 (noisy, not used for the headline) | `(2024 / 2019 - 1) * 100`, rounded to 1 decimal |
| `first_3_years` | `yearly` | Units in 2019 to 2021 | Sum of the 2019, 2020, 2021 columns |
| `last_3_years` | `yearly` | Units in 2022 to 2024 | Sum of the 2022, 2023, 2024 columns |
| `block_change_pct` (M2, sales trend) | `yearly` | Change from the first 3 years to the last 3 years | `(last_3_years / first_3_years - 1) * 100`, rounded to 1 decimal |
| `units_bought` | `bought`, `compare` | Units bought per product on Received purchase lines | Sum of purchase `quantity` per `product_id` |
| `units_sold` | `compare` | Units sold per product | Taken from `velocity['total_units']` |
| `bought_per_1_sold` (M4) | `compare` | Units bought for every 1 unit sold | `units_bought / units_sold`, rounded to 1 decimal |
| `share_of_bought_pct` | `compare` | Product's share of all units bought | `units_bought / sum(units_bought) * 100`, rounded to 2 decimals |
| `avg_per_month` | `steady` | Average monthly units | Mean of `units_sold` per product, rounded to 1 decimal |
| `swing` | `steady` | How much monthly units move around the average | Standard deviation of `units_sold` per product (sample standard deviation) |
| `bumpiness_pct` (M3, steadiness) | `steady` | Swing as a percentage of the average. Lower = steadier | `swing / avg_per_month * 100`, rounded to 1 decimal |

### 6.5 Buying-follows-selling tables

| Feature | Table | Meaning | How it is calculated |
|---------|-------|---------|----------------------|
| `units_sold`, `units_bought`, `bought_per_1_sold` | `by_year` | Sold, bought and the ratio for each year | Sums of `quantity` per `year`, then bought / sold |
| `sold`, `bought` | `by_month` | Units sold and bought in each of the 72 months | Sums of `quantity` per `month` |
| `sales_change_pct`, `buying_change_pct` | `chg` | Per product, change in units sold and in units bought (2019-21 vs 2022-24) | `(last 3 years / first 3 years - 1) * 100` on sales and on purchases |
| Month link (correlation) | scalar | How closely monthly sold and monthly bought move together (-1 to +1) | `by_month['sold'].corr(by_month['bought'])` = 0.28 |
| Product link (correlation) | scalar | How closely the change in a product's sales matches the change in its buying | `chg['sales_change_pct'].corr(chg['buying_change_pct'])` = -0.36 |
| High-sales and low-sales months | `high`, `low` | The 36 months above the median sales and the 36 at or below | Split of `by_month` at `by_month['sold'].median()` |
| Same-way share | scalar | Share of month-to-month moves where sales and buying went the same direction | `moves = by_month.diff()`, count where `(sold > 0) == (bought > 0)` / 71 = 59.2% |

### 6.6 Branch and product-branch tables

| Feature | Table | Meaning | How it is calculated |
|---------|-------|---------|----------------------|
| `units_sold`, `units_bought`, `bought_per_1_sold` | `by_branch` | Sold, bought and the ratio per branch | Sums per `branch_id`, then bought / sold, rounded to 1 decimal |
| `units_sold`, `units_bought` | `pb` (180 rows) | Sold and bought for one product in one branch | Sums per `product_id` and `branch_id` |
| `bought_per_1_sold` | `pb` | Over-buying number per product-branch pair | `units_bought / units_sold`, rounded to 1 decimal |
| `pairs`, `lowest`, `median`, `highest`, `pairs_above_20` | `summary` | Per branch, the spread of its 30 product ratios and how many are above 20 | `groupby('branch_id').agg(...)` on `pb['bought_per_1_sold']` |

### 6.7 Velocity label (M5)

| Feature | Table | Meaning | How it is calculated |
|---------|-------|---------|----------------------|
| `avg_units_per_month` | `pb` | Average units sold per month for the pair | `units_sold / 72`, rounded to 1 decimal (for display) |
| `exact_avg` | `pb` | The same average without rounding. The label uses this | `units_sold / 72` |
| `velocity_group` (M5, final) | `pb` | Slow, Medium or Fast for the product-branch pair | The 180 pairs are cut into equal thirds. Cut-offs are the 1/3 and 2/3 quantiles of `exact_avg` (85.8287 and 102.3287). `pd.cut(exact_avg, [-inf, lo, hi, inf], labels=['Slow','Medium','Fast'])` |
| `velocity_group_old` | `pb` | The first label, kept for the record | `pd.qcut` on the rounded average. This version put P008 Bucket Tooth BT-10 in KOL001 on the cut-off (Medium). The corrected label is Fast |

### 6.8 Check-only features (built to test the KPI, not part of the reported metrics)

| Feature | Meaning | How it is calculated |
|---------|---------|----------------------|
| `pm` | Months with sales (or purchases) per product | `groupby('product_id')['month'].nunique()` |
| `pbm` | Months with sales (or purchases) per product-branch pair | `groupby(['product_id','branch_id'])['month'].nunique()` |
| `out` (gap table) | For each pair with an empty month: months with sales, missing months, total units, average over 72 months, average over active months | Built row by row from `s` |
| `pb2['avg_exact']`, `pb2['group_exact']` | Second-method average and label for Check 5 | `units_sold / 72`, then `pd.cut` with the exact cut-offs |
| `near` | Pairs within 1 unit of a cut-off | `abs(avg_exact - cut-off) < 1` for either cut-off (18 pairs) |
| `chance` | Chance two random lines share a quantity | `sum(p ** 2)` where `p` is the share of each quantity value (5.0% sales, 0.36% purchases) |
| `copies` | Lines that are exact copies of another line, ignoring `line_number` | `duplicated(subset=all columns except line_number, keep=False).sum()` (1,374) |
| `ratio` (line size) | Largest line compared with the average line, per product | `max(quantity) / mean(quantity)` |
| `worst_month`, `trim` | Each product's most extreme month, and steadiness without it | `piv.sub(mean).abs().idxmax(axis=1)`, then steadiness on the other 71 months |
| `piv` | Product-by-month table of units (72 months per product) | `pivot_table(index='product_id', columns='m', values='quantity', aggfunc='sum', fill_value=0)` |
| `bump` | Steadiness recomputed from `piv` | `piv.std(axis=1, ddof=1) / piv.mean(axis=1) * 100` |
| `lines` (per month), `bm` (per branch-month), `pm` (purchases per month) | Number of order lines per month and per branch-month | `groupby(...).size()` |

### 6.9 KPI 5 headline results (for reference)

| Metric | Field | Result |
|--------|-------|--------|
| M1 Average units sold per month | `avg_units_per_month` | Products 548 to 593. Pairs 70.6 to 123.1 |
| M2 Sales trend | `block_change_pct` | Overall -3.4%. 6 up, 24 down |
| M3 Steadiness | `bumpiness_pct` | 13.9% to 19.1%, median 17.06% |
| M4 Bought per 1 sold | `bought_per_1_sold` | Overall 18.1. Products 17.4 to 19.2. Branches 15.0 to 22.0. Pairs 13.4 to 25.5 |
| M5 Velocity label | `velocity_group` | 60 Slow, 60 Medium, 60 Fast |

---

## 7. Features decided in Week 3 but not yet built

These come from the data cleaning log. They are decisions, and the columns have not been created in a notebook yet.

| Feature | Meaning | How it will be calculated | Planned for |
|---------|---------|---------------------------|-------------|
| `margin` (recomputed) | Profit as a share of selling price. The `margin_percentage` column in `products.csv` mixes two definitions, so it is not used | `(unit_price - unit_cost) / unit_price` | Margin and capital KPIs (Weeks 5 and 6) |
| `warehouse_capacity` (numeric) | Capacity as a number. The raw column is text such as "45230 sqft" | Remove " sqft" and convert to a number | KPI 12 Warehouse Space Utilisation |
| Reconstructed stock balance | Real stock level per product and branch, because `current_stock` is inflated | Opening stock plus the movements, built from the order lines and the ledger | KPI 8 (Week 4) |

---

## 8. Notes

1. **Dates are text in the raw files.** Every date column was converted with `pd.to_datetime` before use (all 12 date columns convert with 0 failures).
2. **Units are compared within a product only.** The unit of measure mixes piece (24 products), set (5) and cartridge (1), so totals across products add mixed units.
3. **Delivered and Received only.** Sales features use Delivered orders. Purchase features use Received orders. Cancelled orders keep their lines in the lines files, so the filter is required.
4. **Duplicate IDs.** KPI 4 features are built on the clean tables, after the repeated `invoice_id` and `payment_id` rows found in KPI 2 were excluded.
5. **Labels use unrounded averages.** The rounded average is only for display.
6. **Fields still to be added.** Features for KPI 6 to KPI 22 are added to this list as each KPI is built.
