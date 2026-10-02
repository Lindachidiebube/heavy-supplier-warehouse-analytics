# KPI Proposal and Business Questions

**Project:** Heavy Supplier & Warehouse Analytics
**Sprint:** Week 3 (Phase 1 close-out)
**Purpose:** Before choosing catalogue tasks, we explored the dataset and propose the KPIs and business questions below. Each KPI answers one business question linked to the project problem: reactive warehouse and supply-chain management causing excess inventory, stockouts, poor space usage and delayed decisions.

**Note on formulas:** Formulas are working definitions. Each one is finalised inside its own week, after the data for that KPI has been explored and checked. KPIs 1-4 are already built and verified, so their definitions are final.

---

## Phase 1: Data Foundation (Weeks 1-3)

### KPI 1: Stock Accumulation Ratio (DONE, Week 2)
- **Business question:** Which products are being bought in far faster than they are sold?
- **Formula:** Total IN quantity / Total OUT quantity per product (stock_ledger.csv). ADJUSTMENT movements are excluded.
- **Source:** stock_ledger

### KPI 2: Data Integrity Rate (DONE, Week 2)
- **Business question:** How much of our invoice and payment data can we trust?
- **Formula:** Rows with a duplicate ID / total rows, reported separately for invoice_id and payment_id.
- **Source:** invoices, payments

### KPI 3: Unpaid Invoice Rate (DONE, Week 2)
- **Business question:** How much billed revenue has not been collected?
- **Formula:** Unpaid invoices / total invoices. Also reported: Partially Paid rate and Total Outstanding rate.
- **Source:** invoices

### KPI 4: Late Payment Rate (DONE, Week 2)
- **Business question:** How often do customers pay after the due date, and where is it worst?
- **Formula:** Payments with payment_date > due_date / total clean payments. Also reported: median days late and rate by branch.
- **Source:** payments joined to invoices

---

## Product Movement (Week 3)

### KPI 5: Product Sales Velocity (Week 3)
- **Business question:** Which products are fast movers and which are slow movers?
- **Working formula:** Units sold per product per time period.
- **Source:** sales_orders_lines, sales_orders_header, products

---

## Phase 2: Inventory and Warehouse Performance (Weeks 4-6)

### KPI 6: Dead Stock Identification (Week 4)
- **Business question:** Which products are sitting in stock with no demand?
- **Working formula:** Products with stock on hand and no OUT movement within a set number of days.
- **Source:** stock_ledger, KPI 8

### KPI 7: ABC / Pareto Classification (Week 5)
- **Business question:** Which few products drive most of the revenue?
- **Working formula:** Rank products by revenue and classify A, B, C by cumulative share.
- **Source:** sales_orders_lines, products

### KPI 8: Reconstructed Stock Balance (Week 4)
- **Business question:** What is the real stock level, given that current_stock is unreliable?
- **Working formula:** Rebuilt balance from stock_ledger movements per product and branch (ADJUSTMENT direction resolved first).
- **Source:** stock_ledger, inventory_master

### KPI 9: Inventory Turnover Rate (Week 5)
- **Business question:** How fast does stock turn over?
- **Working formula:** Quantity sold / average stock level over the period.
- **Source:** stock_ledger, KPI 8

### KPI 10: Capital Tied Up in Excess Stock (Week 6)
- **Business question:** How much money is sitting in surplus stock?
- **Working formula:** Excess units x unit cost (products.csv must be confirmed to hold cost or price).
- **Source:** KPI 8, KPI 6, products

### KPI 11: Warehouse Throughput (Week 6)
- **Business question:** How much does each branch move?
- **Working formula:** Units moved (IN + OUT) per branch per period.
- **Source:** stock_ledger, branches

### KPI 12: Warehouse Space Utilisation (Week 6, may slip to Week 7)
- **Business question:** How much of each warehouse's space is actually used?
- **Working formula:** Stock held / branch capacity (branches.csv must be confirmed to hold capacity).
- **Source:** KPI 8, branches

---

## Phase 3: Suppliers, Customers and Forecasting (Weeks 7-9)

### KPI 13: Supplier Lead Time and Reliability (Week 7)
- **Business question:** Which suppliers deliver late, and how risky are they?
- **Working formula:** Lead time = received_date minus order date; reliability = share of orders received on time (purchase_orders_header must be confirmed to hold the needed dates).
- **Source:** purchase_orders_header, suppliers

### KPI 14: RFM Segmentation (Week 8)
- **Business question:** Which customers matter most and which are slipping away?
- **Working formula:** Recency, Frequency and Monetary scores per customer, grouped into segments.
- **Source:** customers, sales_orders_header, invoices

### KPI 15: Historical Customer Value (Week 8)
- **Business question:** How much revenue has each customer generated?
- **Working formula:** Total revenue per customer, ranked.
- **Source:** customers, invoices

### KPI 16: Customer Payment Risk (Week 8)
- **Business question:** Which customers combine high value with late or unpaid behaviour?
- **Working formula:** Combines KPI 3, KPI 4, KPI 14 and KPI 15 per customer.
- **Source:** KPIs 3, 4, 14, 15

### KPI 17: Churn by Payment Behavior (Week 8)
- **Business question:** Are quiet customers also the bad payers?
- **Working formula:** Compare inactivity (from RFM recency) with late payment rate (KPI 4) per customer.
- **Source:** KPI 4, KPI 14

### KPI 18: Product Demand Forecast (Week 9)
- **Business question:** How many units of each product will we need next period?
- **Working formula:** Units sold per product per period (stock_ledger OUT), forecast with trend or moving average.
- **Source:** stock_ledger

### KPI 19: Stockout Risk / Reorder Point (Week 9)
- **Business question:** When should we reorder so we do not run out?
- **Working formula:** Reorder point = average daily demand x supplier lead time + safety stock.
- **Source:** KPI 8, KPI 13, KPI 18

---

## Phase 4: Insight, Risk and Strategy (Weeks 10-12)

### KPI 20: Inventory Movement Anomaly Detection (Week 10)
- **Business question:** Which stock movements look unusual and need checking?
- **Working formula:** Flag outliers in stock_ledger movement quantities.
- **Source:** stock_ledger

### KPI 21: Stock Shrinkage Rate (Week 10)
- **Business question:** How much stock is missing compared with what should be there?
- **Working formula:** Gap between expected balance and reconstructed balance / expected balance.
- **Source:** KPI 8, inventory_master

### KPI 22: Operational Risk Score (Week 10)
- **Business question:** Where is the operation most at risk overall?
- **Working formula:** Combines KPI 4, KPI 6, KPI 20 and KPI 21 into one score per branch or product.
- **Source:** KPIs 4, 6, 20, 21

Week 11 brings all 22 KPIs together in dashboards, and Week 12 turns the findings into strategy recommendations.
