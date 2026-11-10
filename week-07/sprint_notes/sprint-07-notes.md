# Sprint 07 Notes

**Project:** Heavy Supplier & Warehouse Analytics
**Sprint:** Week 7 (Phase 3: Supplier, Customer & Predictive Analytics)

---

## Sprint Goal

- Measure how fast and how reliably each of the 8 suppliers delivers, using the real order dates, so procurement can plan orders and choose between suppliers.

## Work Completed

### KPI 13: Supplier Lead Time and Reliability

- Defined the KPI on the 21,630 Received purchase orders. The 2,370 Cancelled orders never arrived, so they were left out. The KPI covers 8 suppliers and 72 months of orders (January 2019 to December 2024).
- Measured four things for every supplier: real lead time (received date minus order date), days late (received date minus promised date), late share, and average days late.
- About 1 order in 7 is late: 3,267 of 21,630 (15.10%). 17,392 orders (80.41%) arrived on the promised day and 971 (4.49%) arrived early. Days late run from 3 days early to 7 days late, with an average of 0.513. A late order is 3.99 days late on average.
- All 8 suppliers keep the promised date about equally well: late share runs from 14.67% (Tianjin) to 15.80% (Hangzhou). A chi-square test gave a p-value of 0.9702, so the differences are within chance and no supplier is really later than the others.
- The suppliers differ in how long they take: real lead time runs from 10.50 days (Qingdao) to 28.49 days (Tianjin). The typical order arrives exactly on the promised day for all 8 suppliers, and the longest wait is the promised days plus 7.
- Compared the suppliers table with the real orders. The promised days are supported (the real wait is 0.49 to 0.55 days longer for every supplier). The stored reliability score is not: suppliers scored 5, 4 and 3 were late on 15.15%, 15.39% and 14.87% of orders. Reliability is therefore measured from the orders, and the stored score is shown for comparison only.
- Ran all 8 verification checks: missing values (no blanks, every order matched to a supplier), impossible dates (none; smallest real wait 7 days), repeated orders (none), totals reconcile (all match), second method (largest difference 0.0), extreme orders (none; longest wait 1.23 to 1.62 times the average), history length (72 months for every supplier; at least 19 orders in any month) and a raw file check of Qingdao, Tianjin and Hangzhou (identical to the table).
- Recommended planning orders with each supplier's real lead time and keeping a 7-day margin for the two slowest suppliers, Tianjin and Hangzhou.
- Scope decision: Supplier Performance Comparison is included inside KPI 13. The other five Week 7 proposed tasks (Supplier Contribution Analysis, Supplier Dependency Risk Analysis, Critical Supplier Identification, Supplier Diversification Analysis and Warehouse Utilisation Benchmarking) are marked Not selected because they answer other questions.
- Built in the notebook "Week7 KPI 13 Supplier Lead Time and Reliability" and written up in "KPI 13 Supplier Lead Time and Reliability.md".

## Blockers & Resolutions

- **Colab could not assign a runtime for a while.** This was a Google outage, not a problem with the notebook. Work continued when it recovered, and nothing was lost because the data and notebooks are on Drive.
- **The first rows suggested the promised and received dates might be the same.** Tested on all Received orders: 4,238 orders (19.59%) differ from the promised date, so the received date is real data.
- **The stored reliability score did not match the real deliveries.** Resolved by measuring reliability from the orders and keeping the score only as a comparison.

## Next Sprint Focus

- Week 8, Customer Analytics: KPI 14 RFM Segmentation, KPI 15 Historical Customer Value, KPI 16 Customer Payment Risk and KPI 17 Churn by Payment Behavior.
