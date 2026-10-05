# KPI 5: Product Sales Velocity

**Project:** Heavy Supplier & Warehouse Analytics (CadetX) | **Week:** 3 | **Notebook:** Week3_KPI05_Product_Sales_Velocity

---

## 1. KPI Definition

**Business question:** Which products sell fast and which sell slowly, and does our buying follow our selling?

KPI 5 measures how quickly each product sells, per product and per branch, over 72 months (January 2019 to December 2024). It has five metrics. All five were verified.

| # | Metric | Definition | Result |
|---|--------|------------|--------|
| M1 | Average units sold per month (headline) | Total units on Delivered sales lines divided by 72 months, per product, per branch and per product-branch pair. An empty month counts as 0. | Products: 548 to 593 units a month. Product-branch pairs: 70.6 to 123.1 |
| M2 | Sales trend | Units sold in 2022-24 compared with 2019-21, per product | Overall -3.4% (628,201 to 606,738). 6 products up, 24 down. Range +4.3% (P006 Alternator ALT-500) to -9.5% (P028 Light Assembly LA-18) |
| M3 | Steadiness | Standard deviation of monthly units divided by the average, as a percentage (lower is steadier) | 13.9% (P029 Pressure Sensor PS-90) to 19.1% (P005 Fuel Injector FI-220), median 17.06% |
| M4 | Bought per 1 sold (over-buying number) | Units on Received purchase lines divided by units on Delivered sales lines | Overall 18.1. Products 17.4 to 19.2. Branches 15.0 to 22.0. Product-branch pairs 13.4 to 25.5 |
| M5 | Velocity label | Slow, Medium or Fast: the 180 product-branch pairs split into equal thirds by M1 | Cut-offs 85.83 and 102.33 units a month. 60 pairs in each group |

**Why these metrics and not a single volume ranking:** all 30 products sell almost the same volume. Each is 3.20% to 3.46% of units sold (an equal share would be 3.33%). A volume ranking would give grades that are not meaningful. The real differences were found by adding the trend, steadiness, buying and the branch split.

---

## 2. Methodology

### 2.1 Sources and scope

- `sales_orders_lines.csv` joined to `sales_orders_header.csv` on `so_id` (order status, branch, order date)
- `purchase_orders_lines.csv` joined to `purchase_orders_header.csv` on `po_id` (order status, branch, order date)
- `products.csv` (product names)
- Sales: Delivered orders only. 18,033 orders, 117,656 lines, 1,234,939 units. Cancelled sales orders (1,967) keep 12,746 lines in the lines file, so the filter is required (130,402 - 12,746 = 117,656).
- Purchases: Received orders only. 21,630 orders, 140,077 lines, 22,387,123 units. Cancelled purchase orders (2,370) keep 15,418 lines (155,495 - 15,418 = 140,077).
- Months are taken from the order date: 72 months, 2019-01 to 2024-12. Branch comes from the order header. The units column is `quantity` in both lines files.

### 2.2 What was built, step by step

1. **Delivered sales table.** Lines joined to Delivered headers, product names added. Result: 18,033 orders, 117,656 lines, 1,234,939 units, 0 lines without a product name, order dates from 2019-01-01 to 2024-12-31.
2. **Monthly table.** Units per product per month: 2,160 rows = 30 products x 72 months. Every product sold in every month, and the total ties to 1,234,939.
3. **Average per product.** All 30 products sell 548 to 593 units a month (P029 Pressure Sensor PS-90 highest, P024 Track Roller TR-60 lowest), each 3.20% to 3.46% of units sold.
4. **Yearly view and trend.** Units per year, then per product. The 2019-vs-2024 view proved noisy, so the 3-year-block view (2019-21 vs 2022-24) is used.
5. **Buying side.** Received purchase orders: 21,630 orders, 140,077 lines, 22,387,123 units, 30 products.
6. **Bought vs sold per product.** The ratio per product and each product's share of units bought.
7. **Does buying follow selling?** Three tests: by year, by month and by product.
8. **Branch and product-branch split.** Sold, bought and the ratio per branch, then for all 180 product-branch pairs, then within each branch.
9. **Steadiness.** Bumpiness of the monthly units for each of the 30 products over 72 months.
10. **Labels.** Slow, Medium and Fast for the 180 pairs (equal thirds of the average units a month).
11. **Verification.** 8 checks (section 3). Corrections that came out of the work are listed in section 4, notes 4 and 15.

