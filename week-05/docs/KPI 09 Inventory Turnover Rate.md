# KPI 9: Inventory Turnover Rate

Project: Heavy Supplier & Warehouse Analytics | Week 5 | Notebook: Week5_KPI09_Inventory_Turnover_Rate

## Summary

- Inventory turnover fell about 11 times, from 0.114 in 2019 to 0.0102 in 2024.
- The cause is stock growing while sales stay flat. Stock grew from 32,969 units before 2019 to 21,185,215 units at the end of January 2025, while sales stayed between 200,562 and 211,190 units a year. Purchases brought in about 17 to 18 units for every unit sold.
- At the 2024 speed of sales, the average stock would last about 98 years (79 years at Chennai to 118 years at Hyderabad).
- All 180 product-branch pairs and all 30 products are slower in 2024 than in 2019. No pair turns over even 2% of its stock in a year.
- The branch gap comes from sales, not from stock. Chennai and Kolkata are the fastest branches and Hyderabad is the slowest.
- The result holds with a second averaging method, with sales dated by order date, with cancelled orders counted, with ledger-based stock, and with a different cost.
- Inventory Aging is Not selected because the data has no lot or batch IDs.

---

## 1. Definition

Inventory Turnover Rate shows how many times the average stock held in a year is sold in that year. A low rate means stock stays on the shelf for a long time. Years of stock is the opposite view: how many years the average stock would last at the speed of sales in that year.

| Label | Metric | How it is calculated |
|---|---|---|
| M1 | Turnover per product-branch pair per year | Units sold in the year divided by the average stock units in the year. 180 pairs (30 products x 6 branches) x 6 years (2019 to 2024) = 1,080 pair-years |
| M2 | Turnover per branch per year (money-weighted) | For the 30 products of a branch: sum of (units sold x received cost) divided by sum of (average stock units x received cost) |
| M3 | Overall turnover per year | The same money-weighted calculation over all 180 pairs. The units-only version is shown next to it |
| M4 | Years of stock | 1 divided by the turnover |

The per-product figures in this document (all 6 branches added) are M1 added over the branches.

**Business question.** How quickly does the warehouse sell the stock it holds, which products and branches are the slowest, and is the speed getting better or worse over time?

**Why this metric.** Stock only helps the business when it sells. Turnover puts sales and stock into one number: a high number means stock leaves the shelf quickly and a low number means it sits. KPI 8 gave a stock figure we can trace unit by unit, and KPI 6 measured years of cover for the last 12 months; this KPI shows how the speed of selling changed year by year from 2019 to 2024, per pair, branch and product. It is the KPI for the Inventory turnover requirement of Phase 2.

**Scope and choices**
- Products: P001 to P030. Branches: AHM001, CHN001, DEL001, HYD001, KOL001, PUN001. Pairs: 180.
- Period: calendar years 2019 to 2024. January 2025 is a partial month (the last delivery is on 13 January 2025 and the last receipt on 28 January 2025) and is left out of yearly turnover.
- Units sold: Delivered sales order lines only (117,656 lines, 1,234,939 units), dated by delivery date.
- Stock: month-end order-line stock per pair (opening stock + received - delivered + stock corrections), 73 months from 2019-01 to 2025-01.
- Average stock (headline): the mean of the 12 month-end values of the year. Second method: (opening + closing) / 2.
- Cost (headline): weighted average cost paid on Received purchase order lines. The products-table cost is shown as a second view.
- Cancelled orders are not in the headline. Their effect is tested in Check 4.
- Inventory Aging (a task in the CadetX Task Catalogue): Not selected. The data has no lot or batch IDs, so the age of stock would depend on a FIFO (first in, first out) assumption that the data cannot prove. Years of stock (M4) already shows how long stock sits.

---

## 2. Methodology

### Sources and scoping
- Source tables (from the 12 project CSV files): stock_ledger (237,230 rows), inventory_master (180), purchase_orders_header (24,000), purchase_orders_lines (155,495), sales_orders_header (20,000), sales_orders_lines (130,402) and products (30). In this document they are called ledger, inv, po_h, po_l, so_h, so_l and products (see the Glossary).
- Scope: Delivered sales order lines and Received purchase order lines only, calendar years 2019 to 2024. Cancelled orders and the partial month of January 2025 are outside the headline and are tested separately (Check 4 extra and Step 3).

### Step 1: Setup
Seven tables were loaded and their row counts equal the verified counts from Weeks 3 and 4: ledger 237,230 rows x 9 columns; inv 180 x 8; po_h 24,000 x 10; po_l 155,495 x 9; so_h 20,000 x 11; so_l 130,402 x 9; products 30 x 23.

### Step 2: Which cost to use
**Is products.unit_cost usable?** products has 30 rows. Empty unit_cost: 0. Zero or negative: 0. Products with unit_cost at or above unit_price: 0. Cost ranges from 180 to 95,500. Cost is between 54.2% (P027) and 82.8% (P010) of price. Units of measure: 5 sets (P027, P013, P012, P004, P014), 1 cartridge (P026) and 24 pieces, which matches Week 3.

**Purchase order lines against the products-table cost.** 155,495 purchase order lines; after joining with products 155,495 rows; lines with no matching product 0. The unit_cost on the purchase order line differs from products.unit_cost in 155,492 lines (only 3 lines are equal). The purchase order cost is higher in 94,007 lines, lower in 61,485 and equal in 3. The difference per line, in %: mean 6.11, standard deviation 17.31, minimum -27.54, 25% -6.83, median 4.64, 75% 16.39, maximum 66.15.

**By year of the purchase order** (lines, mean difference %): 2019 25,802, 6.24; 2020 26,568, 6.16; 2021 25,951, 6.17; 2022 25,515, 6.02; 2023 26,020, 6.05; 2024 25,639, 6.00. There is no trend over time. For every product the highest purchase cost is about 1.5 times the lowest (ratio between 1.499 and 1.5 for all 30 products).

**Received lines only.** 140,077 lines; 22,387,123 units; total spend 312,027,429,107.70. Lines where line_total differs from quantity x unit_cost by more than 0.01: 0. The weighted average cost paid is close to the simple mean of the purchase order costs for every product (by reading the table; the largest gap seen is P027, 360.20 against 361.0, about 0.2%). Units received per product range from 722,890 (P019) to 764,701 (P002).

**Decision.** Headline cost = weighted average cost paid on Received purchase order lines. The products-table cost is the second view. Why the products-table cost differs from the purchase cost is not in the data.

