# KPI 16 Customer Payment Risk

**Project:** Heavy Supplier & Warehouse Analytics (CadetX virtual internship)
**Week:** 8 (Phase 3, Customer analytics)
**Notebook:** Week8_KPI_16_Customer_Payment_Risk
**Data used:** invoices.csv, payments.csv, customers.csv, sales_orders_header.csv (invoices from 3 Jan 2019 to 13 Jan 2025, payments from 9 Jan 2019 to 20 Mar 2025)
**Money basis:** grand total including GST (payments are made on this basis). Money before GST is shown only to tie back to KPI 15.

---

## Summary

**Question answered:** which valuable customers put our cash flow at risk, and how much money is involved?

**Short answer:** no customer can be named as a bad payer. The cash risk is real, but it is a steady, company-wide leak of about one fifth of the billed money that has not been paid. The share of invoices not fully paid does not differ between due periods, customer types, branches, regions or industries beyond chance, and the concentration of unpaid money in the biggest balances is what customer size and chance produce. The most valuable customers are not riskier than the rest.

- **Billed, paid, outstanding.** Clean invoices total 28,599,637,860.60 billed, 22,795,333,903.87 paid and 5,804,303,956.73 outstanding. That is about 20.3% of the billed money.
- **How many invoices.** 5,376 of 17,640 clean invoices (30.48%) are not fully paid: 1,798 Unpaid and 3,578 Partially Paid.
- **Steady every year.** From invoice year 2019 to 2024 the outstanding share stays between 19.17% and 21.52% of billing, and the share of invoices not fully paid stays between 29.7% and 31.8%. The share of invoices not fully paid does not differ between the six invoice years (chance test p 0.61) and shows no trend up or down (p 0.64).
- **The unpaid money is old.** No payment in the data ever arrived more than 90 days after its invoice. 99.34% of the unpaid money is older than 90 days and 88.01% is older than a year. It is not slow money, it is money that has not come in during the whole history.
- **Lateness follows the due period, not the customer.** The share of late invoices is 78.57% for 15-day due periods, 57.40% for 30 days, 34.04% for 45 days and 7.16% for 60 days (the four rates differ far beyond chance, p below 0.0001). Customers pay at the same pace whatever the deadline: the middle is 34 to 36 days, anywhere from 0 to 90 days, and the four due periods do not differ (p 0.39; no pair is below the strict line 0.0083), so a short due period brings cash in no faster. Inside a due period group, customers do not differ beyond luck.
- **Valuable customers are not riskier.** The 100 most valuable customers (KPI 15 ranking) hold 26.03% of the outstanding money against 26.74% of the billing. Value rank and outstanding share are unrelated (correlation 0.042, p 0.3444).
- **A call list exists.** The 20 valuable customers with the biggest balances hold 461,987,109.14, which is 7.96% of all outstanding money. They are on the list because their balances are big, not because they are shown to be bad payers.
- **No group stands out.** Across 27 groups (due period, customer type, branch, region, industry), the share of invoices not fully paid does not differ from the overall 30.48% in any group. The smallest p-value is 0.0944 and none is below 0.05. Outstanding money is between 18.58% and 22.13% of billing in every group.
- **Verification.** All 8 checks were run and agree (see Verification Log).

---

## 1. KPI Definition

| Item | Definition |
|---|---|
| Business question | Which valuable customers put cash flow at risk, and how much money is involved? |
| Billed | Sum of grand_total over clean invoices (invoice IDs that appear once). Includes GST. |
| Paid | Sum of payment_amount over the payments used (every payment that does not point to a repeated invoice ID). |
| Outstanding | Billed minus paid, per invoice, customer or group. |
| Outstanding share of billed money | Outstanding divided by billed, as a percentage. Always written "of billed money". |
| Not fully paid invoice | An invoice with payment status Unpaid or Partially Paid. Shares of these are always written "of invoices". |
| Late payment | A payment dated after the invoice due date (same rule as KPI 4). |
| Late invoice | An invoice whose last payment is dated after its due date. |
| Late invoice rate | Late invoices divided by invoices that have at least one payment. |
| Due period | Due date minus invoice date, in days (15, 30, 45 or 60). Each customer has exactly one. |
| Open invoice | An invoice with more than 1 of money still outstanding. |
| Ageing | Days from the invoice date to today (21 Mar 2025, the day after the latest date in the data). |
| Valuable customer | A customer in the top 100 of the KPI 15 ranking (money before GST on Delivered orders). A second rule (top 100 by billed money) is used as a check. |
| Priority list | The 20 valuable customers with the most outstanding money. |

**Why this KPI and not another.** KPI 3 counts invoices by status and KPI 4 measures how late payments are. Neither says how much money is stuck or which customers hold it. KPI 16 measures the money and ties it to customer value from KPI 15. The customers table columns credit_limit and current_balance were tested and not used, because they do not match the transactions. The payment_terms column does match the due period on the invoices, but the due period is taken from the invoices themselves, because it comes from the transaction (see Data Quality Notes).

**Important difference in measures.** KPI 3 reports Unpaid 10.19% and Partially Paid 20.30% **of invoices** (all 18,033 invoice rows). KPI 16 reports 20.3% **of billed money** outstanding. The two 20.3 figures measure different things and must not be mixed up.

---

## 2. Methodology

### Sources

| Table | Rows | Used for |
|---|---|---|
| invoices.csv | 18,033 rows, 10 columns | Billed money, due dates, status, customer |
| payments.csv | 19,257 rows, 5 columns | Paid money, payment dates |
| customers.csv | 500 rows, 15 columns | Customer type, branch, region, industry segment; credit_limit, current_balance and payment_terms (tested only) |
| sales_orders_header.csv | 20,000 rows, 11 columns | Delivered order value before GST (KPI 15 money and rank); order payment terms (Advance against Net 30 test) |

### What is reused and what is new

| Reused from | What is reused | What is new in KPI 16 |
|---|---|---|
| KPI 3 | Status classes Paid, Partially Paid, Unpaid (rates of invoices for comparison) | Outstanding money, split by status and customer |
| KPI 4 | Late rule (payment date after due date), clean payment set, late rate 41.50% | Late rate checked with the 395 added-back payments (41.51%), late invoice rate, due period, days to pay |
| KPI 14 | Finding that customer differences equal chance (segments were not rebuilt, see Alternatives considered) | Chance tests on lateness and unpaid share |
| KPI 15 | Customer money before GST and value rank (22,958,987,430 over 500 customers) | Valuable versus others, priority list, outstanding by group, ageing, by invoice year |

### Steps

| Step | What was done | Result |
|---|---|---|
| Keys | Counted repeated invoice and payment IDs and orphans | 393 invoice rows with a repeated ID; 403 payment rows with a repeated ID; 417 payments point to a repeated invoice ID; 0 orphans |
| Clean tables | Kept invoices whose ID appears once | 17,640 clean invoices (Paid 12,264, Partially Paid 3,578, Unpaid 1,798) |
| Money rebuild | Used every payment that does not point to a repeated invoice ID | 18,840 payments used; all four money tests 0 (see Check 4 and Data Quality Notes) |
| Late flag | Payment date after due date | 7,820 late of 18,840 (41.51%) |
| Per invoice | Payments, last payment date, last payment late | 15,842 invoices with a payment, 7,180 with a late last payment (45.32%) |
| Per customer | Invoices, billed, paid, outstanding, status counts, late counts | 500 customers; totals tie to the invoice and payment tables |
| Link to KPI 15 | Joined money before GST and value rank | 22,958,987,430 over 500 customers, difference 0 |
| Chance tests | Compared customers' spread of late rate and unpaid share with the spread luck alone gives | See Business Interpretation |
| Late-rate bands | Customers grouped by late invoice rate | See Business Interpretation |
| Due period | Late rate by due period, chance test inside each due period | See Business Interpretation |
| Days to pay | Days from invoice date to last payment by due period | See Business Interpretation |
| Concentration | Share of outstanding held by top 10, 50, 100 customers | See Business Interpretation |
| Valuable versus others | Top 100 valuable (KPI 15 rank) against the other 400, rank correlation | See Business Interpretation |
| Priority list | 20 valuable customers with the most outstanding, second rule as a check | See Business Interpretation |
| Groups | Outstanding by due period, customer type, branch, region, industry (27 groups) | See Business Interpretation |
| Ageing | Open invoices by age band | See Business Interpretation |
| By invoice year | Billed and outstanding by invoice year | See Business Interpretation |

