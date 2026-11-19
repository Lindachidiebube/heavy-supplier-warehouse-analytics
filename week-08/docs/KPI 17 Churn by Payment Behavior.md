# KPI 17 Churn by Payment Behavior

Week 8, KPI 17 | Heavy Supplier and Warehouse Analytics project | CadetX virtual internship | 18 Nov 2026

## Summary

**Question.** Do customers who pay badly stop ordering? Are the customers who go quiet and the customers who pay badly two different groups? Does payment behaviour warn that a customer is about to go quiet?

**Answer.** No link was found between payment behaviour and going quiet. Quiet customers do not pay later (45.53% late invoices against 44.90%, p 0.9290), do not leave more invoices unpaid (32.01% against 30.37%, p 0.2787) and do not owe more of their billing (21.45% against 20.23%, p 0.3554). Before they went quiet they did not pay worse either. Each result is "no difference found", which is not proof that the groups behave the same.

| Finding | Result |
|---|---|
| Quiet customers | 67 of 500 customers (13.4%, by hand) have been silent for more than twice their own normal gap at 1 Jan 2025. All 23 customers whose last order is on or before 30 Jun 2024 are inside the 67. |
| Late payment | Quiet customers paid 45.53% of their invoices late against 44.90% for the other 433 customers (p 0.9290). No difference found. |
| Unpaid invoices | 32.01% of the quiet customers' invoices are not fully paid against 30.37% (p 0.2787). No difference found. |
| Money still owed, as a share of own billing | Quiet customers owe 21.45% of their billing against 20.23% (p 0.3554). No difference found. |
| Money owed in amounts | The 67 quiet customers owe 804,728,429 (13.86% of all unpaid money). The 23 customers with no order since 30 Jun 2024 owe 266,055,010 (4.58%). |
| Owed share against chance | Random groups of 67 customers reach the quiet group's owed share in 8.3% of draws, and random groups of 23 reach the 23-group's share in 7.5%, so the gap is not proven. |
| Early warning | On invoices dated before 1 Jul 2024, quiet customers did not pay worse: late 45.35% against 44.69% (p 0.8945), not fully paid 31.96% against 30.41% (p 0.2292), owed 21.52% against 20.30% (p 0.3588). No warning found. |
| Bad payers and quiet customers | The share of quiet customers is 12.0% among the top 10% of late payers against 13.56% among the rest, 13.6% against 13.33% for the top 25%, and 13.6% against 13.2% for the top half (p 1.0 each). Bad payers are not more often quiet. |
| Due period | Quiet and other customers have the same due period mix (p 0.9511). The late share depends strongly on the due period, from 78.57% for 15-day invoices to 7.16% for 60-day invoices (p below 0.0001). |
| Silence | Every one of the 500 customers has its last order in 2024 and none has a last order before 2024. 25 customers have been silent for more than 180 days, and the longest silence is 359 days. |
| Robustness | Moving the quiet line to 1.5 or 2.5, removing the borderline customer, removing each customer's largest invoice and removing the 136 invoices dated after 20 Dec 2024 gave no p value below the strict line in any variant. |
| Verification | All 8 checks agree with the tables. No problem was found. |

## 1 KPI Definition

| Item | Definition | Value |
|---|---|---|
| Snapshot date | The day silence is measured to. It is one day after the last order in the data. | 1 Jan 2025 |
| Normal gap | Days between a customer's first and last delivered order, divided by its orders minus 1. | per customer |
| Recency | Days from the customer's last delivered order to the snapshot date. | per customer |
| Silence ratio | Recency divided by the normal gap, never rounded before dividing. | per customer |
| Quiet customer | A customer whose silence ratio is above 2. The line of 2 is a convention. | 67 customers |
| Group of 23 | Customers whose last order is on or before 30 Jun 2024. | 23 customers |
| Late invoice rate | Invoices with at least one payment after the due date, divided by invoices with a payment, as a percentage. | per customer |
| Not fully paid share | Invoices whose payment status is not Paid, divided by all invoices, as a percentage. | per customer |
| Outstanding share of billing | Billed minus paid, divided by billed, as a percentage. | per customer |
| Money basis | Invoice grand total, which includes GST. | all money figures |

