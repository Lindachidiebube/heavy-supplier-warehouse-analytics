# Sprint 1 Notes — Heavy Supplier & Warehouse Analytics

**Sprint:** 1 (Data Foundation & Exploration)
**Scrum Master this sprint:** Linda chidiebube
**Team size:** 4 (2 members introduced so far; 2 pending)

## Sprint Goal
Establish a clean, well-understood data foundation before building KPIs and dashboards — profile all 12 tables for data quality issues, map relationships between tables, and document findings for the team.

## Work Completed (Linda chidiebube — Req 1, 2, 3, 4)
- **Req 1 — Dataset structure/overview:** all 12 tables reviewed (branches, customers, inventory_master, invoices, payments, products, purchase_orders_header, purchase_orders_lines, sales_orders_header, sales_orders_lines, stock_ledger, suppliers)
- **Req 2 — Primary/foreign keys:** identified across all tables; 5 referential integrity checks run, all clean (0 unmatched FK records)
- **Req 3 — Table relationship mapping:** ERD built in dbdiagram.io (live link: https://dbdiagram.io/d/6aa7107436f9982564809e24); standalone master tables vs composite-key junction tables mapped and documented
- **Req 4 — current_stock anomaly investigation:** found `inventory_master.current_stock` inflated 100–850x realistic warehouse capacity; traced exactly to `stock_ledger.running_balance`; root cause is inbound POs (~160 units avg) far exceeding outbound sales (~10 units avg) compounded over 2019–2025 history. Requires team decision before building any stock-related KPIs.

## Work Completed (Deepika Gupta — Req 5, official team submission)
Full 12-section Data Quality Profiling doc covering:
1. Objective & methodology (missing values, duplicates, keys, dates, invalid values, cross-table consistency, calculations)
2. Dataset scope — all 12 tables
3. Missing values — only significant gap: 2,370 missing `received_date` in purchase_orders_header, confirmed tied to cancelled POs (not random)
4. Duplicate analysis — 0 exact duplicate rows; but duplicate key IDs found: invoice_id (393 rows/196 groups), payment_id (403 rows/201 groups) — flagged "High priority"
5. Date consistency — all logical date-order checks clean (order vs delivery, invoice vs due date, etc.)
6. Payment status consistency — 1,837 invoices marked "Unpaid," 35 of those have an associated payment record (inconsistency flagged)
7. Unusual/invalid values — no negative/zero-value issues; all calculation checks (PO lines, SO lines, invoice totals) internally consistent
8. Referential integrity — 0 unmatched FK records, matches Lindachidiebube's independent Req 2 findings
9. Key data quality findings summary table (missing dates, duplicate IDs, late PO receipts, payment timing issues)
10. Overall data quality assessment — dataset usable for analysis; main concerns are duplicate IDs (risk of double-counting in joins)
11. Recommendations — investigate duplicate IDs before aggregation, validate early/late payments, use late deliveries/payments as KPIs, document all issues before dashboard-building
12. Conclusion — dataset structured and usable overall, pending validation of flagged issues

## Cross-Review & Additional Findings (Linda chidiebube, verifying Deepika's Req 5 work)
Independently verified several of Deepika's findings using pandas in Colab:

- **Duplicate ID collisions confirmed as a deeper issue than reported:** both invoice_id (196 groups) and payment_id (201 groups) each contain exactly **one group of 3 rows**, not just pairs. Investigated the actual records:
  - `INV-341709`: 3 completely unrelated invoices (different customers, branches, dates, amounts — all marked "Paid")
  - `PAY-361416`: 3 completely unrelated payments (different invoice_ids, dates, amounts, payment methods)
  - Conclusion: this is genuine **ID collision**, not harmless duplication — invoice_id and payment_id cannot be trusted as standalone unique keys for joins/aggregations. High-severity, needs a fix at the source or a composite-key workaround.

- **Added denominators for context:** total invoices = 18,033; total payments = 19,257
  - Unpaid invoices: 1,837 = **10.2%** of all invoices
  - Late payments: 8,200 = **42.6%** of all payments — flagged as a headline finding, strong candidate for its own KPI in later sprints

- **Investigated the 208 "payment before invoice date" records for clustering:** spread across all 6 branches roughly proportionally (CHN001: 52, KOL001: 41, PUN001: 39, AHM001: 30, DEL001: 27, HYD001: 19); no customer concentration (top customer only 4 occurrences). Conclusion: likely scattered data-entry noise, not a systemic/branch-specific issue — recommend excluding from date-sensitive analysis rather than building a KPI around it.

- **current_stock cross-reference gap:** Deepika's Req 5 doc does not mention the current_stock anomaly (already covered in Lindachidiebube's Req 4 report) — flagged to her as a suggested one-line cross-reference addition, and documented separately in week1_review_notes_lindachidiebube.docx.

## Questions Sent to Deepika — Awaiting Response
1. Heads-up on current_stock anomaly cross-reference
2. Whether she'd identified the 3+ duplicate groups (triplets) in invoice_id/payment_id
3. Denominators for unpaid invoices / late payments context
4. Whether the 208 early-payment records cluster by branch/customer or scatter randomly

**Status:** No response yet as at the time of this sprint's submission. Findings above are independently verified and stand regardless of her reply; her response may add interpretation/context but is not expected to change the underlying numbers. Will update if her response adds new information.

## Blockers
- Teammate response pending
- 2 of 4 team members yet to introduce themselves / confirm task assignments or GitHub usernames (needed to add as repo collaborators)

## Next Sprint Focus
- Finalize Week 1 submission (GitHub repo + CadetX portal link)
- Begin KPI definition based on this week's findings (late payment rate, unpaid invoice rate, supplier delivery delays)
- Resolve invoice_id/payment_id collision issue before any revenue-related aggregation work
