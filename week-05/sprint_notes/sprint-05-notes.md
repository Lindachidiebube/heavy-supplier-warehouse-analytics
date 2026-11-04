# Sprint 05 Notes

Project: Heavy Supplier & Warehouse Analytics
Sprint: Week 5 (Phase 2: Inventory Health and Stock Analysis)

## Sprint Goal
- Find out which products bring in the money (KPI 7, ABC/Pareto Classification) and how fast the warehouse sells the stock it holds (KPI 9, Inventory Turnover Rate), each with 8 verification checks.
- Decide what to do with the Inventory Aging task of the Task Catalogue.

## Work Completed

### KPI 7: ABC/Pareto Classification
- Used the Delivered sales lines only: 18,033 orders, 117,656 lines, 1,234,939 units and 30 products. Revenue is counted before GST (22,958,987,430), because GST is tax and not sales money.
- Ranked the 30 products by revenue and classed them by the running share of revenue before each product: A (under 80%), B (under 95%), C (the rest). Result: 10 A products with 81.53% of revenue, 7 B products with 14.07% and 13 C products with 4.40%.
- Main finding: every product sells about the same number of units (39,486 to 42,702), but list prices run from 310 to 119,000 and every line was sold at its list price. So the revenue rank follows the price tier: the rank correlation with list price is 0.999 and with units 0.171.
- Tested the classes against other choices: the other boundary rule moves only 2 products (P022 and P005); including cancelled orders, including GST, and removing the largest lines change 0 classes; classing by units gives no concentration (24 of 30 products are needed to reach 80%).
- Tested stability: classes by year equal the overall class in 178 of 180 product-years (only P022 and P009 swapped, in 2020) and by branch in 178 of 180 product-branch classes (only P005 and P025 swapped, in Kolkata). The pieces sold explain both swaps.
- Revenue fell 2.76% between 2019-2021 and 2022-2024 while units fell 3.42%, because the units lost were worth less than the average unit.
- All 8 verification checks done. Check 8 rebuilt the figures from the raw CSV files line by line (0 differences) and by hand for 3 products (P015, P005, P026).

### KPI 9: Inventory Turnover Rate
- Measured turnover (units sold divided by average stock) for each of the 180 product-branch pairs for 2019 to 2024, with branch, product and overall views, and years of stock (1 divided by turnover). Sales come from Delivered lines dated by delivery date; stock is the month-end order-line stock rebuilt in KPI 8.
- Main finding: turnover fell about 11 times, from 0.114 in 2019 to 0.0102 in 2024. Stock grew from 32,969 units to 21,185,215 units while sales stayed between 200,562 and 211,190 units a year, and purchases brought in about 17 to 18 units for every unit sold. At the 2024 speed the average stock would last about 98 years.
- All 180 pairs and all 30 products are slower in 2024 than in 2019. The branch gap comes from sales (1.47 times between Hyderabad and Chennai) and not from stock (1.05 times). Hyderabad is the slowest branch, and Chennai and Kolkata are the fastest.
- Tested the result against other choices: sales dated by order date, cancelled orders counted, ledger-based stock, a different averaging method and a different cost all give the same fall. The cost choice changes the result only in the 5th decimal.
- The stock table equals an independent rebuild within 0.031%, and the gap is exactly the 2,454 stock corrections in all 73 months.
- All 8 verification checks done. Check 8 rebuilt 3 pairs from raw rows, and all 36 month-end values equal the table.
- Decided that Inventory Aging is Not selected, because the data has no lot or batch IDs and the age of stock would rest on a first-in-first-out assumption that cannot be proven. Years of stock already shows how long stock sits.

## Blockers & Resolutions
- Cost: the products-table cost differs from the purchase order cost on 155,492 of 155,495 lines. Resolved by using the weighted average cost paid as the headline and testing the products-table cost as a second view.
- Two stock figures: the order lines and the ledger differ by 1,879,221 units (about 9%). Resolved by using the order-line stock as the headline and showing the ledger-based stock beside it. The fall is the same in both.
- Untested remarks that came up during the work (why 2019 differs, why some pairs rose, why 4 pairs hold less stock, why the effect of cancelled orders changes sign, why January 2019 is low) were each tested in the check where they came up, and the results are in the documents.
- In KPI 7 the first version of Check 8 compared only totals. It was replaced by a line-by-line rebuild from the raw files and a hand check of 3 products.

## Next Sprint Focus
- Week 6: KPI 10 Capital Tied Up in Excess Stock, KPI 11 Warehouse Throughput and KPI 12 Warehouse Space Utilisation (KPI 12 may move to Week 7).
- Decide whether understock becomes a section of KPI 10, and whether Product-Warehouse Alignment is a section or Not selected.
- Before starting, check that products.csv has cost and price and that branches.csv has warehouse capacity.
