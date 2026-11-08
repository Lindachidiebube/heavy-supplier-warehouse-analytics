# KPI 11: Warehouse Throughput

Project: Heavy Supplier & Warehouse Analytics | Week 6 | Notebook: Week6_KPI11_Warehouse_Throughput

## Summary

- Warehouse throughput is the number of units a branch warehouse receives (IN) and ships (OUT) each month. It is measured on 72 full months, January 2019 to December 2024, for the 6 branches.
- In total, 22,387,123 units were received and 1,234,939 units were shipped. Receiving is 94.77% of all units moved and shipping is 5.23%, so IN and OUT are reported separately.
- Average monthly units IN are almost equal across branches (50,355 at KOL001 to 52,296 at CHN001). Average monthly units OUT differ by about 1.5 times (2,339 at HYD001 to 3,522 at CHN001).
- Per employee per month, units OUT run from 27.9 (DEL001) to 65.3 (KOL001), and units IN from 565.5 (DEL001) to 1,027.7 (KOL001). Staffing is one current value per branch.
- Yearly units OUT are flat (200,562 to 211,190 a year in total). HYD001 ships the least in all 6 full years. Yearly units IN have no steady leader; the swings come mostly from how many purchase orders arrive (correlation 0.91) and partly from order size (0.67).
- All 8 verification checks were run on both sides (sales and purchases). Every comparison between two methods gave a difference of 0. Stated limits: staffing is a current snapshot, and January 2019 is a start-of-data month (it lowers the averages by under 1% and changes no ranking).

## 1. Definition

| Item | Definition |
|---|---|
| KPI | Warehouse Throughput |
| Units IN | Units on Received purchase order lines, counted in the month of received_date |
| Units OUT | Units on Delivered sales order lines, counted in the month of delivery_date |
| Headline | Average monthly units IN and average monthly units OUT per branch, over 72 full months |
| Supporting figures | Units IN and OUT per branch per year, lowest and busiest month, units IN per 1 unit OUT, units per employee per month |
| Period | January 2019 to December 2024 (72 months). 2025 holds only the first days of January and is left out |
| Branches | AHM001, CHN001, DEL001, HYD001, KOL001, PUN001 |
| Context only | Total movement (IN plus OUT) |

**Business question.** How many units does each branch warehouse receive and ship, how does that change by year and by month, and how much work is that for each employee?

**Why this metric.** Receiving and shipping are different jobs in a warehouse with different drivers (purchase orders in, sales orders out). Reporting them separately shows where the work is. A combined IN plus OUT figure counts the same goods twice, once in and once out, and here it only follows IN because IN is 94.77% of the total. Throughput is also the base for the next KPI, warehouse space use.

**Scope.** Branch warehouses only. Warehouse capacity and space belong to the next KPI.

## 2. Methodology

### Sources and scoping

- Sales side: sales order lines (`so_l`) joined to the sales order headers (`so_h`) on so_id. Only orders with order_status Delivered are counted. The branch comes from the header (the lines have no branch_id).
- Purchase side: purchase order lines (`po_l`) joined to the purchase order headers (`po_h`) on po_id. Only orders with po_status Received are counted. The branch comes from the header.
- Dates: delivery_date for sales and received_date for purchases, because that is when the goods physically move.
- Staffing and warehouse details: the branches table (`branches`): warehouse_type, warehouse_capacity, total_employees, monthly_operational_cost.
- The stock ledger was loaded and not used: the order lines are complete, and the ledger misses about 10% of movements (Week 3 finding).
- Cancelled sales lines and not-received purchase lines are left out because no goods moved.
- 2025 is left out of yearly comparisons and averages because it holds only the first days of January (sales to 2025-01-13, purchases to 2025-01-28).

### Step 1: Load and size check (Cell 1)

| Table | Rows | Columns |
|---|---|---|
| branches | 6 | 13 |
| ledger | 237,230 | 9 |
| so_h | 20,000 | 11 |
| so_l | 130,402 | 9 |
| po_h | 24,000 | 10 |
| po_l | 155,495 | 9 |

All row counts match the verified counts of earlier weeks.

### Step 2: Branch link check (Cell 2)

| Test | Result |
|---|---|
| Sales lines whose so_id is not in the sales headers | 0 |
| Purchase lines whose po_id is not in the purchase headers | 0 |
| Sales headers with a branch_id not in the branches table | 0 |
| Purchase headers with a branch_id not in the branches table | 0 |
| Branches found in sales headers and in purchase headers | AHM001, CHN001, DEL001, HYD001, KOL001, PUN001 (all 6, both) |

Every order line ties to a branch.

### Step 3: Build the sales and purchase line tables (Cell 3)

| Table | Content | Lines | Units |
|---|---|---|---|
| sales | Delivered sales lines with branch_id, order_date, delivery_date | 117,656 | 1,234,939 |
| purch | Received purchase lines with branch_id, order_date, received_date | 140,077 | 22,387,123 |