**Why this KPI.** The brief asks which customers are at risk. None of our four tables has a field that says a customer has left: the customers table has 15 columns, orders 11, invoices 10 and payments 5, and the only status columns, order_status and payment_status, describe an order and an invoice. Churn is therefore measured by silence: a customer is quiet when it has been silent for more than twice its own normal gap. Silence is a stand-in for leaving and is not proof of it. The KPI asks whether late or unpaid invoices warn that a customer is slipping away. A tested answer of "no link" is still a decision answer: it tells the business not to use lateness as a churn signal.

## 2 Methodology

**Sources.**

| Table | Rows | Used for |
|---|---|---|
| so_h, Delivered orders | 20,000 rows, 18,033 delivered | order dates and the silence table |
| customers | 500 | the customer list |
| invoices | 18,033 | invoice dates, due dates, amounts |
| payments | 19,257 | payment dates and amounts |
| clean_invoices | 17,640 | invoices whose invoice_id is not repeated (18,033 minus 17,640 leaves 393 invoices out, by hand) |
| payments_used | 18,840 | payments whose invoice_id is in clean_invoices (19,257 minus 18,840 leaves 417 payment rows out, by hand) |

All 500 customers have at least one clean invoice.

**Reused from earlier KPIs.**

| Earlier KPI | What KPI 17 reuses |
|---|---|
| KPI 4 | the late rule: a payment dated after the due date |
| KPI 14 | the snapshot date and the idea of a customer's own normal gap; KPI 14 reported 67 quiet customers |
| KPI 15 | each customer's last order date |
| KPI 16 | the clean_invoices and payments_used tables and the money basis (grand total including GST) |

**Steps.**

- Build the silence table from delivered orders: orders, first and last order, recency, normal gap, silence ratio, quiet flag.
- Build the invoice table (inv) and the customer table (cust) from clean_invoices and payments_used: billed, paid, late invoices, not fully paid invoices, outstanding share.
- Compare quiet customers with the others on three measures (Mann-Whitney, strict line 0.0167 for three comparisons).
- Compare the money owed by group and test the owed share against random groups of the same size (shuffle test).
- Repeat the comparison on invoices dated before 1 Jul 2024 only (early warning test).
- Test whether bad payers are more often quiet (Fisher exact test).
- Test the due period mix and the effect of the due period on the late share (chi-square).
- Test the silence length, the end of the data and the choice of the quiet line (see the Verification Log).

**Choices made and alternatives.**

- The silence ratio uses the exact gap. The first run rounded the gap to 2 decimals before dividing and found 66 quiet customers instead of 67, because it lost C0299 (recency 124, rounded gap 61.97, exact ratio 2.00098). The exact ratio is used from then on.
- The line of 2 is a convention. The lines 1.5 and 2.5 are shown in the sensitivity results (see Check 6).
- Money is measured on the grand total including GST, as in KPI 16.
- Late means a payment after the due date, as in KPI 4.
- Mann-Whitney compares two groups without assuming a bell shape, so it is used for the three payment measures. Both mean and median are printed.
- Where several comparisons are made, the strict line is 0.05 divided by the number of comparisons: 0.0167 for three, 0.0083 for six and 0.025 for two.

## 3 Verification Log

| # | Check | Method | Result |
|---|---|---|---|
| 1 | Blank, zero and negative values | counted in silence, inv and cust | No blanks and no negative values. The zeros are explained: 1,798 invoices with nothing paid, and 16 customers with no late invoice. |
| 2 | Edge cases | customers with one order, ratio exactly at 2, customers near the line, zero-paid invoices | No customer has one order (fewest 21). No customer sits exactly on 2. The 1,798 zero-paid invoices are exactly the invoices with no payment. |
| 3 | Duplicates | repeated customer ids and invoice ids, payments pointing outside the kept invoices | 0 repeats and 0 payments outside, in every table. |
| 4 | Totals reconcile | counts and money against KPI 16 | 500 = 67 + 433, all 23 inside the 67, billed, paid and outstanding equal the KPI 16 totals with difference 0. |
| 5 | Second method | plain Python loop for the ratio, quiet flag and payment measures | Same 500 customers, same 67 quiet, 0 flag differences. Money differences are rounding traces of about 4.5e-08. |
| 6 | Outliers and sensitivity | quiet line moved, borderline customer removed, largest invoices removed | No p value below the strict line in 15 comparisons. Smallest p 0.0766. |
| 7 | History and dates | data end, snapshot moved later, unpaid share by invoice month | Snapshot sits one day after the last order. Later snapshots give larger quiet counts. |
| 8 | Hand check of 3 customers | C0299, C0002, C0029 from the raw files | Orders, gaps, ratios, invoices, billed, paid and late counts match the tables for all three. |

