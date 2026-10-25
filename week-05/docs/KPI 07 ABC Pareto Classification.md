# KPI 07: ABC / Pareto Classification

Project: Heavy Supplier & Warehouse Analytics | Week 5 | Notebook: Week5_KPI07_ABC_Pareto_Classification

---

## 1. KPI Definition

**What it measures.** KPI 7 ranks the 30 products by the revenue they brought in and sorts them into three classes: A (the products that together make the first 80% of revenue), B (the next 15%) and C (the last 5%). It answers the question: which few products carry most of the sales?

| Metric | Name | Meaning |
|---|---|---|
| M1 | Revenue per product | Sum of line_total (revenue before GST) of the product's Delivered order lines |
| M2 | Share % | M1 of the product divided by total revenue |
| M3 | Cumulative share % | Running total of M2 when products are ranked from the highest revenue to the lowest |
| M4 | A/B/C class | A if the running total before the product is under 80%, B if under 95%, otherwise C |

**Why this metric.** A business cannot give every product the same attention. The classification shows where the money is, so that stock control, purchasing and sales effort can be aimed at the few products that matter most.

**Data used.** Delivered sales orders only: 18,033 orders, 117,656 order lines, 1,234,939 units, 30 products, order dates 2019-01-01 to 2024-12-31 (72 months). Total revenue before GST: 22,958,987,430.

**Result.**

| Class | Products | Revenue (before GST) | Share of revenue |
|---|---|---|---|
| A | 10 | 18,718,459,400 | 81.53% |
| B | 7 | 3,229,679,500 | 14.07% |
| C | 13 | 1,010,848,530 | 4.40% |
| Total | 30 | 22,958,987,430 | 100% |

Class sums tie to the total revenue and to 30 products.

**Full ranking** (list price from the products table; share and running total in %):

| Rank | Product | Name | List price | Share % | Running total % | Class |
|---|---|---|---|---|---|---|
| 1 | P015 | Engine Block EB-600 | 119,000 | 21.16 | 21.16 | A |
| 2 | P014 | Transmission Assembly TA-800 | 83,500 | 14.76 | 35.91 | A |
| 3 | P021 | Boom Cylinder BC-400 | 46,200 | 8.42 | 44.34 | A |
| 4 | P019 | Swing Motor SWM-350 | 39,800 | 7.20 | 51.54 | A |
| 5 | P004 | Track Chain TC-45 | 39,800 | 7.09 | 58.63 | A |
| 6 | P010 | Turbocharger TB-900 | 34,900 | 6.43 | 65.05 | A |
| 7 | P011 | Hydraulic Cylinder HC-200 | 27,600 | 5.06 | 70.11 | A |
| 8 | P001 | Hydraulic Pump HP-300 | 24,500 | 4.41 | 74.52 | A |
| 9 | P030 | Fuel Pump FP-350 | 21,900 | 3.97 | 78.49 | A |
| 10 | P022 | Control Valve CV-75 | 16,900 | 3.04 | 81.53 | A |
| 11 | P009 | Starter Motor SM-700 | 14,900 | 2.63 | 84.16 | B |
| 12 | P023 | Idler Wheel IW-55 | 12,950 | 2.27 | 86.43 | B |
| 13 | P006 | Alternator ALT-500 | 11,200 | 2.04 | 88.47 | B |
| 14 | P018 | Wheel Rim WR-24 | 11,300 | 2.01 | 90.48 | B |
| 15 | P016 | Radiator RD-250 | 10,200 | 1.86 | 92.34 | B |
| 16 | P024 | Track Roller TR-60 | 10,300 | 1.77 | 94.11 | B |
| 17 | P005 | Fuel Injector FI-220 | 8,450 | 1.49 | 95.60 | B |
| 18 | P025 | Cabin Glass CG-20 | 7,850 | 1.40 | 97.00 | C |
| 19 | P029 | Pressure Sensor PS-90 | 3,950 | 0.73 | 97.74 | C |
| 20 | P012 | Brake Pad BP-40 | 2,450 | 0.44 | 98.18 | C |
| 21 | P028 | Light Assembly LA-18 | 2,250 | 0.40 | 98.58 | C |
| 22 | P007 | Hydraulic Hose HH-60 | 1,450 | 0.27 | 98.85 | C |
| 23 | P013 | Seal Kit SK-15 | 1,450 | 0.25 | 99.10 | C |
| 24 | P003 | Air Filter AF-120 | 1,250 | 0.23 | 99.33 | C |
| 25 | P020 | Pivot Pin PP-12 | 1,150 | 0.20 | 99.54 | C |
| 26 | P002 | Engine Oil Filter OF-90 | 720 | 0.13 | 99.66 | C |
| 27 | P008 | Bucket Tooth BT-10 | 580 | 0.10 | 99.77 | C |
| 28 | P017 | Fan Belt FB-30 | 520 | 0.09 | 99.86 | C |
| 29 | P027 | O-Ring Set OR-50 | 480 | 0.09 | 99.95 | C |
| 30 | P026 | Grease Cartridge GC-5 | 310 | 0.05 | 100.00 | C |