| Product | Products-table cost | Unit price | Cost % of price | PO cost min | PO cost mean | PO cost max | PO mean vs table % | Units received | Weighted avg cost paid | Weighted vs table % |
|---|---|---|---|---|---|---|---|---|---|---|
| P001 | 18,950 | 24,500 | 77.3 | 14,701.95 | 18,376.3 | 22,048.91 | -3.03 | 749,292 | 18,380.34 | -3.01 |
| P002 | 450 | 720 | 62.5 | 432.16 | 540.2 | 647.99 | +20.04 | 764,701 | 540.40 | +20.09 |
| P003 | 820 | 1,250 | 65.6 | 750.02 | 938.9 | 1,124.78 | +14.50 | 754,421 | 938.35 | +14.43 |
| P004 | 32,500 | 39,800 | 81.7 | 23,881.60 | 29,811.7 | 35,818.17 | -8.27 | 740,161 | 29,785.03 | -8.35 |
| P005 | 6,350 | 8,450 | 75.1 | 5,070.13 | 6,336.0 | 7,604.51 | -0.22 | 759,066 | 6,332.77 | -0.27 |
| P006 | 8,650 | 11,200 | 77.2 | 6,720.28 | 8,404.3 | 10,078.10 | -2.84 | 751,495 | 8,405.06 | -2.83 |
| P007 | 980 | 1,450 | 67.6 | 870.01 | 1,090.8 | 1,305.00 | +11.31 | 755,069 | 1,091.81 | +11.41 |
| P008 | 340 | 580 | 58.6 | 348.05 | 435.9 | 521.99 | +28.21 | 726,479 | 436.73 | +28.45 |
| P009 | 11,450 | 14,900 | 76.8 | 8,941.20 | 11,200.9 | 13,408.15 | -2.18 | 736,336 | 11,193.77 | -2.24 |
| P010 | 28,900 | 34,900 | 82.8 | 20,940.70 | 26,163.5 | 31,409.01 | -9.47 | 764,284 | 26,193.71 | -9.36 |
| P011 | 21,500 | 27,600 | 77.9 | 16,560.12 | 20,697.9 | 24,838.77 | -3.73 | 754,640 | 20,693.67 | -3.75 |
| P012 | 1,800 | 2,450 | 73.5 | 1,470.16 | 1,845.6 | 2,204.79 | +2.53 | 761,229 | 1,843.48 | +2.42 |
| P013 | 950 | 1,450 | 65.5 | 870.04 | 1,088.2 | 1,304.91 | +14.55 | 740,939 | 1,088.75 | +14.61 |
| P014 | 68,500 | 83,500 | 82.0 | 50,103.87 | 62,703.8 | 75,146.87 | -8.46 | 764,100 | 62,712.84 | -8.45 |
| P015 | 95,500 | 119,000 | 80.3 | 71,411.22 | 89,201.7 | 107,090.25 | -6.60 | 734,499 | 89,239.25 | -6.56 |
| P016 | 7,800 | 10,200 | 76.5 | 6,120.40 | 7,661.1 | 9,178.91 | -1.78 | 733,675 | 7,668.35 | -1.69 |
| P017 | 320 | 520 | 61.5 | 312.01 | 390.0 | 467.94 | +21.88 | 761,754 | 389.95 | +21.86 |
| P018 | 8,200 | 11,300 | 72.6 | 6,780.42 | 8,483.8 | 10,168.94 | +3.46 | 748,671 | 8,481.44 | +3.43 |
| P019 | 31,200 | 39,800 | 78.4 | 23,884.71 | 29,867.8 | 35,806.30 | -4.27 | 722,890 | 29,849.35 | -4.33 |
| P020 | 750 | 1,150 | 65.2 | 690.02 | 863.4 | 1,034.95 | +15.12 | 724,414 | 864.72 | +15.30 |
| P021 | 35,600 | 46,200 | 77.1 | 27,720.93 | 34,568.5 | 41,578.09 | -2.90 | 743,914 | 34,558.29 | -2.93 |
| P022 | 12,800 | 16,900 | 75.7 | 10,140.51 | 12,660.2 | 15,209.53 | -1.09 | 753,909 | 12,661.26 | -1.08 |
| P023 | 9,200 | 12,950 | 71.0 | 7,770.67 | 9,710.0 | 11,654.28 | +5.54 | 758,185 | 9,716.58 | +5.61 |
| P024 | 7,400 | 10,300 | 71.8 | 6,182.67 | 7,733.4 | 9,268.18 | +4.51 | 757,049 | 7,721.04 | +4.34 |
| P025 | 5,600 | 7,850 | 71.3 | 4,710.57 | 5,878.6 | 7,064.92 | +4.97 | 757,080 | 5,871.96 | +4.86 |
| P026 | 180 | 310 | 58.1 | 186.05 | 232.4 | 279.00 | +29.11 | 729,339 | 232.45 | +29.14 |
| P027 | 260 | 480 | 54.2 | 288.01 | 361.0 | 431.99 | +38.85 | 734,376 | 360.20 | +38.54 |
| P028 | 1,450 | 2,250 | 64.4 | 1,350.00 | 1,679.7 | 2,024.95 | +15.84 | 725,809 | 1,681.38 | +15.96 |
| P029 | 2,750 | 3,950 | 69.6 | 2,370.11 | 2,962.7 | 3,554.88 | +7.73 | 745,514 | 2,961.63 | +7.70 |
| P030 | 16,400 | 21,900 | 74.9 | 13,140.76 | 16,404.4 | 19,709.03 | +0.03 | 733,833 | 16,378.56 | -0.13 |

The purchase cost is above the products-table cost in about 60.5% of lines and below it in about 39.5%, so there is no one-direction shift. Averaged over all lines the purchase cost is about 6.1% above the products-table cost, with product-level gaps from -9.47% (P010) to +38.85% (P027).

### Step 3: Units sold and received by calendar year
Sales are dated by delivery date (Delivered lines) and purchases by received date (Received lines). Delivered lines 117,656, units 1,234,939, lines with no delivery date 0. Received lines 140,077, units 22,387,123, lines with no received date 0.

| Year | Units delivered | Units received | Received per delivered unit (about) |
|---|---|---|---|
| 2019 | 204,586 | 3,556,945 | 17.4 |
| 2020 | 211,190 | 3,786,822 | 17.9 |
| 2021 | 206,935 | 3,776,259 | 18.2 |
| 2022 | 202,716 | 3,682,130 | 18.2 |
| 2023 | 205,578 | 3,752,938 | 18.3 |
| 2024 | 200,562 | 3,637,075 | 18.1 |
| 2025 (partial) | 3,372 | 194,954 | not used |

The yearly rows add up to 1,234,939 and 22,387,123. The delivery-date year differs from the order-date year used in KPI 7 (for example 2019 units 207,925 by order date and 204,586 by delivery date); both are correct and answer different questions. The period chosen is 2019 to 2024.

### Step 4: Month-end stock
**Ledger chain re-test** on 237,230 rows: IN rows change the balance by +quantity in 127,611 of 127,611 rows; OUT rows by -quantity in 107,165 of 107,165; ADJUSTMENT rows by plus or minus quantity in 2,454 of 2,454 (same as KPI 8). Movements used: Received lines 140,077 + Delivered lines 117,656 + adjustments 2,454 = 260,187; movements outside the 73 months: 0.

**Stock table**: 180 pairs x 73 months (2019-01 to 2025-01). Total stock at the end of 2025-01 is 21,185,215, equal to the KPI 8 order-line stock. Pair-months below zero: 0. Smallest month-end value of any pair: 67.

| End of December | Total stock (units) | Increase in the year |
|---|---|---|
| Opening (before 2019) | 32,969 | |
| 2019 | 3,384,660 | 3,351,691 |
| 2020 | 6,960,247 | 3,575,587 |
| 2021 | 10,529,943 | 3,569,696 |
| 2022 | 14,009,405 | 3,479,462 |
| 2023 | 17,557,387 | 3,547,982 |
| 2024 | 20,993,633 | 3,436,246 |

January 2025 adds 191,582 units (194,954 received minus 3,372 delivered).

### Step 5: Turnover per year
Columns: units sold, average stock (monthly average, all 180 pairs added), then money-weighted turnover with received cost (monthly average, and opening-and-closing) and with products-table cost (monthly average, and opening-and-closing).

| Year | Units sold | Average stock (units) | Received cost, monthly avg | Received cost, opening-and-closing | Table cost, monthly avg | Table cost, opening-and-closing |
|---|---|---|---|---|---|---|
| 2019 | 204,586 | 1,798,354 | 0.11418 | 0.12003 | 0.11419 | 0.12003 |
| 2020 | 211,190 | 5,302,595 | 0.03963 | 0.04053 | 0.03962 | 0.04052 |
| 2021 | 206,935 | 8,899,310 | 0.02321 | 0.02360 | 0.02320 | 0.02359 |
| 2022 | 202,716 | 12,378,411 | 0.01666 | 0.01683 | 0.01666 | 0.01684 |
| 2023 | 205,578 | 15,890,688 | 0.01301 | 0.01309 | 0.01301 | 0.01309 |
| 2024 | 200,562 | 19,389,482 | 0.01023 | 0.01029 | 0.01022 | 0.01028 |

Total units sold 2019 to 2024: 1,231,567 (1,234,939 minus the 3,372 sold in 2025).