### 2.3 Decisions

1. Order lines are the source for demand and purchases, not the stock ledger (the ledger holds only about 90% of order lines, see note 1).
2. Cancelled orders are excluded.
3. Labels are made at product-branch level (180 pairs), because all 30 products look alike and labels by product would carry no information.
4. Labels are calculated from unrounded averages (units sold divided by 72).
5. Units are compared within a product only, because the unit of measure mixes piece, set and cartridge.

### 2.4 Alternatives rejected

- **Ledger OUT movements as demand:** about 10% of order lines never reached the ledger.
- **Fast/Medium/Slow by volume per product:** products differ by only 8%.
- **Trend from 2019 vs 2024:** single years bounce 5% to 10% per product, and this view misled for three products (Radiator, O-Ring Set and Engine Oil Filter looked up in this view but are down in the block view).
- **Fixed thresholds chosen by hand:** the equal-thirds cut-offs come from the data.
- **"Over-buying is spread evenly" as the main finding:** it held across products but not across branches (note 15).

---

## 3. Verification Log

**Coverage.** Checks 1 to 7 were run on all five metrics wherever they apply, on the sales side and the purchase side. The manual spot-check (Check 8) rebuilt units sold, units bought, the monthly average and the label for three pairs from the raw files. It also rebuilt the trend for the two extreme products (P006 and P028) and the steadiness for the two extreme products (P029 and P005). The trend and steadiness of the other 28 products were recomputed from the raw lines in Check 5 but were not hand-checked.

| # | Check | Method | Result |
|---|-------|--------|--------|
| 1 | Missing, zero and negative values | Counted empty, zero and negative values in every column used | 0 empty in every column used. Sales quantity runs 1 to 20, purchase quantity 20 to 300. 0 zero, 0 negative |
| 2 | Empty months | Counted months with sales and with purchases per product and per product-branch pair | All 30 products have sales and purchases in all 72 months. Sales: 4 of 180 pairs have 1 or 2 empty months. Purchases: 3 of 180 pairs have 1 empty month. The branch was active in every one of these months, so these are quiet months, counted as 0 |
| 3 | Repeated lines are not copies | Compared how often a product repeated on 2 lines of one order has the same quantity with the rate expected by chance | Sales: 11,202 combos with 2 lines, 547 same quantity = 4.88% (chance 5.0%). Purchases: 13,219 combos, 53 = 0.40% (chance 0.36%). Matches are coincidence, not double counting |
| 4 | Totals reconcile | Compared totals in the working tables with the verified Week 3 totals | Sold 18,033 orders / 117,656 lines / 1,234,939 units. Bought 21,630 / 140,077 / 22,387,123. The monthly table and the 180-pair table add up to the same totals |
| 5 | Second method | Recomputed each metric another way | See 3.1. All five metrics matched |
| 6 | Outliers | Largest line vs average line per product. Steadiness without each product's worst month. Pairs near a cut-off | Largest sales line is 1.86 to 1.95 times the average line. Largest purchase line is 1.85 to 1.92 times. Without the worst month, median steadiness goes from 17.06 to 16.44, biggest drop for any product 1.26 points. 18 of 180 pairs are within 1 unit of a cut-off |
| 7 | History length | Counted months and lines per month and per branch-month, sales and purchases | 72 months for both. First and last month are complete. Lowest branch-month is 82 sales lines (HYD001, 2020-02), median 267 |
| 8 | Manual spot-check | Rebuilt three pairs, the trend of two products and the steadiness of two products from the raw CSV files | All equal the saved numbers (see 3.1) |