Revenue of the largest product (P015) is 4,856,985,000 and of the smallest (P026) 12,526,170, a ratio of 387.7.

---

## 2. Methodology

**Sources.** sales_orders_header (order status, order date, branch), sales_orders_lines (product, quantity, unit price, GST rate, line_total, gst_amount, line_grand_total) and the products table (30 products, list price, GST rate).

**Scoping.** Only orders with status Delivered are counted. The data holds two statuses: Delivered (18,033 orders, 117,656 lines, 1,234,939 units) and Cancelled (1,967 orders, 12,746 lines, 133,595 units). Cancelled orders did not bring in money and are left out. Counting them would change no class (see Supporting tests).

**Revenue definition.** Revenue is line_total, the value of the lines before GST. GST (6,295,005,732.40 on Delivered lines, about 27.4% of revenue) is a tax collected for the government, not money the business earned from selling products. Two GST rates exist: 18% (51,119 lines, revenue 1,335,107,480, 5.82% of revenue) and 28% (66,537 lines, revenue 21,623,879,950, 94.18%). Of the 30 products, 17 carry 28% and 13 carry 18%.

**Steps.**
1. Sum line_total per product (M1).
2. Divide by the total to get the share (M2).
3. Rank from the highest to the lowest revenue and add up the shares (M3).
4. Give each product its class (M4) with the rule: the running total *before* the product decides. Under 80% means A, under 95% means B, otherwise C. So A is the group of products needed to reach at least 80% of revenue. The 80% and 95% limits are a common convention, not a business standard.

**Boundary products.** P022 (running total 81.53% after it) is the product that carries A over 80%, and P005 (95.60% after it) carries B over 95%. Under the other common rule (class by the running total *at* the product, A if at most 80%, B if at most 95%), P022 would become B and P005 would become C, and no other product would change. Both rules are shown in Check 5. We keep the rule above and mark P022 and P005 as boundary products.

**Rejected alternatives (each tested, see 3.3).** Counting cancelled orders; ranking with GST included; ranking by units sold instead of money. None changes the result in a useful way, and the reasons are given with the tests.

**Tables used in the notebook.** `sales` = Delivered order lines (117,656), `rev` = the 30 ranked products, `raw_h`, `raw_l`, `raw_p` = the raw files read again fresh for Check 8.

---

## 3. Verification Log

All checks were run on the Delivered lines (`sales`, 117,656 lines). Check 1 was also run on all 130,402 lines, and the two runs agree.

### 3.1 The 8 checks