- Turnover falls from 0.114 in 2019 to 0.0102 in 2024, about 11 times lower, while average stock rises from 1.80 million to 19.39 million units (about 10.8 times) and sales stay flat (200,562 to 211,190 units).
- Units-only by hand: 204,586 / 1,798,354 = 0.1138 (2019) and 200,562 / 19,389,482 = 0.0103 (2024). Opening-and-closing for 2019 by hand: (32,969 + 3,384,660) / 2 = 1,708,815, and 204,586 / 1,708,815 = 0.1197.
- Cost choice: the received cost and the products-table cost differ only in the 5th decimal (largest relative gap about 0.06%, 2022 opening-and-closing: 0.01683 against 0.01684).
- Averaging method: monthly average and opening-and-closing differ by about 5% in 2019 (0.11418 against 0.12003), 2.3% in 2020 and under 1% by 2024, because stock grows steadily and the 12 month-end values leave out the stock at the start of the year. Both tell the same story.
- Years of stock in 2024: 1 / 0.01023 = about 98 years (KPI 6 gave about 110 years using end-of-year stock).
- Headline method: monthly average and received cost. Second method: opening-and-closing. Products-table cost: second view.

### Step 6: Turnover by branch and year (money-weighted, received cost, monthly average)

| Branch | 2019 | 2020 | 2021 | 2022 | 2023 | 2024 |
|---|---|---|---|---|---|---|
| AHM001 | 0.12899 | 0.04328 | 0.02476 | 0.01662 | 0.01450 | 0.00980 |
| CHN001 | 0.15269 | 0.04355 | 0.02833 | 0.02005 | 0.01577 | 0.01222 |
| DEL001 | 0.08946 | 0.03489 | 0.02131 | 0.01682 | 0.01143 | 0.00938 |
| HYD001 | 0.08670 | 0.03269 | 0.01682 | 0.01413 | 0.00994 | 0.00856 |
| KOL001 | 0.13151 | 0.04908 | 0.02979 | 0.01878 | 0.01563 | 0.01232 |
| PUN001 | 0.10017 | 0.03526 | 0.01897 | 0.01380 | 0.01100 | 0.00917 |

- Every branch falls every year.
- Chennai (CHN001) and Kolkata (KOL001) are the two fastest branches in every year (Chennai first in 2019, 2022 and 2023; Kolkata first in 2020, 2021 and 2024). The earlier expectation that Chennai is always the fastest was only partly right.
- Hyderabad (HYD001) is the slowest branch in 5 of 6 years (Pune in 2022).
- In 2024 the fastest branch (KOL001 0.01232) is about 1.44 times the slowest (HYD001 0.00856). In 2019 the ratio was about 1.76.

**2024 turnover per product-branch pair (units sold / average stock), 180 pairs:** mean 0.01038, standard deviation 0.00186, minimum 0.00634, 25% 0.00899, median 0.01026, 75% 0.01161, maximum 0.01530.
- Slowest 5 pairs: P005-HYD001 0.00634; P004-HYD001 0.00699; P019-HYD001 0.00699; P015-PUN001 0.00729; P018-HYD001 0.00744.
- Fastest 5 pairs: P023-CHN001 0.01440; P006-CHN001 0.01444; P016-CHN001 0.01454; P030-CHN001 0.01486; P027-CHN001 0.01530.
- Four of the 5 slowest pairs are in HYD001 and all 5 fastest are in CHN001. The pair spread is tight (maximum / minimum about 2.4, middle half 0.00899 to 0.01161, no pair above 0.016). By hand, years of stock for the slowest pair is 1 / 0.00634 = about 158 and for the fastest 1 / 0.0153 = about 65 (KPI 6 range was 72 to 172). No pair turns over even 2% of its stock in a year.

### Step 7: What drives the branch gap (2024)

| Branch | Units sold | Units received | Average stock | Stock at end of 2024 | Turnover in units | Years of stock |
|---|---|---|---|---|---|---|
| AHM001 | 31,921 | 615,796 | 3,165,998 | 3,442,316 | 0.01008 | 99.2 |
| CHN001 | 40,818 | 642,929 | 3,234,502 | 3,516,571 | 0.01262 | 79.2 |
| DEL001 | 31,967 | 627,088 | 3,293,798 | 3,566,310 | 0.00971 | 103.0 |
| HYD001 | 27,816 | 576,606 | 3,268,726 | 3,517,178 | 0.00851 | 117.5 |
| KOL001 | 36,941 | 568,941 | 3,147,293 | 3,400,815 | 0.01174 | 85.2 |
| PUN001 | 31,099 | 605,715 | 3,279,165 | 3,550,443 | 0.00948 | 105.5 |

Totals for 2024: units sold 200,562, units received 3,637,075, average stock adds to 19,389,482, end stock adds to 20,993,633 (all equal to Steps 3, 4 and 5).

The branch gap comes from sales, not from stock. Units sold run from 27,816 (HYD001) to 40,818 (CHN001), a ratio of about 1.47. Average stock only runs from 3,147,293 (KOL001) to 3,293,798 (DEL001), a ratio of about 1.05. Units received run from 568,941 to 642,929, a ratio of about 1.13. The units-based and the money-based branch turnover differ by up to about 5% (for example KOL001 0.01174 in units against 0.01232 in money, 2024), because the money version gives more weight to expensive products. That is a difference of definition, not a finding.

### Step 8: Turnover per product, all branches (2024, and 2019 for comparison)

| Product | Name | Units sold 2024 | Average stock 2024 | Turnover 2024 | Years of stock 2024 | Turnover 2019 |
|---|---|---|---|---|---|---|
| P001 | Hydraulic Pump HP-300 | 6,636 | 645,793 | 0.01028 | 97.3 | 0.11318 |
| P002 | Engine Oil Filter OF-90 | 7,145 | 664,365 | 0.01075 | 93.0 | 0.11230 |
| P003 | Air Filter AF-120 | 6,728 | 649,329 | 0.01036 | 96.5 | 0.10626 |
| P004 | Track Chain TC-45 | 6,668 | 639,256 | 0.01043 | 95.9 | 0.12962 |
| P005 | Fuel Injector FI-220 | 6,137 | 658,617 | 0.00932 | 107.3 | 0.11501 |
| P006 | Alternator ALT-500 | 7,192 | 651,481 | 0.01104 | 90.6 | 0.09492 |
| P007 | Hydraulic Hose HH-60 | 6,689 | 656,926 | 0.01018 | 98.2 | 0.11227 |
| P008 | Bucket Tooth BT-10 | 6,810 | 631,268 | 0.01079 | 92.7 | 0.11792 |
| P009 | Starter Motor SM-700 | 6,415 | 632,603 | 0.01014 | 98.6 | 0.11060 |
| P010 | Turbocharger TB-900 | 6,464 | 663,710 | 0.00974 | 102.7 | 0.12474 |
| P011 | Hydraulic Cylinder HC-200 | 7,000 | 654,184 | 0.01070 | 93.5 | 0.11435 |
| P012 | Brake Pad BP-40 | 6,280 | 659,547 | 0.00952 | 105.0 | 0.12577 |
| P013 | Seal Kit SK-15 | 6,525 | 642,626 | 0.01015 | 98.5 | 0.11007 |
| P014 | Transmission Assembly TA-800 | 6,717 | 663,326 | 0.01013 | 98.7 | 0.09948 |
| P015 | Engine Block EB-600 | 6,367 | 636,993 | 0.01000 | 100.0 | 0.11784 |
| P016 | Radiator RD-250 | 7,135 | 633,968 | 0.01125 | 88.9 | 0.10201 |
| P017 | Fan Belt FB-30 | 7,012 | 656,811 | 0.01068 | 93.6 | 0.11238 |
| P018 | Wheel Rim WR-24 | 6,399 | 650,910 | 0.00983 | 101.7 | 0.11433 |
| P019 | Swing Motor SWM-350 | 6,198 | 625,236 | 0.00991 | 100.9 | 0.11693 |
| P020 | Pivot Pin PP-12 | 6,470 | 624,578 | 0.01036 | 96.5 | 0.12010 |
| P021 | Boom Cylinder BC-400 | 6,838 | 642,086 | 0.01065 | 93.9 | 0.11678 |
| P022 | Control Valve CV-75 | 6,635 | 652,263 | 0.01017 | 98.3 | 0.12414 |
| P023 | Idler Wheel IW-55 | 6,505 | 661,248 | 0.00984 | 101.6 | 0.11394 |
| P024 | Track Roller TR-60 | 6,681 | 660,966 | 0.01011 | 98.9 | 0.10522 |
| P025 | Cabin Glass CG-20 | 6,371 | 657,798 | 0.00969 | 103.2 | 0.10831 |
| P026 | Grease Cartridge GC-5 | 6,545 | 635,380 | 0.01030 | 97.1 | 0.11379 |
| P027 | O-Ring Set OR-50 | 6,923 | 636,259 | 0.01088 | 91.9 | 0.11017 |
| P028 | Light Assembly LA-18 | 6,441 | 625,150 | 0.01030 | 97.1 | 0.11298 |
| P029 | Pressure Sensor PS-90 | 7,485 | 647,584 | 0.01156 | 86.5 | 0.12644 |
| P030 | Fuel Pump FP-350 | 7,151 | 629,222 | 0.01136 | 88.0 | 0.11769 |

