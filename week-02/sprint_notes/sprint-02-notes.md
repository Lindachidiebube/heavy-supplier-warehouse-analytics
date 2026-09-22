Sprint 2 Notes — Week 2

Sprint: 2
Team status: Solo (all teammates removed/inactive as of Sprint 2 start)
Scrum Master: N/A (solo sprint)

Sprint Goal

Address a gap identified from Sprint 1: no formal KPI/business-question proposal was completed before task selection, as required by the CadetX brief ("Before selecting tasks, explore the dataset and propose your own KPIs and a short list of analysis questions your team finds interesting"). Sprint 2 focuses on defining and fully verifying KPIs, starting with the Stock Accumulation Ratio, following a strict evidence-based verification process before any KPI is trusted for reporting.

Work Completed
Process correction: Reviewed Sprint 1 output and identified that the team's exact-duplicate check had missed key-level duplicates (invoice_id/payment_id collisions) that a teammate's separate review later surfaced. Adopted a new standing rule: every future "duplicate/unique" check must test both row-level AND column/key-level uniqueness.

KPI Roadmap defined: Mapped out 13 KPIs across the full internship, in dependency order (e.g. Data Integrity Rate must precede Unpaid Invoice Rate and Late Payment Rate, since those depend on clean join keys).

KPI 1 — Stock Accumulation Ratio: fully defined and verified

Formula: Total IN quantity ÷ Total OUT quantity, per product, using stock_ledger.csv (237,230 rows, 2019–2025, 30 products) — chosen over the unreliable inventory_master.current_stock field flagged in Week 1.

Ran all 8 verification checks (nulls, zero-OUT/zero-IN edge cases, totals reconciliation, formula cross-check via two independent methods, outlier check, history-length check, manual spot-check on 3 products) — all passed.
Discovered a previously undocumented third movement type in stock_ledger, ADJUSTMENT (2,454 rows), and made a documented methodology decision to exclude it from the ratio as a correction, not real supply/demand.

Business finding: across all 30 products, only ~5–6% of everything ever delivered has actually been sold — direct evidence of the brief's stated problem (reactive, demand-disconnected ordering causing chronic excess inventory).

Full write-up (Definition, Methodology, Verification Log, Data Quality Notes, Business Interpretation) completed and uploaded to week-02/docs/.
Verification code and outputs uploaded to week-02/notebooks/.

Blockers & Resolutions
Colab file path error: Hit a FileNotFoundError when loading stock_ledger.csv in a fresh Colab session — caused by Google Drive not being mounted / no full file path specified. Resolved by mounting Drive and using the full path to the dataset folder. Process fix going forward: every notebook will now start with a setup cell that mounts Drive and defines a reusable BASE_PATH variable, to prevent this recurring on every new file load.
GitHub folder structure error: Notebook was initially uploaded to the wrong path (nested inside docs instead of its own notebooks folder). Corrected by editing the file path directly on GitHub to move it to week-02/notebooks/.

Next Sprint Focus
Begin KPI 2 — Data Integrity Rate, quantifying the invoice_id/payment_id key collisions found in Sprint 1 review as a trackable percentage (affected-row rate vs. duplicate-group rate under discussion).
KPI 2 is a blocking dependency for KPI 3 (Unpaid Invoice Rate) and KPI 4 (Late Payment Rate), both of which need a clean, reliable join key before they can be calculated.