| # | Check | Method | Result |
|---|---|---|---|
| 1 | Empty, zero and negative values | Counted in every column used, on Delivered lines and on all lines | No empty value in any column used; no zero or negative quantity, price or line value |
| 2 | Edge cases: price and months | Compared each of 117,656 lines with the list price; counted months with sales per product | Every line sold at its list price; every product sold in all 72 months |
| 3 | Repeated product lines in one order | Counted repeats and tested whether they are copies | 12,184 repeats; same-quantity rate at chance level; no duplicate line keys, no identical rows, no ties in the ranking |
| 4 | Totals reconcile | Compared totals with earlier verified totals and with the order headers | Totals match to the last unit; header total equals sum of lines for all 18,033 orders |
| 5 | Second method and other boundary rule | Rebuilt revenue as quantity times price; ran the other class rule | Both methods give the same revenue, share and class; the other rule moves only P022 and P005 |
| 6 | Outliers | Largest line against average line per product; classes without the largest lines | No line distorts the ranking; removing 5,852 maximum-quantity lines changes no class |
| 7 | History | Dates, months, years; class recalculated per year | 72 full months; 178 of 180 product-year classes equal the overall class |
| 8 | Manual spot-check | 3 products worked out by hand from the raw rows | All 3 products agree with the table on units, revenue, share, running total and class |

### 3.2 Details per check

**Check 1: empty, zero and negative values.**

| Table (rows) | Columns tested | Empty | Zero | Negative |
|---|---|---|---|---|
| Sales lines, Delivered (117,656) | so_id, product_id, quantity, unit_price, gst_rate, line_total, gst_amount | 0 | 0 | 0 |
| Sales lines, all statuses (130,402) | same columns | 0 | 0 | 0 |
| Sales header | so_id, order_status | 0 | not applicable | not applicable |
| Products (30) | product_id, product_name, unit_price, gst_rate | 0 | not applicable | not applicable |

Quantity runs from 1 to 20, unit price from 310 to 119,000, line_total from 310 to 2,380,000 (20 times 119,000), the same in both runs.

**Check 2: price per line and months with sales.**

| Test | Result |
|---|---|
| Lines checked | 117,656 |
| Lines with no list price found | 0 |
| Lines sold at a price different from the list price | 0 |
| Rows after adding order dates | 117,656 (no line gained or lost) |
| Months in the data | 72 |
| Months with sales per product | lowest 72, highest 72 |

Every line was sold at its list price. This shows the earlier result that average price equals list price is not hiding lines that offset each other.

**Check 3: repeated product lines in one order.**

| Test | Result |
|---|---|
| Order-product combinations on 2 or more lines | 12,184 |
| Lines inside those combinations | 25,422 (about 22% of lines) |
| Lines that copy another line, ignoring line number | 1,374 |
| Chance that two random lines share a quantity | 5.00% |
| Combinations with exactly 2 lines (Check 3, first test) | 11,202; same quantity 547 = 4.88% (about 560 expected by chance) |
| Combinations with 3 or more lines (Check 3 part 2) | 982; 3,018 lines inside; largest has 6 lines; 3,167 pairs of lines; same quantity 143 = 4.52% (about 158 expected, 5.00%) |
| Duplicate order and line number (Check 3 part 3) | 0 |
| Lines identical in every column to another line | 0 |
| Products in the ranking, unique product IDs | 30 and 30 |
| Products with the same revenue as another product | 0 |
| Products table rows and unique product IDs | 30 and 30 |

Both groups sit at or slightly below chance, so repeated lines are coincidence and not double counting. No line is loaded twice and the ranking has no ties. Merges kept their row counts (117,656 before and after; 30 products).

**Check 4: totals reconcile.**

| Test | Result |
|---|---|
| Delivered header rows / unique Delivered order IDs | 18,033 / 18,033 |
| Lines / units | 117,656 / 1,234,939 |
| Revenue from lines / revenue in the ranking table | 22,958,987,430 / 22,958,987,430 |
| Lines where line_total is not quantity times unit_price | 0 |
| Delivered orders compared with their header | 18,033 |
| Orders with no lines | 0 |
| Orders where header total_order_value differs from the sum of its lines | 0 |
| Header total_order_value (Delivered) / sum of lines | 22,958,987,430 / 22,958,987,430 |