### Check 1 Blank, zero and negative values

- silence 500 rows, inv 17,640 rows, cust 500 rows: no blank value in any column.
- silence has no zero and no negative value. Every customer has orders, recency, normal gap and ratio above 0.
- inv: grand_total has no zero and no negative. paid_amount has 1,798 zeros and no negative.
- cust has no negative. late_invoices and late_rate each have 16 zeros: 16 customers have no late invoice. Every customer has at least one invoice with a payment, so the late rate is defined for all 500.

### Check 2 Edge cases

- Customers with one order: 0. The fewest orders by any customer is 21.
- Customers with a ratio of exactly 2: 0. Customers with a ratio between 1.99 and 2.01: C0083 (1.99694, below 2, not quiet), C0299 (2.00098, quiet) and C0417 (2.00810, quiet).
- Invoices with zero paid amount: 1,798. Invoices with no payment recorded: 1,798. Both at once: 1,798. Payment status of all of them: Unpaid. A paid amount of 0 therefore means no payment was made.
- Rounding test: with exact numbers 67 customers are above 2, with rounded numbers 66.

### Check 3 Duplicates

- Repeated customer ids: 0 in customers, silence, cust and by_cust (500 rows each).
- Repeated invoice ids: 0 in clean_invoices and inv (17,640 rows each).
- Payment rows pointing to an invoice that was not kept: 0.
- Invoices in inv but not in clean_invoices: 0. Invoices in clean_invoices but not in inv: 0.

### Check 4 Totals reconcile

| Item | Value |
|---|---|
| Customers | 500 |
| Quiet plus others | 67 + 433 = 500 |
| Last order on or before 30 Jun 2024 | 23, all 23 inside the quiet group |
| Billed, from invoices and from the customer table | 28,599,637,860.60 and 28,599,637,860.60 |
| Paid, from invoices and from the customer table | 22,795,333,903.87 and 22,795,333,903.87 |
| Difference to the KPI 16 totals | billed 0.0, paid 0.0 |
| Outstanding, quiet | 804,728,429 |
| Outstanding, others | 4,999,575,528 |
| Outstanding, together | 5,804,303,957 (804,728,429 + 4,999,575,528, by hand), difference to KPI 16: 0 |

### Check 5 Second method

- A plain Python loop over the order dates found the same 500 customers and 67 quiet customers. The quiet flag differs from the table for 0 customers and the largest ratio difference is 0.0.
- A plain loop over the invoice rows gave the same billed and paid per customer (largest differences 4.47e-08), the same not fully paid share (largest difference 0.0) and the same outstanding share (largest difference 4.97e-14). These are rounding traces far below any practical size.
- The late flag is not redone in this check. It is checked by hand against the raw due dates and payment dates in Check 8.

### Check 6 Outliers and sensitivity

Strict line 0.0167. Mean of the quiet group against the others, with the p value.

| Variant | Quiet / others | Late rate | Not fully paid | Outstanding share |
|---|---|---|---|---|
| Line 2 (normal) | 67 / 433 | 45.53 vs 44.90, p 0.9290 | 32.01 vs 30.37, p 0.2787 | 21.45 vs 20.23, p 0.3554 |
| Line 1.5 | 119 / 381 | 44.50 vs 45.14, p 0.6963 | 31.83 vs 30.20, p 0.0766 | 21.45 vs 20.06, p 0.1012 |
| Line 2.5 | 42 / 458 | 46.44 vs 44.85, p 0.6879 | 32.23 vs 30.44, p 0.2023 | 21.49 vs 20.29, p 0.3258 |
| Line 2 without C0299 | 66 / 433 | 45.41 vs 44.90, p 0.9441 | 32.08 vs 30.37, p 0.2508 | 21.60 vs 20.23, p 0.2773 |
| Line 2 without each customer's largest invoice | 67 / 433 | 45.51 vs 44.83, p 0.9233 | 31.98 vs 30.44, p 0.3022 | 21.29 vs 20.47, p 0.4362 |