The sales figures tie to Week 5. Lines missing a branch or a delivery date: 0. Lines missing a branch or a received date: 0. Units IN are about 18 times units OUT.

### Step 4: Throughput per branch (Cell 4)

| Branch | Units IN | Lines IN | Orders IN | Units OUT | Lines OUT | Orders OUT | Units total | Share of total % |
|---|---|---|---|---|---|---|---|---|
| AHM001 | 3,681,090 | 23,104 | 3,603 | 214,223 | 20,425 | 3,115 | 3,895,313 | 16.49 |
| CHN001 | 3,797,868 | 23,655 | 3,633 | 253,921 | 24,202 | 3,677 | 4,051,789 | 17.15 |
| DEL001 | 3,777,619 | 23,607 | 3,616 | 185,319 | 17,511 | 2,682 | 3,962,938 | 16.78 |
| HYD001 | 3,718,428 | 23,217 | 3,606 | 168,975 | 16,102 | 2,478 | 3,887,403 | 16.46 |
| KOL001 | 3,661,187 | 22,919 | 3,547 | 230,868 | 21,989 | 3,413 | 3,892,055 | 16.48 |
| PUN001 | 3,750,931 | 23,575 | 3,625 | 181,633 | 17,427 | 2,668 | 3,932,564 | 16.65 |

Printed ties: units IN 22,387,123, units OUT 1,234,939, received purchase orders 21,630, delivered sales orders 18,033. All tie.

Reading: units IN are almost equal across branches (3.66 million to 3.80 million). Units OUT differ a lot (HYD001 168,975 is the lowest, CHN001 253,921 the highest, about 1.5 times). The total shares look equal (16.46% to 17.15%) only because IN is about 95% of the total.

Units OUT are 5.23% of total movement (1,234,939 of 23,622,062) and units IN are 94.77%. CHN001's total is only about 4.2% above HYD001's, while its units OUT are about 50% higher. This is why IN and OUT are reported separately.

### Step 5: Units per branch per year (Cell 5)

Sales year comes from delivery_date, purchase year from received_date. 2025 is partial.

Units OUT:

| Branch | 2019 | 2020 | 2021 | 2022 | 2023 | 2024 | 2025 (partial) | Total |
|---|---|---|---|---|---|---|---|---|
| AHM001 | 33,943 | 39,000 | 36,971 | 33,767 | 38,048 | 31,921 | 573 | 214,223 |
| CHN001 | 46,886 | 40,613 | 40,806 | 41,049 | 43,376 | 40,818 | 373 | 253,921 |
| DEL001 | 28,812 | 30,422 | 30,001 | 34,287 | 29,414 | 31,967 | 416 | 185,319 |
| HYD001 | 27,519 | 29,400 | 28,210 | 27,715 | 27,763 | 27,816 | 552 | 168,975 |
| KOL001 | 37,478 | 39,862 | 41,681 | 36,657 | 37,610 | 36,941 | 639 | 230,868 |
| PUN001 | 29,948 | 31,893 | 29,266 | 29,241 | 29,367 | 31,099 | 819 | 181,633 |
| TOTAL | 204,586 | 211,190 | 206,935 | 202,716 | 205,578 | 200,562 | 3,372 | 1,234,939 |

Units IN:

| Branch | 2019 | 2020 | 2021 | 2022 | 2023 | 2024 | 2025 (partial) | Total |
|---|---|---|---|---|---|---|---|---|
| AHM001 | 581,390 | 629,042 | 658,827 | 566,431 | 599,234 | 615,796 | 30,370 | 3,681,090 |
| CHN001 | 606,670 | 676,736 | 567,223 | 635,957 | 635,778 | 642,929 | 32,575 | 3,797,868 |
| DEL001 | 616,681 | 588,384 | 611,358 | 664,321 | 638,003 | 627,088 | 31,784 | 3,777,619 |
| HYD001 | 570,983 | 621,929 | 684,765 | 597,334 | 628,241 | 576,606 | 38,570 | 3,718,428 |
| KOL001 | 592,832 | 613,468 | 588,516 | 636,683 | 625,109 | 568,941 | 35,638 | 3,661,187 |
| PUN001 | 588,389 | 657,263 | 665,570 | 581,404 | 626,573 | 605,715 | 26,017 | 3,750,931 |
| TOTAL | 3,556,945 | 3,786,822 | 3,776,259 | 3,682,130 | 3,752,938 | 3,637,075 | 194,954 | 22,387,123 |

Both total rows tie to Step 4.

Reading (checked by hand against the tables):

- Units OUT: HYD001 is the lowest branch in all 6 full years. CHN001 is the highest in 5 of 6 years (KOL001 is highest in 2021, 41,681 against 40,806). The yearly OUT total is flat (200,562 to 211,190).
- Units IN: no branch leads every year. The leader by year is DEL001, CHN001, HYD001, DEL001, DEL001, CHN001.
- 2025 holds only the first days of January and is left out of yearly comparisons and averages.