Two different tables (header and lines) agree to the last unit. This is the first independent confirmation of the revenue total.

**Check 5: second method and other boundary rule.**

*Part 1, second method.* Revenue rebuilt as quantity times unit_price (not the line_total column) with a pivot table; share, running total and class recomputed separately.

| Test | Result |
|---|---|
| Products compared | 30 |
| Products where revenue differs | 0 |
| Total revenue, second method | 22,958,987,430 |
| Share differs by more than 0.01 | 0 products |
| Running total differs by more than 0.01 | 0 products |
| Class differs | 0 products |

Both methods read the same `sales` table; Check 8 tests the table itself against the raw files.

*Part 2, other boundary rule.* Under the other rule 2 products change: P022 Control Valve CV-75 (A to B) and P005 Fuel Injector FI-220 (B to C). The other 28 stay.

| Rule | A | B | C |
|---|---|---|---|
| Ours (running total before the product) | 10 products, 18,718,459,400 (81.53%) | 7 products, 3,229,679,500 (14.07%) | 13 products, 1,010,848,530 (4.40%) |
| Other (running total at the product) | 9 products, 18,020,540,100 (78.49%) | 7 products, 3,585,542,800 (15.62%) | 14 products, 1,352,904,530 (5.89%) |

Both rows sum to 22,958,987,430. The revenue of P022 (697,919,300) and P005 (342,056,000) accounts exactly for the shifts. Decision: keep our rule (A means the products needed to reach at least 80%) and show both.

**Check 6: outliers.**

| Test | Result |
|---|---|
| Largest line divided by average line, per product | lowest 1.86, highest 1.95 (price is constant per product, so this is the quantity ratio) |
| Largest single line | 2,380,000 = 0.010% of all revenue |
| Lines with the maximum quantity (20) | 5,852 = 4.97% of lines |
| Rows left after removing those lines | 111,804 |
| Products whose class changes without those lines | 0 (including P022 and P005) |

**Check 7: history.**

*Part 1, coverage.*

| Test | Result |
|---|---|
| Lines with no order date | 0 |
| Order dates | 2019-01-01 to 2024-12-31 |
| Months, first and last | 72; 2019-01 and 2024-12 |
| Lines per month | lowest 1,334, median 1,614, highest 2,126 |
| First month / last month | 1,679 / 1,488 lines (both inside the normal range) |

| Year | Lines | Units | Revenue (before GST) |
|---|---|---|---|
| 2019 | 19,710 | 207,925 | 3,870,016,600 |
| 2020 | 20,205 | 212,007 | 3,917,532,120 |
| 2021 | 19,745 | 208,269 | 3,852,479,620 |
| 2022 | 19,264 | 202,242 | 3,816,288,220 |
| 2023 | 19,606 | 204,638 | 3,824,675,310 |
| 2024 | 19,126 | 199,858 | 3,677,995,560 |
| Total | 117,656 | 1,234,939 | 22,958,987,430 |

Years use the order date. The totals tie to the figures in Check 4.

*Part 2, class per year.* Products were ranked again inside each year with the same rule.

| Year | Products whose class differs from the overall class |
|---|---|
| 2019, 2021, 2022, 2023, 2024 | 0 each |
| 2020 | 2 |

Every year has A 10, B 7, C 13. Only 2 products differ in any year, both in 2020, and they swapped: P022 (overall A) was B and P009 (overall B) was A. 178 of 180 product-year classes (30 products times 6 years) equal the overall class, about 98.9%. The reason for the swap is in 3.3.

**Check 8: manual spot-check (3 products, worked out by hand from the raw rows).** The three cover the top of the ranking (P015, rank 1), the bottom (P026, rank 30) and the boundary where a class depends on the running total (P005, rank 17).

