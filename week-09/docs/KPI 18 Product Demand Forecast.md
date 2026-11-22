# KPI 18 Product Demand Forecast

Project: Heavy Supplier & Warehouse Analytics | Week 9 | Notebook: Week9 KPI 18 Product Demand Forecast

## Summary

KPI 18 forecasts how many units of each product each branch will sell, using the 72 months of Delivered sales lines (2019-01 to 2024-12: 117,656 lines, 1,234,939 units, 30 products in 6 branches, which gives 180 product-branch pairs). The method was judged by hiding the last 12 months (2024), forecasting them from the first 60 months, and comparing the forecast with what really sold.

The final method is the long-run level: the average month of all training months for each pair, repeated for every forecast month. In the 2024 test it had the lowest gap of every method tried, at every level of adding up (see Verification Log and Supporting tests).

Measured accuracy in the 2024 test, as the total gap divided by the actual units (199,858):

| Level of adding up | Gap |
|---|---|
| Each pair, each month | 32.2% |
| Each pair, each quarter | 18.7% |
| Each pair, full year | 11.06% |
| Each product, full year (branches added together) | 5.18% |
| All products and branches, full year (by hand: 7,158 / 199,858) | about 3.6% |

The forecast for 2025 is 205,823 units. By hand, this is 5,965 units, or about 3.0%, above the 2024 actual of 199,858. In the 2024 test the same method forecast 3.6% too high, and the 2025 forecast cannot be tested until 2025 happens.

## 1 KPI Definition

- Name: Product Demand Forecast.
- Question: how many units of each product will each branch sell in the coming months?
- Demand source: Delivered sales lines only. The stock ledger is not used as the demand source because about 10% of order lines never reached it (found in earlier KPI work).
- Unit: units as recorded in the sales lines. Products are sold as piece (24 products), set (5 products) and cartridge (1 product), so totals across products add quantities of different kinds of item.
- Scoring measure: total absolute gap divided by total actual units. Months are not divided one by one, so a quiet month does no harm to the score.

## 2 Methodology

### Sources

| Table | Used for |
|---|---|
| sales_orders_header (so_h) | order date, branch, order status |
| sales_orders_lines (so_l) | product and quantity of each line |
| products | the 30 products |

### What was reused and what is new

| Reused | New in this KPI |
|---|---|
| Delivered sales lines and the 72-month history (KPI 5 and KPI 7) | Monthly units per product-branch pair |
| Yearly trend facts (KPI 5) | Forecast methods and their scores on hidden months |
| | Forecast for 2025 |

### Steps

- Built the Delivered sales lines with order date and branch (117,656 lines, 1,234,939 units).
- Built the monthly table: 180 pairs by 72 months, 12,960 cells.
- Split the months: training 2019-01 to 2023-12 (60 months, 1,035,081 units) and test 2024-01 to 2024-12 (12 months, 199,858 units).
- Forecast the 12 test months with three simple methods and one straight-line trend method, and scored each by month, by quarter and by full year per pair.
- Scored the chosen method by product (branches added together).
- Repeated the scoring with other hidden windows (last 6 and last 24 months) and with the median instead of the average.
- Forecast 2025 from all 72 months.

### Choices and alternatives

| Method | Gap by month | Gap by quarter | Gap full year per pair | Forecast total (actual 199,858) |
|---|---|---|---|---|
| Long-run level (average of all 60 training months) | 32.2% | 18.7% | 11.06% | 207,016 (+7,158) |
| Recent level (average of last 12 training months) | 33.6% | 21.14% | 14.41% | 204,638 (+4,780) |
| Same as last year (copy the 2023 months) | 44.42% | 26.96% | 14.41% | 204,638 (+4,780) |
| Straight-line trend per pair (fitted on 60 months, clipped at 0) | 32.99% | 20.07% | 12.98% | 203,656 (+3,798) |

The long-run level has the lowest gap at every level. Same as last year has the highest gap by month and by quarter. The straight-line trend total is closer to actual than the long-run total, but its gaps are higher at every level, so by the rule set beforehand it does not replace the long-run level. Recent level and same as last year give the same full-year gap (14.41%) because both add up to each pair's 2023 units (by hand). By hand, the totals are 2.4% above actual for recent level and same as last year, and 3.6% above actual for the long-run level. The straight-line trend gave a falling line for 103 of 180 pairs.

### Product Demand Trend Analysis

Units sold per year, all products:

| Year | Units |
|---|---|
| 2019 | 207,925 |
| 2020 | 212,007 |
| 2021 | 208,269 |
| 2022 | 202,242 |
| 2023 | 204,638 |
| 2024 | 199,858 |

- The years add up to 1,234,939.
- Units in 2022-24 against 2019-21 changed by -3.42%, which agrees with the -3.4% found in KPI 5.
- 6 products rose and 24 fell. The biggest rise is P006 (+4.27%) and the biggest fall is P028 (-9.52%), so every product moved by less than 10% either way.
- By hand: the peak year is 2020, 2024 is the lowest of the six years, and 2024 is about 3.9% below 2019.

Seasonality, average share of the year by calendar month (an even share is 8.33):

| Jan | Feb | Mar | Apr | May | Jun | Jul | Aug | Sep | Oct | Nov | Dec |
|---|---|---|---|---|---|---|---|---|---|---|---|
| 8.42 | 7.56 | 8.43 | 7.99 | 8.52 | 7.94 | 8.93 | 8.92 | 8.31 | 8.44 | 8.33 | 8.21 |

- The busiest month in each year from 2019 to 2024 was Jul, Jul, Oct, May, Sep, Mar, so no month is busiest every year.
- The quietest month in each year was Feb, Feb, Jun, Feb, Feb, Apr, so February was quietest in 4 of 6 years.
- The shares range from 7.56 to 8.93 around the even share of 8.33, so the seasonal pattern is weak. By hand from the printed table, August is above 8.33 in all six years.
- No cause for the seasonal pattern was tested, and none is stated.

## 3 Verification Log

### Summary

| Check | What it looked at | Result |
|---|---|---|
| Check 1 | Blanks, zeros and negatives in the sales columns, the monthly table and all forecasts | None found |
| Check 2 | Edge cases: smallest pairs, quiet months, orders that are not Delivered, repeated product lines | Handled as described below |
| Check 3 | Duplicates: order ids, line ids, identical lines | No sign of accidental copies |
| Check 4 | Totals tie across lines, monthly table, split, years, products, branches, pairs and forecasts | All tie |
| Check 5 | A second method with plain loops | Same table and same scores |
| Check 6 | Other hidden windows, the median, and the biggest months | Long-run level stays lowest |
| Check 7 | History and dates | Complete, no gap |
| Check 8 | Three pairs rebuilt by hand from the raw files | All match |

### Check 1: blanks, zeros and negatives

- so_id, product_id, branch_id, order_date and quantity in the sales lines: 0 blanks each.
- Quantity: 0 zero values, 0 negative values, smallest 1, largest 20.
- Monthly table: 0 blanks, 0 negative cells, 5 zero cells (real quiet months, listed under Data Quality Notes).
- Recent level, same as last year, long-run level and straight-line trend forecasts: 0 blanks and 0 negative cells.
- 2025 forecast: 0 blanks and 0 negative values.

### Check 2: edge cases

- Smallest 2024 total for any pair: 733. Smallest 72-month total for any pair: 5,081.
- Pairs with at least one quiet month: 4.
- Orders not Delivered: 1,967, all Cancelled. Lines on those orders: 12,746 (by hand 130,402 - 117,656) with 133,595 units. Lines in the sales table that belong to those orders: 0. Delivered lines plus lines on other orders: 130,402, equal to all lines in so_l.
- Order and product combinations in the sales lines: 104,418, of which 12,184 have more than one line. Lines in a repeated combination: 25,422 of 117,656 (by hand, 21.6%).

### Check 3: duplicates

- Orders in the header: 20,000, with 20,000 distinct order ids.
- A line is identified by order id plus line number. Lines with a repeated order id and line number in so_l: 0.
- Delivered lines identical to another line apart from the line number: 1,374 of 117,656. Extra copies if one of each group is kept: 688.
- The same product twice in one order with exactly two lines: 11,202 combinations, of which 547 have the same quantity (4.88%) against 5.0% chance (binomial test p 0.5879). The repeated product lines inside an order are not more often equal in quantity than chance, so there is no sign of accidental copies. No difference found is not proof of no difference.
- Scope: this chance test covers combinations with exactly two lines (11,202 of 12,184 repeated combinations, 91.9% by hand). The 982 combinations with three or more lines (by hand) were not tested separately. The chance of 5.0% was taken from the overall quantity mix, which is an approximation.

### Check 4: totals reconcile