### Alternatives considered

| Alternative | Decision and reason |
|---|---|
| Use all 19,257 payments | Rejected. 417 payments point to a repeated invoice ID, so their invoice is ambiguous. |
| Use the KPI 4 payment set (18,445 payments) for money | Rejected. It leaves out 395 payments that point to a clean invoice, which gave 306 Paid invoices with money outstanding and 88 odd Partially Paid invoices. With the 395 added back, both counts are 0. |
| Use the customers table columns credit_limit and current_balance | Rejected. Tested: they do not match the transactions (see Data Quality Notes). |
| Use the customers table column payment_terms for the due period | Not needed. It matches the invoice due period for all 500 customers and all 17,640 clean invoices, so the due period is taken from the invoices themselves. |
| Rebuild the KPI 14 segments inside KPI 16 | Not done. KPI 14 already found its segments do not predict. KPI 16 tests payment behaviour directly by customer, due period, customer type, branch, region and industry. |
| Rank customers by late rate to find bad payers | Tested and rejected. The differences are explained by the due period (see Business Interpretation). |
| Define valuable by KPI 16 billed money | Used as a second rule only. 91 of 100 customers overlap and the result moves by 0.24%. |

---

## 3. Verification Log

### Summary

| # | Check | Method | Result |
|---|---|---|---|
| 1 | Blank, zero and negative values | Counted blanks, zeros and negatives in clean invoices, payments used, customers and the customer risk table | No blank cells and no negative numbers in any table. Zeros appear only in customer count columns, where zero is a real answer. |
| 2 | Edge cases | Customers without invoices, invoices with several payments, due date before invoice date, payments on the invoice day, due periods per customer | All 500 customers have invoices, no impossible dates, one due period per customer, and the 16 customers with no late payment are the same 16 in every late column. |
| 3 | Duplicates | Repeated IDs, exact copies and look-alike payments | No repeated invoice or order IDs in clean invoices. Repeated payment IDs are unrelated payments (192 of 193 belong to different customers, all 193 point to different invoices), with no exact copies. |
| 4 | Totals reconcile | Billed, paid and outstanding across tables, by status, late plus on time, before-GST tie to KPI 15 | Every difference is 0. The before-GST gap to KPI 15 is fully explained by the 393 left-out invoices. |
| 5 | Second method | Rebuilt every customer from the raw csv files with a plain loop, no pandas groupby | 0 customers differ on any number. Due period and ageing results are identical. |
| 6 | Outliers | Largest invoice and payment against the average, results without the biggest customers, priority list without each customer's largest open invoice | No extreme values (largest invoice 6.4 times the average, largest payment 8.2 times). The 20.3% leak survives removing the biggest customers. Priority ranks 1 to 14 all stay in the top 20; 4 of the 20 change places. |
| 7 | History and dates | Months with invoices, partial edges, late rate by payment year, 90-day payment window, ageing if today moves | 73 months with invoices and no empty month. By invoice year the payment pace is steady 2019 to 2024 (no year differs from the others at the strict line 0.0083). The 2025 late rate comes from the end of the data, not from slower customers. |
| 8 | Hand check of 3 customers | C0045, C0297 and C0006 rebuilt from the raw files and added up by hand | Raw rows equal the customer table on every number for all three. |

### Check 1: Blank, zero and negative values

- Clean invoices (17,640 rows), payments used (18,840), customers (500) and customer risk (500): 0 blank cells and 0 negative numbers in all four.
- Zeros: none in clean invoices, payments used or customers. In the customer risk table, late payments is zero for 16 customers, late invoices for 16, late invoice rate for 16 and unpaid invoices for 13. These are counts, so zero is a real answer. No money column has a zero.

### Check 2: Edge cases

- Customers in the customers table 500; with a clean invoice 500; without 0.
- Invoices with 2 or more payments 2,368; most payments on one invoice 3. Because of this, lateness of an invoice is judged by its last payment.
- Invoices with a due date before the invoice date 0; due date on the invoice day 0.
- Payments made on the invoice day 292 (allowed, counted on time); payments before the invoice date 0.
- Most due period lengths for one customer 1. Customers per due period: 15 days 87, 30 days 193, 45 days 113, 60 days 107 (total 500).
- Customers with no late payments 16, with no late invoices 16, with both 16. They are the same 16 customers.

### Check 3: Duplicates

- Clean invoices: repeated invoice ID 0 and repeated order ID 0.
- Payments used 18,840. Rows that share a payment ID with another used row: 387 in 193 payment IDs. By hand, 395 minus 387 is 8 and 201 minus 193 is 8: the 8 payments removed earlier (they point to a repeated invoice ID) left their partner rows standing alone.
- Exact copies of another row (every column the same): 0.
- Same invoice, date, amount and method under a different payment ID: 0.
- Payment IDs whose rows point to different invoices: 193. Payment IDs whose rows all point to the same invoice: 0. The repeated payment IDs are unrelated payments that share an ID, so adding the 395 back double counts nothing.
- Payment rows in the raw payments table 19,257; used 18,840 (417 removed). Payment IDs repeated in the raw table 201 (403 rows); repeated in payments used 193 (387 rows); repeated in the raw table but single in payments used 8 (confirmed by code: 403 minus 8 is 395, and 387 plus 8 is 395, by hand). Repeated in the raw table and missing from payments used 0.
- Rows per shared payment ID in payments used: 192 IDs with 2 rows and 1 ID with 3 rows (192 x 2 + 3 = 387, by hand). Rows without a customer 0.
- Customers: 192 of the 193 shared payment IDs have rows that belong to different customers. If payment IDs had been reused at random among 500 customers, about 0.4 of the 193 would share one customer (193 / 500 = 0.386, by hand); the observed 1 is consistent with chance.
- The one shared ID with a single customer is PAY-366706 (customer C0475): 56,548.55 by UPI on 21 Feb 2019 for INV-134522 (invoice total 91,273.00) and 48,179.40 in cash on 8 Mar 2024 for INV-882734 (invoice total 48,179.40). They are different invoices, five years apart, paid by different methods, and the two amounts (104,727.95 together, by hand) match neither invoice total, so this is not a split payment.

### Check 4: Totals reconcile

| Figure | Invoice or payment table | Customer table | Difference |
|---|---|---|---|
| Billed | 28,599,637,860.60 | 28,599,637,860.60 | 0.0 |
| Paid | 22,795,333,903.87 | 22,795,333,903.87 | 0.0 |
| Outstanding (billed minus paid) | 5,804,303,956.73 | 5,804,303,956.73 | 0.0 |

| Status | Invoices | Billed | Paid | Outstanding |
|---|---|---|---|---|
| Paid | 12,264 | 19,855,109,591 | 19,855,109,591 | 0.17 |
| Partially Paid | 3,578 | 5,826,085,351 | 2,940,224,313 | 2,885,861,038 |
| Unpaid | 1,798 | 2,918,442,919 | 0 | 2,918,442,919 |

- The 0.17 on Paid invoices is cents left over. Tested: the 12,264 invoices with 1 or less outstanding add up to 0.17, and no invoice is paid more than 1 over its bill (0).
- Late payments 7,820 plus on time payments 11,020 equal 18,840.
- Before GST: clean invoices 22,445,461,170; KPI 15 customer money 22,958,987,430; gap 513,526,260. Customers with a gap 284, with no gap 216, with a negative gap 0.
- The gap is exactly the 393 Delivered orders whose invoice rows were left out (tested: 18,033 Delivered orders, 17,640 with a clean invoice, 393 without; their value 513,526,260, difference 0; most for one customer 3; clean invoices whose order is not Delivered 0). Billed incl. GST divided by billed before GST is 1.2742.

### Check 5: Second method