| Item | P015 Engine Block EB-600 | P005 Fuel Injector FI-220 | P026 Grease Cartridge GC-5 |
|---|---|---|---|
| Rank | 1 | 17 | 30 |
| Raw Delivered lines | 3,851 | 3,896 | 3,888 |
| First raw lines (quantity times price) | SO-431510 line 1: 13 x 119,000 = 1,547,000; SO-495859 line 4: 20 x 119,000 = 2,380,000; SO-508292 line 5: 14 x 119,000 = 1,666,000 | SO-431510 line 5: 3 x 8,450 = 25,350; SO-527363 line 1: 12 x 8,450 = 101,400; SO-527363 line 4: 13 x 8,450 = 109,850 | SO-990591 line 2: 2 x 310 = 620; line 3: 3 x 310 = 930; line 10: 16 x 310 = 4,960 |
| Units x list price by hand | 40,815 x 119,000 = 4,856,985,000 | 40,480 x 8,450 = 342,056,000 | 40,407 x 310 = 12,526,170 |
| Revenue from raw lines | 4,856,985,000 | 342,056,000 | 12,526,170 |
| Revenue in the saved table | 4,856,985,000 | 342,056,000 | 12,526,170 |
| Share % | 21.16 | 1.49 | 0.05 |
| Running total % (raw / saved) | 21.16 / 21.16 | 95.60 / 95.60 | 100.00 / 100.00 |
| Class (raw / saved) | A / A | B / B (running total before it 94.11, under 95) | C / C |

The three products agree with the saved table on every line.

*Additional evidence inside Check 8 (not the check itself).*

(a) Four more products worked out the same way from the raw files, all equal to the saved table:

| Product | Rank | Units x list price = revenue | Share % | Running total % | Class |
|---|---|---|---|---|---|
| P014 Transmission Assembly TA-800 | 2 | 40,574 x 83,500 = 3,387,929,000 | 14.76 | 35.91 | A |
| P022 Control Valve CV-75 | 10 | 41,297 x 16,900 = 697,919,300 | 3.04 | 81.53 | A (running total before it 78.49, under 80) |
| P009 Starter Motor SM-700 | 11 | 40,547 x 14,900 = 604,150,300 | 2.63 | 84.16 | B (first B product) |
| P025 Cabin Glass CG-20 | 18 | 41,091 x 7,850 = 322,564,350 | 1.40 | 97.00 | C (first C product; running total before it 95.60) |

(b) Raw-file rebuild. The raw files were read again fresh: 20,000 header rows, 130,402 line rows, 30 products. Delivered: 18,033 orders, 117,656 lines, 1,234,939 units, revenue 22,958,987,430. Products rebuilt 30; products where revenue differs from the saved table 0; class differs 0. Line by line on (so_id, line_number): 117,656 lines in the rebuild and 117,656 in `sales`; lines only in the raw rebuild 0; only in `sales` 0; compared field by field 117,656; lines where product, quantity, price or line_total differs 0.

(c) Hand check of order SO-990591 (Delivered, header total_order_value 1,345,720): 11 raw lines: P018 19 x 11,300 = 214,700; P026 2 x 310 = 620; P026 3 x 310 = 930; P013 9 x 1,450 = 13,050; P004 3 x 39,800 = 119,400; P001 9 x 24,500 = 220,500; P021 3 x 46,200 = 138,600; P010 11 x 34,900 = 383,900; P001 10 x 24,500 = 245,000; P026 16 x 310 = 4,960; P008 7 x 580 = 4,060. The sum of the raw line totals is 1,345,720, equal to the header total and to the 11 lines in `sales`. The order also shows repeated product lines (P026 on 3 lines, P001 on 2).

### 3.3 Supporting tests

These tests check the claims made in this document. They do not replace any of the 8 checks above.

**T1: cancelled orders included.** If cancelled orders were counted, would any class change?

