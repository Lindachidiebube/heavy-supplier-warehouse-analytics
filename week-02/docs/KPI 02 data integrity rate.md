# KPI 02 — Data Integrity Rate (Invoice & Payment Keys)

## 1. KPI Definition

**Data Integrity Rate** measures how much of the `invoices` and `payments` data sits inside a **non-unique ID** — i.e., an `invoice_id` or `payment_id` value that has been assigned to more than one genuinely different record. It answers the question: *"How much of my data can I trust to join correctly on these ID columns?"*

Reported as **two separate metrics**, not combined:

| Metric | Formula | Result |
|---|---|---|
| **Invoice ID Integrity Rate** | Affected invoice rows ÷ Total invoice rows | 393 ÷ 18,033 = **2.18%** |
| **Payment ID Integrity Rate** | Affected payment rows ÷ Total payment rows | 403 ÷ 19,257 = **2.09%** |

**Why two metrics instead of one combined rate:** `invoice_id` and `payment_id` are independent identifiers that feed different downstream KPIs and may have different root causes. `invoice_id` issues threaten any KPI that reads `invoices.csv` alone (e.g. Unpaid Invoice Rate). `payment_id`/`invoice_id` issues *together* threaten any KPI that must join `payments` to `invoices` (e.g. Late Payment Rate). Combining the two into one blended percentage would let a clean column average out a messier one, hiding exactly the information a reader needs to act on.

**Why "affected-row rate" and not "duplicate-group rate":** a naive metric could count *how many IDs are broken* (duplicate-group rate) rather than *how much data is compromised* (affected-row rate). Group-rate treats a 2-row collision and a 3-row collision as equally "one incident," so it doesn't scale with real exposure. Affected-row rate scales correctly — a group of 3 contributes 3 to the numerator, not 1 — and is therefore the more honest reflection of how much of the dataset is actually exposed to join risk. Duplicate-group rate is retained as a supporting detail below, not the headline number.

## 2. Methodology