### Step 6: Monthly average, lowest and busiest month (Cell 6)

Full months January 2019 to December 2024. Every branch has 72 months on both sides (no missing month).

| Branch | Avg OUT | Lowest OUT month | Busiest OUT month | Avg IN | Lowest IN month | Busiest IN month |
|---|---|---|---|---|---|---|
| AHM001 | 2,967 | 1,294 | 4,150 | 50,704 | 21,887 | 73,945 |
| CHN001 | 3,522 | 1,789 | 5,458 | 52,296 | 20,693 | 72,109 |
| DEL001 | 2,568 | 1,545 | 3,824 | 52,025 | 24,155 | 74,465 |
| HYD001 | 2,339 | 1,340 | 3,267 | 51,109 | 21,406 | 72,092 |
| KOL001 | 3,198 | 2,097 | 4,517 | 50,355 | 27,968 | 69,168 |
| PUN001 | 2,511 | 1,373 | 3,697 | 51,735 | 27,711 | 76,477 |

Hand check: (yearly total minus 2025) divided by 72 reproduces every average. For example CHN001 OUT (253,921 minus 373) / 72 = 3,521.5, and CHN001 IN (3,797,868 minus 32,575) / 72 = 52,296.

Reading (by hand from the table):

- Average monthly units OUT run from HYD001 2,339 to CHN001 3,522 (about 1.5 times). Average monthly units IN are about equal (50,355 to 52,296).
- Units IN per 1 unit OUT: CHN001 14.8, KOL001 15.7, AHM001 17.1, DEL001 20.3, PUN001 20.6, HYD001 21.9.
- Busiest month divided by quietest month: units OUT AHM001 3.2, CHN001 3.1, DEL001 2.5, HYD001 2.4, KOL001 2.2, PUN001 2.7; units IN AHM001 3.4, CHN001 3.5, DEL001 3.1, HYD001 3.4, KOL001 2.5, PUN001 2.8. The quietest IN month is January 2019 for every branch (Check 7).

### Step 7: Branch details check (Cell 7)

6 rows, 0 empty values in every column.

| Branch | Warehouse type | Warehouse capacity | Total employees | Monthly operational cost |
|---|---|---|---|---|
| DEL001 | Central | 45230 sqft | 92 | 3,189,000 |
| PUN001 | Regional | 38750 sqft | 74 | 2,654,000 |
| CHN001 | Regional | 41890 sqft | 81 | 2,739,000 |
| HYD001 | Branch | 36210 sqft | 57 | 1,887,000 |
| KOL001 | Branch | 29880 sqft | 49 | 1,743,000 |
| AHM001 | Service Hub | 34120 sqft | 63 | 2,018,000 |

total_employees and monthly_operational_cost are one current value per branch (see Data Quality Note 6).

### Step 8: Units per employee per month (Cell 8)

Average monthly units divided by total_employees. 0 empty values.

| Branch | Avg OUT | Avg IN | Employees | OUT per employee | IN per employee |
|---|---|---|---|---|---|
| AHM001 | 2,967 | 50,704 | 63 | 47.1 | 804.8 |
| CHN001 | 3,522 | 52,296 | 81 | 43.5 | 645.6 |
| DEL001 | 2,568 | 52,025 | 92 | 27.9 | 565.5 |
| HYD001 | 2,339 | 51,109 | 57 | 41.0 | 896.6 |
| KOL001 | 3,198 | 50,355 | 49 | 65.3 | 1,027.7 |
| PUN001 | 2,511 | 51,735 | 74 | 33.9 | 699.1 |

Reading (by hand): OUT per employee is highest at KOL001 (65.3, the smallest team of 49) and lowest at DEL001 (27.9, the largest team of 92), about 2.3 times apart. IN per employee is highest at KOL001 (1,027.7) and lowest at DEL001 (565.5), about 1.8 times apart. Because units IN are about equal in every branch, the IN per employee differences come only from staff numbers. CHN001 is the busiest in units OUT but third per employee.

### Alternatives considered

| Alternative | Decision | Reason |
|---|---|---|
| Stock ledger as the source of movements | Not used | The ledger misses about 10% of movements (Week 3). The order lines are complete |
| One combined IN plus OUT figure as the headline | Not used | The same goods are counted twice, and the total only follows IN (94.77%). Shown as context only |
| Order date for the month | Not used | delivery_date and received_date are when the goods move. Purchase lead time is 7 to 35 days (Check 7) |
| Include cancelled and not-received lines | Not used | No goods moved. 12,746 sales lines and 15,418 purchase lines are left out (Check 4) |
| Include 2025 | Not used | Only the first days of January exist |
| Drop January 2019 | Not used | It lowers the averages by under 1% and changes no ranking (Check 7) |
| Lines or orders instead of units | Kept as supporting figures | Lines and orders are in Step 4. Units are the headline |
| Warehouse capacity and space use | Left to the next KPI | Space use is a different question from throughput |