| Status | Orders | Lines | Units | Revenue (before GST) | Share |
|---|---|---|---|---|---|
| Delivered | 18,033 | 117,656 | 1,234,939 | 22,958,987,430 | 90.22% |
| Cancelled | 1,967 | 12,746 | 133,595 | 2,488,405,130 | 9.78% |
| Total | 20,000 | 130,402 | | 25,447,392,560 | 100% |

No order or line lacks a status. By hand: 22,958,987,430 + 2,488,405,130 = 25,447,392,560, 18,033 + 1,967 = 20,000 and 117,656 + 12,746 = 130,402. With all statuses counted, 0 products change class. The Delivered-only scope is justified and changes no class.

**T2: GST included.** If products were ranked with GST included (line_grand_total), would any class change? Total revenue with GST is 29,253,993,162.40, which equals 22,958,987,430 plus 6,295,005,732.40. Result: 30 products compared, classes A 10 / B 7 / C 13, and 0 products change class. Why no class moves was not tested, so no reason is claimed.

**T3: units instead of money.** Would ranking by pieces sold pick out the important products? Total units 1,234,939; each product holds 3.2% to 3.46% of units (an equal split would be 3.33%). To reach 80% of units, 24 of the 30 products are needed (by revenue: 10); to reach 95%, 29. Classes by units: A 24, B 5, C 1; 17 products get a class different from ours; our 10 A products are all A by units, which is expected because A by units holds 24 products. Result: units are almost equal across products, so ranking by units cannot pick out the vital few.

**T4: per branch.** Is each product's class the same in every branch? Branch revenue: AHM001 3,986,314,570; CHN001 4,682,202,010; DEL001 3,500,031,300; HYD001 3,077,048,220; KOL001 4,339,861,320; PUN001 3,373,530,010 (sum 22,958,987,430; no line without a branch). Every branch has A 10 / B 7 / C 13. Products with a class different from the overall class: AHM001 0, CHN001 0, DEL001 0, HYD001 0, PUN001 0, KOL001 2. The same 10 products are A in all 6 branches, and 178 of 180 product-branch classes equal the overall class.

**T5: price versus revenue.** Does list price explain the revenue ranking?

| Test | Result |
|---|---|
| Rank correlation, list price and revenue (30 products) | 0.999 |
| Rank correlation, units and revenue | 0.171 |
| Units, smallest / largest / ratio | 39,486 / 42,702 / 1.08 |
| List price, smallest / largest / ratio | 310 / 119,000 / 383.9 |
| Revenue, smallest / largest / ratio | 12,526,170 / 4,856,985,000 / 387.7 |
| Our A products among the 10 highest list prices | 10 of 10 |

The price ranking and the revenue ranking nearly match. The 0.999 (not 1) comes from small swaps: P006 ranks above P018 although P018 costs slightly more (P006 sold 41,843 pieces, P018 40,752), and P019 and P004 share a price of 39,800.

**Revenue-versus-units test.** Why did revenue fall by a different percentage than units? Compared 2019-2021 with 2022-2024: units 628,201 to 606,738 (-3.42%); revenue 11,640,028,340 to 11,318,959,090 (-2.76%). The product changes add up to -321,069,250, equal to the total revenue change. The five biggest revenue falls are all A products: P019 Swing Motor SWM-350 -55,919,000; P015 Engine Block EB-600 -52,003,000; P010 Turbocharger TB-900 -45,439,800; P021 Boom Cylinder BC-400 -44,906,400; P004 Track Chain TC-45 -29,093,800 (together -227,362,000, about 71% of the fall). The five biggest rises: P014 +9,853,000; P006 +9,800,000; P030 +4,708,500; P024 +1,421,400; P029 +1,082,300. By class: A -237,883,400; B -55,193,800; C -27,992,050. By hand, the net 21,463 pieces lost were worth on average about 14,960 each, against an overall average of 18,591 per piece (22,958,987,430 / 1,234,939), which is why revenue fell less than units. Because every product sells at a constant price (Check 2), any gap between revenue and units comes only from which products the units are. The first view (2019 against 2024: revenue about -5.0%, units about -3.9%) is an endpoint effect, since 2024 was a weak year.

