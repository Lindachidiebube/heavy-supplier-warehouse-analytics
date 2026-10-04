# KPI Proposal and Business Questions

**Project:** Heavy Supplier & Warehouse Analytics
**Sprint:** Week 3
**Purpose:** Before choosing catalogue tasks, we explored the dataset and propose the KPIs and business questions below. Each KPI answers one business question linked to the project problem: reactive warehouse and supply-chain management causing excess inventory, stockouts, poor space usage and delayed decisions.

**Note on formulas:** Formulas are working definitions. Each one is finalised inside its own week, after the data for that KPI has been explored and checked. KPIs 1-4 are already built and verified, so their definitions are final. The formulas below were updated after the Week 3 data checks (see `data-cleaning-log.md`).

---

## Rules from the Week 3 data checks

These apply to every KPI below.

- **Source of truth:** the transaction tables (orders, order lines, invoices, payments) agree with each other and are used for every KPI. Summary columns in the master tables (customers dates and totals, products lead time, last purchase date and margin percentage, inventory current_stock) do not match the transactions and are not used. Their values are worked out from the transactions.
- **Sales KPIs** use only Delivered orders. **Purchase KPIs** use only Received orders. Cancelled orders keep their lines in the data and must be filtered out.
- **Demand and purchase quantities** come from the Delivered sales lines and Received purchase lines, not from the stock ledger. About 10% of order lines never reached the ledger.
- **Duplicate invoice and payment IDs:** affected rows are left out only where invoices are joined to payments. KPI 3 keeps all invoices.
- **Dates** are stored as text and are converted before any date arithmetic. Quantities are not added across different products (units differ: piece, set, cartridge).

---

## Phase 1: Data Foundation (Weeks 1-3)

### KPI 1: Stock Accumulation Ratio (DONE, Week 2)
- **Business question:** Which products are being bought in far faster than they are sold?
- **Formula:** Total IN quantity / Total OUT quantity per product (stock_ledger.csv). ADJUSTMENT movements are excluded.
- **Source:** stock_ledger
- **Week 3 check:** the ratio is the same (about 18.1) whether it is calculated from the ledger or from the order lines, so the missing ledger lines do not change this KPI.

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
- **Working formula:** Units sold per product per month (and per branch), from Delivered sales lines only, then grouped into fast, medium and slow movers.
- **Source:** sales_orders_lines, sales_orders_header, products

---

## Phase 2: Inventory and Warehouse Performance (Weeks 4-6)

### KPI 6: Dead Stock Identification (Week 4)
- **Business question:** Which products are sitting in stock with no demand?
- **Working formula:** Product and branch combinations with stock on hand (KPI 8) and no Delivered sales within a set number of days. The number of days is chosen in Week 4 after looking at the data.
- **Source:** KPI 8, sales_orders_lines, sales_orders_header

### KPI 7: ABC / Pareto Classification (Week 5)
- **Business question:** Which few products drive most of the revenue?
- **Working formula:** Rank products by revenue from Delivered sales lines and classify A, B, C by cumulative share. Margin is recomputed as (unit_price - unit_cost) / unit_price, not taken from the margin_percentage column.
- **Source:** sales_orders_lines, sales_orders_header, products

### KPI 8: Reconstructed Stock Balance (Week 4)
- **Business question:** What is the real stock level, given that current_stock is unreliable?
- **Working formula:** Two rebuilds per product and branch, compared side by side. (1) Ledger-based: opening stock + IN - OUT +/- ADJUSTMENT, with the direction of each adjustment read from the change in running_balance. (2) Order-line-based: opening stock + all Received purchase lines - all Delivered sales lines +/- adjustments. Rows linked to cancelled orders are reported both ways (with and without).
- **Source:** stock_ledger, inventory_master (opening stock only), purchase_orders_lines, sales_orders_lines

### KPI 9: Inventory Turnover Rate (Week 5)
- **Business question:** How fast does stock turn over?
- **Working formula:** Quantity sold (Delivered sales lines) / average stock level over the period (KPI 8).
- **Source:** sales_orders_lines, KPI 8

### KPI 10: Capital Tied Up in Excess Stock (Week 6)
- **Business question:** How much money is sitting in surplus stock?
- **Working formula:** Excess units x unit cost (products.unit_cost). The rule for what counts as excess is set in Week 6.
- **Source:** KPI 8, KPI 6, products

### KPI 11: Warehouse Throughput (Week 6)
- **Business question:** How much does each branch move?
- **Working formula:** Units moved (Received purchase lines + Delivered sales lines) per branch per period.
- **Source:** purchase_orders_lines, sales_orders_lines, order headers, branches

### KPI 12: Warehouse Space Utilisation (Week 6, may slip to Week 7)
- **Business question:** How much of each warehouse's space is actually used?
- **Working formula:** Stock held / branch capacity. The capacity is stored as text (for example "45230 sqft") and is converted to a number first. The way to compare stock units with floor space is decided in Week 6.
- **Source:** KPI 8, branches

---

## Phase 3: Suppliers, Customers and Forecasting (Weeks 7-9)

### KPI 13: Supplier Lead Time and Reliability (Week 7)
- **Business question:** Which suppliers deliver late, and how risky are they?
- **Working formula:** Lead time = received_date minus order_date; reliability = share of orders received on or before expected_delivery_date. Received orders only, reported per supplier.
- **Source:** purchase_orders_header, suppliers

### KPI 14: RFM Segmentation (Week 8)
- **Business question:** Which customers matter most and which are slipping away?
- **Working formula:** Recency (days since last Delivered order), Frequency (number of Delivered orders) and Monetary (total invoiced value) scores per customer, grouped into segments. All three come from the transactions, not from the customers table.
- **Source:** sales_orders_header, invoices, customers (customer details only)

### KPI 15: Historical Customer Value (Week 8)
- **Business question:** How much revenue has each customer generated?
- **Working formula:** Total invoiced revenue per customer, ranked.
- **Source:** invoices, sales_orders_header, customers (customer details only)

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
- **Working formula:** Units sold per product per period (Delivered sales lines), forecast with trend or moving average.
- **Source:** sales_orders_lines, sales_orders_header

### KPI 19: Stockout Risk / Reorder Point (Week 9)
- **Business question:** When should we reorder so we do not run out?
- **Working formula:** Reorder point = average daily demand x supplier lead time + safety stock. Lead time comes from the supplier table or the real purchase order dates, not from products.lead_time_days. The final choice is confirmed in Week 9.
- **Source:** KPI 8, KPI 13, KPI 18

---

## Phase 4: Insight, Risk and Strategy (Weeks 10-12)

### KPI 20: Inventory Movement Anomaly Detection (Week 10)
- **Business question:** Which stock movements look unusual and need checking?
- **Working formula:** Flag outliers in stock_ledger movement quantities.
- **Source:** stock_ledger

### KPI 21: Stock Shrinkage Rate (Week 10)
- **Business question:** How much stock is missing compared with what should be there?
- **Working formula:** Gap between the order-line-based balance (expected) and the ledger-based balance from KPI 8, plus the net effect of ADJUSTMENT movements, divided by the expected balance.
- **Source:** KPI 8, stock_ledger

### KPI 22: Operational Risk Score (Week 10)
- **Business question:** Where is the operation most at risk overall?
- **Working formula:** Combines KPI 4, KPI 6, KPI 20 and KPI 21 into one score per branch or product.
- **Source:** KPIs 4, 6, 20, 21

Week 11 brings all 22 KPIs together in dashboards, and Week 12 turns the findings into strategy recommendations.