## 3. Verification Log

### Summary of the 8 checks

| # | Check | Method | Result |
|---|---|---|---|
| 1 | Missing, zero and negative values | Counted empty values in branch_id, product_id, quantity and the dates; looked at quantity range and date conversion, on sales and purchases | No empty values, no zero or negative quantity, no date failed to convert on either side |
| 2 | Edge case: every branch has IN and OUT in all 72 months | Counted months per branch and branch-months with no units or zero units | 72 months in every branch on both sides, no branch-month with no units or zero units |
| 3 | Repeated product lines inside one order | Counted order and product pairs with repeated lines and, among them, exact copies, on both sides, and compared with chance | Sales exact-copy shares match chance; purchases have no exact copy because every repeated line carries a different unit cost |
| 4 | Totals reconcile | All lines equal counted lines plus left-out lines, on lines and units, and counted units equal months 2019 to 2024 plus 2025, on both sides | All differences 0 |
| 5 | Second-method recompute | Rebuilt branch units IN and OUT without joining tables (dictionary of order to branch), then the monthly averages and per-employee figures | All largest differences 0 |
| 6 | Outliers | Largest line against average line per branch on both sides; busiest and quietest month, and how much one line adds | No line above the allowed range; the biggest line is at most 1.55% of a month; months swing with the number of lines |
| 7 | History | First and last dates, 2025 lines, first and last months; test of what drives yearly IN swings; January 2019 tests | 72 full months; the 2025 lines explain the line gaps exactly; January 2019 effect under 1%; yearly IN swings come mostly from the number of orders |
| 8 | Manual spot-check of 3 items | Rebuilt AHM001 June 2019 units OUT, CHN001 January 2019 units IN and PUN001 November 2023 units IN, with each branch's average and per-employee figure, from the raw tables | All differences 0 |

### Check 1: Missing, zero and negative values

| Table | Rows | Empty in branch_id, product_id, quantity and date | Quantity zero or below | Smallest and largest quantity | Dates that failed to convert |
|---|---|---|---|---|---|
| sales | 117,656 | 0 | 0 | 1 and 20 | 0 |
| purchases | 140,077 | 0 | 0 | 20 and 300 | 0 |

By hand: average units per line IN is 22,387,123 / 140,077 = about 160, and OUT is 1,234,939 / 117,656 = about 10.5. Purchase lines are bulk lines.

### Check 2: Every branch has IN and OUT in all 72 months

| Side | Months expected | Months in table | Branch-months with no units | Branch-months at zero | Lowest month |
|---|---|---|---|---|---|
| OUT | 72 | 72 | 0 | 0 | 1,294 |
| IN | 72 | 72 | 0 | 0 | 20,693 |

### Check 3: Repeated product lines inside one order

An exact copy means the same order, the same product, the same quantity and the same price (sales) or cost (purchases).

First run:

| Side | Rows | Pairs with repeated lines | Pairs with an exact copy | Share |
|---|---|---|---|---|
| Sales | 117,656 | 12,184 | 684 | 5.61% |
| Purchases | 140,077 | 14,408 | 0 | 0.0% |

Chance is about 5.0% for sales (2 lines, quantity 1 to 20) and about 0.36% for purchases (quantity 20 to 300). The earlier KPI 5 run had 4.88% (547 of 11,202) and 0.40% (53 of 13,219).

Follow-up run (extra test of Check 3, purchases and sales split):

| Test | Result |
|---|---|
| Purchase pairs with repeated lines | 14,408 |
| Same quantity, cost ignored | 66 (0.46%) |
| Same unit cost, quantity ignored | 0 |
| Lines per purchase pair: 2 / 3 / 4 / 5 | 13,219 / 1,129 / 57 / 3 pairs (sum 14,408) |
| Sales pairs with exactly 2 lines | 11,202; exact copy 547 (4.88%, same as KPI 5) |
| Sales pairs with 3 or more lines | 982; exact copy 137 (13.95%); chance for exactly 3 lines, by hand, is 1 - (19/20 x 18/20) = 14.5% |
| Tie to the first run | 547 + 137 = 684 and 11,202 + 982 = 12,184 |

Result: purchases show 0 exact copies because every repeated line carries a different unit cost, so none can be a copy (the 66 pairs with the same quantity have different costs). Sales exact-copy shares match chance; the 5.61% overall is a mix of 2-line and 3-or-more-line pairs. The reason for the different unit costs is not in the data and no reason is claimed.

### Check 4: Totals reconcile

| Side | Measure | All | Counted | Left out | Difference |
|---|---|---|---|---|---|
| Sales | Lines | 130,402 | 117,656 | 12,746 (cancelled) | 0 |
| Sales | Units | 1,368,534 | 1,234,939 | 133,595 | 0 |
| Purchases | Lines | 155,495 | 140,077 | 15,418 (not received) | 0 |
| Purchases | Units | 24,867,716 | 22,387,123 | 2,480,593 | 0 |