- Delivered lines 117,656 and units 1,234,939.
- Delivered orders with at least one line: 18,033, equal to Delivered orders in the header (18,033).
- Units in the monthly table: 1,234,939. Training plus test: 1,234,939. Years added up: 1,234,939.
- Products 30, branches 6 and pairs 180: the largest difference between the sales lines and the monthly table is 0 at each level.
- 2025 forecast added up by product, by branch and over all pairs: 205,823.17 (by hand 1,234,939 / 72 x 12).
- 2024 long-run forecast added up: 207,016.2 (by hand 1,035,081 / 60 x 12).

### Check 5: second method

- The monthly table was rebuilt cell by cell with plain loops, with no grouping.
- Cells compared: 12,960. Cells that differ: 0. Units in the loop totals: 1,234,939.
- Pair-months with sales: 12,955 in the loop and 12,955 in the table (by hand 12,960 - 5 zero cells).
- Long-run scores reproduced: 32.2 by month, 18.7 by quarter, 11.06 full year per pair, 5.18 full year per product. 2025 forecast total 205,823.17.

### Check 6: other hidden windows, the median, the biggest months

Gap by month / gap for the whole window per pair:

| Hidden window (actual units) | Recent level | Same as last year | Long-run level |
|---|---|---|---|
| Last 6 months (100,993) | 32.61 / 15.6 | 43.96 / 19.72 | 31.67 / 13.08 |
| Last 12 months (199,858) | 33.6 / 14.41 | 44.42 / 14.41 | 32.2 / 11.06 |
| Last 24 months (404,496) | 32.96 / 10.81 | 45.03 / 10.81 | 32.18 / 7.75 |

- The long-run level has the lowest gap in all three windows on both measures.
- For the last 24 months, same as last year repeats the same 12 months, which is a rougher test, and its whole-window gap equals the recent level by construction (by hand).
- Median against average (long-run level, 2024 test):

| Level | Forecast total | Minus actual | By month | By quarter | Full year per pair |
|---|---|---|---|---|---|
| Average of 60 months | 207,016 | +7,158 | 32.2 | 18.7 | 11.06 |
| Median of 60 months | 199,830 | -28 | 32.01 | 18.61 | 11.17 |