Totals for 2024: units sold 200,562; average stock 19,389,483 (Step 5 shows 19,389,482; the product figures were rounded to whole units before adding, so the difference of 1 is rounding only). Products where 2024 turnover is below 2019: 30 of 30.

Product turnover in 2024 only runs from 0.00932 (P005) to 0.01156 (P029), a ratio of 1.24, and years of stock from 86.5 to 107.3. Units sold run from 6,137 (P005) to 7,485 (P029), a ratio of 1.22, while average stock runs from 624,578 (P020) to 664,365 (P002), a ratio of 1.06, so product differences come mainly from sales. By hand: P029 7,485 / 647,584 = 0.01156 and P005 6,137 / 658,617 = 0.00932. In 2019 the product turnover ran from 0.09492 (P006) to 0.12962 (P004). No product stands out: every product turns over about 1% of its average stock a year.

### Alternatives considered

| Alternative | Where it was tested | Result | Decision |
|---|---|---|---|
| Sales dated by order date instead of delivery date | Check 5 extra A and A2 | Yearly turnover differs by 0.23% to 1.63%; same fall | Delivery date kept |
| Cancelled orders counted as sold and received | Check 4 extra | Yearly turnover changes by -1.19% to +1.23%; same fall | Left out of the headline |
| Ledger-based stock instead of order-line stock | Check 5 extra B | Turnover 9.7% to 10.0% higher; same fall (about 11.0 times) | Order-line stock kept (the complete source); ledger-based shown as a second view |
| Opening-and-closing average instead of the monthly average | Step 5 | Differs by about 5% in 2019 and under 1% by 2024; same story | Monthly average kept |
| Products-table cost instead of received cost | Steps 2 and 5, Check 5 extra D | Differs only in the 5th decimal | Received cost kept |
| Units only, or equal weights per product, instead of money weights | Check 5 extra D | The four weightings are within about 0.4% (2019) and 1.3% (2024) | Money weights kept for M2 and M3, units shown beside them |
| An Inventory Aging section | Decision for the Task Catalogue | No lot or batch IDs, so FIFO cannot be proven | Not selected |

---

## 3. Verification Log

Every check was run on the exact tables used by the KPI: sd (Delivered sales lines, 117,656 rows), pr (Received purchase order lines, 140,077 rows), inv, ledger, products and the stock table. Table names and row counts were printed in each check.

### Summary of the 8 checks

| # | Check | Method | Result in plain words |
|---|---|---|---|
| 1 | Empty, zero and negative values | Counted in every numeric column of sd, pr, inv, ledger and products | No empty, zero or negative values in any column used |
| 2 | Edge cases | All 1,080 pair-years checked for sales and receipts; smallest stock value | Every pair-year has sales and receipts; the smallest stock value is 67 units |
| 3 | Repeated or duplicate lines | Repeated keys, exact copies, and same-quantity twins against chance (3b, 3c) | No repeated keys or exact copies; twins at chance (690 observed, 719.3 expected) |
| 4 | Totals reconcile | Units, line totals, and opening + received - delivered + corrections against the stock table and KPI 8 | All totals tie exactly at 21,185,215 units; counting cancelled orders changes turnover by -1.19% to +1.23% |
| 5 | Second method | Stock rebuilt from raw lines without the stock table, plus the order-date, ledger-stock and cost tests | Agrees within 0.031%; the gap is exactly the 2,454 stock corrections; same fall in every version |
| 6 | Outliers | 1.5 x IQR on pair turnover for each year; the 4 outlier pair-years examined (6b, 6c) | 4 of 1,080 pair-years mildly above the limit, explained by fewer receipts; effect under 0.4% |
| 7 | History | Dates, all 73 months, the smallest months (7b, 7c) | All 73 months have data; the small months are explained |
| 8 | Hand spot-check of 3 pairs | P027-CHN001, P005-HYD001 and P015-AHM001 rebuilt from raw rows | All 36 month-end values and all 3 turnover values equal the tables |

The details of each check follow.

### Check 1: Empty, zero and negative values
Tables and rows: sd 117,656; pr 140,077; inv 180; ledger 237,230; products 30. Every numeric column in every table shows 0 empty values, 0 zero values and 0 negative values. Columns checked: in sd and pr, line_number, quantity, unit_price (sd) or unit_cost (pr), gst_rate, line_total, gst_amount, line_grand_total and year; in inv, opening_stock, reorder_level, safety_stock, max_stock and current_stock; in ledger, quantity and running_balance (no negative running balance and no zero quantity); in products, unit_cost, unit_price, margin_percentage, gst_rate, weight_kg, warranty_months, reorder_level, safety_stock, max_stock_level, lead_time_days and last_purchase_price.
Result: no empty, zero or negative values in any column used.

### Check 2: Edge cases
180 pairs x 6 years (2019 to 2024) = 1,080 pair-years.
- sd, rows used (2019 to 2024) 117,316: pair-years 1,080, pair-years with zero units 0, smallest units in a pair-year 593.
- pr, rows used 138,831: pair-years 1,080, with zero units 0, smallest units in a pair-year 13,663.
- Stock table: smallest value of any pair-month 67.
- Rows left out because they are dated 2025: sd 340 (3,372 units) and pr 1,246 (194,954 units).
Result: every pair-year has sales and receipts, and no stock value is zero.

### Check 3: Repeated or duplicate lines
- sd (117,656 rows): repeated (so_id, line_number) keys 0; rows that are exact copies of another row 0; rows where the same product appears twice in one order 25,422 (21.6% of rows).
- pr (140,077 rows): repeated (po_id, line_number) keys 0; exact copies 0; rows where the same product appears twice in one order 30,068 (21.5% of rows).
- **Check 3b** (same order, product, quantity and price or cost, ignoring line_number): in sd, of the 25,422 repeated rows, 1,374 have the same quantity and the same unit_price; quantities range from 1 to 20 (20 different values), so the chance that two lines have the same quantity by luck is about 1 in 20 (0.05). In pr, of the 30,068 repeated rows, 0 have the same quantity and the same unit_cost (quantities range from 20 to 300, 281 different values, chance about 1 in 281).
- **Check 3c** (exact test on sd): the 25,422 rows sit in 12,184 groups (order + product): 11,202 groups of 2 lines, 914 of 3, 65 of 4, 2 of 5 and 1 of 6. That makes 14,369 pairs of lines. The chance that a pair has the same quantity, from the real quantity spread, is 0.0501, so about 719.3 pairs would match by chance. Observed: 690 (about 4% below chance, no excess).
Result: no double-entered lines found. All lines are kept as separate lines. Worst case: if all 690 matching pairs in sd were double entries, at most 690 x 20 = 13,800 units (about 1.1% of 1,234,939 delivered units) would be affected, so turnover would move by about 1% at most; the evidence says that case is not real. Limit: the test shows no excess over chance. It cannot prove that one single coincidental twin is not a double entry, and it is not needed for turnover.

### Check 4: Totals reconcile
- Units delivered 1,234,939 (expected 1,234,939). Units received 22,387,123 (expected 22,387,123).
- sd lines where line_total differs from quantity x unit_price by more than 0.01: 0. pr lines where line_total differs from quantity x unit_cost by more than 0.01: 0.
- Opening stock (inv) 32,969. Stock corrections, net +62. Opening + received - delivered + corrections = 21,185,215. The stock table total for 2025-01 = 21,185,215. KPI 8 order-line stock = 21,185,215.
Result: all totals tie exactly.

