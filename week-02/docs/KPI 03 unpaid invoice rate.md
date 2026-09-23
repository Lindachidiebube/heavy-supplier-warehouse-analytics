# KPI #3: Unpaid Invoice Rate & Outstanding Revenue Exposure

## 1. Definition

This KPI is reported as **three related metrics** rather than a single blended number, because "Unpaid" and "Partially Paid" represent genuinely different business states and combining them would obscure information that should be interpreted separately.

| Metric | Formula | Value |
|---|---|---|
| **Unpaid Invoice Rate** (headline) | Unpaid invoices ÷ Total invoices | 1,837 ÷ 18,033 = **10.19%** |
| **Partially Paid Rate** (supporting) | Partially Paid invoices ÷ Total invoices | 3,660 ÷ 18,033 = **20.30%** |
| **Total Outstanding / At-Risk Revenue Rate** (supporting) | (Unpaid + Partially Paid) ÷ Total invoices | 5,497 ÷ 18,033 = **30.49%** |

**Source table:** `invoices.csv` (18,033 rows). No join required — `payment_status` is a native column on this table.

## 2. Business Question

**"How much of billed revenue is stuck in unpaid or partially unpaid customer accounts, and how exposed is the business financially as a result?"**

This connects directly to the brief's stated business value areas:

- **Data-driven procurement/replenishment decisions** — cash tied up in uncollected revenue directly limits how confidently the business can commit to new stock purchases. This links back to KPI #1's finding of chronic over-ordering: a business already over-invested in excess inventory (only ~5-6% average sell-through) is doubly exposed if a meaningful share of the revenue it *has* billed isn't actually collectable yet.
- **Improve overall warehouse/business reliability & performance** — a ~30% at-risk revenue rate is a material signal for financial planning and risk management, not just an operations metric.

## 3. Methodology