- The line 2 row gives the same p values as the main comparison.
- The smallest p value in all 15 comparisons is 0.0766, and none is below 0.0167.
- The borderline customer C0299 does not drive the result, and neither do the customers' largest invoices.

### Check 7 History and dates

- Last order 31 Dec 2024. Last invoice 13 Jan 2025. Last payment 20 Mar 2025. Snapshot used 1 Jan 2025.
- Quiet customers by snapshot date: 42 at 1 Dec 2024, 67 at 1 Jan 2025, 125 at 1 Feb 2025, 201 at 1 Mar 2025, 494 at 1 Jun 2025. The count rises with every later snapshot.
- Unpaid share by invoice month: 7.17% to 12.20% from Feb to Nov 2024, 11.35% in Dec 2024 and 4.26% in Jan 2025 (47 invoices). Not fully paid share: 23.56% to 35.22% from Feb to Nov 2024, 28.82% in Dec 2024 and 25.53% in Jan 2025.
- The end of the data was tested directly (see Supporting tests, End of the data).

### Check 8 Hand check of 3 customers

| Item | C0299 (borderline) | C0002 (quiet) | C0029 (one of the 23) |
|---|---|---|---|
| Delivered orders | 34 | 51 | 38 |
| First and last order | 2019-01-24 and 2024-08-30 | 2019-01-24 and 2024-09-07 | 2019-01-16 and 2024-06-30 |
| Days between first and last order | 2,045 | 2,053 | 1,992 |
| Normal gap, by hand | 2,045 / 33 = 61.9697 | 2,053 / 50 = 41.06 | 1,992 / 37 = 53.8378 |
| Recency (days to 1 Jan 2025) | 124 | 116 | 185 |
| Silence ratio, by hand | 124 / 61.9697 = 2.00098 | 116 / 41.06 = 2.825 | 185 / 53.8378 = 3.436 |
| Invoices | 33 | 49 | 37 |
| Billed | 43,979,214.80 | 69,350,700.20 | 57,386,622.80 |
| Paid | 38,898,802.15 | 52,300,739.72 | 40,539,611.00 |
| Still owed, by hand | 5,080,412.65 (11.55% of billing) | 17,049,960.48 (24.59% of billing) | 16,847,011.80 (29.36% of billing) |
| Invoices with a payment | 32 | 45 | 32 |
| Invoices with a payment after the due date | 17 | 29 | 11 |
| Late rate, by hand | 17 / 32 = 53.12% (53.125) | 29 / 45 = 64.44% | 11 / 32 = 34.38% (34.375) |

The raw files and the tables agree for all three customers. C0029 has its last order exactly on 30 Jun 2024, so the group of 23 means a last order on or before 30 Jun 2024.

### Supporting tests