**Check 4 extra: cancelled orders included.** so_l 130,402 rows and po_l 155,495 rows were joined with their order headers; the row counts stayed the same after the join. Sales lines by status: Delivered 117,656, Cancelled 12,746 (9.77% of lines). Purchase lines by status: Received 140,077, Cancelled 15,418 (9.92% of lines). Units in cancelled sales lines 133,595 (9.76% of all sales units); in cancelled purchase lines 2,480,593 (9.98% of all purchase units); all are inside the 72 months. Sales lines have only two statuses (Delivered and Cancelled), so no pending orders distort 2024. Cancelled orders were counted as sold and as received, by order year:

| Year | Turnover, completed orders only | Turnover, cancelled counted | Change % |
|---|---|---|---|
| 2019 | 0.11558 | 0.11421 | -1.19 |
| 2020 | 0.03998 | 0.03974 | -0.59 |
| 2021 | 0.02340 | 0.02329 | -0.47 |
| 2022 | 0.01634 | 0.01636 | +0.16 |
| 2023 | 0.01288 | 0.01287 | -0.09 |
| 2024 | 0.01031 | 0.01043 | +1.23 |

**Why the sign changes by year (tested).** Counting cancelled orders pushes units sold up (sold push) and pushes average stock up (stock push) at the same time.

| Year | Completed units sold | Cancelled sales units | Cancelled purchase units | Sold push % | Stock push % | Sold push minus stock push | Change % |
|---|---|---|---|---|---|---|---|
| 2019 | 207,925 | 19,765 | 375,959 | 9.51 | 10.82 | -1.32 | -1.19 |
| 2020 | 212,007 | 21,655 | 449,671 | 10.21 | 10.87 | -0.65 | -0.59 |
| 2021 | 208,269 | 22,063 | 403,488 | 10.59 | 11.11 | -0.52 | -0.47 |
| 2022 | 202,242 | 22,793 | 391,287 | 11.27 | 11.09 | +0.18 | +0.16 |
| 2023 | 204,638 | 22,289 | 424,677 | 10.89 | 10.99 | -0.10 | -0.09 |
| 2024 | 199,858 | 25,030 | 435,511 | 12.52 | 11.16 | +1.37 | +1.23 |

The yearly cancelled units add up to 133,595 (sales) and 2,480,593 (purchases). The stock push is steady (10.8% to 11.2%) and the sold push moves (9.5% to 12.5%), so the sold push decides the sign: turnover goes down when the sold push is below the stock push (2019, 2020, 2021, 2023) and up when it is above (2022, 2024). By hand: change = (sold push - stock push) / (1 + stock push), for example -1.32 / 1.1082 = -1.19 and +1.37 / 1.1116 = +1.23. Cancelled sales are highest in 2024 (25,030 units, 11.13% of all sales units against 8.68% in 2019). Why cancellations rose in 2024 is not in the data.
Result: counting cancelled orders changes yearly turnover by -1.19% to +1.23% and the fall stays the same.

### Check 5: Second method
Total month-end stock was rebuilt from raw sd, pr and opening stock 32,969, without stock corrections and without the stock table. The average is the mean of the 12 month-ends. Turnover here is units sold / average stock (units-based).

| Year | Units sold | Average stock, second method | Average stock, Step 5 | Gap % | Turnover, second method | Turnover, Step 5 (units) |
|---|---|---|---|---|---|---|
| 2019 | 204,586 | 1,798,915 | 1,798,354 | 0.031 | 0.11373 | 0.11376 |
| 2020 | 211,190 | 5,303,361 | 5,302,595 | 0.014 | 0.03982 | 0.03983 |
| 2021 | 206,935 | 8,899,554 | 8,899,310 | 0.003 | 0.02325 | 0.02325 |
| 2022 | 202,716 | 12,378,860 | 12,378,411 | 0.004 | 0.01638 | 0.01638 |
| 2023 | 205,578 | 15,890,459 | 15,890,688 | -0.001 | 0.01294 | 0.01294 |
| 2024 | 200,562 | 19,389,511 | 19,389,482 | 0.0 | 0.01034 | 0.01034 |

By hand the gaps are +561 (2019), +766, +244, +449, -229 and +29 (2024) units; the largest is 0.031%. The units-based turnover (0.01034 in 2024) differs slightly from the money-based figure in Step 5 (0.01023) by definition: the money version gives more weight to expensive products.

**Check 5 extra C: the gap is exactly the stock corrections (tested).** The total month-end stock from the stock table minus the rebuild without corrections is +62 in the last month (2025-01), the smallest gap is -955 and the largest +597 (largest gap 0.079% of the rebuilt stock). At the end of each December the gap is: 2019 -668; 2020 -713; 2021 -341; 2022 -293; 2023 +329; 2024 +62. The 2,454 correction rows (movement_date, signed change) have a net change of +62 and none is dated outside the 73 months. Adding up the corrections month by month and comparing with the gap: the difference is 0 in all 73 months (largest absolute difference 0.0). The cumulative corrections at each December equal the gaps above.

**Check 5 extra A: sales dated by order date instead of delivery date.** Delivered lines, sd joined with the order header: 117,656 rows after joining, lines with no order date 0.

| Year | Units, delivery date | Units, order date | Turnover (delivery) | Turnover (order) | Difference % |
|---|---|---|---|---|---|
| 2019 | 204,586 | 207,925 | 0.11373 | 0.11558 | 1.63 |
| 2020 | 211,190 | 212,007 | 0.03982 | 0.03998 | 0.39 |
| 2021 | 206,935 | 208,269 | 0.02325 | 0.02340 | 0.64 |
| 2022 | 202,716 | 202,242 | 0.01638 | 0.01634 | -0.23 |
| 2023 | 205,578 | 204,638 | 0.01294 | 0.01288 | -0.46 |
| 2024 | 200,562 | 199,858 | 0.01034 | 0.01031 | -0.35 |

The order-date units add up to 1,234,939. The 2019 order-date units (207,925) tie to KPI 7. By order date the fall 2019 to 2024 is 0.11558 / 0.01031 = about 11.2 times, against about 11.0 times by delivery date.

**Check 5 extra A2: why 2019 differs most (tested).** Days from order to delivery: minimum 0, median 8, maximum 15; deliveries before the order date 0. Lines delivered in a different year than ordered: 2,482 (2.1% of lines), 25,449 units (2.06% of 1,234,939). Units ordered in a year and delivered in a later year: 2019 3,339; 2020 4,156; 2021 5,490; 2022 5,016; 2023 4,076; 2024 3,372 (add to 25,449). Orders placed before 2019-01-01: 0. Expected gap for 2019 (207,925 - 204,586) = 3,339 = observed 3,339. Every year ties: delivered year = ordered year - moved out + moved in (for example 2020: 212,007 - 4,156 + 3,339 = 211,190; 2024: 199,858 - 3,372 + 4,076 = 200,562), and the 2025 deliveries (3,372) are the units ordered in 2024. The 2019 gap is explained exactly: the data starts on 1 January 2019, so no orders from 2018 flow into 2019 while the late-2019 orders are delivered in 2020.
Result: the choice of date changes yearly turnover by 0.23% to 1.63% and the fall stays the same.

**Check 5 extra B: ledger-based stock.** Month-end ledger running_balance per pair, carried forward in months without a row, with opening stock before a pair's first row. Table 180 x 73, empty cells 0. Ledger-based stock for all pairs at 2025-01 = 19,305,994 (equal to the KPI 8 ledger-based stock).

| Year | Units sold | Average stock, ledger-based | Average stock, order-line | Turnover, ledger-based | Turnover, order-line | Difference % |
|---|---|---|---|---|---|---|
| 2019 | 204,586 | 1,638,474 | 1,798,915 | 0.12486 | 0.11373 | 9.79 |
| 2020 | 211,190 | 4,819,706 | 5,303,361 | 0.04382 | 0.03982 | 10.03 |
| 2021 | 206,935 | 8,100,272 | 8,899,554 | 0.02555 | 0.02325 | 9.87 |
| 2022 | 202,716 | 11,266,232 | 12,378,860 | 0.01799 | 0.01638 | 9.88 |
| 2023 | 205,578 | 14,478,424 | 15,890,459 | 0.01420 | 0.01294 | 9.75 |
| 2024 | 200,562 | 17,675,030 | 19,389,511 | 0.01135 | 0.01034 | 9.70 |

