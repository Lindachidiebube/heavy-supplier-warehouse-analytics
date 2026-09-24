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

**KPI 4 — Late Payment Rate: fully defined and verified**
- Business question: how often do customers pay after the due date, and how long is the delay when they do — complements KPI 3 (KPI 3 measures how much money is still outstanding, KPI 4 measures how late the money arrives) and ties to the brief's goal of improving overall reliability and performance.
- Defined as Late Payment Rate = late payments ÷ total matched payments, where a payment is late when `payment_date > due_date` (due date taken from the matched invoice). A payment made exactly on the due date counts as on-time. The rate is per payment, not per invoice, and invoices with no payment are outside this KPI (they are covered by KPI 3). Supporting metric: Days Late, measured among late payments only.
- This was the first KPI needing a real payments↔invoices join, so KPI 2's scoping decision was applied: rows affected by duplicate IDs were excluded before joining (393 invoice rows and 403 payment rows), because a duplicate key could match a payment to the wrong invoice's due date. Before joining, ran a new standing check for orphaned keys: 0 of 19,257 payments had an `invoice_id` with no match in invoices, so every payment is joinable.
- After the join, 409 clean payments pointed to excluded invoices and dropped out, leaving 18,445 matched payments. Total excluded is 812 payments (403 + 409), which is 4.22% of 19,257 (95.78% kept).
- Headline results:
  - Late Payment Rate (headline): 7,655 / 18,445 = 41.50%
  - On-time or early payments: 10,790 (271 paid on the due date + 10,519 paid early)
  - Days late (late payments only): mean 20.02 days, median 17 days, middle half between 9 and 28 days, range 1 to 75 days
- Ran all 8 verification checks, on both the headline rate and the days-late metric: null check (0 nulls), cross-method check (same rate and mean across independent methods), row-gap explanation (19,257 − 18,445 = 812 = 403 + 409, fully accounted for), totals reconciliation (7,655 + 10,790 = 18,445), formula re-derivation three ways (identical results), distribution check across branches, years and customers, date-coverage check, and manual spot-check of 7 payments against the raw files (3 late, 3 early, 1 paid exactly on the due date) — all passed.
- Distribution findings: Chennai (CHN001) stands out at 48.61% late, roughly 7 to 11 points above the other five branches (37.72% to 43.15%), while median days late stays at 16 to 19 in every branch. From 2019 to 2024 the rate is stable at 40.2% to 43.1%. There is no customer concentration (500 customers, top customer 0.32% of payments), so lateness is a general habit and not the fault of a few accounts.
- Investigated why 2025 looks lower (30.46%, 371 payments): the late rate falls month by month toward the data cutoff (Jan 35.56%, Feb 18.18%, Mar 7.69%), consistent with a limited observation window since late payments for recent invoices may not have arrived yet. Tested the idea that 2025 simply has more unpaid invoices and it was not supported (346 invoices in 2025: 248 Paid, 64 Partially Paid, 34 Unpaid, a similar unpaid share to the whole table). Kept 41.50% as the headline and reported 41.73% excluding 2025 as a sensitivity check.
- Ran a pre-merge bias check comparing the excluded rows against the full tables: no meaningful skew in invoice amount or payment amount, but mild unevenness of about 2 to 3 percentage points by branch and year (PUN001 over-represented in the excluded set, HYD001 and 2023 under-represented). Judged not large enough to materially bias the headline, and stated in the write-up as a caveat instead of claimed as zero bias.
- Documented the on-due-date convention: counting a payment made exactly on the due date as late (`>=` instead of `>`) would give 7,926 late payments (42.97%). The stricter `>` was kept because paying on the due date meets the payment term.
- Resolved the open Sprint 1 finding about 208 "payment before invoice" records. In the KPI 4 merged table there are 0 such payments (and 0 invoices with a due date before the invoice date), so excluding them changes nothing and the rate stays 41.50%. To explain the difference, ran the same test on the raw, unfiltered payments↔invoices join (19,678 rows, 421 more than the 19,257 payments because duplicate invoice IDs match one payment to two invoices), which reproduces exactly 208. All 208 involve a duplicate `invoice_id` (only 3 involve a duplicate `payment_id`), so they are false matches caused by the ID collisions found in KPI 2, not real timing errors, and are already removed by the KPI 2 exclusion. Sprint 1 recommendation #3 is therefore resolved and needs no separate exclusion.
- Business finding: about 4 in every 10 payments arrive after the due date, typically about 2.5 weeks late, and the pattern is stable across 2019 to 2024, so late payment is persistent and not a one-off problem. Read together with KPI 3 (30.49% of invoices still Unpaid or Partially Paid), money is both slow to arrive and often not fully collected, which supports the brief's concern that decisions are reactive. The findings point to a data-driven collection focus (for example, earlier reminders and closer follow-up on the branch with the highest late rate).
- Full write-up completed and uploaded to `week-02/docs/KPI 04 late payment rate.md`. Verification notebook `week2_KPI_4_late_payment_rate.ipynb` uploaded to `week-02/notebooks/`.

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

**Colab session reset during KPI 4:**
- Mid-way through KPI 4 the Colab session reset and the notebook variables were lost (a `NameError` on `excluded_invoices`).
- Resolved by rebuilding the excluded-ID lists from the duplicate counts, confirming they matched KPI 2 exactly (196 invoice IDs / 393 rows and 201 payment IDs / 403 rows), and using Run all to restore every variable before continuing.

## Next Sprint Focus

- Start Week 3 (Sprint 3): formally write up the KPI proposal and business questions, run the key-level uniqueness check on every remaining ID column in the dataset (KPI 2 only covered `invoice_id` and `payment_id`), build the data dictionary, and begin KPI 5 — Product Sales Velocity.