| Test | What it checks | Result |
|---|---|---|
| Rounding test | whether rounding the gap changes the quiet count | 67 with exact numbers, 66 with rounded numbers. C0299 is the customer lost by rounding. |
| Status columns | whether any table has a field that says a customer has left (column names searched for status, active, inactive, churn, left, closed, end, terminated, stop, flag) | Customers (15 columns): no match. Payments (5 columns): no match. Orders (11 columns): order_status only, Delivered 18,033 and Cancelled 1,967 (20,000 together, by hand). Invoices (10 columns): payment_status only, Paid 12,536, Partially Paid 3,660 and Unpaid 1,837 (18,033 together, by hand). The two matches describe an order and an invoice, not a customer. |
| Due period mix | whether quiet and other customers have the same mix of due periods | Customers with more than one due period: 0. Quiet customers 13, 25, 14, 15 and others 74, 168, 99, 92 for 15, 30, 45 and 60 days (column totals 87, 193, 113, 107, by hand). Chi-square 0.346, 3 degrees of freedom, p 0.9511. Smallest expected count 11.66 (67 x 87 / 500, by hand). The mix is the same. |
| Late share by due period | whether the late share depends on the due period | Invoices 3,094, 6,858, 3,967, 3,721 (17,640 together, by hand). Invoices with a payment 2,772, 6,195, 3,549, 3,326 (15,842 together, by hand). Late invoices 2,178, 3,556, 1,208, 238 (7,180 together, by hand). Late share 78.57%, 57.40%, 34.04%, 7.16%. Chi-square 3738.787, 3 degrees of freedom, p below 0.0001. Smallest expected count 1256.34. |
| Payment behaviour | quiet 67 against others 433 on three measures (Mann-Whitney, strict line 0.0167) | Late rate: mean 45.53 vs 44.90, median 50.00 vs 48.65, p 0.9290. Not fully paid: mean 32.01 vs 30.37, median 31.58 vs 29.73, p 0.2787. Outstanding share: mean 21.45 vs 20.23, median 19.74 vs 19.61, p 0.3554. No difference found. |
| Money owed by group | amounts owed and the three measures for the 23 against the other 477 | Quiet 67: 13.12% of all billed, 804,728,429 owed (13.86% of all unpaid money), 21.45% of own billing. Last order on or before 30 Jun 2024, 23: 4.13% of billed, 266,055,010 owed (4.58%), 22.53% of own billing. All other 433: 86.88% of billed, 4,999,575,528 owed (86.14%), 20.12%. The 23 against the other 477: late rate 45.85 vs 44.94 (medians 51.85 vs 48.65), p 0.9299; not fully paid 32.65 vs 30.49 (medians 34.21 vs 29.73), p 0.1740; outstanding share 22.12 vs 20.31 (medians 19.96 vs 19.61), p 0.2257. No difference found. |
| Shuffle test | whether the owed share of the quiet groups is higher than chance (5,000 random groups from all 500 customers, seed 42, line 250 of 5,000) | Quiet 67: real 21.45, random average 20.29, lowest 17.15, highest 23.29, random groups as high as the real value 414 of 5,000 (8.3%, by hand). The 23: real 22.53, random average 20.29, lowest 15.1, highest 26.0, as high as real 374 of 5,000 (7.5%, by hand). Both are above the line of 250, so the gap is not proven. |
| Early warning | payment behaviour before going quiet (invoices dated before 1 Jul 2024: 16,106 invoices, 500 customers, strict line 0.0083) | Quiet 67 against others: late rate 45.35 vs 44.69 (medians 50.0 vs 48.39), p 0.8945; not fully paid 31.96 vs 30.41 (medians 32.14 vs 30.0), p 0.2292; outstanding share 21.52 vs 20.30 (medians 19.9 vs 20.0), p 0.3588. The 23 against the other 477: late rate 45.84 vs 44.73, p 0.8854; not fully paid 32.59 vs 30.52 (medians 33.33 vs 30.0), p 0.1879; outstanding share 22.12 vs 20.38 (medians 20.68 vs 19.94), p 0.2201. No warning found. |
| Overlap of bad payers and quiet customers | whether bad payers are more often quiet (Fisher exact, strict line 0.0167, quiet share overall 13.4%) | Top 10% by late rate (late rate 77.8 or above): 50 customers, 6 quiet (12.0%), against 61 of 450 (13.56%), p 1.0. Top 25% (64.33 or above): 125 customers, 17 quiet (13.6%), against 50 of 375 (13.33%), p 1.0. Top 50% (49.32 or above): 250 customers, 34 quiet (13.6%), against 33 of 250 (13.2%), p 1.0. |
| Last order dates and silence length | whether any customer has left before 2024 | Earliest last order 2024-01-08, latest 2024-12-31. 500 of 500 customers have their last order in 2024 and 0 before 2024. Ordered within 30 days of the snapshot: 167 (33.4%), within 90 days: 385 (77.0%), within 120 days: 433 (86.6%), within 180 days: 475 (95.0%). Longest silence 359 days. Customers silent for more than 180 days: 25. |
| End of the data, recent invoices | whether invoices from the last 60 days of invoicing (from 14 Nov 2024) look different (Fisher exact, strict line 0.025) | Recent invoices 427, earlier 17,213. Unpaid share 9.37% vs 10.21%, p 0.6272. Late share among invoices with a payment 45.99% vs 45.31%, p 0.7962. Days from invoice to payment (18,840 payments): median 32, 95th percentile 66, 99th percentile 85, longest 90. Payments that came more than 66 days after the invoice: 918 (4.87%). The last invoice had 66 days to be paid (13 Jan to 20 Mar 2025, by hand). |
| End of the data, invoices after 20 Dec 2024 removed | whether the invoices with less than 90 days to be paid drive the comparison (strict line 0.0167) | 136 invoices removed (0.77%), 17,504 kept. Quiet 67 and others 433. Late rate 45.53 vs 44.94, p 0.9359. Not fully paid 32.01 vs 30.38, p 0.2836. Outstanding share 21.45 vs 20.26, p 0.3578. The full-data results were p 0.9290, 0.2787 and 0.3554, so the comparison is unchanged. |

