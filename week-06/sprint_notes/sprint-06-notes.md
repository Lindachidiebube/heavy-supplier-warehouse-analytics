# Sprint 06 Notes

Project: Heavy Supplier & Warehouse Analytics
Sprint: Week 6 (Phase 2: Inventory Health and Stock Analysis)

## Sprint Goal

- Find out how much of the stock the business holds is stock it does not need (KPI 10), how much each branch warehouse moves (KPI 11) and how crowded each warehouse is (KPI 12).
- Build all three KPIs on the same verified stock figure: order-line stock of 21,185,215 units at the end of the data (28 January 2025), so the three results can be read together.

## Work Completed

### KPI 10: Capital Tied Up in Excess Stock

- Defined needed stock as 3 months of sales per product-branch pair, measured on the 12 months from 15 January 2024 to 13 January 2025 (197,300 units sold). Money is counted at the cost actually paid per unit on received purchases.
- Result: 21,135,890 of the 21,185,215 units are excess (99.77%). The money tied up in excess stock is 294,576,746,335, which is 99.77% of the stock value and 94.41% of the 312,027,429,108 spent on received purchases.
- Five other rules for needed stock (1, 6 and 12 months of sales, and the two maximum levels held in the data) give 99.07% to 99.92% excess. The finding does not depend on the rule.
- Branch shares of the excess run from 15.94% (KOL001) to 16.95% (CHN001). Two products, P015 Engine Block and P014 Transmission Assembly, make 36.38% of the excess value.
- The excess value grows from 46.5 billion at the end of 2019 to 291.8 billion at the end of 2024, about 49 billion a year.
- Stock rebuilt from the stock ledger gives an excess value of 268,214,199,098 (91.05% of the headline). The finding holds on either record.
- Understock was tested month by month (no breach) and day by day: 14 of 180 pairs dipped below a reorder or safety level, all between 2 and 25 January 2019, and one pair (P027 CHN001) reached exactly 0 on one day. No pair dipped from February 2019 onward.
- All 8 verification checks were run with their extra tests, and the hand check of 3 pairs from the raw tables matched the table exactly.

### KPI 11: Warehouse Throughput

- Reported units IN (received purchases) and units OUT (delivered sales) separately for each branch, as monthly averages over the 72 full months from January 2019 to December 2024.
- Units IN are 22,387,123 and units OUT are 1,234,939. IN is 94.77% of all units moved, so a combined total would hide the differences between branches.
- Average monthly units OUT run from 2,339 (HYD001) to 3,522 (CHN001), about 1.5 times apart. Average monthly units IN are almost equal everywhere (50,355 to 52,296). Units received per 1 unit delivered run from 14.8 (CHN001) to 21.9 (HYD001).
- Per employee, KOL001 handles the most (65.3 units OUT and 1,027.7 units IN a month) and DEL001 the least (27.9 and 565.5). Staff numbers are one current value per branch.
- Months swing about 3 times between the quietest and the busiest. The swings come from the number of lines, not from a few huge lines. Yearly swings in units IN come mostly from how many orders arrive (correlation 0.91) and partly from order size (0.67).
- January 2019 is a start-of-data month. It lowers the averages by under 1% and changes no ranking.
- All 8 verification checks were run with their extra tests, and 3 branch-months rebuilt by hand from the raw tables matched exactly.

### KPI 12: Warehouse Space Utilisation

- Headline: units of stock per square foot of warehouse. At the end of the data, KOL001 holds 115.0, AHM001 101.8, HYD001 98.2, PUN001 92.3, CHN001 84.7 and DEL001 79.5. The most crowded branch holds 1.45 times more per square foot than the least crowded.
- The ranking is the same at every year end from 2019 to 2024. The gap comes from the size of the warehouse: stock differs only 1.047 times between branches while warehouse size differs 1.51 times (29,880 to 45,230 square feet).
- Every branch became about 6 times denser between 2019 and 2024 (KOL001 18.8 to 113.8, DEL001 13.1 to 78.8), because units received are about 18 times units delivered every year (17.4 to 18.3) while the warehouse size is one current value.
- Supporting view from product sizes: total stock volume is 1,429,369.3 cubic metres. If it all stood in one layer on the floor, the average pile would be 56.6 m (DEL001) to 82.0 m (KOL001). Two products make 67.2% of the volume. Without them, or without the top 5, the order and the gap stay the same. This view is a comparison only.
- All 8 verification checks were run with their extra tests, including a full test of the 120-row difference between the stock ledger and the order lines in 2025: 135 counted lines have no ledger row of their own and 15 ledger rows belong to cancelled orders. Three items rebuilt by hand from the raw tables matched exactly.
- Product-Warehouse Alignment was considered and recorded as Not selected.

## Blockers & Resolutions

- Two different maximum stock levels exist (inventory master and products table) and the data does not say which one is official. Resolution: the headline uses a rule built from real sales, and both levels are shown as sensitivity tests.
- The first daily understock run gave impossible results (negative stock). Cause: date columns held different types, so movements were added out of date order. Resolution: all date columns were converted, the run was repeated and the first result was not used.
- Claims about January 2019 (dips in stock, low receipts) were first stated without a test. Resolution: each was tested at its check, with the receipt dates and lead times shown.
- In KPI 11 purchases show 0 exact copies of repeated lines. Resolution: every repeated purchase line carries a different unit cost, so none can be a copy; sales copies match chance.
- Stock ledger adjustment quantities carry no sign. Resolution: the direction comes from the change in running balance, tested on all 2,454 rows (1,227 up, 1,227 down, net 62).
- Warehouse size and staff numbers are one current value per branch, and the size of sets and cartridges cannot be confirmed from the data. Resolution: both are stated as limits and the volume view is used for comparison only.
- Notebooks are cleaned and run from the first cell to the last before they are saved to GitHub.

## Next Sprint Focus

- KPI 13 Supplier Lead Time and Reliability (Week 7): confirm that the purchase orders header holds the expected and the actual delivery dates, count Received orders only, and measure which suppliers deliver late.
- Decide whether Warehouse Utilisation Benchmarking becomes a section inside an existing KPI or is Not selected.