- Every customer was rebuilt from the raw invoices.csv and payments.csv with a plain loop. Customers in the loop table 500, in the customer table 500, in only one 0.
- Customers where invoices, late payments or late invoices differ: 0, 0, 0. Customers where billed, paid or outstanding differ by more than one cent: 0, 0, 0.
- Late invoices by due period (invoices with a payment, late): 15 days 2,772 and 2,178; 30 days 6,195 and 3,556; 45 days 3,549 and 1,208; 60 days 3,326 and 238. Same as the first method.
- Today used 21 Mar 2025. Open invoices 5,376. Age bands: 0 to 30 days 0; 31 to 90 days 41 invoices, 38,263,531; 91 to 180 days 209, 216,057,037; 181 to 365 days 416, 441,440,520; over 365 days 4,710, 5,108,542,868. Same as the first method.
- Limit: the loop uses the same rule of leaving out repeated invoice IDs, so it confirms the calculation, not the rule. The rule is tested in Check 3 and in the Week 3 data checks.

### Check 6: Outliers

- Invoice size: average 1,621,295; middle 1,285,161; largest 10,451,982 (6.4 times the average).
- Payment size: average 1,209,943; middle 820,354; largest 9,970,301 (8.2 times the average). Payments bigger than their own invoice: 0.
- Outstanding share of billed money: 20.3% for all customers; 20.36% without the 10 customers with the most billed; 20.43% without the top 50; 20.53% without the top 100; 19.91% without the 10 customers with the most outstanding.
- Extra test, priority list: after removing each valuable customer's largest open invoice, the customers ranked 1 to 14 all stay in the top 20 and 4 of the 20 change places (16 of 20 stay). The four who leave were ranked 15, 17, 18 and 19 and have 24.0% to 32.8% of their outstanding in one invoice (average 27.0%). The four who come in were ranked 22 to 26 and have 14.0% to 19.9% in their largest invoice (average 16.8%). Positions inside the top 20 were not tested.
- Customers ranked 15 to 26 are bunched: balances from 18,383,128 to 20,945,200 (12.2% apart) and the 20th and 21st customers are 0.53% apart (19,727,968 and 19,622,487), so the cut at 20 is arbitrary and ranks 15 to 26 are best treated as one group.
- Across the 20 customers, the share of outstanding sitting in one invoice is 20.5% on average and 32.8% at most. Their outstanding drops from 461,987,109 to 368,081,554 (93,905,555 removed, 20.3%, added by hand). The list is a first group to contact, not an exact ranking.

### Check 7: History and dates

- First invoice 3 Jan 2019, last 13 Jan 2025. Months with invoices 73; empty months 0. Invoices per month (without the last, partial month): fewest 170, middle 244, most 285. Last month (January 2025) 47 invoices.
- Five months with the fewest invoices: 2019-01 170, 2023-02 212, 2020-04 213, 2019-02 214, 2020-02 217. January 2019 is the first month (the data starts on 3 Jan, which explains only about 6%). The rest of the dip is unexplained; it is one month of 73 and 0.96% of the invoices.
- First payment 9 Jan 2019, last payment 20 Mar 2025.
- Late rate by payment year (payments, late rate): 2019 2,808, 39.78%; 2020 3,122, 41.42%; 2021 3,155, 40.03%; 2022 3,171, 43.20%; 2023 3,172, 41.20%; 2024 3,070, 41.40%; 2025 342, 58.19% (partial year, to 20 Mar).
- Test of the 2025 edge, by payment month (payments, late rate, middle days to pay): 2024-07 240, 35.42%, 29.0; 2024-08 244, 39.75%, 32.0; 2024-09 240, 47.50%, 34.5; 2024-10 310, 40.65%, 32.0; 2024-11 237, 45.57%, 37.0; 2024-12 286, 37.06%, 32.0; 2025-01 234, 50.85%, 39.5; 2025-02 91, 71.43%, 52.0; 2025-03 17, 88.24%, 60.0. By hand, without 2025 the late rate is 7,621 of 18,498 = 41.20%, against 41.51% with it.

**The 2025 late rate comes from the end of the data, not from customers paying slower.** Three tests:

- **Invoices stop on 13 Jan 2025.** No payment dated after that day can be quicker than the days since the last invoice. The first payment in Feb 2025 came 19 days after the last invoice and the shortest time from invoice to payment that month was 21 days; in Mar 2025 the figures are 47 and 52 days.

| Payment month | Payments | Shortest days from invoice to payment | Days from the last invoice to the month's first payment |
|---|---|---|---|
| 2024-07 | 240 | 0 | 0 |
| 2024-08 | 244 | 0 | 0 |
| 2024-09 | 240 | 0 | 0 |
| 2024-10 | 310 | 0 | 0 |
| 2024-11 | 237 | 0 | 0 |
| 2024-12 | 286 | 0 | 0 |
| 2025-01 | 234 | 0 | 0 |
| 2025-02 | 91 | 21 | 19 |
| 2025-03 | 17 | 52 | 47 |

- **2025 payments took 10 to 13 days longer than Jul to Dec 2024 payments in every due period,** with a similar due period mix. The late rate rose in the 15, 30 and 45 day groups and hardly moved in the 60 day group.

| Due period | Payments Jul to Dec 2024 | Share % | Late % | Average days | Payments 2025 | Share % | Late % | Average days |
|---|---|---|---|---|---|---|---|---|
| 15 days | 232 | 14.9 | 75.86 | 32.7 | 55 | 16.1 | 90.91 | 45.5 |
| 30 days | 615 | 39.5 | 53.66 | 33.5 | 144 | 42.1 | 74.31 | 43.6 |
| 45 days | 352 | 22.6 | 29.83 | 33.3 | 77 | 22.5 | 48.05 | 45.9 |
| 60 days | 358 | 23.0 | 6.98 | 33.6 | 66 | 19.3 | 7.58 | 43.9 |

- **By invoice year the pace is steady 2019 to 2024.** Each invoice year was compared with all other years 2019 to 2024 (invoices with a payment; Mann-Whitney test for days, Fisher exact test for the late rate; strict line 0.0083 = 0.05 / 6).

| Invoice year | Invoices | With a payment | Middle days to last payment | Average days | Late % | Average days, other years | Late %, other years | p-value days | p-value late |
|---|---|---|---|---|---|---|---|---|---|
| 2019 | 2,911 | 2,612 | 35.0 | 35.2 | 45.25 | 35.2 | 45.35 | 0.9451 | 0.9314 |
| 2020 | 2,970 | 2,656 | 36.0 | 35.2 | 45.03 | 35.1 | 45.40 | 0.9549 | 0.7325 |
| 2021 | 2,952 | 2,666 | 34.0 | 34.7 | 44.34 | 35.2 | 45.54 | 0.1950 | 0.2582 |
| 2022 | 2,885 | 2,578 | 36.0 | 35.7 | 47.40 | 35.1 | 44.94 | 0.1114 | 0.0219 |
| 2023 | 2,975 | 2,682 | 33.5 | 34.2 | 43.89 | 35.3 | 45.63 | 0.0096 | 0.1012 |
| 2024 | 2,900 | 2,603 | 37.0 | 36.0 | 46.22 | 35.0 | 45.16 | 0.0185 | 0.3324 |
| 2025 (to 13 Jan) | 47 | 45 | 35.0 | 33.2 | 40.00 | not tested | not tested | not tested | not tested |

No year differs from the others at the strict line 0.0083. Three of the twelve p-values are under 0.05 (2023 days 0.0096, faster; 2024 days 0.0185, slower; 2022 late 0.0219), none under the strict line. The year 2024 is about 1 day slower (36.0 against 35.0, not proven) and 2023 about 1 day faster (34.2 against 35.3): similar size in opposite directions. Invoices dated Jul to Sep 2024 against 2019 to 2023 (672 against 13,194 invoices with a payment): 36.1 against 35.0 days (p 0.1627) and late 47.02 against 45.16% (p 0.3609). A gap of about 1 day cannot account for 10 to 13 days.