Counted units equal the months 2019 to 2024 plus 2025: sales 1,231,567 + 3,372 = 1,234,939; purchases 22,192,169 + 194,954 = 22,387,123 (differences 0). By hand: the left-out share of lines is 12,746 / 130,402 = 9.8% (sales) and 15,418 / 155,495 = 9.9% (purchases); of units 9.8% and 10.0%.

### Check 5: Second-method recompute

The second method does not join tables. A dictionary maps each order id to its branch, the lines are filtered with the sets of Delivered and Received order ids, and the totals are summed by branch.

| Test | Result |
|---|---|
| Lines used by the second method | 117,656 OUT, 140,077 IN |
| Units OUT per branch | AHM001 214,223; CHN001 253,921; DEL001 185,319; HYD001 168,975; KOL001 230,868; PUN001 181,633 |
| Units IN per branch | AHM001 3,681,090; CHN001 3,797,868; DEL001 3,777,619; HYD001 3,718,428; KOL001 3,661,187; PUN001 3,750,931 |
| Largest difference, units OUT and IN | 0 and 0 |
| Largest difference, monthly average OUT and IN | 0.0 and 0.0 |
| Largest difference, OUT and IN per employee | 0.0 and 0.0 |

All branch totals equal Step 4.

### Check 6: Outliers

Full months January 2019 to December 2024 (sales 117,316 lines, purchases 138,831 lines).

| Side | Branch | Average line | Largest line | Largest / average |
|---|---|---|---|---|
| Sales | AHM001 | 10.5 | 20 | 1.9 |
| Sales | CHN001 | 10.5 | 20 | 1.9 |
| Sales | DEL001 | 10.6 | 20 | 1.9 |
| Sales | HYD001 | 10.5 | 20 | 1.9 |
| Sales | KOL001 | 10.5 | 20 | 1.9 |
| Sales | PUN001 | 10.4 | 20 | 1.9 |
| Purchases | AHM001 | 159.3 | 300 | 1.9 |
| Purchases | CHN001 | 160.6 | 300 | 1.9 |
| Purchases | DEL001 | 160.1 | 300 | 1.9 |
| Purchases | HYD001 | 160.2 | 300 | 1.9 |
| Purchases | KOL001 | 159.7 | 300 | 1.9 |
| Purchases | PUN001 | 159.1 | 300 | 1.9 |

Busiest and quietest month per branch (average lines per month in brackets):

| Side | Branch | Month type | Month | Units | Lines | Avg lines per month | Biggest line | Biggest line share of month % |
|---|---|---|---|---|---|---|---|---|
| Sales | AHM001 | busiest | 2019-06 | 4,150 | 381 | 283 | 20 | 0.48 |
| Sales | AHM001 | quietest | 2022-11 | 1,294 | 123 | 283 | 20 | 1.55 |
| Sales | CHN001 | busiest | 2019-08 | 5,458 | 499 | 336 | 20 | 0.37 |
| Sales | CHN001 | quietest | 2021-09 | 1,789 | 168 | 336 | 20 | 1.12 |
| Sales | DEL001 | busiest | 2022-01 | 3,824 | 355 | 243 | 20 | 0.52 |
| Sales | DEL001 | quietest | 2019-01 | 1,545 | 137 | 243 | 20 | 1.29 |
| Sales | HYD001 | busiest | 2021-12 | 3,267 | 310 | 223 | 20 | 0.61 |
| Sales | HYD001 | quietest | 2022-07 | 1,340 | 142 | 223 | 20 | 1.49 |
| Sales | KOL001 | busiest | 2023-08 | 4,517 | 443 | 304 | 20 | 0.44 |
| Sales | KOL001 | quietest | 2022-07 | 2,097 | 183 | 304 | 20 | 0.95 |
| Sales | PUN001 | busiest | 2022-07 | 3,697 | 361 | 241 | 20 | 0.54 |
| Sales | PUN001 | quietest | 2022-09 | 1,373 | 135 | 241 | 20 | 1.46 |
| Purchases | AHM001 | busiest | 2021-01 | 73,945 | 451 | 318 | 300 | 0.41 |
| Purchases | AHM001 | quietest | 2019-01 | 21,887 | 132 | 318 | 300 | 1.37 |
| Purchases | CHN001 | busiest | 2020-08 | 72,109 | 440 | 326 | 299 | 0.41 |
| Purchases | CHN001 | quietest | 2019-01 | 20,693 | 143 | 326 | 299 | 1.44 |
| Purchases | DEL001 | busiest | 2019-03 | 74,465 | 468 | 325 | 300 | 0.40 |
| Purchases | DEL001 | quietest | 2019-01 | 24,155 | 154 | 325 | 299 | 1.24 |
| Purchases | HYD001 | busiest | 2021-01 | 72,092 | 445 | 319 | 300 | 0.42 |
| Purchases | HYD001 | quietest | 2019-01 | 21,406 | 131 | 319 | 300 | 1.40 |
| Purchases | KOL001 | busiest | 2020-08 | 69,168 | 428 | 315 | 300 | 0.43 |
| Purchases | KOL001 | quietest | 2019-01 | 27,968 | 167 | 315 | 298 | 1.07 |
| Purchases | PUN001 | busiest | 2023-11 | 76,477 | 492 | 325 | 296 | 0.39 |
| Purchases | PUN001 | quietest | 2019-01 | 27,711 | 172 | 325 | 300 | 1.08 |