## 4 Data Quality Notes

1. No table has a field that says a customer has left (customers 15 columns, orders 11, invoices 10, payments 5). The only status columns are order_status (Delivered 18,033 and Cancelled 1,967) and payment_status (Paid 12,536, Partially Paid 3,660 and Unpaid 1,837), and they describe an order and an invoice. Quiet means silent for more than twice the customer's own normal gap, and silence is a stand-in for leaving, not proof of it.
2. The silence ratio must not be rounded before the count. Rounding the gap to 2 decimals gave 66 quiet customers instead of 67.
3. The line of 2 is a convention. Three customers sit within 0.01 of it: C0083 (1.99694), C0299 (2.00098) and C0417 (2.00810). With the line at 1.5 there are 119 quiet customers and at 2.5 there are 42.
4. The money figures use clean_invoices (17,640 of 18,033 invoices, 393 left out, by hand) and payments_used (18,840 of 19,257 payment rows, 417 left out, by hand), as in KPI 16. Money includes GST.
5. 1,798 invoices have no payment and the status Unpaid. They count as invoices with nothing paid. The late rate is worked out only over invoices with a payment, and all 500 customers have at least one.
6. 16 customers have no late invoice, so their late rate is 0.
7. The data ends on 31 Dec 2024 for orders, 13 Jan 2025 for invoices and 20 Mar 2025 for payments. The snapshot is set one day after the last order. The quiet count rises with later snapshots (42, 67, 125, 201, 494).
8. The two quiet groups are small (67 and 23 customers). A result of no difference found does not prove that the groups behave the same.
9. The money gap in the owed share (21.45% against 20.12% for the others, 22.53% for the 23) was tested against random groups and is not proven (8.3% and 7.5% of random groups reach it).
10. Money differences of 4.47e-08 between the pandas tables and the plain loop are rounding traces and do not change any figure shown.
11. A pandas warning about fillna downcasting appeared when the customer table was built. All totals tied out to KPI 16 (17,640 invoices, billed 28,599,637,860.60, paid 22,795,333,903.87).
12. Numbers marked by hand are arithmetic on printed counts. Every other number comes from a notebook cell.

## 5 Business Interpretation

- Payment behaviour is not a churn signal here. Quiet customers pay no later, leave no more invoices unpaid and owe no more of their billing than the other customers, and the same holds for invoices dated before they went quiet. Treat quiet customers and slow payers as separate follow-up lists.
- Bad payers are not more often quiet: the quiet share is between 12.0% and 13.6% whichever cut-off defines a bad payer.
- Lateness follows the due period. The late share is 78.57% on 15-day invoices and 7.16% on 60-day invoices, and quiet and other customers have the same due period mix, so their late rates can be compared fairly.
- The money owed by quiet customers is large in amount (804,728,429, which is 13.86% of all unpaid money, and 266,055,010 for the 23), but it is plain money owed. The higher owed share of quiet customers is not proven.
- No customer has a last order before 2024, but 25 customers have been silent for more than 180 days and the longest silence is 359 days. These are the first customers to contact.
- The quiet list depends on the snapshot date and on the line of 2. It should be rebuilt at the data end with the exact ratio.

