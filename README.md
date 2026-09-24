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

- `data-raw/` copies of the raw dataset files
- `week-XX/docs/` KPI write-ups and documentation
- `week-XX/notebooks/` verification notebooks (Google Colab, Python and pandas)
- `week-XX/sprint_notes/` weekly sprint notes

## Progress by week

| Week | Focus | Status |
|---|---|---|
| 1 | Dataset profiling, integrity checks, ERD, current stock investigation | Done |
| 2 | First four KPIs, built and fully verified | Done |
| 3 | KPI proposal write-up, key-uniqueness checks on all ID columns, data dictionary, KPI 5 | Next |
| 4 to 12 | Product, inventory and warehouse analytics; supplier and customer analytics; forecasting; dashboards; strategy and final presentation | Planned |

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
| 1 | Stock Accumulation Ratio | Only about 5 to 6% of the stock delivered has been sold, so stock arrives far faster than it sells | [KPI 01](week-02/docs/KPI%2001%20stock%20accumulation%20ratio.md) |
| 2 | Data Integrity Rate | Duplicate IDs affect 2.18% of invoice rows and 2.09% of payment rows | [KPI 02](week-02/docs/KPI%2002%20data%20integrity%20rate.md) |
| 3 | Unpaid Invoice Rate | 10.19% of invoices are Unpaid and 20.30% are Partially Paid, so 30.49% are not fully collected | [KPI 03](week-02/docs/KPI%2003%20unpaid%20invoice%20rate.md) |
| 4 | Late Payment Rate | 41.50% of payments arrive after the due date, typically about 17 days late (median); Chennai is highest at 48.61% | [KPI 04](week-02/docs/KPI%2004%20late%20payment%20rate.md) |

Read together, the Week 2 results point to a compounding risk: capital is tied up in stock that sells slowly, while a large share of billed revenue is slow to arrive or not fully collected.

## Data quality decisions

- Invoice and payment IDs contain genuine collisions (unrelated records sharing one ID). Rows affected are excluded only where a payments-to-invoices join is needed (KPI 4), and a bias check confirmed this does not distort the results.
- The current stock field in the inventory table is unreliable, so stock movements are calculated from the stock ledger instead.

## Running the notebooks

The notebooks run in Google Colab. Each one starts with a setup cell that mounts Google Drive and sets the dataset folder path, so the raw CSV files must be available in that Drive folder first.