- `payment_status` was read directly from `invoices.csv`; no join to `payments.csv` was needed, since payment status already lives on the invoice record itself.
- The 393 rows affected by the invoice_id duplicate-ID collision (identified in KPI #2) were **not excluded** from this calculation. Rationale: a duplicate ID only threatens operations that need to *match* rows across tables (joins). This KPI only *reads* a value already sitting in each row — the duplicate ID doesn't make that row's own `payment_status` any less real or trustworthy. (Exclusion of these rows is reserved for KPI #4, which does require a payments↔invoices join.)
- Three independent calculation methods (boolean mask + `.sum()`, `.groupby().size()`, and `.value_counts(normalize=True)`) were cross-checked and returned an identical result to 14 decimal places, confirming the rate is not an artifact of any one coding approach.

## 4. Verification Log

Checks #1, #3, #4, and #7 are **structural, whole-table checks** — they validate `payment_status` and `invoices.csv` as a whole, so a single pass covers all three reported metrics (Unpaid, Partially Paid, and by extension Total Outstanding). Checks #2, #5, #6, and #8 test the actual **rate calculations and their real-world distribution**, so these were run separately for both the headline metric (Unpaid) and its main supporting metric (Partially Paid) to give each the same level of scrutiny — Total Outstanding is a straight sum of these two independently-verified numbers and does not require its own separate re-derivation, distribution check, or spot-check.

| # | Check | Method | Result | Status |
|---|---|---|---|---|
| 1 | Null check | `.isnull().sum()` on `payment_status` | 0 nulls (whole column) | ✅ Pass |
| 2 | Cross-method count | `value_counts()` vs boolean-filter `.sum()` | Both confirm 1,837 Unpaid and 3,660 Partially Paid; full breakdown 12,536 Paid / 3,660 Partially Paid / 1,837 Unpaid | ✅ Pass |
| 3 | No-dedup check | `len(invoices)` vs `invoices['invoice_id'].nunique()` | 18,033 − 17,836 = 197, fully explained by the 195 duplicate pairs (1 excess row each) + 1 triplet (2 excess rows) from KPI #2 = 197. Confirms no accidental row collapsing occurred (whole table). | ✅ Pass |
| 4 | Totals reconciliation | `value_counts()` sum vs `len(invoices)`; check for hidden/typo categories via `.unique()` | 12,536 + 3,660 + 1,837 = 18,033, exact match. Only 3 clean category values present: `Paid`, `Partially Paid`, `Unpaid` — no typos, casing issues, or stray categories. | ✅ Pass |
| 5a | Re-derive 3 ways — **Unpaid** | Boolean mask vs `.groupby().size()` vs `.value_counts(normalize=True)` | All three methods return 0.10186879609604614 | ✅ Pass |
| 5b | Re-derive 3 ways — **Partially Paid** | Boolean mask vs `.groupby().size()` vs `.value_counts(normalize=True)` | All three methods return 0.2029612377308268 | ✅ Pass |
| 6a | Distribution / scatter — **Unpaid** | Broken down by branch, customer, year | Branch: even spread across all 6 branches (240–400 range, sums to 1,837). Customer: top customer accounts for only 0.65% of all Unpaid invoices (12 of 1,837), long tail with no concentration. Year: stable 293–321/year across 2019–2024, no anomalous spike or drop. | ✅ Pass |
| 6b | Distribution / scatter — **Partially Paid** | Broken down by branch, customer, year | Branch: even spread across all 6 branches (478–763 range, sums to 3,660), same relative ordering as Unpaid. Customer: top customer accounts for only 0.46% of all Partially Paid invoices (17 of 3,660), long tail with no concentration. Year: stable 577–638/year across 2019–2024, no anomalous spike or drop. | ✅ Pass |
| 7 | History / coverage check | Min/max `invoice_date`, per-year row counts | Range: 2019-01-03 to 2025-01-13 (2,202 days). Six full years at a consistent ~2,950–3,035 invoices/year; 2025 shows only 50 rows, consistent with a partial-year dataset cutoff (expected ~107 for 13 days at the yearly rate — no unexplained gap). Sums to 18,033, matching total row count (whole table, covers both metrics). | ✅ Pass |
| 8a | Manual spot-check — **Unpaid** | Hand-verified 3 randomly sampled Unpaid invoices | All 3 rows show positive, realistic `grand_total` values, valid `customer_id`/`branch_id`, and `due_date` on or after `invoice_date` in every case. One showed a 60-day payment term vs 30 for the others. No logic violations found. | ✅ Pass |
| 8b | Manual spot-check — **Partially Paid** | Hand-verified 3 randomly sampled Partially Paid invoices | All 3 rows show positive, realistic `grand_total` values, valid `customer_id`/`branch_id`, and `due_date` on or after `invoice_date` in every case. One showed a 15-day payment term vs 30 for the others. No logic violations found. | ✅ Pass |

## 5. Data Quality Notes

- **197-row gap in `nunique()` vs `len()` is expected, not an error.** It is fully explained by the invoice_id duplicate-ID collision documented in KPI #2 (195 pairs + 1 triplet = 197 "excess" rows collapsed under `nunique()`). This KPI's calculation uses row counts throughout, not `nunique()`, so the duplicate IDs do not affect the 10.19% / 20.30% / 30.49% figures.
- **2025 is a partial year** (only 13 days of data, through 2025-01-13), consistent with the same dataset cutoff observed in KPI #1 and KPI #2. The low 2025 invoice count (50) is expected and does not indicate missing or corrupted data.
- **Payment terms vary slightly by invoice** — spot-checks surfaced a 60-day due date window on one Unpaid invoice and a 15-day window on one Partially Paid invoice, versus the more common 30-day term elsewhere. This is a normal business variation (likely by customer or contract type) and does not affect the validity of `payment_status` values.
- **Partially Paid shows the same healthy distribution pattern as Unpaid** — same relative branch ordering, similarly low customer concentration (0.46% vs 0.65% for the top contributor), and the same stable year-over-year pattern. This consistency is itself evidence that both categories reflect the same underlying collections dynamics rather than being driven by unrelated causes.
- **No exclusion applied for the invoice_id duplicate collision** in this KPI, unlike KPI #4 (Late Payment Rate), which will require excluding those rows because it depends on a payments↔invoices join. This is a deliberate scoping decision, not an oversight — see KPI #2's Data Quality Notes for the full reasoning and the bias-check that confirmed exclusion is safe where it is needed.

## 6. Business Interpretation

Roughly **30% of all billed revenue (5,497 of 18,033 invoices) has not been fully collected** — either sitting entirely unpaid (10.19%) or partially collected with a balance still outstanding (20.30%). This pattern is not driven by a handful of problem customers, a single struggling branch, or a one-off bad year: it is evenly spread across all six branches, shows no meaningful customer concentration (the single largest contributor accounts for under 1% of all Unpaid invoices), and has held steady year over year from 2019 through 2024.

This is a structural, ongoing characteristic of how the business collects revenue — not a temporary anomaly. Combined with KPI #1's finding that the business is already carrying chronic excess inventory (average sell-through of only 5–6%), this paints a compounding risk picture: capital is simultaneously being over-committed to stock that isn't moving, while a meaningful share of billed revenue that could otherwise fund more disciplined purchasing decisions remains uncollected. Addressing either issue in isolation — tightening procurement without improving collections, or vice versa — would leave the business exposed on the other side.
