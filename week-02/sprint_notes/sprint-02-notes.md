# Sprint 2 Notes — Week 2

**Sprint:** 2
**Team status:** Solo (all teammates removed/inactive as of Sprint 2 start)
**Scrum Master:** N/A (solo sprint)

## Sprint Goal

Address a gap identified from Sprint 1: no formal KPI/business-question proposal was completed before task selection, as required by the CadetX brief ("Before selecting tasks, explore the dataset and propose your own KPIs and a short list of analysis questions your team finds interesting"). Sprint 2 focuses on defining and fully verifying KPIs, starting with the Stock Accumulation Ratio, following a strict evidence-based verification process before any KPI is trusted for reporting.

## Work Completed

**Process correction:**
- Reviewed Sprint 1 output and identified that the team's exact-duplicate check had missed key-level duplicates (invoice_id/payment_id collisions) that a teammate's separate review later surfaced.
- Adopted a new standing rule: every future "duplicate/unique" check must test both row-level AND column/key-level uniqueness.

**KPI Roadmap defined:**
- Mapped out 13 KPIs across the full internship, in dependency order (e.g. Data Integrity Rate must precede Unpaid Invoice Rate and Late Payment Rate, since those depend on clean join keys).

**KPI 1 — Stock Accumulation Ratio: fully defined and verified**
- Formula: Total IN quantity ÷ Total OUT quantity, per product, using `stock_ledger.csv` (237,230 rows, 2019–2025, 30 products) — chosen over the unreliable `inventory_master.current_stock` field flagged in Week 1.
- Ran all 8 verification checks (nulls, zero-OUT/zero-IN edge cases, totals reconciliation, formula cross-check via two independent methods, outlier check, history-length check, manual spot-check on 3