**P009 / P022 swap test (2020).** Why did P009 and P022 swap class in 2020? P022 costs 16,900 and P009 14,900, so P009 needs 1.134 pieces for each piece of P022 to match its revenue.

| Year | P009 revenue | P022 revenue | P009 units | P022 units | P009 units per P022 unit | P009 minus P022 revenue |
|---|---|---|---|---|---|---|
| 2019 | 104,508,600 | 124,637,500 | 7,014 | 7,375 | 0.951 | -20,128,900 |
| 2020 | 111,899,000 | 107,129,100 | 7,510 | 6,339 | 1.185 | +4,769,900 |
| 2021 | 98,235,700 | 119,094,300 | 6,593 | 7,047 | 0.936 | -20,858,600 |
| 2022 | 96,894,700 | 113,568,000 | 6,503 | 6,720 | 0.968 | -16,673,300 |
| 2023 | 98,295,300 | 121,443,400 | 6,597 | 7,186 | 0.918 | -23,148,100 |
| 2024 | 94,317,000 | 112,047,000 | 6,330 | 6,630 | 0.955 | -17,730,000 |

Only in 2020 did the ratio (1.185) go above 1.134. 2020 was P022's year with the fewest pieces (6,339) and P009's year with the most (7,510). The price did not change. Why customers bought differently is not in the data.

**Kolkata swap test.** Why did P005 and P025 swap class in KOL001? P005 costs 8,450 and P025 7,850, so P025 needs 1.076 pieces for each piece of P005 to match its revenue.

| Branch | P005 pieces | P025 pieces | P025 per P005 piece | P025 minus P005 revenue |
|---|---|---|---|---|
| AHM001 | 7,158 | 6,974 | 0.974 | -5,739,200 |
| CHN001 | 8,368 | 8,563 | 1.023 | -3,490,050 |
| DEL001 | 6,234 | 5,798 | 0.930 | -7,163,000 |
| HYD001 | 5,663 | 5,977 | 1.055 | -932,900 |
| KOL001 | 7,251 | 7,956 | 1.097 | +1,183,650 |
| PUN001 | 5,806 | 5,823 | 1.003 | -3,350,150 |

Only Kolkata is above 1.076 (P025 revenue 62,454,600 against P005 61,270,950). The swap there comes from the mix of pieces, not from price. Why Kolkata customers bought relatively more Cabin Glass is not in the data.

### 3.4 Glossary of labels used in the notebook and in this document

| Label | Plain name | Meaning |
|---|---|---|
| T1 | Cancelled orders included | Would classes change if cancelled orders were counted? |
| T2 | GST included | Would classes change if revenue included GST? |
| T3 | Units instead of money | Would ranking by pieces sold pick out the important products? |
| T4 | Per branch | Is each product's class the same in every branch? |
| T5 | Price versus revenue | Does list price explain the revenue ranking? |
| Check 3 part 2 | Orders with 3 or more lines of the same product | Test for copied lines in the 982 larger combinations |
| Check 3 part 3 | Line key and ties | No duplicate line numbers, no identical rows, no ties in the ranking |
| Check 5 part 1 | Second method | Revenue rebuilt as quantity times price, classes recomputed |
| Check 5 part 2 | Other boundary rule | Class by the running total at the product instead of before it |
| Check 7 part 1 | History coverage | Dates, 72 months, yearly totals |
| Check 7 part 2 | Class per year | Classes recalculated for each year 2019 to 2024 |
| Revenue-versus-units test | | Why revenue fell less than units (2019-2021 against 2022-2024) |
| P009 / P022 swap test | | Why those two products swapped class in 2020 |
| Kolkata swap test | | Why P005 and P025 swapped class in Kolkata |
| Check 8 raw rebuild | | Everything rebuilt from the raw files and compared line by line with `sales` |
| Check 8 hand spot-check | | Three products worked out by hand from raw rows |
| M1 to M4 | Metrics | Revenue per product, share %, running total %, A/B/C class |
| Rule "before the product" | | A product is A if the running total before it is under 80%, B if under 95%, otherwise C |
| `sales` | | Delivered order lines (117,656) |
| `rev` | | The 30 products ranked by revenue |
| `raw_h`, `raw_l`, `raw_p` | | Raw header, line and product files read fresh for Check 8 |