The ledger average stock is about 8.8% to 8.9% lower than the order-line stock every year, so the ledger-based turnover is 9.7% to 10.0% higher. The fall 2019 to 2024 is about 11.0 times in both versions. The gap between order-line and ledger stock at 2025-01 is 21,185,215 - 19,305,994 = 1,879,221 units (traced in KPI 8). The headline stays on the order-line stock (the complete source) and the ledger-based figure is the second view.

**Check 5 extra D: does the cost choice matter, and why (tested).** The combined turnover was calculated with four weightings next to the slowest and fastest product. The reason tested: products turn over at similar speeds, so no weighting can move a combined figure far, because a combined figure is an average of the 30 product turnovers and cannot go outside their range.

| | 2019 | 2024 |
|---|---|---|
| Slowest product turnover | 0.09492 | 0.00932 |
| Units, no cost | 0.11376 | 0.01034 |
| Received cost | 0.11418 | 0.01023 |
| Products-table cost | 0.11419 | 0.01022 |
| Equal weights per product | 0.11398 | 0.01035 |
| Fastest product turnover | 0.12962 | 0.01156 |
| Fastest / slowest | 1.366 | 1.240 |

All four combined values lie between the slowest and the fastest product. The four weightings differ by about 0.38% in 2019 (0.11376 to 0.11419) and about 1.3% in 2024 (0.01022 to 0.01035). The received cost and the products-table cost differ by 0.00001 in both years (about 0.01% in 2019 and about 0.1% in 2024 at 5 decimals), so the cost choice changes the combined turnover only in the 5th decimal. Limit: the narrow range caps how much any weighting can matter; it does not by itself explain why the two cost sets give nearly the same figure.
Result for Check 5: the stock table agrees with an independent rebuild to within 0.031% every year, the gap is exactly the 2,454 stock corrections, and turnover agrees to 4 or 5 decimals. The order-date test, the ledger-based test and the cost test show the same story.

### Check 6: Outliers (1.5 x IQR on pair turnover)
Pair turnover = units sold / monthly average stock per product-branch pair, 180 pairs each year; pairs with no sales row after matching: 0.

| Year | Pairs | Min | Median | Max | Lower limit | Upper limit | Pairs below | Pairs above |
|---|---|---|---|---|---|---|---|---|
| 2019 | 180 | 0.05804 | 0.11159 | 0.20234 | 0.02952 | 0.20049 | 0 | 1 |
| 2020 | 180 | 0.02233 | 0.03926 | 0.06249 | 0.01729 | 0.06198 | 0 | 1 |
| 2021 | 180 | 0.01361 | 0.02305 | 0.04156 | 0.00764 | 0.03877 | 0 | 1 |
| 2022 | 180 | 0.00960 | 0.01637 | 0.02699 | 0.00738 | 0.02533 | 0 | 1 |
| 2023 | 180 | 0.00666 | 0.01260 | 0.02076 | 0.00380 | 0.02221 | 0 | 0 |
| 2024 | 180 | 0.00634 | 0.01026 | 0.01530 | 0.00505 | 0.01555 | 0 | 0 |

The 2024 values tie to Step 6 (mean 0.01038, minimum 0.00634, maximum 0.0153). There are 4 pair-years above the upper limit out of 1,080 (0.37%), none below, and none in 2023 or 2024. The maximum / minimum ratio falls from 3.49 (2019) to 2.41 (2024), and the median from 0.11159 (2019) to 0.01026 (2024).

**Check 6b: the 4 pairs above the limit.**

| Pair-year | Turnover (limit) | Units sold | Average stock | Typical pair: units sold and average stock |
|---|---|---|---|---|
| P001-CHN001, 2019 | 0.20234 (0.20049) | 1,689 | 8,347 | 1,083 and 9,990 |
| P003-AHM001, 2020 | 0.06249 (0.06198) | 1,576 | 25,220 | 1,175 and 29,415 |
| P021-KOL001, 2021 | 0.04156 (0.03877) | 1,771 | 42,609 | 1,125 and 49,563 |
| P001-CHN001, 2022 | 0.02699 (0.02533) | 1,596 | 59,129 | 1,125 and 68,729 |

Each pair sold 1.34 to 1.57 times the typical pair (1.56, 1.34, 1.57, 1.42) and held 14% to 16% less stock (0.836, 0.857, 0.860, 0.860 of the typical stock). Yearly turnover (units) for all pairs against without these pairs: 2019 0.11376 against 0.11335; 2020 0.03983 against 0.03972; 2021 0.02325 against 0.02316; 2022 0.01638 against 0.01633; 2023 and 2024 unchanged (none removed). Removing them changes yearly turnover by -0.36%, -0.28%, -0.39% and -0.31% (2019 to 2022).

**Check 6c: why these pairs hold less stock (tested).** At the end of the year, this pair against the typical pair (median of 180 pairs): opening stock, units received so far, units delivered so far, stock without corrections.

| Pair, end of year | Opening | Received to date | Delivered to date | Stock without corrections |
|---|---|---|---|---|
| P001-CHN001, 2019 | 187 against 178 | 18,870 against 19,630 | 1,689 against 1,084 | 17,368 against 18,741 |
| P003-AHM001, 2020 | 238 against 178 | 36,575 against 40,724 | 2,722 against 2,263 | 34,091 against 38,502 |
| P021-KOL001, 2021 | 280 against 178 | 54,616 against 62,150 | 4,164 against 3,390 | 50,732 against 58,674 |
| P001-CHN001, 2022 | 187 against 178 | 74,470 against 82,628 | 6,083 against 4,514 | 68,574 against 78,228 |

Differences, this pair minus the typical pair: opening +9, +60, +102, +9; received -760, -4,149, -7,534, -8,158 (96.1%, 89.8%, 87.9% and 90.1% of the typical pair); delivered +605, +459, +774, +1,569; stock -1,373, -4,411, -7,942, -9,654 (92.7%, 88.5%, 86.5% and 87.7% of the typical pair, end-of-year stock, not the yearly average above). The medians of the parts do not add exactly to the median of the stock. The four pairs hold less stock mostly because they received fewer units than the typical pair (about 4% to 12% fewer): in 2019 receipts explain about half of the gap and the extra sales about 44%; in 2020 to 2022 receipts explain about 85% to 95% and the extra sales about 10% to 16%. Their opening stock was slightly higher, which does not explain the gap. Why purchasing gave these pairs fewer units is not in the data.
Result: no pair is below the lower limit; 4 pair-years (0.37%) are mildly above the upper limit in 2019 to 2022 and none in 2023 and 2024. They are busier-than-typical pairs with a little less stock, not errors, and they stay in the data. Their effect on yearly turnover is under 0.4%. Chennai (CHN001) appears twice, consistent with Chennai selling the most.

### Check 7: History
- sd delivery_date: first 2019-01-03, last 2025-01-13. pr received_date: first 2019-01-11, last 2025-01-28.
- Months expected 73 (2019-01 to 2025-01): months with deliveries 73, months with receipts 73. Rows outside the 73 months: sd 0, pr 0.
- Smallest month between 2019-01 and 2024-12: units delivered 11,734 and units received 143,820. January 2025 (partial): delivered 3,372 and received 194,954 (194,954 - 3,372 = 191,582, equal to the January 2025 stock rise in Step 4).
- Stock table: 180 pairs x 73 months, empty cells 0.
- Information only: pair-months with no delivery 10 of 12,960 (0.08%); with no receipt 4 of 12,960 (0.03%).

**Check 7b: which months are the smallest (tested).** 72 months from 2019-01 to 2024-12. The 3 smallest months by units delivered: 2019-01 11,734; 2019-02 14,442; 2023-02 14,971. By units received: 2019-01 143,820; 2022-02 253,937; 2023-06 266,164. The typical month (median) delivers 17,229.5 and receives 309,938.5 units, so the smallest month is 68.1% of typical for deliveries and 46.4% for receipts. By units per day, the 3 smallest for deliveries are 2019-01 (379), 2020-10 (494) and 2023-06 (505); for receipts 2019-01 (4,639), 2023-06 (8,872) and 2022-02 (9,069). The Februaries (2019-02, 2023-02, 2022-02) are short months and are not among the smallest per day. January 2019 is the only clearly low month.