### Recommended actions for the business owners

1. Head of Sales: contact the 25 customers silent for more than 180 days first, then the rest of the 67 quiet customers, to find out whether they still buy from the business. Silence is the only signal in the data, and payment behaviour does not add to it.
2. Credit Control Manager: do not use late or unpaid invoices as a churn warning, and keep chasing late payment by due period, since the late share falls from 78.57% for 15-day invoices to 7.16% for 60-day invoices.
3. Finance Controller: track the money owed by quiet customers as plain money owed (804,728,429 by the 67 quiet customers and 266,055,010 by the 23), without treating it as proof that they are leaving.
4. Customer Account Managers: review the borderline customers (C0083, C0299 and C0417) personally, since a different line would change which of them are called quiet.
5. Reporting Lead: rebuild the quiet list at each data end using the exact silence ratio and a snapshot one day after the last order, because later snapshots give larger counts (125 at 1 Feb 2025 and 494 at 1 Jun 2025).

## Glossary

| Label | Meaning |
|---|---|
| Quiet customer | a customer whose silence ratio is above 2 (67 customers) |
| Group of 23 | customers whose last order is on or before 30 Jun 2024 |
| Snapshot | the date silence is measured to, 1 Jan 2025 |
| Recency | days from a customer's last delivered order to the snapshot |
| Normal gap | days between a customer's first and last delivered order divided by its orders minus 1 |
| Silence ratio | recency divided by the normal gap |
| ratio_exact | the silence ratio without any rounding |
| Late invoice | an invoice with at least one payment dated after its due date (KPI 4 rule) |
| Late invoice rate | late invoices divided by invoices with a payment, as a percentage |
| Not fully paid share | invoices whose payment status is not Paid, divided by all invoices, as a percentage |
| Outstanding share | billed minus paid, divided by billed, as a percentage |
| Outstanding money | billed minus paid |
| Bad payer | a customer in the top 10%, 25% or 50% by late invoice rate (used in the overlap test) |
| Due period | days between invoice date and due date: 15, 30, 45 or 60 |
| GST | the tax included in the invoice grand total |
| p value | the chance of a gap at least this large if the groups were really the same |
| Strict line | the p value a result must fall below: 0.05 divided by the number of comparisons (0.0167, 0.0083, 0.025) |
| Mann-Whitney | a test that compares two groups on a number without assuming a bell shape |
| Fisher exact | a test on counts in a 2 by 2 table |
| Chi-square | a test of whether counts in groups differ from what the group sizes would give |
| Shuffle test | repeating the comparison on 5,000 random groups of the same size to see how often chance reaches the real value |
| No difference found | the test did not find a gap; this is not proof that the groups behave the same |
| Early warning test | the payment comparison on invoices dated before 1 Jul 2024 only |
| Data end | 31 Dec 2024 for orders, 13 Jan 2025 for invoices, 20 Mar 2025 for payments |
| Recent invoices | invoices dated from 14 Nov 2024, the last 60 days of invoicing |
| Rounding trace | a difference of about 0.00000004 caused by how computers store decimals |
| by hand | arithmetic done on printed counts, not printed by a notebook cell |
| silence | notebook table with one row per customer: orders, first and last order, recency, normal gap, ratio, quiet flag |
| inv | notebook table with one row per clean invoice: customer, grand total, payment status, paid amount, late flag, has-payment flag, not-fully-paid flag |
| cust | notebook table with one row per customer: invoices, invoices with a payment, late invoices, not fully paid, billed, paid, late rate, not fully paid share, outstanding share, quiet flag |
| by_cust | notebook table like cust, built from invoices dated before 1 Jul 2024 only |
| dated | notebook table inv with the invoice date added |
| clean_invoices | invoices whose invoice_id is not repeated (17,640) |
| payments_used | payments whose invoice_id is in clean_invoices (18,840) |
| order_status | field of the orders table: Delivered or Cancelled |
| payment_status | field of the invoices table: Paid, Partially Paid or Unpaid |