---

## 4. Data Quality Notes

1. **Scope.** Only Delivered orders are counted. Cancelled orders (1,967 orders, 12,746 lines, 9.78% of ordered value) are excluded; counting them would change no class (T1).
2. **No discounts.** Every Delivered line was sold at its list price (Check 2), so revenue per product equals units times list price.
3. **Repeated product lines.** 12,184 order-product combinations have 2 or more lines (25,422 lines, about 22%). The rate of equal quantities is at chance level in both the 2-line and the 3-or-more-line groups, so the repeats look like coincidence and not double counting (Check 3). Order SO-990591 is an example: P026 on 3 lines and P001 on 2.
4. **Boundary products.** P022 (A) and P005 (B) sit exactly at the limits. Under the other common rule they would be B and C. The class limits are a convention, so these two classes are the least firm.
5. **A is a third of the products.** 10 of 30 products make up class A, not 20%. The reason is the price tiers described in section 5.
6. **Year and branch swaps.** P009 and P022 swapped in 2020 and P005 and P025 swapped in Kolkata. The pieces sold explain both (3.3). Why customers bought differently is not in the data.
7. **Revenue before GST.** GST is about 27.4% of revenue. Ranking with GST included gives the same classes (T2).
8. **Years use order date.** Yearly figures are by order date, 2019-01-01 to 2024-12-31.
9. **Scope of the classification.** KPI 7 classifies products by recorded sales revenue only. It uses no cost, margin or stock figures.
10. **Revenue trend.** 2019 against 2024 shows revenue about -5.0% and units about -3.9%. This is an endpoint effect; on 3-year blocks revenue fell 2.76% and units 3.42%.

---

## 5. Business Interpretation

**Concentration.** 10 of the 30 products (class A) bring in 81.53% of the revenue. P015 Engine Block EB-600 alone brings in 21.16%, and P015 with P014 Transmission Assembly TA-800 bring in 35.91%. The 13 products in class C together bring in 4.40%.

**What drives the ranking.** Revenue rank follows list price almost exactly (rank correlation 0.999), because every product sells about the same number of pieces (39,486 to 42,702, a gap of 8%) while list prices run from 310 to 119,000. Our 10 A products are exactly the 10 most expensive products. The data therefore shows a price tier, not a demand difference: units differ too little between products to show whether customers want some products more than others.

**Stable picture.** The same 10 products are A in all 6 branches (except a swap of two neighbouring products in Kolkata) and in every year (except a swap of two neighbouring products in 2020). Removing cancelled orders, adding GST, or removing the largest lines changes no class.

**Trend.** Revenue fell 2.76% between 2019-2021 and 2022-2024 while units fell 3.42%. All five of the biggest revenue falls are A products (P019, P015, P010, P021, P004), together about 71% of the fall.

**Ranking by units does not work here.** Because units are nearly equal, 24 of 30 products would be needed to reach 80% of units. Revenue is the measure that separates the important products.

### Recommended actions for the business owners

1. **Inventory planning team:** give the 10 A products the closest stock attention, since together they bring in 81.53% of revenue.
2. **Sales manager:** review the five A products with the biggest revenue falls between 2019-2021 and 2022-2024 (P019, P015, P010, P021, P004), which account for about 71% of the fall.
3. **Purchasing manager and inventory planning team:** do not treat class C products as low priority because of volume. Their unit sales are as high as those of A products; they are in class C because their prices are low (310 to 7,850).
4. **Data owner:** if class is used in a business rule, treat P022 and P005 as boundary products, since a different common rule would move them to B and C.