**Check 7c: why January 2019 is low (tested).** Received purchase order lines 140,077, lines with no order date 0. The first purchase order is dated 2019-01-01. Days from order to receipt: smallest 7, median 18, largest 35. Lines received before they were ordered: 0. No purchase order exists before 2019, and an order takes at least 7 days to arrive, so the earliest possible receipt is 2019-01-08 (the first receipt in the data is 2019-01-11). With a median wait of 18 days only orders placed in roughly the first 13 days of January can arrive in January (13 / 31 = about 42%, a rough calculation), against the observed 46.4% of a typical month. Deliveries in 2019-01 (68.1% of typical) are partly the start of the data (first delivery on 2019-01-03).
Result: all 73 months have deliveries and receipts, nothing lies outside the period, no stock cell is empty, and the small months are explained. January 2025 is partial and is left out of yearly turnover.

### Check 8: Hand spot-check of 3 pairs
Each pair was rebuilt month by month from raw rows (opening stock + raw receipts - raw deliveries + ledger adjustment rows) up to 2024-12.

| Pair | Why chosen | Opening | Sales lines | Receipt lines | Adjustment rows | Units sold 2024 | Units received 2024 | Turnover 2024, raw rows | Turnover 2024, table |
|---|---|---|---|---|---|---|---|---|---|
| P027-CHN001 | Fastest in 2024 | 84 | 805 | 754 | 8 | 1,538 | 17,883 | 0.0153 | 0.0153 |
| P005-HYD001 | Slowest in 2024 | 178 | 543 | 805 | 19 | 714 | 20,561 | 0.00634 | 0.00634 |
| P015-AHM001 | Closest to the 2024 median | 85 | 690 | 790 | 16 | 1,091 | 19,191 | 0.01027 | 0.01027 |

The 12 month-end stocks of 2024 rebuilt from raw rows, all equal to the stock table:
- P027-CHN001: 93,711; 94,562; 95,920; 96,982; 98,127; 99,063; 101,083; 102,180; 104,330; 105,560; 106,791; 107,621.
- P005-HYD001: 103,234; 105,169; 107,042; 108,175; 109,746; 111,288; 113,583; 116,085; 116,991; 119,162; 120,228; 121,716.
- P015-AHM001: 98,923; 100,043; 101,599; 102,797; 103,287; 105,184; 106,885; 108,219; 109,242; 110,694; 112,763; 115,010.

By hand, the averages of the 12 month-ends are 100,494.2 (P027-CHN001), 112,701.6 (P005-HYD001) and 106,220.5 (P015-AHM001), which give 0.0153, 0.00634 and 0.01027. Received units per unit sold in 2024 are about 11.6, 28.8 and 17.6.
Result: all 36 month-end values (3 pairs x 12 months) equal the stock table, and all 3 turnover values equal the turnover table.

### Audit: do the yearly falls hold for every pair and every product?
- Pairs with 2024 turnover below 2019: 180 of 180.
- Pairs falling in every one of the 5 year steps: 142 of 180. Pairs falling by step: 2020 180; 2021 180; 2022 169; 2023 167; 2024 159.
- Products (all branches added): 30; falling in every one of the 5 steps: 30 of 30.
- The 38 pairs that rose in at least one year: 31 rose once and 7 rose twice, 45 rise-events in total (31 + 14), which matches the 11 + 13 + 21 rises of 2022, 2023 and 2024. Rises per branch (out of 30 pairs per branch): AHM001 5, CHN001 1, DEL001 11, HYD001 8, KOL001 5, PUN001 8. By product the most are P024 4 (of 6 pairs), P002 3, P017 3, P022 2 and P009 2. The size of the 45 rises in %: mean 9.997, standard deviation 9.38, minimum 0.089, 25% 2.82, median 6.68, 75% 14.08, maximum 36.03.
- **Why the rises happen (tested).** At the 45 rise-events units sold went up by 21% to 68% (mean 39.83, median 38.28) while average stock went up by 18% to 43% (mean 27.24, median 24.44). Units sold went up in 45 of 45 rise-events and average stock went down in 0 of 45. The typical yearly change over all 180 pairs (median) is units sold +5.01%, -2.99%, -4.63%, +3.58%, -2.53% (2020 to 2024) and average stock +196.62%, +67.11%, +38.59%, +28.27%, +22.06%. So every rise comes from a sales jump, never from stock dropping, and every one of the 180 pairs is still below its 2019 level in 2024.
- **Are the branch counts luck? (tested).** If the 38 pairs with a rise were spread evenly, each branch would have about 6.33. Chance of this many or more | this few or fewer: AHM001 5 | 0.814 | 0.352; CHN001 1 | 1.0 | 0.004; DEL001 11 | 0.024 | 0.992; HYD001 8 | 0.277 | 0.855; KOL001 5 | 0.814 | 0.352; PUN001 8 | 0.277 | 0.855. Allowing for the 6 branches (6 x the chance, a rough rule), Delhi's 11 is about 14%, which is within what luck can give, and Chennai's 1 is about 2.4%.
- **Why Chennai has so few rises (tested).** Per branch, the standard deviation of the yearly change in units sold (%): AHM001 18.88, CHN001 15.65, DEL001 21.41, HYD001 21.76, KOL001 15.43, PUN001 18.29. Pair-years with a sales jump above 20% (150 pair-years per branch, 2020 to 2024): AHM001 23, CHN001 12, DEL001 35, HYD001 28, KOL001 15, PUN001 25. The median yearly change in average stock is almost the same in every branch (for example 2024: 22.24, 22.69, 22.56, 21.32, 21.44 and 21.90%), so faster stock growth is not the reason for Chennai. A rise could only happen in 2022 to 2024 (stock grew by 63% or more in 2020 and 2021). In those 3 years (90 pair-years per branch): sales jumps above 20%: AHM001 10, CHN001 7, DEL001 22, HYD001 17, KOL001 7, PUN001 16; rise-events: 5, 1, 13, 11, 5, 10; share of jumps that became a rise: 50.0%, 14.3%, 59.1%, 64.7%, 71.4%, 62.5%. The size of the jumps above 20% in 2022 to 2024, with the stock growth in the same pair-years (jumps | median jump % | median stock growth %): AHM001 10 | 30.5 | 27.9; CHN001 7 | 24.4 | 32.8; DEL001 22 | 33.4 | 37.5; HYD001 17 | 28.7 | 29.3; KOL001 7 | 29.7 | 24.4; PUN001 16 | 30.9 | 24.3. Chennai has few jumps (7, tied lowest with Kolkata), its jumps are the smallest (median 24.4%) and they land in pair-years where stock grew more (median 32.8%), so a jump rarely beats the stock growth (1 of 7 against 50% to 71% in the other branches). Limits: only 7 jumps, medians only, and chance cannot be ruled out (for 7 jumps, 1 rise or fewer has a chance of about 2.8%, about 17% allowing for 6 branches). Delhi's median gap is negative (-4.1) and still 13 of its 22 jumps became rises, so the median gap shows a direction and is not a strict rule.
Result: the fall in turnover holds for all 180 pairs and all 30 products between 2019 and 2024. It holds year after year for all 30 products and for 142 of 180 pairs.

### Test labels used in this document and in the notebook