- Source tables: `invoices.csv` (18,033 rows, 10 columns) and `payments.csv` (19,257 rows, 5 columns)
- An ID is classified as "broken" if it appears on more than one row in its table
- All rows sharing a broken ID are counted as "affected" (not just the "extra" copies)
- **Scoping decision:** exclusion of affected rows applies **only to calculations that require a cross-table join between `payments` and `invoices`** (i.e. KPI #4 — Late Payment Rate). It does **not** apply blanket-wide to KPI #3 (Unpaid Invoice Rate), because `payment_status` already lives inside `invoices.csv` itself and does not require a join to `payments` to be computed.
- **Why a composite key was rejected as a fix:** a composite key (e.g. `invoice_id` + `so_id`) would make rows unique *within* `invoices`, but `payments.csv` has no shared disambiguating field (no `so_id`, no `customer_id`) — so the ambiguity cannot be resolved from the payments side of any join. With no shared field available to disambiguate, exclusion of the affected rows is the only defensible option for any calculation requiring the join.

## 3. Verification Log

| # | Check | Method | Result | Status |
|---|-------|--------|--------|--------|
| 1 | Null check | `.isnull().sum()` on `invoice_id` / `payment_id` | 0 nulls in both columns | ✅ Pass |
| 2 | Cross-method duplicate count | `duplicated(keep=False)` vs `groupby().size()` | Both methods agree: invoices 393 affected rows / 196 groups; payments 403 affected rows / 201 groups | ✅ Pass |
| 3 | Group-size distribution | `value_counts()` on group sizes | Invoices: 195 groups of 2 + 1 group of 3. Payments: 200 groups of 2 + 1 group of 3. No groups of 4+ found | ✅ Pass |
| 4 | Content-difference check | Manual inspection of all duplicate-ID rows, sorted by ID | Every duplicate pair/triplet shows genuinely different records — different customer_id, branch_id, so_id/invoice_id, dates, and amounts. Confirms true ID collision, not accidental exact-row duplication | ✅ Pass |
| 5 | Scatter/concentration check | `value_counts()` on branch_id (invoices) and payment_method (payments) for affected rows | Invoices: proportional spread across all 6 branches (85/74/71/67/53/43). Payments: even spread across all 5 methods (90/90/80/72/71). No single branch or method dominates | ✅ Pass |
| 6 | Temporal check | Year extracted from invoice_date/payment_date for affected rows | Even spread across 2019–2024 (~60–77 rows/year each table); 2025 lower on both (3 and 6 rows) consistent with the dataset's partial-year cutoff already established in KPI #1, not an anomaly | ✅ Pass |
| 7 | Denominator sanity check | Compare total row counts and affected-row counts to Week 1 findings | Total rows unchanged (18,033 / 19,257, confirmed via `.shape`). Affected-row counts unchanged (393 / 403, confirmed via Check #2's independent cross-method verification). No new computation needed — satisfied by evidence already gathered in earlier checks | ✅ Pass (verified via earlier checks) |
| 8 | Manual spot-check | Hand-inspection of the one triplet in each table (INV-341709, PAY-361416) | Both triplets confirmed to be three genuinely unrelated records each (different customers, branches, dates, amounts, payment methods) | ✅ Pass |

## 4. Data Quality Notes

1. **Root cause is systemic, not localized.** Checks #5 and #6 together rule out a single branch, payment method, or time period as the cause — the collisions are scattered proportionally across the entire 6-year dataset. This points to an ongoing weakness in how `invoice_id`/`payment_id` are generated or assigned, not a one-time incident or a single faulty system.

2. **Confirmed genuine collision, not duplication.** Check #4 (backed by the formal spot-check in #8) rules out the milder explanation — these are not the same row saved twice. Every affected group contains fully independent records that happen to share an ID. This is a more serious classification and should be described precisely as such in any downstream reporting, rather than as "duplicate rows."

3. **Bias check on exclusion (payment_status distribution):** before excluding affected rows from any join-dependent KPI, we tested whether the exclusion would disproportionately remove any one `payment_status` category. Result:

   | payment_status | Within excluded rows | Dataset-wide |
   |---|---|---|
   | Paid | 69.21% | 69.52% |
   | Partially Paid | 20.87% | 20.30% |
   | Unpaid | 9.92% | 10.19% |

   The distributions are nearly identical (largest gap: 0.57 percentage points). This confirms the exclusion is close to random with respect to payment status and will not systematically bias Unpaid Invoice Rate (KPI #3) or Late Payment Rate (KPI #4) in either direction.

4. **Composite key evaluated and rejected as a fix.** A composite key using `invoice_id` + `so_id` was considered to disambiguate collisions, but `payments.csv` contains no `so_id` (or any other shared field beyond `invoice_id`) to supply the other half of that key from the payments side. The ambiguity is therefore not resolvable through key engineering — it is a genuine gap in the source data, and exclusion (scoped only to join-dependent calculations) is the correct, defensible response.

## 5. Business Interpretation

`invoice_id` and `payment_id` cannot be relied upon as unique join keys anywhere in this dataset — 2.18% of invoices and 2.09% of payments are each involved in a genuine identifier collision, where the same ID has been assigned to multiple unrelated records. This is a systemic and ongoing data quality issue (confirmed present across all branches, all payment methods, and all six-plus years of history) rather than a single fixable event, and it is not something a composite key can resolve given the fields currently captured in `payments.csv`.

The practical implication for the rest of this project: **any KPI that requires matching a payment to its invoice must scope its exclusions to only the rows actually affected, and only where a join is genuinely required** — not applied as a blanket exclusion across every KPI. Because `payment_status` already lives inside `invoices.csv`, KPI #3 (Unpaid Invoice Rate) is only lightly touched by this issue. KPI #4 (Late Payment Rate), which must match a payment date against an invoice's due date across both tables, is the one most exposed and will apply the row exclusion directly, with the bias check above as documented justification that doing so does not distort the resulting rate.
