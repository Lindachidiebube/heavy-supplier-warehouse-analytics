# KPI 04: Late Payment Rate

**Sprint:** Week 2

## Business Question

How often do customers pay after the due date, and how long is the delay when they do?

This ties to the brief's goal of improving overall reliability and performance. It complements KPI 03 (Unpaid Invoice Rate): KPI 03 measures how much money is still outstanding, and KPI 04 measures how late the money arrives.

## Definition

**Late Payment Rate = late payments ÷ total matched payments**

- A payment is late when `payment_date > due_date`, where the due date comes from the matched invoice.
- A payment made exactly on the due date counts as on-time.
- The rate is per payment, not per invoice. Invoices with no payment are outside this KPI (they are covered by KPI 03).

**Supporting metric: Days Late.** Measured among late payments only (payment_date minus due_date, in days).

## Headline Results

| Metric | Value |
|---|---|
| Matched payments analysed | 18,445 |
| Late payments | 7,655 |
| **Late Payment Rate** | **41.50%** |
| On-time or early payments | 10,790 (271 paid on the due date + 10,519 paid early) |
| Days late (late payments only), mean | 20.02 days |
| Days late, median | 17 days |
| Days late, IQR | 9 to 28 days |
| Days late, min / max | 1 / 75 days |

## Methodology

1. Loaded `invoices.csv` (18,033 rows) and `payments.csv` (19,257 rows). Both match the previously verified row counts.
2. Per the KPI 02 scoping decision, excluded rows affected by duplicate IDs before joining, because a duplicate key could match a payment to the wrong invoice's due date:
   - 393 invoice rows (196 duplicated `invoice_id` values) removed, leaving `invoices_clean` = 17,640.
   - 403 payment rows (201 duplicated `payment_id` values) removed, leaving `payments_clean` = 18,854.
3. Orphan check: 0 of 19,257 payments have an `invoice_id` with no match in invoices (0.0%), so every payment is joinable.
4. Inner-joined `payments_clean` to `invoices_clean` on `invoice_id` (validated many-to-one). 409 clean payments pointed to excluded invoices and dropped out, giving **18,445 rows** (18,854 − 409).
5. Converted `payment_date` and `due_date` with `pd.to_datetime` (0 nulls) and defined `is_late = payment_date > due_date`.
6. Calculated the rate and, for late payments only, the days-late statistics.

Total excluded: 812 payments (403 + 409) = 4.22% of 19,257; 95.78% kept.

## Verification Log (8 checks)

| # | Check | Result |
|---|---|---|
| 1 | Null check | 0 nulls in `is_late`, `payment_date` and `due_date` after the merge. Pass. |
| 2 | Cross-method check | Late count and rate agree across independent methods (7,655 / 18,445; mean days late 20.02). Pass. |
| 3 | Row gap explained | 19,257 − 18,445 = 812 = 403 (duplicate payment_id) + 409 (payments pointing to excluded invoices). Fully accounted for. Pass. |
| 4 | Totals reconcile | 7,655 late + 10,790 on-time/early = 18,445. Pass. |
| 5 | Re-derived 3 ways | Rate 0.4150176199512063 and mean days late 20.018811234487263 identical across 3 derivations. Pass. |
| 6 | Distribution | See below. Pass, with caveats noted. |
| 7 | Date coverage | payment_date 2019-01-09 to 2025-03-20; due_date 2019-01-20 to 2025-03-14. Pass, with the 2025 caveat below. |
| 8 | Manual spot-check | 7 payments hand-checked against raw `payments.csv` and `invoices.csv`. All matched code output. Pass. |

**Check 6 detail: distribution**

- By branch (late rate): CHN001 48.61% (3,732 payments, median 19 days late), PUN001 43.15%, KOL001 39.74%, AHM001 39.61%, HYD001 38.20%, DEL001 37.72%. Median days late is 16 to 19 in every branch.
- By due year: 2019 to 2024 is stable at 40.2% to 43.1% (median 17 to 18 days late). 2025 is 30.46% (371 payments), explained below.
- Customers: 500 customers; the top customer accounts for 0.32% of payments, so there is no concentration.
- Pre-merge bias check of excluded vs full tables: see Data Quality Notes.