The units of every busiest and quietest month equal Step 6.

Reading (by hand): the ratio of 1.9 equals 20 / 10.5 and 300 / 160, the same as KPI 5. The largest line in every branch is the top of the allowed range (1 to 20 for sales, 20 to 300 for purchases, Check 1), so no line is above the normal range. The biggest line is at most 1.55% of a month. Busy months have more lines and quiet months fewer: AHM001 sales in 2019-06 had 381 lines against an average of 283 (1.35 times) and 4,150 units against 2,967 (1.40 times); in 2022-11 it had 123 lines (0.43 times) and 1,294 units (0.44 times). Months swing with the number of lines, not with a few huge lines.

### Check 7: History

| Side | Earliest order_date | Latest order_date | Earliest date used | Latest date used | Lines dated 2025 | 2025 dates run |
|---|---|---|---|---|---|---|
| Sales | 2019-01-01 | 2024-12-31 | delivery 2019-01-03 | delivery 2025-01-13 | 340 | 2025-01-01 to 2025-01-13 |
| Purchases | 2019-01-01 | 2024-12-31 | received 2019-01-11 | received 2025-01-28 | 1,246 | 2025-01-01 to 2025-01-28 |

| Side | Lines in 2019-01 / 02 / 03 | Lines in 2024-10 / 11 / 12 |
|---|---|---|
| Sales | 1,132 / 1,385 / 1,715 | 1,790 / 1,558 / 1,559 |
| Purchases | 899 / 1,831 / 2,197 | 1,831 / 1,954 / 2,045 |

Ties: Check 6 counted 117,316 sales lines and 138,831 purchase lines in full months. 117,656 minus 117,316 is 340 and 140,077 minus 138,831 is 1,246, exactly the 2025 lines. December 2024 is complete on both sides.

**Extra test of Check 7: January 2019 effect on the averages.** Average monthly units with all 72 months against 71 months without January 2019:

| Branch | OUT 72 months | OUT without Jan 2019 | OUT change % | IN 72 months | IN without Jan 2019 | IN change % |
|---|---|---|---|---|---|---|
| AHM001 | 2,967 | 2,985 | 0.58 | 50,704 | 51,110 | 0.80 |
| CHN001 | 3,522 | 3,533 | 0.34 | 52,296 | 52,741 | 0.85 |
| DEL001 | 2,568 | 2,583 | 0.56 | 52,025 | 52,418 | 0.75 |
| HYD001 | 2,339 | 2,351 | 0.50 | 51,109 | 51,527 | 0.82 |
| KOL001 | 3,198 | 3,209 | 0.37 | 50,355 | 50,670 | 0.63 |
| PUN001 | 2,511 | 2,520 | 0.34 | 51,735 | 52,073 | 0.65 |

The 72-month averages equal Step 6. January 2019 lowers the averages by 0.34% to 0.58% (OUT) and 0.63% to 0.85% (IN). The ranking of the branches is the same with and without it (OUT: CHN001, KOL001, AHM001, DEL001, PUN001, HYD001; IN: CHN001, DEL001, PUN001, HYD001, AHM001, KOL001). Decision: keep the 72 full calendar months (the same window as KPI 5).

**Extra test of Check 7: why January 2019 is the quietest IN month.** Purchase lead time (received_date minus order_date): smallest 7 days, average 17.9, largest 35.

| Month | Purchase lines ordered | Purchase lines received |
|---|---|---|
| 2019-01 | 2,002 | 899 |
| 2019-02 | 2,019 | 1,831 |
| 2019-03 | 2,132 | 2,197 |
| 2019-04 | 1,961 | 2,049 |

Ordering in January 2019 was normal (2,002 against 2,019 in February). Receipts were low (899, 0.45 times the lines ordered) because goods take 7 to 35 days to arrive and no orders exist before 2019-01-01. No reason is claimed for the lower sales in January 2019 (1,132 lines against 1,385 in February); its effect on the averages was measured above (under 0.6%).

**Extra test of Check 7: what drives the yearly IN swings.** Units IN per branch and year (2019 to 2024) equal the number of purchase orders received times the units per order. How much bigger the highest year is than the lowest year, in %:

| Branch | Orders received | Units per order | Units IN |
|---|---|---|---|
| AHM001 | 11.2 | 4.9 | 16.3 |
| CHN001 | 15.1 | 4.9 | 19.3 |
| DEL001 | 10.3 | 3.7 | 12.9 |
| HYD001 | 13.3 | 6.4 | 19.9 |
| KOL001 | 9.0 | 6.6 | 11.9 |
| PUN001 | 15.0 | 7.3 | 14.5 |