### 3.1 Details of each check

**Check 1 (missing, zero, negative).** Files checked: sales lines (130,402 rows), sales header (20,000 rows), purchase lines (155,495 rows), purchase header (24,000 rows), products (30 rows, no empty names). Sales quantity min 1, max 20. Purchase quantity min 20, max 300.

**Check 2 (empty months).**
- Products: months with sales per product min 72, max 72. Months with purchases per product min 72, max 72.
- Sales pairs (180): months with sales per pair min 70, max 72. The four pairs with an empty month:

| Pair | Months with sales | Empty month(s) | Total units | Average over 72 months | Average over active months |
|------|------|------|------|------|------|
| P004 Track Chain TC-45, HYD001 | 70 | 2021-01 and 2024-06 | 5,148 | 71.50 | 73.54 |
| P008 Bucket Tooth BT-10, KOL001 | 71 | 2023-02 | 7,369 | 102.35 | 103.79 |
| P018 Wheel Rim WR-24, HYD001 | 71 | 2020-02 | 5,455 | 75.76 | 76.83 |
| P021 Boom Cylinder BC-400, HYD001 | 71 | 2023-08 | 5,999 | 83.32 | 84.49 |

- Was the branch active in those months? Branch lines in the empty month vs lines of that product: P004 HYD001 2021-01 172 vs 0, P004 HYD001 2024-06 180 vs 0, P008 KOL001 2023-02 246 vs 0, P018 HYD001 2020-02 82 vs 0, P021 HYD001 2023-08 233 vs 0. An average branch-month has about 272 sales lines (117,656 / 432). The branch was active and only that product had no sales: a quiet month, not lost data.
- Purchase pairs (180): months with purchases per pair min 71, max 72. The three pairs with an empty month: P003 Air Filter AF-120 in KOL001 in 2021-06 (the branch had 214 purchase lines that month, this product 0), P016 Radiator RD-250 in KOL001 in 2023-08 (282 vs 0), P023 Idler Wheel IW-55 in DEL001 in 2019-09 (210 vs 0). An average branch-month has about 324 purchase lines (140,077 / 432).
- The saved averages already divide by 72, so an empty month counts as 0 and nothing changed. Bought per 1 sold uses period totals, so an empty month does not affect it.

**Check 3 (repeated lines).**
- 12,184 order-product combos appear on 2 or more lines, involving 25,422 lines. This ties to the earlier pair count (25,422 - 12,184 = 13,238 = 117,656 lines - 104,418 order-product pairs).
- 1,374 lines are exact copies of another line (ignoring the line number).
- Sales: among the 11,202 combos with exactly 2 lines, 547 have the same quantity = 4.88%. Quantities run 1 to 20, so two random lines share a quantity 5.0% of the time (about 560 expected). The observed rate equals the chance rate.
- Purchases: among the 13,219 combos with exactly 2 lines, 53 have the same quantity = 0.40%. Chance rate 0.36% (about 48 expected).
- Combos with 3 or more lines were not tested separately (sales: 982 of the 12,184 repeated combos).

**Check 4 (totals reconcile).** Sold: 18,033 orders, 117,656 lines, 1,234,939 units. Bought: 21,630 orders, 140,077 lines, 22,387,123 units. Monthly table units_sold 1,234,939. Pair table units_sold 1,234,939 and units_bought 22,387,123. Yearly sales add to 1,234,939. Yearly purchases add to 22,387,123. Branch totals add to the same figures (0 lines without a branch).