| Label | Plain name | Question it answers | Result | Where it sits |
|---|---|---|---|---|
| Check 3b | Same order, product, quantity and price, ignoring the line number | Are there double entries with a different line number? | sd 1,374 matching rows, pr 0 | Check 3 |
| Check 3c | Exact chance test on sd | Do same-quantity twins occur more often than chance? | 690 observed against 719.3 expected | Check 3 |
| Check 4 extra | Cancelled orders included | Does counting cancelled orders change the story, and why does the sign change by year? | -1.19% to +1.23%; the sold push decides the sign | Check 4 |
| Check 5 extra A | Order date instead of delivery date | Does the date choice change the result? | 0.23% to 1.63%; same fall | Check 5 |
| Check 5 extra A2 | Order-to-delivery lag | Why is 2019 the biggest gap? | 3,339 units ordered in 2019 were delivered in 2020; explained exactly | Check 5 |
| Check 5 extra B | Ledger-based stock | Does the ledger stock give the same story? | 9.7% to 10.0% higher turnover; same fall | Check 5 |
| Check 5 extra C | Gap equals the stock corrections | Is the 0.031% gap exactly the corrections? | 0 of 73 months differ | Check 5 |
| Check 5 extra D | Cost weighting | Does the cost choice matter, and why not? | The four weightings are within about 0.4% (2019) and 1.3% (2024); the cost choice only in the 5th decimal | Check 5 |
| Check 6b | The 4 pairs above the limit | Who are they and do they change turnover? | Busier pairs with less stock; effect under 0.4% | Check 6 |
| Check 6c | Why these pairs hold less stock | Opening stock, receipts or deliveries? | Mostly fewer units received | Check 6 |
| Check 7b | The smallest months | Which months are the smallest? | January 2019; the Februaries are short months | Check 7 |
| Check 7c | Why January 2019 receipts are low | Is data missing? | No orders before 2019-01-01 and at least 7 days to arrive; start of the data, not missing data | Check 7 |
| Audit: yearly falls | Does the fall hold for every pair and product? | 180 of 180 pairs lower in 2024 than 2019; 142 fall every year; 30 of 30 products | After Check 8 |
| Audit: the 38 pairs that rose | Why and where did pairs rise? | Sales jumps, never stock drops; Delhi within chance; Chennai few and small jumps | After Check 8 |

---

## 4. Data Quality Notes

1. The products-table cost differs from the purchase cost for 155,492 of 155,495 purchase lines (average +6.1%, product gaps from -9.47% to +38.85%). Why is not in the data. The weighted average cost paid is used as the headline and the products-table cost as the second view; the combined turnover differs only in the 5th decimal.
2. January 2025 is a partial month (deliveries end on 13 January, receipts on 28 January) and is left out of yearly turnover. The data starts on 1 January 2019, which makes January 2019 receipts low (no earlier orders can arrive).
3. Cancelled orders are 9.77% of sales lines and 9.92% of purchase lines. They are not in the headline; counting them changes yearly turnover by -1.19% to +1.23%.
4. About 21.6% of sales rows and 21.5% of purchase rows are products repeated in one order. They are kept as separate lines because no sign of double entry was found.
5. The 2,454 stock corrections add up to +62 units and explain exactly the gap between the stock table and the independent rebuild.
6. The headline uses the order-line stock. The ledger-based stock is about 9% lower (the gap at 2025-01 is 1,879,221 units, traced in KPI 8); ledger-based turnover is 9.7% to 10.0% higher with the same fall.
7. Sales are dated by delivery date. Dating them by order date changes yearly turnover by 0.23% to 1.63%.
8. Not in the data, so stated as limits and not as reasons: why purchasing gave the 4 fastest pair-years fewer units; why cancelled sales rose in 2024; why the products-table cost differs from the purchase cost; why Chennai has so few rises (7 sales jumps only, chance cannot be ruled out).
9. Inventory Aging is Not selected because the data has no lot or batch IDs.

---

## 5. Business Interpretation

**Business question:** How quickly does the warehouse sell the stock it holds, which products and branches are the slowest, and is the speed getting better or worse over time?

**Answer:** The warehouse sells only about 1% of its average stock in a year, the speed has fallen about 11 times since 2019 because stock keeps arriving about 18 times faster than it sells, and Hyderabad is the slowest branch.

- **Stock piles up while sales stay flat.** Order-line stock grows from 33 thousand units before 2019 to 21.2 million units at the end of January 2025, while sales stay at about 200,000 to 211,000 units a year. Purchases bring in about 17 to 18 units for every unit sold (17.4 in 2019, 18.1 in 2024).
- **Turnover falls about 11 times.** It goes from 0.114 in 2019 to 0.0102 in 2024. All 180 product-branch pairs and all 30 products are slower in 2024 than in 2019. In 2024 the average stock would last about 98 years at the speed of sales in that year (79 years at Chennai to 118 years at Hyderabad). KPI 6 gave about 110 years using end-of-year stock; the two figures are in the same range.
- **No pair turns over even 2% of its stock in a year.** The pair spread is tight: the slowest pair is P005-HYD001 (0.00634, about 158 years of stock) and the fastest is P027-CHN001 (0.0153, about 65 years).
- **The branch gap comes from sales, not from stock.** Chennai and Kolkata are the fastest branches and Hyderabad is the slowest (in 5 of 6 years). In 2024 units sold differ by about 1.47 times between Hyderabad and Chennai, while average stock differs by only about 1.05 times.
- **Products are alike.** Every product turns over about 1% of its average stock a year (0.00932 for P005 Fuel Injector to 0.01156 for P029 Pressure Sensor), and years of stock run from 86.5 to 107.3.
- **The result is robust.** The same fall appears with the second averaging method, with sales dated by order date, with cancelled orders counted, with ledger-based stock, with a different cost, and without the 4 mild outlier pairs.
- **What the number does not say.** The data shows that stock is far larger than sales; whether this is a planned buffer or excess stock is a business decision. The stock records also differ by about 9% between the order lines and the ledger.

## Recommended actions for the business owners

1. **Procurement Manager:** review the purchase quantities and reorder levels for all 30 products, because purchases are about 18 times the units sold and stock has grown every year since 2019.
2. **Inventory Manager:** set a target for years of stock per product and branch and report it every quarter next to turnover.
3. **Hyderabad Branch Manager:** review the stock held at HYD001, which has the lowest sales (27,816 units in 2024) and the highest years of stock (about 118).
4. **Finance Controller:** estimate the money tied up in the extra stock, using the weighted average cost paid in the product table above.
5. **Data Owner (the team that keeps the stock records):** confirm which stock record is correct, since the order-line stock and the ledger stock differ by 1,879,221 units at the end of January 2025.

---

## Glossary

| Label | Meaning |
|---|---|
| Pair | One product at one branch (30 products x 6 branches = 180 pairs) |
| Pair-year | One pair in one calendar year (180 x 6 = 1,080) |
| M1, M2, M3, M4 | The four metrics defined in section 1 |
| Turnover | Units sold in a year divided by the average stock in that year |
| Years of stock | 1 divided by the turnover |
| Order-line stock | Stock rebuilt from order lines: opening stock + received - delivered + stock corrections |
| Stock table (stock) | Month-end order-line stock per pair, 180 pairs x 73 months |
| sd | Sales delivered: the Delivered sales order lines (117,656 rows) |
| pr | Purchases received: the Received purchase order lines (140,077 rows) |
| inv | The inventory master table (180 rows, one per pair) |
| ledger | The stock book (237,230 movement rows) |
| products | The product master table (30 rows) |
| so_h, so_l | Sales order headers and sales order lines |
| po_h, po_l | Purchase order headers and purchase order lines |
| adj | The 2,454 stock-correction rows of the ledger (signed change) |
| cs | Cancelled sales units per month, by order month (72 months, 133,595 units in total) |
| cp | Cancelled purchase units per month, by order month (72 months, 2,480,593 units in total) |
| alt | Month-end stock with cancelled orders counted |
| avg_alt | Yearly average of alt |
| avg2 | Yearly average stock for completed orders, second method (rebuilt without the stock corrections) |
| by_order | Units sold by order year, completed orders |
| gap | Stock table total minus the rebuild without stock corrections |
| Stock corrections | Ledger rows of type ADJUSTMENT that add or remove units (2,454 rows, net +62) |
| Monthly average | Mean of the 12 month-end stock values of a year (headline) |
| Opening-and-closing | (Stock at the start of the year + stock at the end of the year) / 2 (second method) |
| Received cost | Weighted average cost paid on Received purchase order lines (spend divided by units) |
| Products-table cost | The unit_cost column of the products table |
| Sold push | Percentage by which counting cancelled sales raises the units sold |
| Stock push | Percentage by which counting cancelled purchases raises the average stock |
| Rise-event | One pair in one year where turnover went up compared with the year before |
| Sales jump | A year where a pair's units sold rose by more than 20% compared with the year before |
| FIFO | First in, first out: the assumption that the oldest stock is sold first |
| 1.5 x IQR | Outlier rule: a value more than 1.5 times the middle-half spread above the upper quartile or below the lower quartile |