- By hand, the median is 0.19 and 0.09 points lower by month and by quarter and 0.11 points higher for the full year. Every difference is under 0.2 points, so there is no clear gain, and the final method stays the long-run average, as decided beforehand.
- The five biggest pair-months: P001 CHN001 2019-07 (347 units, 2.88 times the pair's average month of 120.6), P005 CHN001 2020-05 (323, 2.78 times, average 116.2), P027 CHN001 2019-07 (315, 2.63 times, average 119.8), P018 KOL001 2021-01 (305, 2.73 times, average 111.5), P025 CHN001 2020-07 (297, 2.5 times, average 118.9). All are in 2019 to 2021, and none is above 3 times its pair's average month.

### Check 7: history and dates

- The monthly table has 72 months, from 2019-01 to 2024-12, matching the full list with none missing.
- First order date 2019-01-01 and last order date 2024-12-31.
- Months with zero total units: 0, including the 12 hidden 2024 months. The smallest month total is 14,117 (2019-02).
- Every month has orders on 28 to 31 distinct days (fewest 28 in 2019-02, most 31 in 2019-01).
- The first month of the 2025 forecast is 2025-01, which is after the last order date.

### Check 8: three pairs by hand from the raw files

Rebuilt from so_h and so_l with plain loops (Delivered orders only):

| Pair | Units in 60 training months | Average month | 2024 actual | Forecast for 2024 | Gap | Gap as % of the pair's actual | Zero months | Match with tables |
|---|---|---|---|---|---|---|---|---|
| P004 HYD001 | 4,383 | 73.05 | 765 | 876.6 | 111.6 | 14.59% | 2021-01, 2024-06 | Yes |
| P019 AHM001 | 6,252 | 104.2 | 928 | 1,250.4 | 322.4 | 34.74% | None | Yes |
| P001 CHN001 | 7,429 | 123.82 | 1,252 | 1,485.8 | 233.8 | 18.67% | None | Yes |

The single-pair gap for P019 AHM001 (34.74%) is much larger than the gap for the product P019 with branches added together (15.67%).

### Supporting tests

| Test | Result |
|---|---|
| Seasonality | Weak, no month busiest every year (see Product Demand Trend Analysis) |
| Split | 60 training months and 12 test months, units tie to 1,234,939 |
| Three forecasts | Long-run level 32.2%, recent level 33.6%, same as last year 44.42% by month |
| Adding-up ladder | The gap shrinks from month to quarter to year for all three methods |
| Straight-line trend method | Higher gaps than the long-run level at every level |
| Product accuracy | 5.18% full year per product |
| Pairs offsetting | 104 pairs too high, 76 too low, pair-year gap 11.06% falls only to 10.65% after removing the overall level |
| Decline link | Products that fell more tended to be forecast too high by more, but the link is not proven |
| Hidden-window sensitivity | Long-run level lowest in the last 6, 12 and 24 months |
| Median test | Under 0.2 points different on every measure |

Adding-up ladder (gap by month / by quarter / full year per pair):

| Method | By month | By quarter | Full year per pair |
|---|---|---|---|
| Recent level | 33.6 | 21.14 | 14.41 |
| Same as last year | 44.42 | 26.96 | 14.41 |
| Long-run level | 32.2 | 18.7 | 11.06 |

Product accuracy (long-run level, 2024, branches added together):

- Actual units sum to 199,858 and the forecast to 207,016.
- Gap under 5% for 17 products, under 10% for 26 products, and 20% or more for 0 products.
- The forecast was too high for 20 products and too low for 10. The total gap across products is 5.18% of actual.
- Smallest gaps: P017 0.37% (6,841 actual, 6,816 forecast), P016 0.40%, P014 0.45%, P013 0.85%, P024 1.03%.
- Largest gaps: P019 15.67% (6,124 actual, 7,084 forecast), P005 11.99%, P012 11.63%, P010 11.08%, P028 9.20%, P025 8.99%, P015 8.75%. The four products above 10% were all forecast too high.

Pairs offsetting (long-run level, 2024, 180 pairs):

- Too high 104 pairs and too low 76 pairs.
- Units over-forecast in the too-high pairs 14,630, and units under-forecast in the too-low pairs 7,471. The net is 7,158.
- The full-year gap per pair is 11.06%. After removing the overall level (forecast rescaled to the actual total) it is 10.65%, a fall of 0.41 points (by hand). With 76 pairs forecast too low against 104 too high, the pair-year gap is mostly pairs missing in opposite directions.

Decline link (training years only, product change = 2022-23 average year against 2019-21 average year, compared with forecast minus actual percent for the 30 products, strict line 0.05):

- 23 of 30 products fell. Spearman correlation -0.303, p 0.1033 (above 0.05).
- The four products over 10%: P019 (change -2.59, error +15.67), P005 (-5.93, +11.99), P012 (-4.12, +11.63), P010 (-3.37, +11.08); all four fell.
- Products that fell more tended to be forecast too high by more, but the link is not proven (rho -0.30, p 0.10, only 30 products). The decline is not stated as the reason for the over-forecast.

## 4 Data Quality Notes

1. Cancelled orders are left out of demand: 1,967 orders, 12,746 lines and 133,595 units, which is about 9.8% of all lines (by hand).
2. Five pair-months have zero sales, in 4 pairs: P004 HYD001 2021-01, P004 HYD001 2024-06, P008 KOL001 2023-02, P018 HYD001 2020-02 and P021 HYD001 2023-08. By hand from this list, 4 of the 5 zero months and 3 of the 4 pairs are at HYD001. The latest zero month is 2024-06. No cause was tested, and none is stated.
3. Repeated product lines inside one order exist (12,184 repeated combinations, 25,422 lines). They are kept as real sales because no sign of accidental copies was found (see Check 3). The test covers combinations of exactly two lines only.
4. The stock ledger is not the demand source, because about 10% of order lines never reached it (earlier KPI work).
5. Totals add quantities across products sold as piece, set and cartridge, so a total is a count of units as recorded and not of equal-sized items.
6. The 2025 forecast cannot be tested until 2025 happens. In the 2024 test the same method was 3.6% too high, and the 2025 forecast is about 3.0% above the 2024 actual (by hand).

## 5 Business Interpretation

How close the forecast is depends on how much is added up:

| Level of use | Measured gap in the 2024 test |
|---|---|
| Per product, full year | 5.18%, with 26 of 30 products under 10% and none at 20% or more |
| Per pair, full quarter | 18.7% |
| Per pair, full year | 11.06% |
| Per pair, one month | 32.2% |

The forecast is closest per product over a year and loosest per pair in one month. The gap shrinks as months and branches are added together for all three simple methods, so a large part of the monthly miss is swings that cancel out.

Products to check before ordering: P019 (15.67% too high), P005 (11.99%), P012 (11.63%) and P010 (11.08%), all four forecast too high in 2024.

Demand in 2024 (199,858 units) was the lowest of the six years, and 23 of 30 products fell between the 2019-21 and 2022-23 averages. The link between the fall and the over-forecast is not proven.

### The 2025 forecast (average month over all 72 months per pair, times 12)

- Total 205,823 units for 180 pairs. By hand this is 5,965 units (about 3.0%) above the 2024 actual of 199,858.

By branch:

| Branch | Units |
|---|---|
| AHM001 | 35,704 |
| CHN001 | 42,320 |
| DEL001 | 30,886 |
| HYD001 | 28,162 |
| KOL001 | 38,478 |
| PUN001 | 30,272 |

The branches add to 205,822 by hand because of rounding.

By product (branches added together; smallest P024 6,581, largest P029 7,117):

| Product | Units | Product | Units | Product | Units |
|---|---|---|---|---|---|
| P001 | 6,884 | P011 | 7,016 | P021 | 6,977 |
| P002 | 6,820 | P012 | 6,925 | P022 | 6,883 |
| P003 | 6,952 | P013 | 6,654 | P023 | 6,713 |
| P004 | 6,815 | P014 | 6,762 | P024 | 6,581 |
| P005 | 6,747 | P015 | 6,802 | P025 | 6,848 |
| P006 | 6,974 | P016 | 6,962 | P026 | 6,734 |
| P007 | 7,026 | P017 | 6,820 | P027 | 6,812 |
| P008 | 6,843 | P018 | 6,792 | P028 | 6,882 |
| P009 | 6,758 | P019 | 6,924 | P029 | 7,117 |
| P010 | 7,049 | P020 | 6,819 | P030 | 6,932 |

## Recommended actions for the business owners

1. Purchasing manager: use the 2025 forecast per product (see the by product table above) as the starting point for the yearly buying plan, since the product-year gap was 5.18% in the 2024 test.
2. Branch managers: use the quarterly forecast per product and branch for branch stock planning and expect a gap of about 18.7%, as measured in the 2024 test; do not use the single-month forecast for exact order sizes, since its gap was 32.2%.
3. Purchasing manager: check the current stock position of P019, P005, P012 and P010 before ordering against their forecasts, since all four were forecast too high by 11.08% to 15.67% in 2024.
4. Inventory planner: plan 2025 with the forecast of 205,823 units and keep in mind that the same method was 3.6% too high in the 2024 test.

## Glossary

- Pair: one product in one branch (180 pairs).
- Gap: the total of the differences between forecast and actual, ignoring direction, divided by the total actual units.
- Forecast total minus actual: the forecast total with the actual total taken away; a positive number means the forecast was too high.
- Hidden months or test months: the last 12 months (2024), kept out of the forecast and used to score it.
- Training months: the first 60 months (2019-01 to 2023-12), used to build the forecast.
- Long-run level: the average of all training months for a pair, repeated for every forecast month.
- Recent level: the average of the last 12 training months for a pair.
- Same as last year: the 2023 months copied into the 2024 forecast.
- Straight-line trend: a straight line fitted to the 60 training months of a pair and projected 12 months, clipped at 0.
- Delivered sales lines: lines of orders with the status Delivered; Cancelled orders are left out.
- Zero month: a pair-month with no units sold.
- so_h, so_l, products: the sales order header, the sales order lines and the products tables.
- delivered: the Delivered orders from the header.
- sales: the Delivered lines with order date and branch.
- monthly: units per pair (rows) by month (columns), 180 by 72.
- train, test: the first 60 months and the last 12 months of monthly.
- yearly: units per year.
- forecast_recent, forecast_last_year, forecast_long, forecast_trend: the forecasts of the four methods for the 12 test months.
- pair_average: the average month of each pair over all 72 months.
- forecast_2025: the 2025 forecast per pair (average month times 12).
- by_product_2025, by_branch_2025: the 2025 forecast added up by product and by branch.
- signed_percent: forecast minus actual as a percent of actual, with direction.
- quarters: the 2024 months grouped in threes.
- actual_total: the 2024 actual units (199,858).
- window_scores: the function that scores a forecast for a chosen hidden window.