**Check 5 (second method).**
- M1: average per product from the monthly table vs raw lines divided by 72, 30 products, largest difference 0.0. Per pair (180 pairs) the largest difference is 0.05, which is rounding to 1 decimal.
- M4: bought per 1 sold from raw lines vs the saved table, 0 pairs without a ratio, largest difference 0.05 (rounding).
- M2: recomputed from raw lines. First 3 years 628,201, last 3 years 606,738, -3.4%, 6 products up, 24 down, biggest rise +4.3% (P006 Alternator ALT-500), biggest fall -9.5% (P028 Light Assembly LA-18).
- M3: recomputed from a product-by-month table (72 months per product): lowest 13.9 (P029 Pressure Sensor PS-90), median 17.06, highest 19.1 (P005 Fuel Injector FI-220).
- M5: relabelled from unrounded averages. Cut-offs 85.8287 and 102.3287, groups 60 / 60 / 60, 1 label corrected (P008 Bucket Tooth BT-10 in KOL001, exact average 102.347, Medium to Fast). All 180 saved averages equal units sold / 72.

**Check 6 (outliers).**
- Sales: largest line 20, average line 10.5. Largest line / average line per product from 1.86 to 1.95.
- Purchases: largest line 300, average line 159.82. Largest line / average line per product from 1.85 to 1.92.
- Steadiness with all 72 months: lowest 13.9, median 17.06, highest 19.1. Without each product's worst month: lowest 13.5, median 16.44, highest 18.4. Biggest drop for any product 1.26 points.
- 18 of 180 pairs lie within 1 unit of a cut-off.

**Check 7 (history).**
- Sales: 72 months, 2019-01 to 2024-12. Lines per month: lowest 1,334, median 1,614, highest 2,126. First month 1,679, last month 1,488. Lines per branch-month: lowest 82 (HYD001, 2020-02), median 267, highest 539.
- Purchases: 72 months, 2019-01 to 2024-12. Lines per month: lowest 1,637, median 1,944, highest 2,193. First month 2,002, last month 2,082.

**Check 8 (manual spot-check).** Rebuilt from the raw CSV files (Delivered and Received only):

| Pair | Role | Sold | Bought | Average a month | Label |
|------|------|------|------|------|------|
| P023 Idler Wheel IW-55, CHN001 | Fastest pair | 8,861 | 124,901 | 123.07 (saved 123.1) | Fast |
| P023 Idler Wheel IW-55, HYD001 | Slowest pair | 5,081 | 119,764 | 70.57 (saved 70.6) | Slow |
| P002 Engine Oil Filter OF-90, HYD001 | Highest bought per 1 sold of all 180 pairs (25.5) | 5,413 | 138,227 | 75.18 (saved 75.2) | Slow |

All sold and bought figures equal the saved table. The averages differ only by rounding to 1 decimal.

Trend rebuilt from the raw files (Delivered orders joined to the header dates; 117,656 rows, 1,234,939 units): P006 Alternator ALT-500 had 20,484 units in 2019-21 and 21,359 in 2022-24 (+4.3%). P028 Light Assembly LA-18 had 21,678 units in 2019-21 and 19,614 in 2022-24 (-9.5%). Steadiness rebuilt from the raw files over all 72 months: P029 Pressure Sensor PS-90 averages 593.1 units a month with bumpiness 13.9%, and P005 Fuel Injector FI-220 averages 562.2 with bumpiness 19.1%. All equal the verified numbers.

---

## 4. Data Quality Notes

