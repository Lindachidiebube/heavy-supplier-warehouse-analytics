# Heavy Supplier, Inventory & Warehouse Analytics

My CadetX Virtual Internship project, completed independently as a Data Analyst. This repository holds my weekly work for a 12-week analytics project on warehouse, inventory and supplier data.

## About me

Hi, I'm **Chidiebube Linda**, a Data Analyst. I am carrying out this project on my own, from the first data check through KPIs, forecasting and dashboards to the final recommendations. The work so far uses Python, pandas, Google Colab, Google Drive and GitHub, with dbdiagram.io and Mermaid for the table diagrams.

Connect with me on LinkedIn: [Chidiebube Linda](https://www.linkedin.com/in/chidiebube-linda-24185b224)

## The business problem

Warehouses and supply-chain teams often operate reactively, relying on manual reports or intuition to manage inventory, storage and space. This leads to excess inventory, frequent stockouts, poor space usage and delayed decisions.

**Goal:** build a unified analytics framework that uses historical warehouse and supplier data to analyse, predict and optimise product movement, inventory levels and warehouse performance.

## How the work is done

- KPIs and analysis questions are proposed first; tasks are then chosen to answer them.
- Every KPI goes through an 8-check verification (nulls, cross-method confirmation, row-gap explanation, totals reconciliation, re-derivation three ways, distribution and bias, date coverage, manual spot-check).
- Every KPI write-up has the same sections: Definition, Methodology, Verification Log, Data Quality Notes, Business Interpretation.
- Each KPI is uploaded as soon as it is finished, and each week ends with sprint notes.

## Repository structure

- `data/raw/` copies of the raw dataset files
- `week-XX/docs/` KPI write-ups and documentation
- `week-XX/notebooks/` verification notebooks (Google Colab, Python and pandas)
- `week-XX/sprint_notes/` weekly sprint notes

## Progress by week

| Week | Focus | Status |
|---|---|---|
| 1 | Dataset profiling, integrity checks, ERD, current stock investigation | Done |
| 2 | First four KPIs, built and fully verified | Done |
| 3 | KPI proposal, key-uniqueness checks on all ID columns, data cleaning log, table join map, KPI 5, feature list, data dictionary | Done |
| 4 | Stock balance reconstruction and dead stock identification (KPIs 8 and 6) | Done |
| 5 | ABC/Pareto classification and inventory turnover (KPIs 7 and 9) | Done |
| 6 | Excess stock, warehouse throughput and warehouse space (KPIs 10, 11 and 12) | Done |
| 7 | Supplier lead time and reliability (KPI 13) | Done |
| 8 | Customer analytics: RFM segmentation, historical customer value, customer payment risk and churn by payment behavior (KPIs 14, 15, 16 and 17) | Done |
| 9 | Forecasting and reorder points: product demand forecast and stockout risk (KPIs 18 and 19) | Done |
| 10 to 12 | Risk and anomaly detection; dashboards; strategy and final presentation | Planned |

## Week 1: Data exploration

No KPIs were built in Week 1. The week was spent checking whether the data can be trusted.

- **Profiling:** all 12 tables were profiled. There were no nulls or duplicate rows, except the purchase order header table, where the received date is empty for 2,370 rows (all of them Cancelled orders).
- **Referential integrity:** 5 checks on how the tables link together all came back clean.
- **Table relationships:** an entity relationship diagram (ERD) was built to show how the tables connect: [view the ERD](https://dbdiagram.io/d/6aa7107436f9982564809e24).
- **Current stock problem:** the recorded current stock in the inventory table is inflated by roughly 100 to 850 times. It matches the running balance in the stock ledger exactly, because stock delivered per movement (about 160 units on average) is far larger than stock sold (about 10 units) and this compounds from 2019 to 2025. This is why the stock ledger is used instead of the current stock field.
- **Duplicate IDs found:** 196 invoice IDs and 201 payment IDs are shared by unrelated records. This led to the Week 2 data integrity KPI.

## Week 2 KPIs

| # | KPI | Headline result | Write-up |
|---|---|---|---|
| 1 | Stock Accumulation Ratio | Only about 5 to 6% of the stock delivered has been sold, so stock arrives far faster than it sells | [KPI 01](https://github.com/Lindachidiebube/heavy-supplier-warehouse-analytics/blob/main/week-02/docs/KPI%2001%20stock%20accumulation%20ratio.md) |
| 2 | Data Integrity Rate | Duplicate IDs affect 2.18% of invoice rows and 2.09% of payment rows | [KPI 02](https://github.com/Lindachidiebube/heavy-supplier-warehouse-analytics/blob/main/week-02/docs/KPI%2002%20data%20integrity%20rate.md) |
| 3 | Unpaid Invoice Rate | 10.19% of invoices are Unpaid and 20.30% are Partially Paid, so 30.49% are not fully collected | [KPI 03](https://github.com/Lindachidiebube/heavy-supplier-warehouse-analytics/blob/main/week-02/docs/KPI%2003%20unpaid%20invoice%20rate.md) |
| 4 | Late Payment Rate | 41.50% of payments arrive after the due date, typically about 17 days late (median); Chennai is highest at 48.61% | [KPI 04](https://github.com/Lindachidiebube/heavy-supplier-warehouse-analytics/blob/main/week-02/docs/KPI%2004%20late%20payment%20rate.md) |

Read together, the Week 2 results point to a compounding risk: capital is tied up in stock that sells slowly, while a large share of billed revenue is slow to arrive or not fully collected.

## Week 3: Phase 1 close-out and KPI 5

Week 3 closed Phase 1 (data foundation and preparation). Every ID column of the 12 tables was tested, every cleaning decision was recorded, the table joins were mapped, every column and every created field was documented, and KPI 5 was built and verified.

| Document | What it contains |
|---|---|
| [KPI-proposal.md](https://github.com/Lindachidiebube/heavy-supplier-warehouse-analytics/blob/main/week-03/docs/KPI-proposal.md) | Roadmap of 22 KPIs with business questions, updated with the Week 3 findings |
| [data-cleaning-log.md](https://github.com/Lindachidiebube/heavy-supplier-warehouse-analytics/blob/main/week-03/docs/data-cleaning-log.md) | Every data-quality finding and the decision taken |
| [table-join-map.md](https://github.com/Lindachidiebube/heavy-supplier-warehouse-analytics/blob/main/week-03/docs/table-join-map.md) | How the 12 tables join, with a diagram, orphan checks and a comparison with the Week 1 ERD |
| [KPI N05 Product Sales Velocity.md](https://github.com/Lindachidiebube/heavy-supplier-warehouse-analytics/blob/main/week-03/docs/KPI%20N05%20Product%20Sales%20Velocity.md) | KPI 5 write-up (Definition, Methodology, Verification Log, Data Quality Notes, Business Interpretation) |
| [feature-list.md](https://github.com/Lindachidiebube/heavy-supplier-warehouse-analytics/blob/main/week-03/docs/feature-list.md) | Every field created in the KPI notebooks, with its meaning and calculation |
| [data-dictionary.md](https://github.com/Lindachidiebube/heavy-supplier-warehouse-analytics/blob/main/week-03/docs/data-dictionary.md) | All 134 columns of the 12 tables: type, empty count, distinct count, range, examples and meaning (marked as inferred from the data) |

Sprint notes: [sprint-03-notes.md](https://github.com/Lindachidiebube/heavy-supplier-warehouse-analytics/blob/main/week-03/sprint_notes/sprint-03-notes.md)

Notebooks:
- [Week3_ID_Uniqueness_Check.ipynb](https://github.com/Lindachidiebube/heavy-supplier-warehouse-analytics/blob/main/week-03/Notebooks/Week3_ID_Uniqueness_Check.ipynb)
- [Week3_KPI_N05_Product_Sales_Velocity.ipynb](https://github.com/Lindachidiebube/heavy-supplier-warehouse-analytics/blob/main/week-03/Notebooks/Week3_KPI_N05_Product_Sales_Velocity.ipynb)
- [Week3 Data Dictionary Profile.ipynb](https://github.com/Lindachidiebube/heavy-supplier-warehouse-analytics/blob/main/week-03/Notebooks/Week3%20Data%20Dictionary%20Profile.ipynb)

**Key-uniqueness and link checks**

- Every real primary key is unique with no empty values. The only duplicate keys are invoice IDs (393 rows in repeated groups) and payment IDs (403 rows in repeated groups).
- 18 cross-table links were tested and none has a real orphan. The 2,454 stock ledger rows (1.03%) that match no order are exactly the ADJUSTMENT rows, which is by design.
- Every Delivered order has an invoice, and no invoice belongs to a non-Delivered order.

**Week 3 KPI**

| # | KPI | Headline result | Write-up |
|---|---|---|---|
| 5 | Product Sales Velocity | All 30 products sell at a similar pace (548 to 593 units a month), but speed depends on the branch, not the product. About 18 units are bought for every 1 sold, and buying stays almost the same in every branch, so the lowest-selling branches (Hyderabad, Delhi, Pune) carry the most over-buying. Sales fell 3.4% between 2019-21 and 2022-24. | [KPI 05](https://github.com/Lindachidiebube/heavy-supplier-warehouse-analytics/blob/main/week-03/docs/KPI%20N05%20Product%20Sales%20Velocity.md) |

The Week 3 results add a fourth strand to the earlier picture: stock arrives far faster than it sells (KPI 1), and buying does not follow selling, either by product or by branch (KPI 5).

## Week 4: Stock balance and dead stock

Week 4 started Phase 2 (inventory health). KPI 8 rebuilt how much stock each product really holds in each branch, and KPI 6 used that stock to test for dead stock.

| # | KPI | Headline result | Write-up |
|---|---|---|---|
| 8 | Reconstructed Stock Balance | Rebuilding stock from the order lines gives 21,185,215 units across the 180 product-branch pairs. That is 1,879,221 units (about 9.7%) more than the stock ledger shows, and the gap is fully explained by order lines missing from the ledger and ledger rows from cancelled orders. Every pair holds at least 177 times its maximum stock level. | [KPI 08](https://github.com/Lindachidiebube/heavy-supplier-warehouse-analytics/blob/main/week-04/docs/KPI%2008%20Reconstructed%20Stock%20Balance.md) |
| 6 | Dead Stock Identification | No pair passes the classic dead stock tests: every pair sold within 27 days of the end of the sales data, and the slowest pair still sold 703 units in 12 months. Ranked by years of cover, every pair holds 72 to 172 years of stock at its current selling rate. Hyderabad has the highest cover because it sells the least, not because it holds more stock. | [KPI 06](https://github.com/Lindachidiebube/heavy-supplier-warehouse-analytics/blob/main/week-04/docs/KPI%2006%20Dead%20Stock%20Identification.md) |

Sprint notes: [sprint-04-notes.md](https://github.com/Lindachidiebube/heavy-supplier-warehouse-analytics/blob/main/week-04/sprint_notes/sprint-04-notes.md)

Notebooks:
- [Week4 KPI NO8 Reconstructed Stock Balance.ipynb](https://github.com/Lindachidiebube/heavy-supplier-warehouse-analytics/blob/main/week-04/notebooks/Week4%20KPI%20NO8%20Reconstructed%20Stock%20Balance.ipynb)
- [Week4 KPI NO6 Dead Stock Identification.ipynb](https://github.com/Lindachidiebube/heavy-supplier-warehouse-analytics/blob/main/week-04/notebooks/Week4%20KPI%20NO6%20Dead%20Stock%20Identification.ipynb)

The Week 4 results add a fifth strand: the stock on record is not the real stock level, and the real level is far above what each branch should hold (KPI 8). That excess is spread across every pair, so there is no small group of dead products to clear out (KPI 6).

## Week 5: Revenue classes and inventory turnover

Week 5 continued Phase 2 (inventory health). KPI 7 ranked the products by the revenue they bring in, and KPI 9 measured how fast the stock turns over.

| # | KPI | Headline result | Write-up |
|---|---|---|---|
| 7 | ABC/Pareto Classification | 10 of the 30 products (class A) bring 81.53% of revenue (22,958,987,430 before GST), 7 (class B) bring 14.07% and 13 (class C) bring 4.40%. Every product sells about the same number of units (39,486 to 42,702), so revenue follows the price tier: list prices run from 310 to 119,000. The classes are almost the same in every year and every branch (only two swaps each) and do not change if cancelled orders or GST are counted or the largest lines are removed. | [KPI 07](https://github.com/Lindachidiebube/heavy-supplier-warehouse-analytics/blob/main/week-05/docs/KPI%2007%20ABC%20Pareto%20Classification.md) |
| 9 | Inventory Turnover Rate | Turnover fell about 11 times, from 0.114 in 2019 to 0.0102 in 2024, because stock grew from 32,969 to 21,185,215 units while sales stayed at about 200,000 units a year. At the 2024 speed the average stock would last about 98 years. All 180 product-branch pairs and all 30 products are slower in 2024 than in 2019, and the branch gap comes from sales (Hyderabad slowest, Chennai and Kolkata fastest), not from stock. | [KPI 09](https://github.com/Lindachidiebube/heavy-supplier-warehouse-analytics/blob/main/week-05/docs/KPI%2009%20Inventory%20Turnover%20Rate.md) |

Sprint notes: [sprint-05-notes.md](https://github.com/Lindachidiebube/heavy-supplier-warehouse-analytics/blob/main/week-05/sprint_notes/sprint-05-notes.md)

Notebooks:
- [Week5 KPI N07 ABC Pareto Classification.ipynb](https://github.com/Lindachidiebube/heavy-supplier-warehouse-analytics/blob/main/week-05/notebooks/Week5%20KPI%20N07%20ABC%20Pareto%20Classification.ipynb)
- [Week5 KPI N09 Inventory Turnover Rate.ipynb](https://github.com/Lindachidiebube/heavy-supplier-warehouse-analytics/blob/main/week-05/notebooks/Week5%20KPI%20N09%20Inventory%20Turnover%20Rate.ipynb)

The Week 5 results add a sixth strand: revenue is concentrated in 10 high-price products (KPI 7), and stock turns over more slowly every year because it keeps arriving far faster than it sells (KPI 9).

## Week 6: Excess stock, warehouse throughput and warehouse space

Week 6 continued Phase 2 (inventory health and warehouse analytics). KPI 10 measured how much of the stock is not needed, KPI 11 measured how much each branch warehouse moves, and KPI 12 compared the stock with the space available.

| # | KPI | Headline result | Write-up |
|---|---|---|---|
| 10 | Capital Tied Up in Excess Stock | 21,135,890 of the 21,185,215 units in stock (99.77%) are more than 3 months of sales. At the cost paid per unit, 294,576,746,335 is tied up in excess stock, which is 94.41% of everything spent on received purchases. Five other rules for needed stock give 99.07% to 99.92% excess, so the finding does not depend on the rule. Two products (Engine Block and Transmission Assembly) make 36.38% of the excess value, and the excess grows by about 49 billion a year, from 46.5 billion (end of 2019) to 291.8 billion (end of 2024). | [KPI 10](https://github.com/Lindachidiebube/heavy-supplier-warehouse-analytics/blob/main/week-06/docs/KPI%2010%20Capital%20Tied%20Up%20in%20Excess%20Stock.md) |
| 11 | Warehouse Throughput | Units received are 94.77% of all units moved, so received and delivered units are reported separately. Average monthly units delivered run from 2,339 (Hyderabad) to 3,522 (Chennai), about 1.5 times apart, while average monthly units received are almost equal in every branch (50,355 to 52,296). Per employee, Kolkata handles the most and Delhi the least (65.3 against 27.9 units delivered a month). | [KPI 11](https://github.com/Lindachidiebube/heavy-supplier-warehouse-analytics/blob/main/week-06/docs/KPI%2011%20Warehouse%20Throughput.md) |
| 12 | Warehouse Space Utilisation | Kolkata holds 115.0 units per square foot of warehouse and Delhi 79.5, so the most crowded branch holds 1.45 times more. The order is the same at every year end from 2019 to 2024. The gap comes from warehouse size (1.51 times apart), not from stock (1.047 times apart). Every branch is about 6 times denser than in 2019 because stock grows 3.4 to 3.6 million units a year. A volume view of the stock gives the same order. | [KPI 12](https://github.com/Lindachidiebube/heavy-supplier-warehouse-analytics/blob/main/week-06/docs/KPI%2012%20Warehouse%20Space%20Utilisation.md) |

Sprint notes: [sprint-06-notes.md](https://github.com/Lindachidiebube/heavy-supplier-warehouse-analytics/blob/main/week-06/sprint_notes/sprint-06-notes.md)

Notebooks:
- [Week6 KPI 10 Capital Tied Up in Excess Stock.ipynb](https://github.com/Lindachidiebube/heavy-supplier-warehouse-analytics/blob/main/week-06/notebooks/Week6%20KPI%2010%20Capital%20Tied%20Up%20in%20Excess%20Stock.ipynb)
- [Week6 KPI 11 Warehouse Throughput.ipynb](https://github.com/Lindachidiebube/heavy-supplier-warehouse-analytics/blob/main/week-06/notebooks/Week6%20KPI%2011%20Warehouse%20Throughput.ipynb)
- [Week6 KPI 12 Warehouse Space Utilisation.ipynb](https://github.com/Lindachidiebube/heavy-supplier-warehouse-analytics/blob/main/week-06/notebooks/Week6%20KPI%2012%20Warehouse%20Space%20Utilisation.ipynb)

The Week 6 results add a seventh strand: almost all of the stock is more than the business needs (KPI 10), the branches receive almost equal amounts but deliver different amounts (KPI 11), and the same stock sits in warehouses of different sizes, so the smallest warehouses are the most crowded (KPI 12).

## Week 7: Supplier lead time and reliability

Week 7 started Phase 3 (supplier, customer and predictive analytics). KPI 13 measured how long each of the 8 suppliers really takes to deliver and how often it misses the promised delivery date.

| # | KPI | Headline result | Write-up |
|---|---|---|---|
| 13 | Supplier Lead Time and Reliability | Of 21,630 Received purchase orders, 3,267 (15.10%) arrived after the promised date, 17,392 (80.41%) on the promised day and 971 (4.49%) early; a late order is 1 to 7 days late. All 8 suppliers are late about equally often (14.67% to 15.80%) and a test shows the differences are within chance (p-value 0.9702), so no supplier is really less reliable. The suppliers differ in how long they take: from 10.50 days (Qingdao) to 28.49 days (Tianjin). The promised days in the suppliers table are supported by the orders, but the stored reliability score is not: suppliers scored 5, 4 and 3 were late on 15.15%, 15.39% and 14.87% of orders. | [KPI 13](https://github.com/Lindachidiebube/heavy-supplier-warehouse-analytics/blob/main/week-07/docs/KPI%2013%20Supplier%20Lead%20Time%20and%20Reliability.md) |

Sprint notes: [sprint-07-notes.md](https://github.com/Lindachidiebube/heavy-supplier-warehouse-analytics/blob/main/week-07/sprint_notes/sprint-07-notes.md)

Notebooks:
- [Week7 KPI 13 Supplier Lead Time and Reliability.ipynb](https://github.com/Lindachidiebube/heavy-supplier-warehouse-analytics/blob/main/week-07/notebooks/Week7%20KPI%2013%20Supplier%20Lead%20Time%20and%20Reliability.ipynb)

The Week 7 results add an eighth strand: the suppliers keep their promised delivery date equally well, about 85% of the time, and differ only in how long they take, so the real difference between suppliers is the length of the wait and not broken promises (KPI 13).

## Week 8: Customer analytics

Week 8 continued Phase 3 (supplier, customer and predictive analytics). KPI 14 sorted the customers into segments, KPI 15 measured how much money each customer has brought in, KPI 16 measured the money customers have not paid, and KPI 17 tested whether the customers who go quiet are the ones who pay badly.

| # | KPI | Headline result | Write-up |
|---|---|---|---|
| 14 | RFM Segmentation | The 500 customers were scored on recency, frequency and money and sorted into six segments. 70 customers are At risk (valuable but quiet) and hold 16.8% of revenue. 67 customers are quiet for more than twice their own normal gap (13.13% of revenue), and a firm group of 21 stays At risk under every rule tested (5.07% of revenue). The Delhi branch is the one tested difference: 60.6% of its customers are high value against 41.4% overall (p-value 0.0015). Segments are a point-in-time view: 155 of 500 customers (31.0%) kept the same segment a year later. | [KPI 14](https://github.com/Lindachidiebube/heavy-supplier-warehouse-analytics/blob/main/week-08/docs/KPI%2014%20RFM%20Segmentation.md) |
| 15 | Historical Customer Value | Customer money (before GST, 22,958,987,430 in all) runs from 17,721,530 to 84,966,970, with an average of 45,917,975. Money is spread out: the top 10 customers hold 3.23%, the top 50 hold 14.33% and the top 100 hold 26.63%, against an even spread of 2%, 10% and 20%. Money per year is flat between 3.68 and 3.92 billion, and 2024 is the lowest year. | [KPI 15](https://github.com/Lindachidiebube/heavy-supplier-warehouse-analytics/blob/main/week-08/docs/KPI%2015%20Historical%20Customer%20Value.md) |
| 16 | Customer Payment Risk | Billed 28,599,637,860.60, paid 22,795,333,903.87 and outstanding 5,804,303,956.73, about 20.3% of billed money. The late share depends on the due period (78.57% late for 15-day invoices, 7.16% for 60-day invoices), customers pay at the same pace whatever the due period (median 34 to 36 days), and no customer or customer group stands out beyond luck. The 100 most valuable customers hold 26.03% of the outstanding money against 26.74% of the billing, so they are not riskier. 99.34% of the unpaid money is older than 90 days. | [KPI 16](https://github.com/Lindachidiebube/heavy-supplier-warehouse-analytics/blob/main/week-08/docs/KPI%2016%20Customer%20Payment%20Risk.md) |
| 17 | Churn by Payment Behavior | 67 of 500 customers (13.4%) are quiet (silent for more than twice their own normal gap), and all 23 customers whose last order is on or before 30 Jun 2024 are among them. No link was found between payment behaviour and going quiet: quiet customers paid 45.53% of invoices late against 44.90% for the others (p-value 0.9290), and bad payers are not more often quiet. 25 customers have been silent for more than 180 days, the longest for 359 days. | [KPI 17](https://github.com/Lindachidiebube/heavy-supplier-warehouse-analytics/blob/main/week-08/docs/KPI%2017%20Churn%20by%20Payment%20Behavior.md) |

Sprint notes: [sprint-08-notes.md](https://github.com/Lindachidiebube/heavy-supplier-warehouse-analytics/blob/main/week-08/sprint_notes/sprint-08-notes.md)

Notebooks:
- [Week8 KPI 14 RFM Segmentation.ipynb](https://github.com/Lindachidiebube/heavy-supplier-warehouse-analytics/blob/main/week-08/notebooks/Week8%20KPI%2014%20RFM%20Segmentation.ipynb)
- [Week8 KPI 15 Historical Customer Value.ipynb](https://github.com/Lindachidiebube/heavy-supplier-warehouse-analytics/blob/main/week-08/notebooks/Week8%20KPI%2015%20Historical%20Customer%20Value.ipynb)
- [Week8 KPI 16 Customer Payment Risk.ipynb](https://github.com/Lindachidiebube/heavy-supplier-warehouse-analytics/blob/main/week-08/notebooks/Week8%20KPI%2016%20Customer%20Payment%20Risk.ipynb)
- [Week8 KPI 17 Churn by Payment Behavior.ipynb](https://github.com/Lindachidiebube/heavy-supplier-warehouse-analytics/blob/main/week-08/notebooks/Week8%20KPI%2017%20Churn%20by%20Payment%20Behavior.ipynb)

The Week 8 results add a ninth strand: customer value is spread across many customers (KPI 15), customers do not differ in how they pay beyond what the due period and chance explain (KPI 16), and going quiet is not linked to paying late or leaving invoices unpaid (KPI 17).

## Week 9: Forecasting and reorder points

Week 9 continued Phase 3 (supplier, customer and predictive analytics). KPI 18 forecast how much of each product will sell in each branch, and KPI 19 used that demand with the real supplier waiting times to work out when each product should be reordered and whether any product is close to a stockout.

| # | KPI | Headline result | Write-up |
|---|---|---|---|
| 18 | Product Demand Forecast | The last 12 months (2024) were hidden and forecast from 2019 to 2023, using each product-branch pair's average month. The forecast total was 207,016 units against 199,858 sold. The gap is 5.18% per product over the year, 11.06% per pair over the year, 18.7% per pair over a quarter and 32.2% per pair per month, so the forecast is reliable per product over a year and weak per pair per month. 26 of 30 products are within 10% and none is 20% or more off; P019, P005, P012 and P010 are more than 10% off, all forecast too high. The 2025 forecast is 205,823 units, about 3.0% above 2024. | [KPI 18](https://github.com/Lindachidiebube/heavy-supplier-warehouse-analytics/blob/main/week-09/docs/KPI%2018%20Product%20Demand%20Forecast.md) |
| 19 | Stockout Risk and Reorder Point | No product-branch pair is near a stockout. Reorder points run from 89.9 to 144.5 units (average 116.2), while stock runs from 98,642 to 132,967 units, so every pair holds at least 745.7 times its reorder point. The closest pair (P019 in Kolkata) would need 74.8 years of average sales to reach it. The answer holds at 90% and 99% service levels, with every wait at 35 days, and against the most any pair ever sold in 35 days (383 units). The stored reorder levels are below the calculated ones in 175 of 180 pairs, and the stored lead times differ from the real waits by more than 1 day in 27 of 30 products. | [KPI 19](https://github.com/Lindachidiebube/heavy-supplier-warehouse-analytics/blob/main/week-09/docs/KPI%2019%20Stockout%20Risk%20and%20Reorder%20Point.md) |

Sprint notes: [sprint-09-notes.md](https://github.com/Lindachidiebube/heavy-supplier-warehouse-analytics/blob/main/week-09/sprint_notes/sprint-09-notes.md)

Notebooks:
- [Week9 KPI 18 Product Demand Forecast.ipynb](https://github.com/Lindachidiebube/heavy-supplier-warehouse-analytics/blob/main/week-09/notebooks/Week9%20KPI%2018%20Product%20Demand%20Forecast.ipynb)
- [Week9 KPI 19 Stockout Risk and Reorder Point.ipynb](https://github.com/Lindachidiebube/heavy-supplier-warehouse-analytics/blob/main/week-09/notebooks/Week9%20KPI%2019%20Stockout%20Risk%20and%20Reorder%20Point.ipynb)

The Week 9 results add a tenth strand: demand can be forecast reliably per product over a year but not per pair per month (KPI 18), and stock is far above every reorder point, at least 745.7 times (KPI 19).

## Data quality decisions

- Invoice and payment IDs contain genuine collisions (unrelated records sharing one ID). Rows affected are excluded only where a payments-to-invoices join is needed (KPI 4), and a bias check confirmed this does not distort the results.
- The current stock field in the inventory table is inflated, but Week 4 showed it matches the stock ledger for 166 of 180 product-branch pairs. The ledger itself is about 10% short of the order lines, so reconstructed stock (KPI 8) is calculated from the order lines, with the ledger-based figure shown beside it.
- About 10% of order lines never reached the stock ledger, so Delivered sales lines and Received purchase lines are the source for demand and purchases. Cancelled orders keep their lines in the lines tables and are filtered out by order status.
- ADJUSTMENT rows in the stock ledger stay out of KPI 1 and are used with their direction in the stock balance reconstruction (KPI 8).
- Master-table summary columns that do not match the transactions (customer purchase dates and totals, product last purchase date and lead time) are not used. Margin is recomputed from unit price and unit cost. Text columns such as warehouse capacity and all dates are converted before use.
- Revenue (KPI 7) is counted before GST and from Delivered sales lines only. Inventory turnover (KPI 9) uses the order-line stock from KPI 8, with the ledger-based figure shown beside it, and the weighted average cost paid on Received purchase lines, because the products-table cost differs from the purchase cost on 155,492 of 155,495 lines.
- Warehouse size (KPI 12) is read from text such as "45230 sqft" and, like the staff numbers used in KPI 11, is one current value per branch, so the same size is used for every past year. The size of sets and cartridges cannot be confirmed from the data, so the volume view of the stock is a comparison only.
- Excess stock (KPI 10) is measured on the order-line stock with needed stock set at 3 months of sales per product and branch, and the ledger-based stock and five other rules are shown as tests. In 2025, 135 of the 1,586 counted order lines have no stock ledger row and 15 ledger rows belong to cancelled orders, which is why the ledger is not used as the stock source.
- Supplier lead time and reliability (KPI 13) use the 21,630 Received purchase orders only, because Cancelled orders have no received date. Real lead time and lateness are worked out from the order, promised and received dates. The promised days in the suppliers table are supported by the orders (the real wait is 0.49 to 0.55 days longer for every supplier), but the reliability score is not (suppliers scored 3, 4 and 5 are late equally often), so it is not used as a measure.
- Customer analytics (KPIs 14 to 17) uses the snapshot date 1 Jan 2025, the day after the last order. The order data ends on 31 Dec 2024 and the quiet count rises with every later date (67 at 1 Jan 2025, 125 at 1 Feb 2025 and 201 at 1 Mar 2025). Customer money in KPIs 14 and 15 is counted before GST from Delivered orders. The payment KPIs (16 and 17) use the 17,640 invoices whose invoice ID is not repeated and the 18,840 payments that do not point to a repeated invoice ID, with money including GST, so the 393 invoices left out (about 2.2% of the money) are not in the unpaid money. The silence ratio is never rounded before counting, because a rounded gap lost one customer.
- The demand forecast and reorder points (KPIs 18 and 19) use Delivered sales lines for demand and Received purchase lines for waiting times, with the stock from KPI 8. Every product comes from all 8 suppliers, so each product uses its own real waiting time. The reorder levels and safety stocks stored in the inventory table and the lead times stored in the products table are not used, because they differ from the real data (175 of 180 pairs; 27 of 30 products). The 2025 forecast cannot be tested until 2025 happens; the same method was 3.6% too high on 2024.
- The full list of decisions is in [data-cleaning-log.md](https://github.com/Lindachidiebube/heavy-supplier-warehouse-analytics/blob/main/week-03/docs/data-cleaning-log.md).

## Running the notebooks

The notebooks run in Google Colab. Each one starts with a setup cell that mounts Google Drive and sets the dataset folder path, so the raw CSV files must be available in that Drive folder first.
