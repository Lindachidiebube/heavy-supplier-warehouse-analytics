# Sprint 2 Notes — Week 2

**Sprint:** 2
**Team status:** Solo (all teammates removed/inactive as of Sprint 2 start)
**Scrum Master:** N/A (solo sprint)

## Sprint Goal

Address a gap identified from Sprint 1: no formal KPI/business-question proposal was completed before task selection, as required by the CadetX brief ("Before selecting tasks, explore the dataset and propose your own KPIs and a short list of analysis questions your team finds interesting"). Sprint 2 focuses on defining and fully verifying KPIs, following a strict evidence-based verification process before any KPI is trusted for reporting.

## Work Completed

**Process correction:**
- Reviewed Sprint 1 output and identified that the team's exact-duplicate check had missed key-level duplicates (invoice_id/payment_id collisions) that a teammate's separate review later surfaced.
- Adopted a new standing rule: every future "duplicate/unique" check must test both row-level AND column/key-level uniqueness.

**KPI Roadmap defined:**
- Mapped out 13 KPIs across the full internship, in dependency order (e.g. Data Integrity Rate must precede Unpaid Invoice Rate and Late Payment Rate, since those depend on clean join keys).

**KPI 1 — Stock Accumulation Ratio: fully defined and verified**
- Formula: Total IN quantity ÷ Total OUT quantity, per product, using `stock_ledger.csv` (237,230 rows, 2019–2025, 30 products) — chosen over the unreliable `inventory_master.current_stock` field flagged in Week 1.
- Ran all 8 verification checks (nulls, zero-OUT/zero-IN edge cases, totals reconciliation, formula cross-check via two independent methods, outlier check, history-length check, manual spot-check on 3 products) — all passed.
- Discovered a previously undocumented third movement type in stock_ledger, `ADJUSTMENT` (2,454 rows), and made a documented methodology decision to exclude it from the ratio as a correction, not real supply/demand.
- Business finding: across all 30 products, only ~5–6% of everything ever delivered has actually been sold — direct evidence of the brief's stated problem (reactive, demand-disconnected ordering causing chronic excess inventory).
- Full write-up completed and uploaded to `week-02/docs/KPI 01 stock accumulation ratio.md`. Verification code and outputs uploaded to `week-02/notebooks/`.

**KPI 2 — Data Integrity Rate (Invoice & Payment Keys): fully defined and verified**
- Investigated the invoice_id/payment_id key collisions first flagged in Sprint 1: `invoice_id` has 196 duplicate groups (393 affected rows out of 18,033), `payment_id` has 201 duplicate groups (403 affected rows out of 19,257).
- Defined as two separate metrics (not combined) to avoid masking which ID column is affecting which downstream KPI: Invoice ID Integrity Rate (2.18%) and Payment ID Integrity Rate (2.09%), using affected-row rate rather than duplicate-group rate, since row rate scales correctly with actual data exposure.
- Ran all 8 verification checks: null check, cross-method duplicate-count reconciliation, group-size distribution (confirmed only pairs + one triplet per table, nothing larger), content-difference check (confirmed genuine ID collisions between unrelated records, not accidental row duplication), scatter/concentration check across branches and payment methods, temporal check across 2019–2025, denominator sanity check, and manual spot-check of both triplets — all passed.
- Evaluated and rejected a composite-key fix (e.g. invoice_id + so_id), since `payments.csv` has no shared field to disambiguate from its side of the join — exclusion of affected rows was concluded to be the only defensible approach for join-dependent calculations.
- Ran a bias check comparing payment_status distribution in the affected rows vs. the full dataset (Paid, Partially Paid, Unpaid all within ~0.6 percentage points of each other) to confirm excluding these rows from downstream KPIs would not distort results.
- Scoping decision: exclusion applies only where a payments↔invoices join is genuinely required (i.e. Late Payment Rate), not blanket-applied everywhere, since payment_status already lives inside `invoices.csv` and doesn't need the join.
- Full write-up completed and uploaded to `week-02/docs/KPI 02 data integrity rate.md`. Verification notebook uploaded to `week-02/notebooks/`.