1. **Order lines, not the stock ledger.** The ledger holds only about 90% of order lines, missing at random. Delivered sales lines are 117,656 against 105,890 non-cancelled OUT rows in the ledger (90.0%). Received purchase lines are 140,077 against 126,069 IN rows (90.0%). Units: 1,234,939 on sales lines against 1,111,598 in the ledger OUT (ledger 10.0% lower), and 22,387,123 on purchase lines against 20,149,239 in the ledger IN. About 9.0% of order-product pairs never reached the ledger. Order lines are complete, so all five metrics use them. The ratio of in to out is the same from either source (18.13).
2. **Cancelled orders excluded.** 1,967 sales orders (12,746 lines) and 2,370 purchase orders (15,418 lines).
3. **Empty months.** 4 sales pairs and 3 purchase pairs have a month with no sales or no purchases (section 3.1). The branch was active in those months, so they are counted as 0. Bought per 1 sold uses period totals and is not affected.
4. **A rounding tie was fixed.** P008 Bucket Tooth BT-10 in KOL001 has an exact average of 102.347, just above the Fast cut-off of 102.33. The saved average was rounded to 1 decimal (102.3), which put it on the cut-off and gave it Medium. The groups were 61 / 60 / 59 before the fix and are now 60 / 60 / 60. Labels now use unrounded averages.
5. **Labels are relative.** Equal thirds always give 60 / 60 / 60 pairs, and 18 of 180 pairs lie within 1 unit of a cut-off. Between the fastest and slowest pair the gap is about 74% (70.6 to 123.1), against 8% between products.
6. **Mixed units.** Products are counted in pieces (24 products), sets (5) and cartridges (1). Comparisons are made within a product, and the labels per product-branch pair are not affected. Branch totals add mixed units, so the pair-level ratios (13.4 to 25.5) are used to confirm the branch pattern.
7. **Weak links.** The month-by-month correlation of sales and buying is 0.28 (72 months). The product-level correlation of sales change and buying change is -0.36 over only 30 products. Sales and buying moved in the same direction in 42 of 71 month-to-month moves (59.2%, against 50% by chance). These are not strong evidence and no cause is claimed.
8. **No 2025.** Sales orders and purchase orders end on 2024-12-31, so there is no 2025 data in this KPI.
9. **Low branch-month.** HYD001 in 2020-02 has only 82 sales lines (typical branch-month: 267). The cause is not known. It is 1 of 432 branch-months and was kept.
10. **Coverage of Check 3.** The repeated-lines test covered combos with exactly 2 lines. Combos with 3 or more lines were not tested separately (sales: 982 of 12,184 repeated combos).
11. **Coverage of Check 6.** The trend (M2) and the over-buying ratio (M4) had no outlier test of their own beyond the line-size test and the second-method recompute.
12. **Coverage of Check 8.** The spot-check rebuilt units sold, units bought, the monthly average and the label for three pairs. It also rebuilt the trend (P006, P028) and the steadiness (P029, P005) for the extreme products. The trend and steadiness of the other 28 products were recomputed from raw lines in Check 5 but were not hand-checked.
13. **Steadiness median.** The first run gave a median of 17.05 and the recompute gave 17.06. The 0.01 gap is most likely rounding of a value between the two. It was not tested further. The value 17.06 is used throughout this document.
14. **Single years are noisy.** Product sales in a single year bounce 5% to 10%, which is why the 3-year-block trend is used. The 2019-vs-2024 view misled for three products (section 2.4).
15. **Corrections made during the work.**
    - The no-gap assumption at pair level was wrong for 4 sales pairs and 3 purchase pairs (note 3).
    - P008 KOL001 was first described as Fast either way, but the saved table had it as Medium because of the rounding tie (note 4).
    - An early reading said over-buying is spread evenly. That held across products (17.4 to 19.2) but not across branches (15.0 to 22.0), which became the main finding after the branch split.
16. **Product names.** The product IDs used in this document were matched against `products.csv` (a lookup run in the notebook), so each ID is paired with its verified name.

---

## 5. Business Interpretation

### 5.1 Products sell alike

All 30 products sell between 548 and 593 units a month, and each is 3.20% to 3.46% of units sold. P029 Pressure Sensor PS-90 is highest and P024 Track Roller TR-60 is lowest. Volume alone does not separate fast and slow products.

### 5.2 Sales trend

| Year | Units sold |
|------|-----------|
| 2019 | 207,925 |
| 2020 | 212,007 |
| 2021 | 208,269 |
| 2022 | 202,242 |
| 2023 | 204,638 |
| 2024 | 199,858 |