**Check 8 detail: spot-checked payments**

| Payment | Days vs due date | Counted as |
|---|---|---|
| PAY-324335 | +3 | Late |
| PAY-920424 | +19 | Late |
| PAY-706069 | +20 | Late |
| PAY-548776 | −8 | On-time (early) |
| PAY-967482 | −2 | On-time (early) |
| PAY-949408 | −28 | On-time (early) |
| PAY-352541 | 0 | On-time (paid on due date) |

## Supplementary Check: "Payment Before Invoice" (Sprint 1 finding)

Sprint 1 found 208 payments dated before their own invoice and recommended excluding them from date-sensitive analysis. This was tested against KPI 04:

- In the KPI 04 merged table (18,445 rows): payments before invoice = **0**, invoices with due date before invoice date = **0**. Removing them changes nothing: the late rate stays 41.50%.
- To explain the difference, the same test was run on the raw, unfiltered payments-to-invoices join. That join has 19,678 rows (421 more than the 19,257 payments, because duplicate invoice IDs match one payment to two invoices). It reproduces exactly **208** payments before invoice.
- **All 208** involve a duplicate `invoice_id` (only 3 involve a duplicate `payment_id`).

**Conclusion:** the 208 are not real timing errors. They are false matches, where a payment was joined to the wrong copy of a duplicated invoice ID. They are a symptom of the ID collisions found in KPI 02 and are already removed by the KPI 02 exclusion. Sprint 1 recommendation #3 is therefore resolved and needs no separate exclusion.

## Data Quality Notes

1. **Exclusion of duplicate-ID rows (4.22%).** 812 payments were excluded or dropped. A pre-merge bias check showed no meaningful skew in amount (invoice grand_total: full mean 1,622,248 vs excluded 1,665,026; payment_amount: full mean 1,210,411 vs excluded 1,267,410, with similar spread and quartiles). Branch and year show mild unevenness of about 2 to 3 percentage points: PUN001 is over-represented in the excluded set (18.07% vs 14.80%, +3.3pp), HYD001 under-represented (10.94% vs 13.74%, −2.8pp), and 2023 due dates under-represented (13.99% vs 16.63%, −2.6pp). This is not large enough to materially bias the headline, but it is stated here rather than claimed as zero bias.
2. **2025 is a partial year.** The late rate falls toward the data cutoff (last payment_date is 2025-03-20). By due month: Jan 35.56% (270 payments), Feb 18.18% (88), Mar 7.69% (13). This is consistent with a limited observation window, since late payments for recent invoices may not have arrived yet. The 41.50% headline is kept. **Sensitivity:** excluding 2025 gives **41.73%** (18,074 rows). The idea that 2025 simply has more unpaid invoices was tested and not supported (346 invoices in 2025: 248 Paid, 64 Partially Paid, 34 Unpaid, a similar unpaid share to the whole table).
3. **On-due-date convention.** 271 payments made exactly on the due date count as on-time. Using `>=` instead would give 7,926 late payments (42.97%). The stricter definition (`>`) is used because paying on the due date meets the payment term.
4. **Scope.** The KPI measures payments that exist. Invoices with no payment yet are not in the denominator (see KPI 03).
5. **Payment-before-invoice records:** none in the merged table; see the Supplementary Check above.

## Business Interpretation

About 4 in every 10 payments (41.50%) arrive after the due date, and when they are late they are typically about 2.5 weeks late (median 17 days, with the middle half of late payments between 9 and 28 days). This is stable from 2019 to 2024, so late payment is a persistent pattern and not a one-off problem.

The Chennai branch (CHN001) stands out at 48.61%, roughly 7 to 11 points above the other branches, which range from 37.72% to 43.15%. The other five branches are close to each other. Lateness is spread across customers with no concentration (top customer 0.32%), so it is a general habit and not the fault of a few accounts.

Read together with KPI 03 (30.49% of invoices still Unpaid or Partially Paid), this supports the brief's concern that decisions are reactive: money is both slow to arrive and often not fully collected. The findings point to a data-driven collection focus (for example, earlier reminders and closer follow-up on the branch with the highest late rate) instead of relying on intuition.
