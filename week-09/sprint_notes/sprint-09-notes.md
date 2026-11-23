# Sprint 09 Notes

Project: Heavy Supplier & Warehouse Analytics
Sprint: Week 9 (Phase 3: Supplier, Customer & Predictive Analytics)

## Sprint Goal

- Forecast how much of each product will sell in each branch (KPI 18), and use that demand with the real supplier waiting times to work out when each product should be reordered and whether any product is close to a stockout (KPI 19).

## Work Completed

### KPI 18 Product Demand Forecast

- Demand comes from Delivered sales lines only: 117,656 lines and 1,234,939 units over 72 months (2019-01 to 2024-12), for 180 product-branch pairs. The monthly table is 180 pairs x 72 months and adds up to 1,234,939. Only 5 of the 12,960 pair-months have no sales (P004 HYD001 in 2021-01 and 2024-06, P008 KOL001 in 2023-02, P018 HYD001 in 2020-02, P021 HYD001 in 2023-08).
- Yearly units were 207,925 (2019), 212,007, 208,269, 202,242, 204,638 and 199,858 (2024). Units fell 3.42% between 2019-21 and 2022-24; 6 products rose and 24 fell. The share of the year by calendar month ran from 7.56% to 8.93% (8.33% if even) and the busiest month changed from year to year.
- The last 12 months (2024) were hidden and each method was scored against what really sold. The long-run level (average of all 60 training months per pair) had the lowest gap: 32.2% by pair-month, 18.7% by pair-quarter, 11.06% by pair-year and 5.18% by product-year. The recent level scored 33.6 / 21.14 / 14.41, same as last year 44.42 / 26.96 / 14.41, and a straight-line trend 32.99 / 20.07 / 12.98. The long-run level stayed the method.
- The 2024 forecast total was 207,016 units against 199,858 sold (7,158 too high). By product, 17 products were within 5%, 26 within 10% and none was 20% or more off. P019, P005, P012 and P010 were more than 10% off, all forecast too high. 104 pairs were forecast too high and 76 too low, so the pair-level gap is mostly pairs missing in opposite directions; removing the overall level only lowered the pair-year gap from 11.06% to 10.65%. The link between falling products and larger over-forecasts was not proven (Spearman -0.30, p = 0.10).
- The 2025 forecast is 205,823 units (about 3.0% above 2024). By branch: CHN001 42,320, KOL001 38,478, AHM001 35,704, DEL001 30,886, PUN001 30,272 and HYD001 28,162.
- All 8 verification checks were run: no blanks, zeros or negatives; every order ID used once and identical lines no more common than chance (4.88% against 5.0%, p = 0.5879); every total reconciled; a second method with plain loops matched all 12,960 cells; the long-run level had the lowest gap when the last 6, 12 and 24 months were hidden; the 72-month history is complete; three pairs rebuilt by hand from the raw files matched.
- The document is week-09/docs/KPI 18 Product Demand Forecast.md and the notebook is "Week9 KPI 18 Product Demand Forecast.ipynb" in week-09/notebooks.

### KPI 19 Stockout Risk and Reorder Point

- Every one of the 30 products and every one of the 180 pairs is supplied by all 8 suppliers, so each product uses its own real waiting time from its Received purchase lines (140,077 lines). Waits run from 7 to 35 days with an average of 17.9, and product averages run from 17.63 to 18.15 days.
- A pair sells on average 3.130 units a day and nothing at all on 76.7% of days, so sales come in lumps (the daily swing is 7.088 units).
- The reorder point (demand during the wait plus safety stock for a 95% service level) runs from 89.9 to 144.5 units per pair, average 116.2. Stock rebuilt as in KPI 8 runs from 98,642 to 132,967 units per pair (21,185,215 in total).
- No pair is at or below its reorder point (0 of 180). The closest pair, P019 at KOL001, holds 745.7 times its reorder point and would take 74.8 years of average sales with no new deliveries to reach it. By branch, KOL001 (74.8 years) and CHN001 (75.8) are closest and HYD001 is furthest (shortest 109.2, median 124.8 years).
- The answer held at 90%, 95% and 99% service levels (smallest stock 840.9, 745.7 and 615.1 times the reorder point), with every wait at 35 days (485.1 times), and against the most any pair really sold in any 35 days (383 units; every pair's stock is at least 285.5 times its own largest 35-day demand).
- The stored reorder levels in the inventory table are below the calculated reorder points in 175 of 180 pairs. The stored lead times in the products table run from 4 to 40 days and differ from the real average wait by more than 1 day in 27 of 30 products and by more than 5 days in 19.
- All 8 verification checks were run: no blanks, zeros or negatives; every order ID used once and no sign of accidental copies on either the sales or the purchase side; every total reconciled and tied to KPI 8; a second method with plain loops matched all 180 pairs; outliers and sensitivity were tested; history and dates were checked; three pairs rebuilt by hand from the raw files matched.
- The document is week-09/docs/KPI 19 Stockout Risk and Reorder Point.md and the notebook is "Week9 KPI 19 Stockout Risk and Reorder Point.ipynb" in week-09/notebooks.

## Blockers & Resolutions

- Percent error breaks on the 5 pair-months with no sales. The forecast was scored by total gap over total units sold instead, so quiet months do no harm.
- The same product appears on more than one line of the same order (12,184 combinations). Equal quantities were tested against chance (547 of 11,202 two-line combinations, 4.88% against 5.0%) and no sign of accidental copies was found.
- Purchase lines have no supplier column, and every product comes from all 8 suppliers. The waiting time therefore comes from each product's own Received lines, not from one supplier.
- The stored reorder levels, safety stocks and lead times in the tables do not match the real data. They were not used as inputs; the reorder points were calculated from the real demand and waits and compared with the stored values.
- The stock ledger is 1,879,221 units below the order-line stock (the KPI 8 finding). The order-line stock was used, as in KPI 8.

## Next Sprint Focus

- Week 10 (Phase 3 close, risk and control): KPI 20 Inventory Movement Anomaly Detection, KPI 21 Stock Shrinkage Rate and KPI 22 Operational Risk Score.