Sales fell 3.9% from 2019 to 2024. Per product, 2019 to 2024 gives 8 up, 1 flat and 21 down. The biggest changes in that view were Alternator ALT-500 (+14.7%, a jump in 2020 then flat), Brake Pad BP-40 (-15.5%, a steady fall) and Swing Motor SWM-350 (-14.0%, bumpy).

That two-year view is noisy, so the block view is used: 2019-21 had 628,201 units and 2022-24 had 606,738 units, a fall of 3.4%. 6 products grew and 24 fell. 14 products are within +/-3%. Alternator ALT-500 (P006) was the only clear riser (+4.3%). The biggest falls were Light Assembly LA-18 (P028, -9.5%), Cabin Glass (-9.0%), Fuel Injector (-8.2%) and Starter Motor (-8.0%).

### 5.3 Steadiness

Bumpiness runs from 13.9% (P029 Pressure Sensor PS-90, also the top seller at 593.1 units a month) to 19.1% (P005 Fuel Injector FI-220, 562.2 units a month), with a median of 17.06%. The next bumpiest are FP-350 (18.6%), Radiator RD-250 (18.5%) and Cabin Glass CG-20 (18.4%). Products differ by only about 5 points, so no product is clearly erratic.

### 5.4 Bought vs sold per product

Overall, 18.1 units are bought for every 1 sold (consistent with KPI 1, about 18 units in for every unit out). Across products the ratio runs only from 17.4 (Swing Motor SWM-350) to 19.2 (Track Roller TR-60). Each product is 3.23% to 3.42% of units bought (an equal share is 3.33%), and no product has zero purchases. So no particular set of products is over-bought: across products the over-buying is spread evenly.

### 5.5 Does buying follow selling?

**By year**

| Year | Units sold | Units bought | Bought per 1 sold |
|------|-----------|--------------|-------------------|
| 2019 | 207,925 | 3,736,316 | 18.0 |
| 2020 | 212,007 | 3,805,018 | 17.9 |
| 2021 | 208,269 | 3,759,011 | 18.0 |
| 2022 | 202,242 | 3,687,415 | 18.2 |
| 2023 | 204,638 | 3,741,449 | 18.3 |
| 2024 | 199,858 | 3,657,914 | 18.3 |

Sold and bought moved in the same direction in all 5 year-to-year steps. From 2019 to 2024 buying fell 2.1% and selling fell 3.9%, so the ratio crept up from 18.0 to 18.3.

**By month.** The correlation of monthly sold and bought units over 72 months is 0.28, a weak link. In the 36 high-sales months, average units sold were 18,191 and units bought 314,089. In the 36 low-sales months, units sold were 16,113 and units bought 307,775. Sales were about 13% higher in the high months but buying was only 2.1% higher. Of 71 month-to-month moves, sales and buying moved the same way in 42 (59.2%, against about 50% for a coin flip). Buying runs near a steady 311,000 units a month.

**By product (2019-21 vs 2022-24).** The correlation between the change in a product's sales and the change in its buying is -0.36 over 30 products: no positive link. 9 products had falling sales and rising buying, for example Cabin Glass CG-20 (sales -9.0%, buying +1.7%) and Fuel Injector FI-220 (sales -8.2%, buying +2.0%). Alternator ALT-500 had sales +4.3% but buying -9.9%.

**Reading.** Total buying follows total demand loosely by year, but buying is not set product by product from sales. It stays near 18 times sales whatever happens.

### 5.6 The branch picture

| Branch | Units sold | Units bought | Bought per 1 sold |
|--------|-----------|--------------|-------------------|
| CHN001 | 253,921 | 3,797,868 | 15.0 |
| KOL001 | 230,868 | 3,661,187 | 15.9 |
| AHM001 | 214,223 | 3,681,090 | 17.2 |
| DEL001 | 185,319 | 3,777,619 | 20.4 |
| PUN001 | 181,633 | 3,750,931 | 20.7 |
| HYD001 | 168,975 | 3,718,428 | 22.0 |