Across the 36 branch-years the correlation of units IN with orders received is 0.91, and with units per order 0.67. In all six branches the spread of orders received is bigger than the spread of units per order, so the yearly swings in units IN come mostly from how many purchase orders arrive and partly from order size. Why the order counts differ between years is not in the data.

### Check 8: Manual spot-check of 3 items

Each item was rebuilt from the raw tables (`so_h`, `so_l`, `po_h`, `po_l`), without the sales and purch tables, and compared with the reported figures.

| Item | Measure | By hand | Reported | Difference |
|---|---|---|---|---|
| 1. AHM001, June 2019, units OUT | Lines in the month | 381 | 381 | 0 |
| | Units in the month | 4,150 | 4,150 | 0 |
| | Monthly average, 72 months | 2,967 | 2,967 | 0 |
| | OUT per employee | 47.1 | 47.1 | 0 |
| 2. CHN001, January 2019, units IN | Lines in the month | 143 | 143 | 0 |
| | Units in the month | 20,693 | 20,693 | 0 |
| | Monthly average, 72 months | 52,296 | 52,296 | 0 |
| | IN per employee | 645.6 | 645.6 | 0 |
| 3. PUN001, November 2023, units IN | Lines in the month | 492 | 492 | 0 |
| | Units in the month | 76,477 | 76,477 | 0 |
| | Monthly average, 72 months | 51,735 | 51,735 | 0 |
| | IN per employee | 699.1 | 699.1 | 0 |

### Test labels

| Label | Plain name | Question | Result | Where it sits |
|---|---|---|---|---|
| Check 3 follow-up | Repeated purchase and sales lines split | Why do purchases show 0 exact copies, and is the sales share explained by chance? | Every repeated purchase line has a different unit cost; sales shares match chance | Check 3 |
| Check 6 busiest and quietest | Busiest and quietest month per branch | Could one big line drive a month? | No, the biggest line is at most 1.55% of a month | Check 6 |
| Check 7 January 2019 effect | Averages with and without January 2019 | Does the first month change the averages or the ranking? | Under 1%, ranking unchanged | Check 7 |
| Check 7 January 2019 reason | Lead time and lines ordered against received | Why is January 2019 the quietest IN month? | Goods take 7 to 35 days and no orders exist before 2019-01-01 | Check 7 |
| Check 7 yearly IN swing drivers | Orders received against units per order | What drives the yearly IN swings? | Mostly the number of orders (0.91), partly order size (0.67) | Check 7 |

## 4. Data Quality Notes

1. **Source.** Throughput is built from order lines joined to order headers. The lines have no branch_id, so the branch comes from the header. All lines tie to a branch (Step 2). The stock ledger is not used because it misses about 10% of movements.
2. **Lines left out.** 12,746 cancelled sales lines (133,595 units, 9.8% of lines) and 15,418 purchase lines that were not received (2,480,593 units, 9.9% of lines) are not counted because no goods moved. Both tie exactly (Check 4).
3. **2025 is partial.** It holds 340 sales lines (to 2025-01-13) and 1,246 purchase lines (to 2025-01-28). They are left out of yearly comparisons and averages, which cover 72 full months.
4. **January 2019 is a start-of-data month.** The first order is dated 2019-01-01 and the first purchase was received on 2019-01-11. It is the quietest IN month for every branch. It lowers the monthly averages by under 1% and changes no ranking, so the 72 months are kept (Check 7).
5. **Repeated purchase lines.** In 14,408 purchase order and product pairs the product appears on more than one line, always with a different unit cost. There is no exact copy. The reason for the different costs is not in the data and no reason is claimed. Units are not affected.
6. **Staffing is one current value per branch.** total_employees and monthly_operational_cost are one value per branch in the branches table. There is no other table to check them against and no history. The per-employee figures describe current staffing only.
7. **Units are counted equally.** A unit of any product counts as one unit, so throughput measures units, not value or weight.
8. **Sales and purchase lines differ in size.** Sales lines hold 1 to 20 units (about 10.5 on average) and purchase lines hold 20 to 300 units (about 160 on average). Units IN are about 18 times units OUT.
9. **Sales in January 2019.** Sales lines are lower in January 2019 (1,132) than in February (1,385). No reason is claimed. Its effect on the averages is under 0.6%.
10. **Reasons not in the data.** The data does not show why order counts differ between years or why the purchase unit costs differ within one order.

## 5. Business Interpretation

**Business question.** How many units does each branch warehouse receive and ship, how does that change by year and by month, and how much work is that for each employee?

**Answer.** Every branch receives about 50,000 to 52,000 units a month, so receiving is almost the same everywhere. Shipping is small and differs more: from 2,339 units a month at HYD001 to 3,522 at CHN001. Per employee, KOL001 carries the most work (65.3 units OUT and 1,027.7 units IN per month) and DEL001 the least (27.9 and 565.5).

