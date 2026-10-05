# Heavy Supplier, Inventory & Warehouse Analytics

CadetX Virtual Internship project. This repository holds the weekly work for a 12-week analytics project on warehouse, inventory and supplier data.

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
| 4 | Stock balance reconstruction and dead stock identification (KPIs 8 and 6) | Next |
| 5 to 12 | Product, inventory and warehouse analytics; supplier and customer analytics; forecasting; risk and anomaly detection; dashboards; strategy and final presentation | Planned |

## Week 1: Data exploration

No KPIs were built in Week 1. The week was spent checking whether the data can be trusted.

- **Profiling:** all 12 tables were profiled. There were no nulls or duplicate rows, except the purchase order header table, where the received date is empty for 2,370 rows (all of them Cancelled orders).
- **Referential integrity:** 5 checks on how the tables link together all came back clean.
- **Table relationships:** an entity relationship diagram (ERD) was built to show how the tables connect: view the ERD.
- **Current stock problem:** the recorded current stock in the inventory table is inflated by roughly 100 to 850 times. It matches the running balance in the stock ledger exactly, because stock delivered per movement (about 160 units on average) is far larger than stock sold (about 10 units) and this compounds from 2019 to 2025. This is why the stock ledger is used instead of the current stock field.
- **Duplicate IDs found:** 196 invoice IDs and 201 payment IDs are shared by unrelated records. This led to the Week 2 data integrity KPI.

## Week 2 KPIs

| # | KPI | Headline result | Write-up |
|---|---|---|---|
| 1 | Stock Accumulation Ratio | Only about 5 to 6% of the stock delivered has been sold, so stock arrives far faster than it sells | KPI 01 |
| 2 | Data Integrity Rate | Duplicate IDs affect 2.18% of invoice rows and 2.09% of payment rows | KPI 02 |
| 3 | Unpaid Invoice Rate | 10.19% of invoices are Unpaid and 20.30% are Partially Paid, so 30.49% are not fully collected | KPI 03 |
| 4 | Late Payment Rate | 41.50% of payments arrive after the due date, typically about 17 days late (median); Chennai is highest at 48.61% | KPI 04 |

Read together, the Week 2 results point to a compounding risk: capital is tied up in stock that sells slowly, while a large share of billed revenue is slow to arrive or not fully collected.

## Week 3: Phase 1 close-out and KPI 5

Week 3 closed Phase 1 (data foundation and preparation). Every ID column of the 12 tables was tested, every cleaning decision was recorded, the table joins were mapped, every column and every created field was documented, and KPI 5 was built and verified. All documents are in `week-03/docs/`, the notebooks in `week-03/notebooks/` and the sprint notes in `week-03/sprint_notes/`.

| Document | What it contains |
|---|---|
| `KPI-proposal.md` | Roadmap of 22 KPIs with business questions, updated with the Week 3 findings |
| `data-cleaning-log.md` | Every data-quality finding and the decision taken |
| `table-join-map.md` | How the 12 tables join, with a diagram, orphan checks and a comparison with the Week 1 ERD |
| `KPI-05-Product-Sales-Velocity.md` | KPI 5 write-up (Definition, Methodology, Verification Log, Data Quality Notes, Business Interpretation) |
| `feature-list.md` | Every field created in the KPI notebooks, with its meaning and calculation |
| `data-dictionary.md` | All 134 columns of the 12 tables: type, empty count, distinct count, range, examples and meaning (marked as inferred from the data) |

Notebooks: `Week3_ID_Uniqueness_Check`, `Week3_KPI05_Product_Sales_Velocity` and `Week3_Data_Dictionary_Profile`.

**Key-uniqueness and link checks**

- Every real primary key is unique with no empty values. The only duplicate keys are invoice IDs (393 rows in repeated groups) and payment IDs (403 rows in repeated groups).
- 18 cross-table links were tested and none has a real orphan. The 2,454 stock ledger rows (1.03%) that match no order are exactly the ADJUSTMENT rows, which is by design.
- Every Delivered order has an invoice, and no invoice belongs to a non-Delivered order.

**Week 3 KPI**

| # | KPI | Headline result | Write-up |
|---|---|---|---|
| 5 | Product Sales Velocity | All 30 products sell at a similar pace (548 to 593 units a month), but speed depends on the branch, not the product. About 18 units are bought for every 1 sold, and buying stays almost the same in every branch, so the lowest-selling branches (Hyderabad, Delhi, Pune) carry the most over-buying. Sales fell 3.4% between 2019-21 and 2022-24. | KPI 05 |

The Week 3 results add a fourth strand to the earlier picture: stock arrives far faster than it sells (KPI 1), and buying does not follow selling, either by product or by branch (KPI 5).

## Data quality decisions

- Invoice and payment IDs contain genuine collisions (unrelated records sharing one ID). Rows affected are excluded only where a payments-to-invoices join is needed (KPI 4), and a bias check confirmed this does not distort the results.
- The current stock field in the inventory table is unreliable, so stock movements are calculated from the stock ledger instead.
- About 10% of order lines never reached the stock ledger, so Delivered sales lines and Received purchase lines are the source for demand and purchases. Cancelled orders keep their lines in the lines tables and are filtered out by order status.
- ADJUSTMENT rows in the stock ledger stay out of KPI 1 and are used with their direction in the stock balance reconstruction (KPI 8).
- Master-table summary columns that do not match the transactions (customer purchase dates and totals, product last purchase date and lead time) are not used. Margin is recomputed from unit price and unit cost. Text columns such as warehouse capacity and all dates are converted before use.
- The full list of decisions is in `week-03/docs/data-cleaning-log.md`.

## Running the notebooks

The notebooks run in Google Colab. Each one starts with a setup cell that mounts Google Drive and sets the dataset folder path, so the raw CSV files must be available in that Drive folder first.