**KPI 3 — Unpaid Invoice Rate & Outstanding Revenue Exposure: fully defined and verified**
- Business question: how much of billed revenue is stuck in unpaid or partially unpaid customer accounts, and how exposed is the business financially as a result — connects to the brief's procurement/replenishment decision-making value area, since uncollected revenue affects how confidently the business can commit cash to new stock.
- Resolved an open definitional question on the numerator: rather than picking a single "Unpaid Rate" number, defined the KPI as three separate metrics, since "Unpaid" and "Partially Paid" are genuinely different business states and blending them would obscure a distinction worth investigating on its own — the same reasoning used to keep invoice_id/payment_id rates separate in KPI 2.
  - Unpaid Invoice Rate (headline): 1,837 / 18,033 = 10.19%
  - Partially Paid Rate (supporting): 3,660 / 18,033 = 20.30%
  - Total Outstanding / At-Risk Revenue Rate (supporting): 5,497 / 18,033 = 30.49%
- Reasoned through whether the 393 invoice_id-affected rows from KPI 2 needed exclusion here, and concluded no: a duplicate ID only threatens operations requiring a join across tables, not a straight read of a value (`payment_status`) already sitting in the row. Exclusion stays scoped to KPI 4, per KPI 2's decision.
- Ran all 8 verification checks: null check, cross-method count reconciliation, a no-dedup check confirming `len(invoices)` vs. `invoices['invoice_id'].nunique()` produces exactly the 197-row gap expected from KPI 2's known duplicate groups (195 pairs + 1 triplet), totals reconciliation (all three categories sum exactly to 18,033 with no hidden/typo categories), formula re-derivation via three independent methods, distribution/scatter check across branches/customers/years, date-range and coverage check, and manual spot-checks — all passed.
- Deliberately ran the re-derivation, distribution, and spot-check tests separately for **both** Unpaid and Partially Paid, not just the headline metric, so the supporting number received the same scrutiny rather than inheriting confidence for free from the raw counts. Both categories showed the same healthy pattern: even spread across all 6 branches, no customer concentration (top contributor under 1% of either category), stable year-over-year 2019–2024.
- Business finding: roughly 30% of billed revenue has not been fully collected. Combined with KPI 1's finding of chronic excess inventory (5–6% average sell-through), this points to a compounding risk — capital tied up in unsold stock while a meaningful share of billed revenue sits uncollected.
- Full write-up completed and uploaded to `week-02/docs/KPI 03 unpaid invoice rate.md`. Verification notebook uploaded to `week-02/notebooks/` and confirmed.

## Blockers & Resolutions

**Colab file path error:**
- Hit a `FileNotFoundError` when loading `stock_ledger.csv` in a fresh Colab session — caused by Google Drive not being mounted / no full file path specified.
- Resolved by mounting Drive and using the full path to the dataset folder.
- Process fix going forward: every notebook now starts with a setup cell that mounts Drive and defines a reusable `BASE_PATH` variable, to prevent this recurring on every new file load.

**GitHub folder structure error:**
- KPI 1's notebook was initially uploaded to the wrong path (nested inside `docs` instead of its own `notebooks` folder).
- Corrected by editing the file path directly on GitHub to move it to `week-02/notebooks/`.

**Markdown formatting error:**
- An earlier draft of this sprint notes file was pasted into GitHub without proper Markdown structure (no blank lines between headers/lists), causing it to render as one unbroken block of text.
- Corrected by rewriting with proper header, bullet, and spacing syntax and re-committing.

## Next Sprint Focus

- Begin KPI 4 — Late Payment Rate, which requires a genuine payments↔invoices join. Will apply the row-exclusion approach from KPI 2 (the 393 invoice_id-affected rows), which the bias check already confirmed is safe.