Branch sales differ a lot (Chennai sold about 50% more than Hyderabad), but each branch buys almost the same (3.66 to 3.80 million units, a 3.7% spread). So the slowest-selling branches carry the highest over-buying.

**Product-branch pairs (180 pairs, none missing).** The ratio runs from 13.4 (lowest) to 18.55 (median) to 25.5 (highest). 4 of the 5 highest pairs are in Hyderabad:

| Pair | Bought per 1 sold |
|------|-------------------|
| P002 Engine Oil Filter OF-90, HYD001 (5,413 sold, 138,227 bought) | 25.5 |
| P024 Track Roller TR-60, DEL001 | 25.1 |
| P017 Fan Belt FB-30, HYD001 | 24.6 |
| P004 Track Chain TC-45, HYD001 | 24.3 |
| P014 Transmission Assembly TA-800, HYD001 | 23.8 |

**Within each branch (30 product ratios per branch)**

| Branch | Lowest | Median | Highest | Pairs above 20 |
|--------|--------|--------|---------|----------------|
| AHM001 | 15.7 | 17.2 | 19.5 | 0 |
| CHN001 | 13.6 | 15.05 | 16.6 | 0 |
| DEL001 | 18.0 | 20.35 | 25.1 | 19 |
| HYD001 | 19.2 | 21.7 | 25.5 | 27 |
| KOL001 | 13.4 | 16.1 | 17.3 | 0 |
| PUN001 | 18.8 | 20.7 | 23.2 | 19 |

65 of the 180 pairs are above 20, all in Hyderabad, Delhi and Pune (the three lowest-selling branches). None are in Ahmedabad, Chennai or Kolkata. Each branch's products cluster around the branch's own ratio: Chennai's highest (16.6) is below Hyderabad's lowest (19.2). Delhi and Hyderabad also have the widest spread inside the branch (7.1 and 6.3 points). The over-buying is a branch effect.

### 5.7 Slow, Medium and Fast

Average units a month for the 180 pairs run from 70.6 (slowest) to 123.1 (fastest), a gap of about 74%, against 8% between products. Cut-offs: Slow below 85.83, Fast above 102.33.

| Branch | Slow | Medium | Fast |
|--------|------|--------|------|
| AHM001 | 0 | 24 | 6 |
| CHN001 | 0 | 0 | 30 |
| DEL001 | 13 | 17 | 0 |
| HYD001 | 29 | 1 | 0 |
| KOL001 | 0 | 6 | 24 |
| PUN001 | 18 | 12 | 0 |

The labels follow the branch: every product is Fast in Chennai and 29 of 30 are Slow in Hyderabad. The same product, P023 Idler Wheel IW-55, is both the fastest pair (Chennai, 8,861 units sold) and the slowest pair (Hyderabad, 5,081 units sold). Velocity depends on the branch, not the product.

### 5.8 What this suggests

- Products behave alike. Speed differs by branch.
- Buying runs near a steady level (about 311,000 units a month and about 3.7 million units per branch) whatever each branch sells, so Hyderabad, Pune and Delhi carry the over-buying.
- Buying is not set product by product from sales.
- Sales are drifting down (-3.4%), and buying has fallen less.
- Buying should be set per branch from that branch's own sales, starting with a review of Hyderabad, Pune and Delhi.
- This links to KPI 1 (about 18 units in for every unit out) and to KPI 8 and KPI 6 in Week 4, which will show how much surplus stock is really sitting in each branch.

### 5.9 What this data cannot tell us

- Why Hyderabad, Pune and Delhi sell less, or why the branches buy almost the same amount.
- Whether the extra stock is physically sitting in those branches (KPI 8 will test this).
- Why Hyderabad had only 82 sales lines in February 2020.