By invoice month (invoices, with a payment, middle days, average days, late %): 2024-07 253, 226, 35.0, 35.4, 44.25; 2024-08 247, 223, 39.0, 37.4, 51.57; 2024-09 242, 223, 36.0, 35.6, 45.29; 2024-10 265, 246, 37.5, 37.6, 46.34; 2024-11 251, 232, 38.0, 38.0, 48.71; 2024-12 229, 203, 36.0, 34.1, 44.33; 2025-01 47, 45, 35.0, 33.2, 40.00. The last invoice months (Dec 2024 and Jan 2025) look faster only because not every payment has arrived.
- 90-day payment window (cutoff 20 Dec 2024): invoices up to the cutoff 17,504, billed 28,370,323,313, outstanding share 20.32%; invoices after the cutoff 136, billed 229,314,547, outstanding share 16.69%. The recent group is not higher, and at about 0.8% of the money it is too small to move the headline.
- Ageing if today moves: today as used, 99.34% older than 90 days and 88.01% older than a year; moved 30 days later, 100% and 89.34%; moved 365 days later, 100% and 100%. The 41 invoices aged 31 to 90 days (0.66% of unpaid money) are still inside the 90-day window and may yet be paid.

### Check 8: Hand check of 3 customers (from the raw csv files)

| Customer | Why chosen | Raw rows, left out, used | Billed | Paid | Outstanding | Late payments, late invoices |
|---|---|---|---|---|---|---|
| C0045 | 60-day terms, value rank 52, biggest outstanding | 42, 0, 42 | 75,275,350.60 | 44,826,515.59 | 30,448,835.01 | 6, 6 |
| C0297 | 15-day terms, value rank 68 | 34, 2, 32 | 68,772,003.60 | 40,130,815.38 | 28,641,188.22 | 26, 24 |
| C0006 | Most left-out invoices (3) | 40, 3, 37 | 58,289,157.00 | 50,285,980.33 | 8,003,176.67 | 2, 2 |

- For all three customers the raw rows equal the customer table on every number, and invoices whose status does not match their payments: 0.
- C0045, added by hand from the printed lines: 6 unpaid invoices total 13,705,074.20; 14 partly paid remainders total 16,743,760.81; together 30,448,835.01. 22 invoices are Paid. 6 late lines counted. Each due date is 60 days after the invoice date. One invoice paid exactly on its due date counts as on time.
- C0297, added by hand: 7 unpaid invoices total 17,030,749.20; 11 partly paid remainders total 11,610,439.02; together 28,641,188.22. Of 25 invoices with a payment, 24 were late (96.0%, as in the priority list). All 14 Paid invoices were paid late, tested by code: the last payment came after the due date for 14 of 14, by 37, 37, 3, 6, 44, 16, 35, 40, 1, 28, 45, 36, 42 and 22 days (1 to 45 days). Two of them (INV-898040 and INV-644291) had a first payment on or before the due date and are late because a later payment came after it. In the whole customer, 24 invoices and 26 payments came after the due date: 16 of those payments sit on the 14 Paid invoices and 10 on the other invoices (26 minus 16, by hand), so the other 10 late invoices are Partially Paid ones (24 minus 14, by hand; Unpaid invoices have no payment). Left out: INV-509998 (642,464.00, Paid) and INV-628478 (3,650,548.00, Partially Paid), together 4,293,012.00, not in billed.
- C0006, added by hand: 3 unpaid invoices total 4,274,942.60; 5 partly paid remainders total 3,728,234.07; together 8,003,176.67. The 2 late lines equal the 2 late invoices. Left out: INV-706527 (2,156,800.00), INV-429049 (1,458,370.00) and INV-264378 (3,578,478.40), all with raw status Paid, together 7,193,648.40, not in billed.
- Limit: the payments belonging to the left-out invoices were not printed in this check. Their effect is covered by the before-GST gap explanation in Check 4.

### Supporting tests (run where the question came up)

| Test | Question | Result |
|---|---|---|
| Status against payment sums | Does payment status agree with payment sums? | With the 395 payments left out only for a repeated payment ID, 306 Paid invoices had money outstanding and 88 Partially Paid invoices had payments that were not between nothing and the full bill. All 306 and all 88 have one of those payments. With the 395 added back, all mismatch counts are 0. |
| Late rate with the 395 added back | Does KPI 4 still stand? | KPI 4 payments: 18,445, 7,655 late, 41.50%, middle 17.0 days late. The 395 added back: 395, 165 late, 41.77%, middle 16.0. All 18,840: 7,820 late, 41.51%, middle 17.0. KPI 4 stands. |
| Gap to KPI 15 money | What is the 2.2% gap before GST? | The 393 Delivered orders whose invoice rows were left out, worth 513,526,260, spread over 284 customers, at most 3 each. |
| Chance test on late rate | Do customers differ in lateness beyond luck? | Late invoice rate 45.32% overall. Real spread between customers 0.2568 against 0.0899 expected from luck (about 2.9 times). Customers with p below 0.05: 265 (about 25 expected); below 0.0001: 130. |
| Chance test on unpaid share | Does any customer stand out for unpaid invoices? | Unpaid invoice share 10.19% overall. Real spread 0.0528 against 0.0517 from luck. Customers with p below 0.05: 20 (about 25 expected); below 0.0001: 0. Chance only. |
| Chance test inside due period | Is the lateness difference a customer habit or the due period? | Inside each due period group, customers do not differ beyond luck (see Business Interpretation). The 265 earlier was the due period. |
| Days to pay by due period | Do customers pay faster under a short deadline? | No. Middle 34.0, 35.0, 35.0 and 36.0 days (average 34.55, 35.21, 35.25, 35.44) for 15, 30, 45 and 60 days. The four groups do not differ (p 0.3861) and no pair is below the strict line 0.0083 (15 against 60 days, p 0.0919, is the smallest). |
| Late rate by due period | Do the late rates differ between due periods? | Yes, far beyond chance: 78.57%, 57.40%, 34.04% and 7.16% (chi-square 3,738.8, 3 degrees of freedom, p below 0.0001; computed from the printed counts). |
| Steadiness by invoice year | Does the share of invoices not fully paid change between invoice years 2019 to 2024? | No. Shares 29.71, 31.75, 30.59, 30.64, 30.29 and 29.93% of 17,593 invoices: chi-square 3.584, 5 degrees of freedom, p 0.6107; trend z -0.468, p 0.6401 (computed from the printed counts). |
| Late rate band by due period | Are the late rate bands the due period? | Yes. All 83 customers in the 0 to 10% band have the 60-day due period, and no band mixes 15-day and 60-day customers (see Business Interpretation). |
| Priority list without the largest open invoice | Does the call list survive removing each customer's largest open invoice? | Ranks 1 to 14 all stay; 4 leave and 4 enter; ranks 15 to 26 are bunched (see Check 6). |
| Customers table columns | Do credit_limit and current_balance match the transactions? | No. current_balance matches outstanding for 0 of 500 customers (within 10%), total 123,340,274 against 5,804,303,957 (2.12%), rank correlation -0.006; credit_limit against billed money rank correlation -0.076 (p 0.090) and below each customer's own outstanding for all 500. |
| payment_terms against due period | Does payment_terms match the invoices? | Yes, for all 500 customers and all 17,640 clean invoices; 0 of 20,000 orders differ from the customer's terms (see Business Interpretation). |
| Concentration against chance | Is unpaid money concentrated beyond customer size and chance? | No. Real top 10, 50, 100 shares and Gini sit inside the range 500 random shuffles give (see Business Interpretation). |
| Late rate against unpaid share | Do customers who pay late more often have more of their billing unpaid? | No. Rank correlation 0.022 (p 0.6283); inside the four due periods 0.154, 0.020, 0.195 (p 0.0384) and 0.007, none under the strict line 0.0125. |
| 2025 late rate | Is the 58.19% a change in customer behaviour? | No. It comes from the end of the data (see Check 7). |
| Repeated payment IDs and customers | Do rows that share a payment ID belong to different customers? | 192 of 193 shared IDs belong to different customers; the other is two unrelated payments of one customer (see Check 3). |
| C0297 Paid invoices | Were all 14 Paid invoices paid late? | Yes, 14 of 14, by 1 to 45 days (see Check 8). |
| Advance against Net 30 | Do Advance customers pay in advance? | No. They behave like Net 30 customers (see Business Interpretation). |
| Left-out invoices bound | Could the 393 left-out invoices change the 20.3%? | By hand, the outstanding share would lie between 19.84% (all paid) and 22.08% (all unpaid) at the usual 1.2742 ratio of billing incl. GST to before GST. |
| Two rules for valuable | Does the result depend on how valuable is defined? | No. 91 of 100 customers overlap and the outstanding they hold differs by 0.24%. |
| Cents and overpayment | Is the ageing gap of 0.17 real money? | No. It is cents on the 12,264 Paid invoices. Invoices paid more than 1 over the bill: 0. |