### Findings

1. **Receiving dominates.** Units IN are 22,387,123 and units OUT 1,234,939, so 94.77% of the units moved are received and about 18 units are received for every 1 shipped.
2. **Receiving is even, shipping is not.** Average monthly units IN range from 50,355 to 52,296 across branches (about 1.04 times). Average monthly units OUT range from 2,339 to 3,522 (about 1.5 times).
3. **HYD001 ships the least.** It is the lowest in units OUT in all 6 full years, and it receives 21.9 units for every 1 it ships (CHN001 14.8).
4. **Shipping is stable over the years, receiving is not.** The yearly OUT total stays between 200,562 and 211,190. In units IN, no branch leads every year and the best year of a branch is 11.9% to 19.9% above its worst year.
5. **The yearly IN swings come mostly from the number of purchase orders received** (correlation 0.91), and partly from order size (0.67).
6. **Months are uneven.** The busiest month is 2.2 to 3.2 times the quietest on the OUT side and 2.5 to 3.5 times on the IN side, and busy months have more lines, not bigger lines.
7. **The work per employee differs a lot.** OUT per employee is about 2.3 times higher at KOL001 (65.3, 49 employees) than at DEL001 (27.9, 92 employees). IN per employee is about 1.8 times higher (1,027.7 against 565.5).
8. **About 10% of the lines are not counted.** Cancelled sales lines and purchase lines that were not received make up 9.8% and 9.9% of lines.

### Recommended actions for the business owners

1. **Warehouse Operations Manager:** plan receiving staff and dock time around the monthly peaks (busiest IN month up to 3.5 times the quietest), because staffing for the average month leaves the busy months short.
2. **Procurement Manager:** review the purchase order calendar of each branch, because the yearly swing in units IN follows the number of orders received, and spreading orders more evenly through the year would smooth the receiving workload.
3. **Branch Managers of DEL001 and KOL001:** compare staffing with workload. DEL001 has 92 employees for 2,568 units OUT a month, KOL001 has 49 for 3,198. Confirm the current staff numbers before moving people, because staffing in the data is a current snapshot.
4. **Branch Manager of HYD001:** review why shipping is the lowest of all branches in every full year while receiving matches the other branches (21.9 units received for each unit shipped).
5. **Data Owner (the team that keeps the stock records):** keep staffing and operational cost by month for every branch, so that throughput per employee can be followed over time instead of at one point.

## Glossary of short names used in the notebook

| Name | Meaning |
|---|---|
| IN, units IN | Units received on Received purchase order lines |
| OUT, units OUT | Units shipped on Delivered sales order lines |
| branches | Branch master table (6 rows, 13 columns) |
| ledger | Stock ledger table (237,230 rows); loaded, not used for throughput |
| so_h, so_l | Sales order headers (20,000) and sales order lines (130,402; no branch_id) |
| po_h, po_l | Purchase order headers (24,000) and purchase order lines (155,495; no branch_id) |
| sales | Delivered sales lines with branch_id, order_date and delivery_date (117,656 lines) |
| purch | Received purchase lines with branch_id, order_date and received_date (140,077 lines) |
| t | Per-branch table of Step 4: units_in, lines_in, orders_in, units_out, lines_out, orders_out, units_total, share_pct |
| out_y, in_y | Units OUT and units IN per branch per year, with a TOTAL column (Step 5) |
| sales_m, purch_m | Units per branch per month for sales and purchases (Step 6) |
| mo, mi | Month by branch tables of units OUT and units IN (Step 6, Check 2) |
| res | Result table of Step 6: months_out, months_in, avg_out, min_out, max_out, avg_in, min_in, max_in |
| per | Per-employee table of Step 8: avg_out, avg_in, total_employees, out_per_employee, in_per_employee |
| emp | total_employees by branch |
| dup_test | Function of Check 3 that counts order and product pairs with repeated lines and, among them, exact copies |
| so_branch, po_branch | Dictionaries that map each order id to its branch (Check 5) |
| so_ok, po_ok | Sets of Delivered sales order ids and Received purchase order ids (Check 5) |
| ls, lp | Sales lines and purchase lines of those orders (Check 5) |
| out2, in2 | Branch units OUT and IN from the second method (Check 5) |
| avg_out2, avg_in2 | Monthly averages from the yearly totals minus 2025, divided by 72 (Check 5) |
| check6 | Function of Check 6 that prints average line, largest line and the busiest and quietest month per branch |
| o, r, lead | Purchase order dates, received dates and the days between them (Check 7) |
| g, s | Branch-year table of units, orders and units per order, and its highest-against-lowest summary (Check 7) |
| hand, items | Function that rebuilds a branch-month, the 72-month average and the per-employee figure from the raw tables, and the 3 items it is run on (Check 8) |
| 72 months | January 2019 to December 2024 |