---

## 4. Data Quality Notes

1. **393 invoice rows left out.** Invoice IDs that appear more than once (393 rows in 196 groups) are not used. They are 393 Delivered orders worth 513,526,260 before GST (about 2.2% of the money) spread over 284 customers, at most 3 each. Customer billed and outstanding figures leave them out. The data cannot show which row of a repeated ID is the real one.
2. **417 payments left out.** Payments that point to a repeated invoice ID cannot be given to one invoice. Payments used: 19,257 minus 417 = 18,840.
3. **395 payments kept in.** 403 payment rows share a payment ID. 8 of them also point to a repeated invoice ID (already out). The other 395 point to a clean invoice and are used. They are unrelated payments that share an ID (193 IDs, every group points to different invoices, 192 of the 193 belong to different customers, 0 exact copies). The one ID with a single customer is two payments on different invoices five years apart (see Check 3).
4. **KPI 4 is not affected.** Adding the 395 payments moves the late rate from 41.50% to 41.51% and the middle days late stays at 17. The KPI 4 result stands.
5. **Money basis.** Payments are on the grand total basis (incl. GST), so outstanding money is reported on that basis only. Before-GST money appears only to tie to KPI 15.
6. **Two different 20.3 figures.** KPI 3's Partially Paid 20.30% is a share of invoices. KPI 16's 20.3% is a share of billed money. Always write "of invoices" or "of billed money".
7. **Customers' own columns.** credit_limit and current_balance were tested and are not used: current_balance matches the outstanding money we calculated for 0 of 500 customers (within 10%), its total is 123,340,274 against 5,804,303,957 (2.12%) and the rank correlation is -0.006 (chance level); credit_limit has no relation to billed money (rank correlation -0.076, p 0.090) and is below each customer's own outstanding for all 500 customers (average 491,525 against average outstanding 11,608,608). The unit of these two columns is not known, so the rank correlations, which do not depend on the unit, are the evidence. payment_terms was also tested and matches the due period on the invoices for all 500 customers and all 17,640 clean invoices (0 of 20,000 orders differ from the customer's terms); Net 15, Net 30, Net 45 and Net 60 mean 15, 30, 45 and 60 days, and the 103 customers marked Advance are invoiced with a 30-day due period (3,661 invoices), the same as Net 30. KPI 16 takes the due period from the invoices themselves. customer_since, last_purchase_date and total_purchase_value were also not used (Week 3 findings).
8. **Cents on Paid invoices.** 12,264 Paid invoices carry 0.17 in total outstanding because of rounding.
9. **Today's date.** Ageing uses 21 Mar 2025 (the day after the latest date in the data). If today moves, the unpaid money only gets older.
10. **Data end effect.** Payments in 2025 (342 payments, to 20 Mar) show a late rate of 58.19%. The rise comes from the end of the data, not from customers paying slower: invoices stop on 13 Jan 2025, so no payment dated after that can be quicker than the days since the last invoice, and by invoice year the payment pace is steady (tests in Check 7). Invoices after 20 Dec 2024 (136) do not yet have a full 90-day payment window; they are about 0.8% of the money and the outstanding share without them is 20.32% against 20.30% with them. The 41 invoices aged 31 to 90 days (0.66% of unpaid money) may still be paid.
11. **January 2019 is low.** 170 invoices against a middle of 244. The first month starts on 3 Jan, which explains only about 6%. The cause is unexplained and it affects 0.96% of invoices.
12. **Priority list is a first group.** The 20 customers were picked for having the biggest balances, so high shares of billing are expected. 4 of 20 change places when each customer's largest open invoice is removed (16 of 20 stay; ranks 1 to 14 all stay). Ranks 15 to 26 are bunched (12.2% apart), so the list is a first group to contact, not an exact ranking (see Check 6).
13. **Small groups.** Central and North East have 11 customers each, so their figures are less reliable.
14. **Left-out payments not printed in the hand check.** Check 8 did not print the payments of the left-out invoices.
15. **No reason field.** The data has no field that says why money is unpaid.

---

## 5. Business Interpretation

### The money

| Figure | Value |
|---|---|
| Clean invoices | 17,640 |
| Billed (incl. GST) | 28,599,637,860.60 |
| Paid | 22,795,333,903.87 |
| Outstanding | 5,804,303,956.73 (about 20.3% of billed money) |
| Invoices not fully paid | 5,376 (30.48% of invoices): Unpaid 1,798, Partially Paid 3,578 |
| Unpaid invoices, money outstanding | 2,918,442,919 |
| Partially Paid invoices, money outstanding | 2,885,861,038 |
| Late payments | 7,820 of 18,840 (41.51%), middle 17.0 days late, average 20.0 days late (late payments only) |
| Late invoice rate | 7,180 of 15,842 invoices with a payment (45.32%) |

Top 5 customers by outstanding money (all customers, incl. GST): C0045 (42 invoices, 30,448,835.01, late invoice rate 16.7%); C0297 (32, 28,641,188.22, 96.0%); C0319 (33, 26,789,362.48, 54.2%); C0421 (38, 25,737,384.59, 53.3%); C0135 (35, 25,528,785.79, 7.1%).

### Is the leak steady? (by invoice year)

| Invoice year | Invoices | Billed | Outstanding | Open invoices | Outstanding % of billed | Open invoice % |
|---|---|---|---|---|---|---|
| 2019 | 2,911 | 4,771,199,102 | 914,664,887 | 865 | 19.17 | 29.71 |
| 2020 | 2,970 | 4,863,343,984 | 1,046,512,696 | 943 | 21.52 | 31.75 |
| 2021 | 2,952 | 4,757,345,306 | 949,467,312 | 903 | 19.96 | 30.59 |
| 2022 | 2,885 | 4,780,701,603 | 1,008,071,697 | 884 | 21.09 | 30.64 |
| 2023 | 2,975 | 4,789,278,680 | 956,718,676 | 901 | 19.98 | 30.29 |
| 2024 | 2,900 | 4,553,561,874 | 915,144,399 | 868 | 20.10 | 29.93 |
| 2025 | 47 | 84,207,312 | 13,724,290 | 12 | 16.30 | 25.53 |

The leak is steady: in every invoice year 2019 to 2024, 19.2% to 21.5% of billing and 29.7% to 31.8% of invoices are not fully paid (about 0.91 to 1.05 billion a year). The share of invoices not fully paid does not differ between the six years (chi-square 3.584, 5 degrees of freedom, p 0.6107) and shows no trend up or down (p 0.6401); both tests use the invoice counts in the table (17,593 invoices, 2019 to 2024). The money share is a range only (19.17% to 21.52%) and was not chance-tested. 2025 has only 47 invoices (to 13 Jan 2025) and is not reliable.

### How old is the unpaid money? (today 21 Mar 2025)

| Age band | Open invoices | Outstanding | Share of unpaid money |
|---|---|---|---|
| 0 to 30 days | 0 | 0 | 0.00% |
| 31 to 90 days | 41 | 38,263,531 | 0.66% |
| 91 to 180 days | 209 | 216,057,037 | 3.72% |
| 181 to 365 days | 416 | 441,440,520 | 7.61% |
| Over 365 days | 4,710 | 5,108,542,868 | 88.01% |

Open invoices 5,376; oldest 2,268 days; middle 1,182 days. No payment in the data arrived later than 90 days after its invoice (the most is 90 days in every due period group). So 99.34% of the unpaid money (5,766,040,425 across 5,335 invoices) is older than 90 days: it is not slow money, it has not come in during the whole history.

### Lateness follows the due period

| Due period | Invoices with a payment | Late | Late rate |
|---|---|---|---|
| 15 days | 2,772 | 2,178 | 78.57% |
| 30 days | 6,195 | 3,556 | 57.40% |
| 45 days | 3,549 | 1,208 | 34.04% |
| 60 days | 3,326 | 238 | 7.16% |

Every customer has one due period. The shorter the due period, the later the payments: the four late rates differ far beyond chance (chi-square 3,738.8, 3 degrees of freedom, p below 0.0001). The guess that short terms would give on-time payers was tested and was wrong.

**Days from invoice date to last payment, by due period:**

| Due period | Invoices | Middle | Average | Fewest | Most |
|---|---|---|---|---|---|
| 15 days | 2,772 | 34.0 | 34.5 | 0 | 90 |
| 30 days | 6,195 | 35.0 | 35.2 | 0 | 90 |
| 45 days | 3,549 | 35.0 | 35.3 | 0 | 90 |
| 60 days | 3,326 | 36.0 | 35.4 | 0 | 90 |

Customers pay at the same pace whatever the deadline: the middle is 34 to 36 days (average 34.55 to 35.44), anywhere from 0 to 90 days, and the four due periods do not differ (Kruskal-Wallis test p 0.3861). No pair of due periods is below the strict line 0.0083 (15 against 60 days, p 0.0919, is the smallest). The due period only decides whether that counts as late. A short due period brings cash in no faster than a long one. Payments dated before the invoice date: 0.

### Do customers differ in how they pay? (chance tests)

Overall, customers differ in lateness beyond luck (265 customers below p 0.05, about 25 expected; 130 below 0.0001). But inside each due period group they do not:

| Due period | Customers | Late rate | Real spread | Spread from luck | p below 0.05 (expected) | p below 0.0001 |
|---|---|---|---|---|---|---|
| 15 days | 87 | 78.57% | 0.0767 | 0.0740 | 4 (about 4.4) | 0 |
| 30 days | 193 | 57.40% | 0.0900 | 0.0887 | 9 (about 9.7) | 0 |
| 45 days | 113 | 34.04% | 0.0900 | 0.0860 | 7 (about 5.7) | 0 |
| 60 days | 107 | 7.16% | 0.0496 | 0.0470 | 5 (about 5.4) | 0 |

In total 25 customers are below 0.05 against about 25 expected. The earlier gap was the due period, not a customer habit. The share of unpaid invoices is also chance only (see Supporting tests in the Verification Log: 20 customers below 0.05 against about 25 expected, 0 below 0.0001). **No list of bad payers can be justified.**

**Customers by late invoice rate band:**

| Late rate band | Customers | Invoices | Billed | Outstanding |
|---|---|---|---|---|
| 0 to 10% | 83 | 2,868 | 4,773,268,078 | 941,034,025 |
| 10 to 20% | 30 | 1,054 | 1,666,290,392 | 373,772,806 |
| 20 to 30% | 25 | 872 | 1,389,157,345 | 251,655,876 |
| 30 to 40% | 60 | 2,110 | 3,440,116,289 | 709,816,665 |
| 40 to 50% | 60 | 2,104 | 3,412,036,871 | 712,661,474 |
| 50 to 60% | 81 | 2,882 | 4,621,875,337 | 919,703,324 |
| 60 to 70% | 69 | 2,498 | 4,023,378,174 | 735,504,616 |
| 70 to 80% | 58 | 2,049 | 3,372,049,510 | 701,154,842 |
| 80 to 90% | 28 | 1,027 | 1,586,894,333 | 390,300,529 |
| 90 to 100% | 6 | 176 | 314,571,532 | 68,699,799 |

Lowest late rate 0.0, middle 0.493, highest 1.0.

**The late rate bands follow the due period** (customers by late rate band and due period):

| Late rate band | 15 days | 30 days | 45 days | 60 days | Total |
|---|---|---|---|---|---|
| 0 to 10% | 0 | 0 | 0 | 83 | 83 |
| 10 to 20% | 0 | 0 | 8 | 22 | 30 |
| 20 to 30% | 0 | 0 | 23 | 2 | 25 |
| 30 to 40% | 0 | 6 | 54 | 0 | 60 |
| 40 to 50% | 0 | 36 | 24 | 0 | 60 |
| 50 to 60% | 1 | 76 | 4 | 0 | 81 |
| 60 to 70% | 8 | 61 | 0 | 0 | 69 |
| 70 to 80% | 45 | 13 | 0 | 0 | 58 |
| 80 to 90% | 27 | 1 | 0 | 0 | 28 |
| 90 to 100% | 6 | 0 | 0 | 0 | 6 |
| Total | 87 | 193 | 113 | 107 | 500 |

All 83 customers in the 0 to 10% band have the 60-day due period (83 of 83), and 83 of the 107 customers with 60 days (77.6%) are in that band (the other 24: 22 in 10 to 20%, 2 in 20 to 30%). No band mixes 15-day and 60-day customers. Each band is made of one or two neighbouring due periods, except 50 to 60% (76 with 30 days, 1 with 15 days, 4 with 45 days). The 15-day customers sit from 50% up, mostly in 70 to 90% (72 of 87); the 30-day customers mostly in 40 to 70% (173 of 193); the 45-day customers mostly in 20 to 50% (101 of 113); the 60-day customers all in 0 to 30%.

**Money stuck and lateness.** The outstanding share of billed money is about 18% to 25% in every band (0 to 10%: 19.7%, 80 to 90%: 24.6%). Customers who pay late more often do not have more of their billing unpaid: the rank correlation of the late invoice rate and outstanding % of billing is 0.022 (p 0.6283). Inside the four due periods it is 0.154 (15 days, 87 customers, p 0.1556), 0.020 (30 days, 193, p 0.7834), 0.195 (45 days, 113, p 0.0384) and 0.007 (60 days, 107, p 0.9447). Only the 45-day group is under 0.05 and it is not under the strict line 0.0125 (0.05 divided by 4 groups), so it is a weak hint at most, not proven; with four groups the chance of at least one under 0.05 by luck is about 18.5% (1 minus 0.95 to the power 4, by hand). The late invoice rate counts only invoices that had a payment.

### Where does the unpaid money sit? Concentration and valuable customers

- Top 10 customers by outstanding hold 4.48% (even share 2.0%); top 50 hold 18.38% (even 10.0%); top 100 hold 32.83% (even 20.0%). They hold 2.62%, 12.42% and 23.41% of billing. The biggest 100 customers by billing hold 26.80% of billing and 25.97% of the outstanding money, so they owe in proportion to their size. Gini: 0.2328 for outstanding, 0.1285 for billing (billing itself is uneven).
- **Concentration against chance.** To see what luck alone gives, which invoices are unpaid was shuffled 500 times (invoice sizes and customers kept). Real top 10 share 4.48% against a luck average of 4.57% (range 3.93 to 5.26; 325 of 500 shuffles as high as the real one). Top 50: 18.38% against 18.66% (17.21 to 20.28; 379 of 500). Top 100: 32.83% against 33.21% (31.53 to 34.99; 393 of 500). Gini: 0.2328 against 0.2431 (0.2177 to 0.2652; 462 of 500). The real concentration sits inside the range random shuffling gives, slightly below its average, so it is what customer size and chance produce: no customer owes more than chance explains. Ranking by outstanding always puts customers with lucky high balances on top, which is why the top of the list looks concentrated.

| Group | Customers | Billed | Outstanding | Share of all billing | Share of all outstanding | Outstanding % of own billing |
|---|---|---|---|---|---|---|
| 100 most valuable (KPI 15 rank 1 to 100) | 100 | 7,648,156,846.40 | 1,510,885,496.85 | 26.74% | 26.03% | 19.75% |
| Other 400 | 400 | 20,951,481,014.20 | 4,293,418,459.88 | 73.26% | 73.97% | 20.49% |

Rank correlation of value rank and outstanding share of billing: 0.042 (p 0.3444). Valuable customers are not riskier: their share of the outstanding money (26.03%) is close to their share of billing (26.74%) and value rank and outstanding share are not proven related. Their exposure is large because they are billed more, and the biggest 100 customers by billing owe in proportion to their size (see the concentration test above). Rule two (valuable = top 100 by KPI 16 billed money): 100 customers, 91 in both rules, outstanding held 1,507,270,908.09 against 1,510,885,496.85 (0.24% apart), so the result does not depend on the rule.

### Priority list: 20 valuable customers with the most outstanding money

| Customer | Value rank | Billed | Outstanding | Unpaid invoices | Partly paid invoices | Due period | Outstanding % of billing |
|---|---|---|---|---|---|---|---|
| C0045 | 52 | 75,275,351 | 30,448,835 | 6 | 14 | 60 days | 40.4 |
| C0297 | 68 | 68,772,004 | 28,641,188 | 7 | 11 | 15 days | 41.6 |
| C0421 | 96 | 69,466,243 | 25,737,385 | 8 | 7 | 30 days | 37.1 |
| C0207 | 14 | 83,475,657 | 25,369,012 | 7 | 12 | 30 days | 30.4 |
| C0288 | 2 | 97,218,404 | 24,774,937 | 6 | 11 | 30 days | 25.5 |
| C0475 | 4 | 94,883,375 | 24,558,266 | 4 | 17 | 45 days | 25.9 |
| C0356 | 33 | 79,690,881 | 24,179,551 | 6 | 13 | 45 days | 30.3 |
| C0134 | 18 | 85,243,025 | 24,072,778 | 6 | 11 | 60 days | 28.2 |
| C0153 | 1 | 103,578,606 | 23,749,609 | 8 | 9 | 30 days | 22.9 |
| C0076 | 31 | 78,054,532 | 22,323,798 | 5 | 9 | 30 days | 28.6 |
| C0375 | 28 | 81,286,326 | 22,284,511 | 5 | 13 | 60 days | 27.4 |
| C0394 | 89 | 69,936,379 | 21,760,366 | 8 | 8 | 45 days | 31.1 |
| C0178 | 11 | 84,320,986 | 21,497,012 | 8 | 14 | 60 days | 25.5 |
| C0311 | 99 | 69,282,135 | 21,307,303 | 6 | 11 | 45 days | 30.8 |
| C0084 | 66 | 67,587,341 | 20,945,200 | 5 | 10 | 60 days | 31.0 |
| C0242 | 67 | 68,650,975 | 20,691,698 | 3 | 14 | 30 days | 30.1 |
| C0060 | 42 | 77,163,010 | 20,019,107 | 5 | 9 | 60 days | 25.9 |
| C0482 | 61 | 69,582,785 | 20,014,759 | 6 | 6 | 60 days | 28.8 |
| C0272 | 50 | 75,550,923 | 19,883,826 | 7 | 6 | 45 days | 26.3 |
| C0165 | 34 | 78,017,958 | 19,727,968 | 7 | 10 | 60 days | 25.3 |

These 20 hold 461,987,109.14 = 7.96% of all outstanding money. They were selected for having the biggest balances, so high shares of their billing are expected; they are not shown to be bad payers. Ranks 15 to 26 are bunched (12.2% apart), so the cut at 20 is arbitrary and this is a first group to contact, not an exact ranking (see Check 6).

### Outstanding money by group (27 groups)

Overall share of invoices not fully paid is 30.48%. The p-value compares each group's share of invoices not fully paid with the overall share. The strict line is 0.0019 (0.05 divided by 27 groups).

| Group | Customers | Invoices | Not fully paid invoices | Billed | Outstanding | Outstanding % of billed | Share of all outstanding % | Not fully paid invoices % | p-value |
|---|---|---|---|---|---|---|---|---|---|
| Due period 15 days | 87 | 3,094 | 948 | 4,982,798,598 | 1,054,341,481 | 21.16 | 18.16 | 30.64 | 0.8452 |
| Due period 30 days | 193 | 6,858 | 2,031 | 11,058,093,373 | 2,181,376,525 | 19.73 | 37.58 | 29.62 | 0.1217 |
| Due period 45 days | 113 | 3,967 | 1,221 | 6,422,965,018 | 1,320,936,435 | 20.57 | 22.76 | 30.78 | 0.6790 |
| Due period 60 days | 107 | 3,721 | 1,176 | 6,135,780,871 | 1,247,649,516 | 20.33 | 21.50 | 31.60 | 0.1347 |
| Corporate | 104 | 3,589 | 1,127 | 5,865,847,363 | 1,256,285,235 | 21.42 | 21.64 | 31.40 | 0.2314 |
| Dealer | 99 | 3,567 | 1,109 | 5,867,524,564 | 1,172,186,965 | 19.98 | 20.20 | 31.09 | 0.4236 |
| Fleet Owner | 95 | 3,379 | 1,000 | 5,495,099,289 | 1,124,720,952 | 20.47 | 19.38 | 29.59 | 0.2703 |
| Government | 114 | 3,975 | 1,178 | 6,256,016,038 | 1,246,265,410 | 19.92 | 21.47 | 29.64 | 0.2555 |
| Retail | 88 | 3,130 | 962 | 5,115,150,606 | 1,004,845,395 | 19.64 | 17.31 | 30.73 | 0.7561 |
| Branch AHM001 | 88 | 3,048 | 897 | 4,962,367,844 | 970,588,677 | 19.56 | 16.72 | 29.43 | 0.2151 |
| Branch CHN001 | 102 | 3,592 | 1,134 | 5,824,464,334 | 1,266,391,646 | 21.74 | 21.82 | 31.57 | 0.1574 |
| Branch DEL001 | 71 | 2,629 | 826 | 4,366,346,167 | 872,652,013 | 19.99 | 15.03 | 31.42 | 0.2992 |
| Branch HYD001 | 68 | 2,435 | 704 | 3,859,076,692 | 738,934,684 | 19.15 | 12.73 | 28.91 | 0.0944 |
| Branch KOL001 | 96 | 3,339 | 1,060 | 5,410,782,244 | 1,125,422,046 | 20.80 | 19.39 | 31.75 | 0.1143 |
| Branch PUN001 | 75 | 2,597 | 755 | 4,176,600,580 | 830,314,891 | 19.88 | 14.31 | 29.07 | 0.1248 |
| Region Central | 11 | 399 | 117 | 632,374,897 | 129,351,923 | 20.45 | 2.23 | 29.32 | 0.6636 |
| Region East | 87 | 3,090 | 964 | 5,048,097,617 | 1,054,734,933 | 20.89 | 18.17 | 31.20 | 0.3899 |
| Region North | 82 | 2,973 | 902 | 4,911,101,772 | 939,004,120 | 19.12 | 16.18 | 30.34 | 0.8891 |
| Region North East | 11 | 367 | 110 | 580,270,014 | 107,785,581 | 18.58 | 1.86 | 29.97 | 0.8650 |
| Region South | 158 | 5,601 | 1,742 | 9,063,664,206 | 1,895,045,062 | 20.91 | 32.65 | 31.10 | 0.3097 |
| Region West | 151 | 5,210 | 1,541 | 8,364,129,354 | 1,678,382,337 | 20.07 | 28.92 | 29.58 | 0.1616 |
| Construction | 90 | 3,146 | 961 | 5,002,587,783 | 1,038,016,226 | 20.75 | 17.88 | 30.55 | 0.9383 |
| Infrastructure | 79 | 2,788 | 835 | 4,571,102,777 | 892,432,843 | 19.52 | 15.38 | 29.95 | 0.5509 |
| Logistics | 96 | 3,387 | 1,012 | 5,476,062,935 | 1,040,808,058 | 19.01 | 17.93 | 29.88 | 0.4554 |
| Manufacturing | 70 | 2,590 | 791 | 4,246,350,168 | 877,672,589 | 20.67 | 15.12 | 30.54 | 0.9489 |
| Mining | 98 | 3,328 | 1,018 | 5,308,250,691 | 1,071,416,824 | 20.18 | 18.46 | 30.59 | 0.8951 |
| Rental | 67 | 2,401 | 759 | 3,995,283,508 | 883,957,414 | 22.13 | 15.23 | 31.61 | 0.2313 |

No group differs from the overall share of invoices not fully paid (30.48%): the smallest p-value is 0.0944 (HYD001) and none is below 0.05 in 27 groups (about 1.4 expected by luck). Outstanding is between 18.58% and 22.13% of billing in every group (a range; it was not chance-tested on money). South and West hold 32.65% and 28.92% of all outstanding, together 61.6%, with 309 of 500 customers. On the share of invoices not fully paid no segment stands out, so there is no segment-level policy target.

### Does the label Advance mean payment in advance?

The customers table column payment_terms matches the due period on the invoices exactly (customers by payment terms and invoice due period):

| Payment terms | 15 days | 30 days | 45 days | 60 days | Customers |
|---|---|---|---|---|---|
| Advance | 0 | 103 | 0 | 0 | 103 |
| Net 15 | 87 | 0 | 0 | 0 | 87 |
| Net 30 | 0 | 90 | 0 | 0 | 90 |
| Net 45 | 0 | 0 | 113 | 0 | 113 |
| Net 60 | 0 | 0 | 0 | 107 | 107 |
| All customers | 87 | 193 | 113 | 107 | 500 |

Orders whose payment terms differ from the customer's: 0 of 20,000. Invoices by order payment terms: Advance 3,661 (all 30-day), Net 15 3,094, Net 30 3,197, Net 45 3,967, Net 60 3,721 (total 17,640, with 3,661 plus 3,197 = 6,858 invoices on 30 days, by hand). So the 103 Advance customers are invoiced with a 30-day due period. Do they pay like Net 30 customers?

| Measure | Advance | Net 30 | p-value |
|---|---|---|---|
| Customers | 103 | 90 | |
| Invoices | 3,661 | 3,197 | |
| Invoices with a payment | 3,334 | 2,861 | |
| Late invoice rate | 57.35% | 57.46% | 0.9384 |
| Middle days to last payment | 35.0 | 36.0 | 0.6810 |
| Average days to last payment | 35.3 | 35.1 | |
| First payment on the invoice day | 70 (2.10%) | 65 (2.27%) | |
| Payment before the invoice date | 0 | 0 | |
| Not fully paid invoices | 29.25% | 30.03% | 0.4907 |
| Outstanding % of billed | 19.32% | 20.19% | |

Advance customers behave like Net 30 customers: no payment came before an invoice date, only 2.1% of Advance invoices with a payment had their first payment on the invoice day (2.3% for Net 30), and none of the three differences is proven (p 0.94, 0.49, 0.68; Fisher exact and Mann-Whitney tests). So the label Advance shows no payment in advance in the invoices and payments data. No difference found is not proof of identical behaviour.

### What this KPI cannot tell us

- It cannot tell us why the money is unpaid (disputes, losses, write-offs). The data has no reason field. No later KPI in the plan can answer this either, because the reason was never recorded. The owner has to find the reasons outside the data, starting with the customers on the priority list.
- 393 invoices (about 2.2% of the money, 513,526,260 before GST) and 417 payments are left out because of repeated IDs. No later KPI can tell us what they are: the repeated IDs were measured in the Week 3 data checks and in KPI 2, and the data cannot show which row of a repeated ID is the real one. Only a correction at the source can settle them. They are 2.2% of the money, spread over 284 customers, at most 3 each. Their payments are unknown, so their effect was bounded by hand rather than measured: at the usual 1.2742 ratio of billing incl. GST to before GST they are worth about 654 million incl. GST (2.24% of billing with them), and the outstanding share of billed money would lie between 19.84% (all paid) and 22.08% (all unpaid), against 20.30% now.
- It cannot tell us why the old balances were not paid (99.34% of unpaid money is older than 90 days).
- It describes the past. It does not predict which customer will not pay (customer differences in lateness inside a due period group, and in unpaid share, are within chance).
- The ageing depends on today's date (21 Mar 2025); the 41 invoices aged 31 to 90 days may still be paid.

### Recommended actions for the business owners

1. **Head of Collections:** Call the customers on the priority list first, starting with the largest balances (C0045, C0297, C0421, C0207, C0288). Together the 20 hold 461,987,109.14, which is 7.96% of all outstanding money. Ask each one why the balance is open, because the data cannot tell us.
2. **Finance Director:** Decide on a firm collection or write-off review for the old invoices. 5,335 invoices worth 5,766,040,425 (99.34% of the unpaid money) are older than 90 days, and no payment in the data ever arrived later than 90 days after its invoice.
3. **Commercial Director:** Review the 15-day due period. 78.57% of those invoices are paid late and customers pay at the same pace (about 35 days) under every due period, so the short deadline brings cash in no faster.
4. **Credit Control Manager:** Do not blacklist customers or single out a branch, region, customer type or industry. No customer stands out beyond luck inside its due period group, and none of the 27 groups differs in the share of invoices not fully paid (smallest p-value 0.0944).
5. **Finance Controller:** Keep reporting the due period and the outstanding money every month, with the age bands, so a change in the steady leak (about 20% of billed money) is seen early.

---

## Glossary

| Label | Meaning |
|---|---|
| billed | Sum of grand_total (incl. GST) of clean invoices |
| paid | Sum of payment_amount of payments used |
| outstanding | Billed minus paid |
| clean_invoices | The 17,640 invoices whose invoice ID appears once |
| repeated_invoice_rows, repeated_payment_rows | True/False marks on rows whose invoice ID or payment ID appears more than once |
| repeated_ids | The set of invoice IDs that appear more than once |
| removed_invoice_ids | The repeated invoice IDs left out |
| points_to_repeated | Payments that point to a repeated invoice ID (417) |
| clean_payments | The KPI 4 payment set (18,445 payments) |
| left_out | The 395 payments left out of KPI 4 only because their payment ID is repeated; used in KPI 16 |
| payments_used | Every payment that does not point to a repeated invoice ID (18,840) |
| paid_per_invoice, inv_money | Paid and outstanding money per invoice |
| pay_late | payments_used with customer, due date, days late and a late flag |
| per_inv, inv_late | Per invoice: number of payments, last payment date, whether the last payment was late |
| customer_risk | The per-customer table (500 rows): invoices, billed, paid, outstanding, status counts, late counts, late invoice rate, value before GST, value rank |
| value_before_gst, value_rank | Customer money before GST on Delivered orders and rank (KPI 15) |
| late payment | A payment dated after the due date |
| late invoice | An invoice whose last payment is dated after its due date |
| late invoice rate | Late invoices divided by invoices that have a payment |
| due period (terms_days) | Due date minus invoice date in days |
| open invoice | An invoice with more than 1 of money outstanding |
| ageing, age band | Days from invoice date to today (21 Mar 2025), grouped into bands |
| valuable customer | Top 100 in the KPI 15 ranking (rule one); top 100 by billed money is rule two |
| priority list | The 20 valuable customers with the most outstanding money |
| chance test, p-value | A test of whether a difference could be luck. A p-value below 0.05 means unlikely to be luck; about 5% of customers fall below 0.05 by luck alone |
| strict line 0.0019 | 0.05 divided by the 27 groups tested |
| strict line 0.0083 | 0.05 divided by 6 comparisons (six invoice years, or six pairs of due periods) |
| strict line 0.0125 | 0.05 divided by the 4 due period groups tested |
| rank correlation | A number from -1 to 1 showing whether two lists move together by rank; near 0 means no link |
| chi-square test | A test of whether shares differ between groups by more than chance would give |
| Kruskal-Wallis test, Mann-Whitney test | Tests of whether groups differ in their typical values (here days to pay); Mann-Whitney compares two groups |
| Fisher exact test | A test of whether two rates (here late invoice rate) differ by more than chance |
| Gini | A number from 0 (everyone equal) to 1 (one holds everything) showing how unevenly a total is spread over customers |
| luck test (shuffle) | The calculation repeated 500 times with the unpaid invoices shuffled, to see what chance alone gives |
| data-end effect | A change that appears only because the data stops on a date |
| left-out invoices | The 393 invoice rows whose invoice ID repeats, which are not used |
| Advance, Net 15, Net 30, Net 45, Net 60 | payment_terms labels in the customers and orders tables; Net 15 to Net 60 mean 15 to 60 days; Advance is invoiced on a 30-day due period |
| selection effect | A list picked for having big balances will show big shares by design |
| show_customer, show_customer_short | Notebook functions that print one customer's invoices and payments from the raw files for the hand check |
| GST | Tax added to the order value. Grand total = order value plus GST |
| not fully paid invoice | An invoice that is Unpaid or Partially Paid |
