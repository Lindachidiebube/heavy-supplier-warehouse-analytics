# KPI 14 RFM Segmentation

Project: Heavy Supplier & Warehouse Analytics | Week 8 | Notebook: Week8_KPI14_RFM_Segmentation

## Summary

- This KPI scores each of the 500 customers on how recently (Recency), how often (Frequency) and how much (Monetary, before GST) they bought, using the 18,033 Delivered orders from 2019-01-01 to 2024-12-31, measured on the snapshot date 2025-01-01.
- Each score runs from 1 to 5 and the customers fall into six segments: Quiet and low value 127 customers (21.9% of money), Recent but small 114 (19.6%), Loyal 71 (15.7%), At risk (valuable but quiet) 70 (16.8%), Champions 66 (16.8%) and Average 52 (9.1%).
- The At risk segment holds 3,859,948,290 of money, the same as Champions, but its customers last ordered about 104 days ago against 18 days for Champions. Three views of "slipping away" are reported: the 70 rule-based customers, the 67 customers quiet beyond twice their own normal gap (13.13% of money) and the firm 21 customers who stay At risk under every cut point tested (5.07% of money, the first call list).
- Branch DEL001 (Delhi) holds 20 of the 70 At risk customers (28.2% against 14.0% overall, p-value 0.0017), partly because it has more high-value customers (60.6% against 41.4%, p-value 0.0015). Delhi is not later than its own rhythm (p-value 0.86). No other branch, customer type, region, payment term or industry showed a tested difference.
- All 8 verification checks agree: no blanks or duplicates, every total ties to 22,958,987,430, a second method and the raw files give identical results for every customer checked, and no extreme order decides a segment.
- Tested limits: the segments are a snapshot (only 31.0% of customers keep their segment after one year), and customers do not differ in how often or how much they order beyond what chance produces (random customers give the same segment sizes). The segments guide follow-up; they are not proof that a customer is leaving.

## 1. KPI Definition

| Metric | Definition | Unit |
|---|---|---|
| Recency | Days from the customer's last Delivered order to the snapshot date 2025-01-01 | days |
| Frequency | Number of Delivered orders of the customer | orders |
| Monetary | Sum of total_order_value (before GST) of the customer's Delivered orders | money |
| Recency score | 1 to 5, five groups of about equal size on the recency values; fewest days = 5 | score |
| Frequency score | 1 to 5, five groups on the order counts; most orders = 5 | score |
| Monetary score | 1 to 5, five groups on the money values; most money = 5 | score |
| FM | Average of the frequency score and the money score | 1 to 5 |
| Segment | One of six segments from rules on the recency score and FM (see Methodology) | label |
| Silence ratio (supporting test) | Recency days divided by the customer's own average gap between orders; gap = (last order - first order) / (orders - 1) | ratio |

**Business question:** Which customers matter most to the business, and which valuable customers are slipping away?

**Why this metric:** An owner cannot call 500 customers every week. RFM turns three simple facts (how recently, how often, how much) into a short, ranked list: who to protect, who to follow up first, and who needs little attention.

**Scope:** Delivered orders only (a Cancelled order is not a sale). Customer facts are built from the orders, not from the customers table columns customer_since, last_purchase_date and total_purchase_value, which Week 3 found unreliable. Money is before GST. The snapshot date is 2025-01-01, the day after the last order in the data.

## 2. Methodology

### Sources and scoping

- sales_orders_header (so_h): 20,000 rows x 11 columns (so_id, customer_id, branch_id, order_date, delivery_date, order_status, payment_terms, total_order_value, total_gst_amount, grand_total, sales_channel). customers: 500 rows x 15 columns (customer_id, customer_type, industry_segment, city, state, pincode, region, branch_id, credit_limit, current_balance, payment_terms, customer_since, last_purchase_date, total_purchase_value, customer_rating). Both match Week 3. The first 3 header rows were hand-checked: value plus GST equals the grand total for all 3.
- sales_orders_lines (130,402 rows x 9 columns; 117,656 lines belong to Delivered orders) is used in the verification of totals and of the raw-file check.
- Delivered orders: 18,033 (ties to Week 3); 1,967 Cancelled orders are left out; Delivered + Cancelled = 20,000 = all orders. 0 Delivered orders have a blank customer, date or value; 0 orders belong to a customer missing from the customers table.
- Order dates run from 2019-01-01 to 2024-12-31. All 500 customers have at least one Delivered order, so every customer gets Recency, Frequency and Monetary and no rule is needed for customers without orders.
- Smallest order value 310 (equals the P026 list price, one unit); largest 8,169,760. Average about 36 orders per customer (18,033 / 500).
- Snapshot date 2025-01-01: the day after the last order. Using today's date would make everyone look inactive.

### Steps

| Name | What it did | Result |
|---|---|---|
| Load tables | Loaded so_h and customers, printed columns, row counts and first rows | Row counts and columns as above |
| Delivered orders table | Kept Delivered rows of so_h, order_date as a date | 18,033 orders; 0 blanks; 0 unknown customers; 500 of 500 customers have orders |
| Customer table (rfm) | One row per customer: last order, Frequency, Monetary, Recency | 500 rows; Frequency total 18,033; Monetary total 22,958,987,430 = Delivered total = KPI 7 revenue (before GST) |
| Distribution | Spread of Recency, Frequency, Monetary and customers per recency band | See below |
| Scores | Five equal-size groups on the values (same value = same score), Recency reversed | See below |
| Segments | Six segments from rules on the recency score and FM | See below |
| At risk claim test | Silence ratio by segment | See below |
| Late-group sizing | Size of the customers quiet beyond twice their own gap | See below |
| Who the At risk customers are | At risk share by customer type, region, branch, payment terms, industry | See Business Interpretation |
| Chance test of 28 groups | Binomial test per group against the overall At risk share 14.0% | See Business Interpretation |
| Delhi At risk group | List and size of the 20 DEL001 At risk customers | See Business Interpretation |
| Stricter branch test | Share quiet beyond twice own gap by branch | See Business Interpretation |
| Value mix versus quietness | High-value share and quiet share by branch | See Business Interpretation |
| Quiet share among high-value customers | Chance test by branch | See Business Interpretation |
| High-value share by branch | Chance test by branch | See Business Interpretation |
| Correlation | Rank correlation of Recency, Frequency and Monetary | See Business Interpretation |

**Customer table (first 3 rows, recounted by hand):** C0001 last order 2024-11-15, 34 orders, 43,201,680, recency 47; C0002 last order 2024-09-07, 51 orders, 56,261,100, recency 116; C0003 last order 2024-10-30, 38 orders, 35,112,260, recency 63.

**Ranges:** Recency 1 to 359 days (so every customer ordered in 2024; 359 days means a last order about 8 January 2024); Frequency 21 to 56 orders; Monetary 17,721,530 to 84,966,970 (about 4.8 times).

**Distribution (500 customers):**

| Measure | Mean | Std | Min | 25% | Median | 75% | Max |
|---|---|---|---|---|---|---|---|
| Recency days | 61.9 | 55.1 | 1 | 21 | 47 | 85 | 359 |
| Frequency | 36.1 | 6.0 | 21 | 32 | 36 | 40 | 56 |
| Monetary | 45,917,974.9 | 10,298,204.4 | 17,721,530 | 38,446,610 | 45,321,980 | 52,438,192.5 | 84,966,970 |

Recency is right-skewed (mean above median), Frequency is symmetric with a narrow middle, Monetary has a few big spenders. Means tie by hand: 18,033 / 500 = 36.07 and 22,958,987,430 / 500 = 45,917,974.86.

Customers per recency band: 1 to 30 days 167; 31 to 90 days 218; 91 to 180 days 90; 181 to 270 days 20; 271 to 366 days 5 (sum 500).

**Scores (customers per group and the values they cover):**

| Score | Recency (days) | Frequency (orders) | Monetary |
|---|---|---|---|
| 5 | 101 customers (1 to 17) | 92 (42 to 56) | 100 (54,324,600 to 84,966,970) |
| 4 | 103 (18 to 37) | 92 (38 to 41) | 100 (47,587,690 to 54,295,880) |
| 3 | 99 (38 to 59) | 101 (35 to 37) | 100 (42,934,640 to 47,504,560) |
| 2 | 98 (60 to 98) | 96 (32 to 34) | 100 (37,042,540 to 42,914,200) |
| 1 | 99 (99 to 359) | 119 (21 to 31) | 100 (17,721,530 to 37,034,600) |

Each metric sums to 500 and the ranges do not overlap. Frequency groups are uneven (119 in group 1) because customers with the same order count must share a score, and each Frequency group is only 3 to 4 orders wide. Hand check: C0001 R3 F2 M3; C0002 R1 F5 M5; C0003 R2 F4 M1.

**Segment rules** (FM = average of the frequency score and the money score; rules applied in this order):

| Segment | Rule |
|---|---|
| Champions | Recency score 4 or 5 and FM 4 or more |
| At risk (valuable but quiet) | Recency score 1 or 2 and FM 3.5 or more |
| Quiet and low value | Recency score 1 or 2 (rest) |
| Loyal | Recency score 3 or more and FM 3.5 or more |
| Recent but small | Recency score 4 or 5 (rest) |
| Average | Everyone else |

**Segment results:**

| Segment | Customers | Avg recency days | Avg orders | Avg spend | Total spend | Share of money |
|---|---|---|---|---|---|---|
| Quiet and low value | 127 | 120.1 | 32.2 | 39,616,765 | 5,031,329,160 | 21.9% |
| Recent but small | 114 | 17.6 | 32.3 | 39,548,701 | 4,508,551,960 | 19.6% |
| Loyal | 71 | 38.8 | 40.2 | 50,849,015 | 3,610,280,070 | 15.7% |
| At risk (valuable but quiet) | 70 | 103.8 | 40.8 | 55,142,118 | 3,859,948,290 | 16.8% |
| Champions | 66 | 18.0 | 43.5 | 58,583,847 | 3,866,533,880 | 16.8% |
| Average | 52 | 47.9 | 32.4 | 40,045,078 | 2,082,344,070 | 9.1% |

Counts sum to 500; spend sums to 22,958,987,430; shares sum to 99.9% (rounding). C0001 is Average, C0002 is At risk, C0003 is Quiet and low value. At risk has about the same money as Champions but last ordered about 104 days ago against 18. The spend gap between the best and the weakest segment is only about 1.5 times (58.6 million against 39.5 million) because spending is clustered.

**At risk claim test** (is the At risk customer really quiet against their own normal rhythm?):

| Segment | Customers | Median gap (days) | Median recency (days) | Median ratio | Customers with ratio above 2 |
|---|---|---|---|---|---|
| At risk | 70 | 51.81 | 96.0 | 1.94 | 28 |
| Quiet and low value | 127 | 64.22 | 101.0 | 1.61 | 39 |
| Loyal | 71 | 53.75 | 42.0 | 0.82 | 0 |
| Average | 52 | 66.22 | 47.5 | 0.72 | 0 |
| Champions | 66 | 50.38 | 18.5 | 0.35 | 0 |
| Recent but small | 114 | 67.00 | 16.5 | 0.25 | 0 |

The At risk label holds for the segment (median customer about 1.9 times their own normal gap against 0.35 for Champions), but only 28 of 70 are above 2. Quiet and low value is also elevated (39 above 2), so 67 customers in total are above twice their own gap. Decision: keep the six segments and report the ratio as a supporting test, not as part of the scoring.

**Late-group sizing** (customers with ratio above 2):

| Segment | Customers | Spend | Share of all money |
|---|---|---|---|
| At risk | 28 | 1,524,468,230 | 6.64% |
| Quiet and low value | 39 | 1,490,920,520 | 6.49% |
| Together | 67 | 3,015,388,750 | about 13.13% |

The 28 At risk customers hold about 39% of the whole At risk spend. The top 10 by spend in this group are all At risk (spend 55,400,350 to 71,230,450):

| Customer | Days quiet | Normal gap | Ratio | Spend |
|---|---|---|---|---|
| C0110 | 194 | 48.15 | 4.03 | 71,230,450 |
| C0254 | 98 | 40.70 | 2.41 | 70,227,620 |
| C0221 | 118 | 47.86 | 2.47 | 64,336,800 |
| C0103 | 125 | 51.69 | 2.42 | 59,164,240 |
| C0154 | 133 | 46.16 | 2.88 | 58,187,170 |
| C0337 | 129 | 59.36 | 2.17 | 57,585,510 |
| C0297 | 184 | 59.55 | 3.09 | 57,279,920 |
| C0384 | 114 | 53.76 | 2.12 | 56,692,240 |
| C0002 | 116 | 41.06 | 2.83 | 56,261,100 |
| C0489 | 137 | 42.43 | 3.23 | 55,400,350 |

Nobody is gone (all 500 ordered in 2024), so the wording is "quiet beyond their own rhythm", not "lost".

### Alternatives considered

| Alternative | Decision | Reason |
|---|---|---|
| Customer facts from the customers table columns (customer_since, last_purchase_date, total_purchase_value) | Rejected | Week 3: customer_since is after the real first order for 293 of 500 customers; last_purchase_date is before the real last order for 467 of 500; total_purchase_value differs by more than 1% for 500 of 500 |
| Snapshot date today, or later than the last order | Rejected | Orders end on 2024-12-31. At 2025-01-31 no segment changes but the count quiet beyond twice their gap rises from 67 to 123 only because the data stops (see Check 7) |
| Include Cancelled orders | Rejected | A Cancelled order is not a sale; 1,967 Cancelled orders are left out (18,033 Delivered + 1,967 Cancelled = 20,000) |
| Score on the values (same value = same score) | Chosen | Customers with the same value must get the same score; Frequency groups end up uneven (92 to 119) |
| Other cut points: Champions need FM 3.5 or 4.5; valuable starts at 3.0 or 4.0; quiet means recency score 1 only | Standard cut points kept | The Champions line does not touch At risk; the At risk group size depends on the other two lines (33 to 92 around 70). Reported as a range (see Check 5) |
| Silence ratio as the scoring measure | Kept as a supporting test | It depends strongly on the snapshot date (67 against 123), and only 28 of the 70 At risk customers are above 2 |

## 3. Verification Log

### Summary of the 8 checks

| Check | What it tests | Result in plain words |
|---|---|---|
| Check 1 | Blank, zero and negative values | None in any column used |
| Check 2 | Unusual cases: customers without orders, with one order, same-day orders | Every customer has orders; none has one order; same-day orders exist and none are copies |
| Check 3 | Repeated orders | Each order appears once; no repeated ids; no identical rows |
| Check 4 | Totals reconcile | Everything adds up; the header value equals the sum of its lines for all 18,033 orders |
| Check 5 | Second method and cut points | Identical for all 500 customers; the At risk size depends on the cut points (33 to 92); firm group of 21 identified |
| Check 6 | Extreme orders | None extreme; no single order decides a segment |
| Check 7 | History and dates | 72 months complete; segments are a snapshot; customers do not differ beyond chance; the snapshot date is tested |
| Check 8 | Manual check of 3 customers from the raw files | Identical in the raw files and in the table |

### Check 1: Blank, zero and negative values

- sales_h (Delivered orders) 18,033 rows: 0 blank values in so_id, customer_id, order_date, total_order_value and order_status; 0 empty-text order ids and customer ids; 0 order values of zero or less.
- rfm (500 rows): 0 blank values in customer_id, recency_days, frequency, monetary, R_score, F_score, M_score and segment; 0 recency, frequency or monetary values of zero or less.
- customers (500 rows): 0 blank customer ids; 0 repeated customer ids; 0 customers of the score table missing from the customers table.

Result in plain words: no blanks, no zero or negative numbers, every customer matches.

### Check 2: Unusual cases

- Customers in the customers table 500; with at least one Delivered order 500; with none 0.
- Fewest orders by one customer 21; customers with only one order 0 (so every customer has a normal gap).
- Customer and day pairs with more than one order: 132, creating 133 extra orders (0.74% of 18,033). The most orders by one customer on one day is 3, so 131 days have 2 orders and 1 day has 3 (a + b = 132 and 2a + b = 133 give a = 1; confirmed by the repeated-orders result, see Check 3).
- Orders matching another order on customer, day and value: 0.

Result in plain words: every customer has orders, nobody has one order only, and same-day orders exist but none are copies. Frequency counts each order separately.

### Check 3: Repeated orders

- Rows in Delivered orders 18,033; different order ids 18,033; repeated order ids 0 in the Delivered table and 0 in the full 20,000-row orders table; fully identical rows 0; most orders by one customer on one day 3.

Result in plain words: every order appears once and there are no copies.

### Check 4: Totals reconcile

- Orders in sales_h 18,033 = orders added over all customers 18,033.
- Order value in sales_h 22,958,987,430 = money added over all customers 22,958,987,430 (difference 0) = revenue total from KPI 7 (difference 0).
- Segments: customers 500, orders 18,033, money 22,958,987,430 over the six segments (customers 70 + 52 + 66 + 71 + 127 + 114 = 500; orders 2,854 + 1,685 + 2,870 + 2,856 + 4,089 + 3,679 = 18,033; money adds to 22,958,987,430).
- Order statuses: Delivered 18,033 + Cancelled 1,967 = 20,000 = all orders.
- Order lines: 130,402 lines in total; 117,656 belong to Delivered orders; the lines add to 22,958,987,430 = the header total (difference 0); Delivered orders with no lines 0; orders where the header value differs from the sum of its lines 0; largest difference on one order 0.

Result in plain words: everything adds up and nothing is lost or counted twice. The header value equals the sum of its own lines for every Delivered order, so the money in KPI 14 agrees with the revenue in KPI 7.

### Check 5: Second method and cut points

**Second method.** The customer table was rebuilt from the raw orders file with a plain loop (not pandas), keeping Delivered orders only: 500 customers rebuilt; customers where recency differs 0, order count differs 0, money differs 0, recency score differs 0, frequency score differs 0, money score differs 0, segment differs 0. Limit: both methods use the same snapshot date and the same rules, so those choices are tested separately.

**Cut-point sensitivity** (each cut point moved one at a time; the normal rules reproduce the table with 0 differences; At risk now 70):

| Variation | Customers who change segment | At risk group | Of today's 70 still At risk |
|---|---|---|---|
| Champions need average score 3.5 | 24 | 70 | 70 |
| Champions need average score 4.5 | 15 | 70 | 70 |
| Valuable starts at 3.0 | 64 | 92 | 70 |
| Valuable starts at 4.0 | 62 | 49 | 49 |
| Quiet means recency score 1 only | 98 | 33 | 33 |

The Champions line does not touch At risk (Champions need recency score 4 or 5, At risk needs 1 or 2). The At risk group size depends on the other two lines: loosened versions (92) contain all 70, tightened versions (49 and 33) are subsets of the 70. The size 70 is a convention and is reported with the range 33 to 92.

**Firm At risk group** (At risk under the strictest rules: valuable starts at 4.0 and quiet means recency score 1 only, therefore At risk under every setting tested): 21 customers; 19 of them quiet beyond twice their own normal gap (the other two are just under: C0084 1.945, C0464 1.971); spend 1,164,484,450, which is 5.07% of all money and about 30.2% of the At risk segment spend.

| Customer | Days quiet | Average gap | Ratio | Orders | Money |
|---|---|---|---|---|---|
| C0110 | 194 | 48.15 | 4.03 | 42 | 71,230,450 |
| C0221 | 118 | 47.86 | 2.47 | 44 | 64,336,800 |
| C0103 | 125 | 51.69 | 2.42 | 40 | 59,164,240 |
| C0154 | 133 | 46.16 | 2.88 | 45 | 58,187,170 |
| C0084 | 101 | 51.92 | 1.95 | 39 | 57,484,480 |
| C0384 | 114 | 53.76 | 2.12 | 39 | 56,692,240 |
| C0002 | 116 | 41.06 | 2.83 | 51 | 56,261,100 |
| C0489 | 137 | 42.43 | 3.23 | 47 | 55,400,350 |
| C0025 | 136 | 54.81 | 2.48 | 38 | 55,220,350 |
| C0373 | 121 | 53.49 | 2.26 | 38 | 54,767,460 |
| C0421 | 194 | 51.43 | 3.77 | 38 | 54,492,210 |
| C0311 | 106 | 51.33 | 2.07 | 41 | 54,370,990 |
| C0225 | 168 | 41.47 | 4.05 | 48 | 54,324,600 |
| C0192 | 125 | 52.54 | 2.38 | 40 | 54,295,880 |
| C0041 | 204 | 53.46 | 3.82 | 38 | 53,358,620 |
| C0464 | 110 | 55.81 | 1.97 | 38 | 52,908,510 |
| C0176 | 147 | 49.56 | 2.97 | 42 | 52,504,460 |
| C0428 | 128 | 48.57 | 2.64 | 43 | 52,052,170 |
| C0006 | 121 | 49.13 | 2.46 | 40 | 51,412,280 |
| C0438 | 122 | 50.08 | 2.44 | 39 | 48,432,400 |
| C0024 | 106 | 48.02 | 2.21 | 44 | 47,587,690 |

Result in plain words: two different views agree (19 of 21 above twice their own gap).

### Check 6: Extreme orders

- Average order value 1,273,165 (22,958,987,430 / 18,033); largest order 8,169,760, which is 6.4 times the average.
- Top 10 orders (10 different customers, spread over 2020 to 2024, only 1.5 million between the first and the tenth):

| Order | Customer | Date | Value |
|---|---|---|---|
| SO-246196 | C0268 | 2020-02-18 | 8,169,760 |
| SO-624611 | C0102 | 2021-02-13 | 7,791,450 |
| SO-929145 | C0359 | 2023-04-24 | 7,752,220 |
| SO-786016 | C0349 | 2024-10-03 | 7,731,900 |
| SO-667806 | C0030 | 2023-07-18 | 7,146,240 |
| SO-713731 | C0200 | 2022-11-07 | 7,001,250 |
| SO-431481 | C0162 | 2022-05-31 | 6,998,530 |
| SO-585493 | C0234 | 2024-10-22 | 6,927,700 |
| SO-609293 | C0153 | 2024-02-22 | 6,716,250 |
| SO-271147 | C0225 | 2024-07-17 | 6,635,650 |

- A customer's biggest order as a share of their own money: smallest 5.15%, average 9.43%, largest 17.38% (an even share with about 36 orders would be about 2.8%).
- Without each customer's own biggest order: the money score changes for 60 customers (12.0%) and the segment changes for 20 customers (4.0%). The At risk group stays 70; 68 customers are in both groups (97.1%); C0102 and C0198 leave; C0321 and C0408 join; all 21 firm At risk customers stay At risk.

Result in plain words: no order is extreme, a customer's biggest order does not decide their segment, and the At risk list changes by two names only.

### Check 7: History and dates

**Months.** 72 months with Delivered orders (2019-01 to 2024-12); fewest orders in a month 209, most 312; the months add to 18,033. All 500 customers have orders before 2024-01-01.

**Segments one year earlier** (segments rebuilt as at 1 January 2024 with the orders before that date, same scoring and rules, compared with today's segments):
- Customers with the same segment a year later: 155 of 500 (31.0%). By chance (random reshuffling with these segment sizes) only about 18.1% (about 91 customers) would keep the same segment (45,285 / 250,000), so 155 is above chance but far from steady.
- At risk on 1 January 2024: 75. Of today's 70 At risk customers, 22 were already At risk then (about 10.5 by chance: 75 x 70 / 500).
- Of the 75 At risk customers a year ago, 38 (22 + 16) are Champions or Loyal today and only 14 are Quiet and low value or Recent but small. No contact data exists, so no cause can be stated for these recoveries.

| Segment a year ago (rows) / today (columns) | At risk | Average | Champions | Loyal | Quiet and low value | Recent but small |
|---|---|---|---|---|---|---|
| At risk | 22 | 1 | 22 | 16 | 10 | 4 |
| Average | 1 | 12 | 0 | 1 | 23 | 23 |
| Champions | 23 | 2 | 27 | 18 | 1 | 1 |
| Loyal | 17 | 0 | 14 | 18 | 10 | 7 |
| Quiet and low value | 5 | 14 | 1 | 14 | 43 | 46 |
| Recent but small | 2 | 23 | 2 | 4 | 40 | 33 |

The diagonal adds to 155; the rows add to 75, 60, 72, 66, 123 and 104 (500); the columns add to today's segment sizes.

**Which score moves** (score a year ago against today; chance for five equal groups is 20% the same and 52% within one step):

| Score | Same score | Within one step | Rank correlation |
|---|---|---|---|
| Recency | 94 of 500 (18.8%) | 249 (49.8%) | -0.01 |
| Frequency | 315 (63.0%) | 490 (98.0%) | 0.89 |
| Money | 288 (57.6%) | 480 (96.0%) | 0.86 |

Recency has no memory (equal to chance), so the segment movement comes from recency. Caveat: the counts a year ago and today share five of the six years of orders, so the high Frequency and Money correlations are partly automatic.

**Year against year with different orders (2023 against 2024):** all 500 customers ordered in 2023 or 2024. Orders per customer 2023 average 6.04, spread 2.5; 2024 average 5.91, spread 2.42. Rank correlation of orders -0.04 and of money 0.04. Over all six years the average is 36.07 orders with a spread of 6.04, against a spread of 6.01 expected from chance alone (the square root of the average). By hand the chance spread for 2023 is 2.46 and for 2024 is 2.43.

Finding: customers do not differ in how often or how much they order beyond what chance produces.

**Simulation** (200 runs of 500 random customers with Poisson order counts at the real average, random order days over 2019 to 2024, order values drawn from the real orders, same scoring and rules):

| Measure | Real | Random customers: average | Lowest | Highest |
|---|---|---|---|---|
| Customers quiet beyond twice their own normal gap | 67 | 72.0 | 51 | 93 |
| Customers in the At risk group | 70 | 73.7 | 54 | 88 |
| Spread of order counts | 6.04 | 5.98 | | |

Every real number lies inside the range that random customers give.

**Snapshot-date sensitivity** (each snapshot uses only the orders before that date):

| Snapshot date | Customers who change segment against our table | At risk group | Quiet beyond twice their normal gap |
|---|---|---|---|
| 2024-07-01 | 340 (68%) | 69 | 69 |
| 2024-12-01 | 270 (54%) | 66 | 69 |
| 2025-01-01 (our date) | 0 | 70 | 67 |
| 2025-01-31 | 0 | 70 | 123 |

A later date changes no segment because every recency grows by the same number of days; only the count quiet beyond twice their gap rises (67 to 123). It is not a fair view, because the orders end on 2024-12-31 and customers would look quiet only because the data stops. Earlier dates change 54% to 68% of the segments (a different moment with less data, consistent with recency having no memory) while the At risk size stays between 66 and 70.

Result in plain words: the history is complete, the segments are a snapshot of one date and must be refreshed, and the size of the groups found is what random customers would produce.

### Check 8: Manual check of 3 customers from the raw files

C0001, C0002 and C0003 were rebuilt from the raw orders file and the raw order lines file with a plain loop and compared with the customer table:

| Customer | Orders | First order | Last order | Recency | Money (headers = lines) | Average gap | Silence ratio | Scores R, F, M | Segment |
|---|---|---|---|---|---|---|---|---|---|
| C0001 | 34 | 2019-01-26 | 2024-11-15 | 47 | 43,201,680 | 64.24 | 0.73 | 3, 2, 3 | Average |
| C0002 | 51 | 2019-01-24 | 2024-09-07 | 116 | 56,261,100 | 41.06 | 2.83 | 1, 5, 5 | At risk (valuable but quiet) |
| C0003 | 38 | 2019-04-05 | 2024-10-30 | 63 | 35,112,260 | 55.0 | 1.15 | 2, 4, 1 | Quiet and low value |

The table values are identical to the raw files on every field. By hand: the spans 2,120 / 33 = 64.24, 2,053 / 50 = 41.06 and 2,035 / 37 = 55.0; the scores fall inside the score ranges; the segments follow the rules.

Result in plain words: the three customers are identical in the raw files and in the table.

### Supporting tests (build and business tests)

- Load tables, Delivered orders table, customer table, distribution, scores, segments: see Methodology.
- At risk claim test (silence ratio) and late-group sizing: see Methodology.
- Who the At risk customers are, chance test of 28 groups, Delhi At risk group, stricter branch test, value mix versus quietness, quiet share among high-value customers, high-value share by branch, correlation of Recency, Frequency and Money: see Business Interpretation.
- Score stability, year against year, simulation and snapshot sensitivity: see Check 7.

## 4. Data Quality Notes

1. The customers table columns customer_since, last_purchase_date and total_purchase_value are unreliable (Week 3) and are not used; all customer facts come from the Delivered orders.
2. 1,967 Cancelled orders are left out by design (Delivered 18,033 + Cancelled 1,967 = 20,000).
3. Same-day orders: 132 customer and day pairs have more than one order (133 extra orders, 0.74%; one day has 3 orders). They are separate orders, not copies, and each counts in Frequency.
4. Frequency groups are uneven (92 to 119 customers) because customers with the same order count must share a score; each group is 3 to 4 orders wide.
5. Money is before GST. The data holds no currency name, so amounts are shown as money.
6. The snapshot date is 2025-01-01, the day after the last order. A later date makes customers look quiet only because the data stops (67 against 123 quiet beyond twice their gap at 2025-01-31).
7. The segment cut points are conventions. The At risk group is 70 under the standard rules and 33 to 92 when the cut points move.
8. The segments are a snapshot: only 31.0% of customers keep their segment after one year (about 18% would by chance).
9. Customers do not differ in how often or how much they order beyond what chance produces (spread of order counts 6.04 against 6.01 from chance; 2023 and 2024 unrelated with correlations -0.04 and 0.04; random customers give the same group sizes). The scoring is correct (all 8 checks agree), but Champions and Loyal describe the past six years and do not predict next year.
10. Frequency and Money are correlated at 0.70, below the 0.8 line, so the FM average is fair; Recency is almost independent of both (-0.14 and -0.10).
11. Branch and region are not the same thing in the customers table (for example C0003 is branch DEL001 but region West); Delhi (a branch) and North (a region) were separated in the tests.
12. The regions Central and North East have only 11 customers each, too few to read.
13. Several early remarks were tested and are not supported: Advance payers being less often quiet (p-value 0.154), and any difference by Dealer type (p-value 0.146) or Retail (p-value 0.219). Customer type, payment terms and industry show no evidence of a difference.
14. The Delhi finding is just inside the strict line for the At risk share (0.0017 against 0.0018) and clear for the high-value share (0.0015); a higher quiet share among its valuable customers is possible but not proven (p-value 0.1055).
15. No outreach or contact data exists, so no cause can be stated for the At risk customers who later became Champions or Loyal (38 of 75).

## 5. Business Interpretation

**Business question:** Which customers matter most to the business, and which valuable customers are slipping away?

**Answer:** Spending is clustered: the best segment (Champions, 58.6 million per customer on average) spends only about 1.5 times the weakest (39.5 million), so no small group carries the business. The valuable customers who are quiet are the At risk segment: 70 customers, 3,859,948,290 (16.8% of money), last order about 104 days ago against a median normal gap of about 52 days. They fall into three views: the 70 rule-based customers; the 67 customers quiet beyond twice their own normal gap (13.13% of money); and the firm 21 customers (5.07% of money) who are At risk under every setting tested and form the first call list. The tests also show that these groups are what random ordering would produce, so they are signals for cheap follow-up, not proof that customers are leaving.

### Finding: the six segments

See the segment table in Methodology: Champions 66 customers (16.8% of money), Loyal 71 (15.7%), At risk 70 (16.8%), Average 52 (9.1%), Recent but small 114 (19.6%), Quiet and low value 127 (21.9%).

### Finding: who the At risk customers are

Overall 14.0% of customers are At risk (70 of 500). Each attribute's customers and At risk counts add to 500 and 70.

| Attribute | Group | Customers | At risk | Share |
|---|---|---|---|---|
| Customer type | Dealer | 99 | 19 | 19.2% |
| Customer type | Fleet Owner | 95 | 14 | 14.7% |
| Customer type | Corporate | 104 | 15 | 14.4% |
| Customer type | Government | 114 | 14 | 12.3% |
| Customer type | Retail | 88 | 8 | 9.1% |
| Region | North | 82 | 19 | 23.2% |
| Region | South | 158 | 22 | 13.9% |
| Region | West | 151 | 19 | 12.6% |
| Region | East | 87 | 8 | 9.2% |
| Region | Central | 11 | 1 | 9.1% |
| Region | North East | 11 | 1 | 9.1% |
| Branch | DEL001 | 71 | 20 | 28.2% |
| Branch | AHM001 | 88 | 13 | 14.8% |
| Branch | CHN001 | 102 | 14 | 13.7% |
| Branch | HYD001 | 68 | 8 | 11.8% |
| Branch | KOL001 | 96 | 10 | 10.4% |
| Branch | PUN001 | 75 | 5 | 6.7% |
| Payment terms | Net 45 | 113 | 21 | 18.6% |
| Payment terms | Net 60 | 107 | 18 | 16.8% |
| Payment terms | Net 15 | 87 | 11 | 12.6% |
| Payment terms | Net 30 | 90 | 11 | 12.2% |
| Payment terms | Advance | 103 | 9 | 8.7% |
| Industry | Infrastructure | 79 | 13 | 16.5% |
| Industry | Rental | 67 | 11 | 16.4% |
| Industry | Construction | 90 | 13 | 14.4% |
| Industry | Manufacturing | 70 | 10 | 14.3% |
| Industry | Logistics | 96 | 13 | 13.5% |
| Industry | Mining | 98 | 10 | 10.2% |

DEL001 holds 20 of the 70 At risk customers (28.6%) with 14.2% of the customers (71 of 500), about twice the overall share.

**Chance test** (binomial, two-sided, against 14.0%, 28 groups; strict line p-value below 0.0018 = 0.05 / 28; between 0.0018 and 0.05 is possible and treated with caution):

| Group | Customers | At risk | Expected | p-value | Reading |
|---|---|---|---|---|---|
| Branch DEL001 | 71 | 20 | 9.9 | 0.0017 | Strong (just under 0.0018) |
| Region North | 82 | 19 | 11.5 | 0.0244 | Possible, caution |
| Branch PUN001 | 75 | 5 | 10.5 | 0.0676 | Not shown |
| Dealer | 99 | 19 | 13.9 | 0.1462 | Not shown |
| Payment Advance | 103 | 9 | 14.4 | 0.1540 | Not shown |
| Payment Net 45 | 113 | 21 | 15.8 | 0.1737 | Not shown |
| Retail | 88 | 8 | 12.3 | 0.2194 | Not shown |
| Region East | 87 | 8 | 12.2 | 0.2771 | Not shown |

Customer type, payment terms and industry show no evidence of a difference (it could be luck). DEL001 customers by region (customers, At risk): East 4 and 1, North 59 and 17, North East 1 and 0, South 5 and 1, West 2 and 1 (sums 71 and 20). 59 of the 82 North customers are in DEL001; the 23 North customers outside DEL001 have 2 At risk (8.7%, below 14.0%), so North's high share comes from the Delhi branch and not from the region.

### Finding: branch DEL001 (Delhi)

The 20 Delhi At risk customers hold 1,119,969,650 (4.88% of all money, 29.0% of the At risk segment spend); 9 of them are quiet beyond twice their own normal gap and 11 are between 0.97 and 1.96 (C0246 0.97, C0129 1.02 and C0230 1.10 are right on their normal rhythm).

| Customer | Type | Region | Days quiet | Normal gap | Ratio | Spend |
|---|---|---|---|---|---|---|
| C0288 | Dealer | North | 75 | 38.26 | 1.96 | 76,291,160 |
| C0083 | Dealer | North | 89 | 44.57 | 2.00 | 70,378,740 |
| C0221 | Dealer | North | 118 | 47.86 | 2.47 | 64,336,800 |
| C0246 | Retail | North | 60 | 61.97 | 0.97 | 60,399,770 |
| C0129 | Corporate | North | 62 | 60.71 | 1.02 | 59,941,820 |
| C0154 | Fleet Owner | North | 133 | 46.16 | 2.88 | 58,187,170 |
| C0476 | Fleet Owner | North | 94 | 63.21 | 1.49 | 57,777,740 |
| C0337 | Corporate | North | 129 | 59.36 | 2.17 | 57,585,510 |
| C0342 | Dealer | North | 84 | 50.60 | 1.66 | 57,066,210 |
| C0311 | Government | North | 106 | 51.32 | 2.07 | 54,370,990 |
| C0225 | Fleet Owner | North | 168 | 41.47 | 4.05 | 54,324,600 |
| C0192 | Fleet Owner | East | 125 | 52.54 | 2.38 | 54,295,880 |
| C0119 | Government | West | 67 | 42.38 | 1.58 | 52,748,640 |
| C0341 | Fleet Owner | North | 86 | 58.23 | 1.48 | 52,480,080 |
| C0428 | Retail | North | 128 | 48.57 | 2.64 | 52,052,170 |
| C0116 | Dealer | North | 157 | 59.65 | 2.63 | 50,227,070 |
| C0135 | Retail | South | 127 | 58.76 | 2.16 | 49,182,970 |
| C0230 | Dealer | North | 62 | 56.24 | 1.10 | 48,757,760 |
| C0318 | Corporate | North | 76 | 60.26 | 1.26 | 48,754,200 |
| C0090 | Government | North | 71 | 48.74 | 1.46 | 40,810,370 |

By region 17 North, 1 East, 1 West, 1 South; by type Dealer 6, Fleet Owner 5, Corporate 3, Retail 3, Government 3 (no single type).

Because the At risk rule needs a high FM, three further tests separate value from quietness.

**Stricter branch test** (quiet = more than twice their own normal gap; overall 67 of 500 = 13.4%; strict line 0.05 / 6 = 0.0083):

| Branch | Customers | Quiet beyond 2x | Share | Expected | p-value | Median recency days | Average FM |
|---|---|---|---|---|---|---|---|
| PUN001 | 75 | 4 | 5.3% | 10.0 | 0.0405 | 48.0 | 2.77 |
| CHN001 | 102 | 20 | 19.6% | 13.7 | 0.0793 | 50.0 | 2.98 |
| HYD001 | 68 | 11 | 16.2% | 9.1 | 0.4766 | 47.0 | 2.92 |
| AHM001 | 88 | 10 | 11.4% | 11.8 | 0.7535 | 47.0 | 2.85 |
| DEL001 | 71 | 10 | 14.1% | 9.5 | 0.8613 | 46.0 | 3.36 |
| KOL001 | 96 | 12 | 12.5% | 12.9 | 1.0000 | 44.5 | 2.83 |

Delhi is not quieter than the other branches (14.1% against 13.4% overall, p-value 0.86; median recency 46 days against 44.5 to 50 elsewhere). Its high At risk count comes partly from higher customer value (average FM 3.36 against 2.77 to 2.98 elsewhere). No branch is strongly different on quietness (PUN001 0.0405 and CHN001 0.0793 are not strong).

**Value mix versus quietness by branch** (high value = FM 3.5 or more; quiet = recency score 1 or 2, more than 59 days since the last order):

| Branch | Customers | High value | High-value share | Of these quiet | Quiet share of high value |
|---|---|---|---|---|---|
| AHM001 | 88 | 33 | 37.5% | 13 | 39.4% |
| CHN001 | 102 | 42 | 41.2% | 14 | 33.3% |
| DEL001 | 71 | 43 | 60.6% | 20 | 46.5% |
| HYD001 | 68 | 24 | 35.3% | 8 | 33.3% |
| KOL001 | 96 | 36 | 37.5% | 10 | 27.8% |
| PUN001 | 75 | 29 | 38.7% | 5 | 17.2% |

All branches: 207 high-value customers (41.4% of 500), of whom 70 are quiet (33.8%); these 70 are the At risk segment. For Delhi the expected At risk count is 9.9 from its size, 14.5 when its own 43 high-value customers are used with the overall quiet rate of 33.8%, and the actual count is 20. Of the 10.1 extra At risk customers, about 4.6 come from more high-value customers and about 5.5 from a higher quiet share among them (roughly half and half). Quietness here is raw days, while the stricter test used each customer's own normal gap, so the two views differ.

**Chance test of the quiet share among high-value customers** (overall 33.8%; strict line 0.0083):

| Branch | High value | Quiet | Expected quiet | p-value |
|---|---|---|---|---|
| PUN001 | 29 | 5 | 9.8 | 0.0757 |
| DEL001 | 43 | 20 | 14.5 | 0.1055 |
| KOL001 | 36 | 10 | 12.2 | 0.4869 |
| AHM001 | 33 | 13 | 11.2 | 0.5812 |
| HYD001 | 24 | 8 | 8.1 | 1.0000 |
| CHN001 | 42 | 14 | 14.2 | 1.0000 |

Delhi's higher quiet share among its valuable customers (46.5% against 33.8%) is not proven (p-value 0.1055, could be luck).

**Chance test of the high-value share by branch** (overall 41.4%; strict line 0.0083):

| Branch | Customers | High value | Share | Expected | p-value |
|---|---|---|---|---|---|
| DEL001 | 71 | 43 | 60.6% | 29.4 | 0.0015 |
| HYD001 | 68 | 24 | 35.3% | 28.2 | 0.3271 |
| KOL001 | 96 | 36 | 37.5% | 39.7 | 0.4695 |
| AHM001 | 88 | 33 | 37.5% | 36.4 | 0.5165 |
| PUN001 | 75 | 29 | 38.7% | 31.0 | 0.7253 |
| CHN001 | 102 | 42 | 41.2% | 42.2 | 1.0000 |

Tested Delhi findings: Delhi has significantly more high-value customers (60.6% against 41.4%, p-value 0.0015); its At risk share is 28.2% against 14.0% (p-value 0.0017); it is not later than its own rhythm (14.1% against 13.4%, p-value 0.86) and its median recency (46 days) is normal; a higher quiet share among its valuable customers is possible but not proven (p-value 0.1055). Delhi is the branch most dependent on high-value customers: 20 of its 43 high-value customers have been quiet for more than 59 days.

### Finding: Frequency, Money and Recency overlap

Rank correlation: Recency with Frequency -0.14; Recency with Monetary -0.10; Frequency with Monetary 0.70. Frequency and Money move together strongly but stay below 0.8, so the FM average is fair (each still adds information, though a good part overlaps). Recency is almost independent of both (more recent customers are slightly more frequent and higher spending).

### Finding: the segments are a snapshot and not a prediction

Only 31.0% of customers keep their segment after one year; recency has no memory (correlation -0.01); 2023 and 2024 are unrelated in orders (-0.04) and money (0.04); the spread of order counts equals the chance spread; random customers give 51 to 93 quiet customers (real 67) and 54 to 88 At risk customers (real 70). The measurement is correct, and the groups are useful for follow-up, but the data show no evidence that At risk customers are really leaving or that Champions are really better customers.

### Recommended actions for the business owners

1. **Head of Sales:** call the 21 firm At risk customers (see the list in Check 5) this week. They normally order every 41 to 56 days and have been silent for 101 to 204 days; together they hold 1,164,484,450 (5.07% of money). Start with C0225 (4.05 times its normal gap), C0110 (4.03), C0041 (3.82), C0421 (3.77) and C0489 (3.23), and log the answer of each call.
2. **Delhi Branch Manager:** review the 20 At risk customers of branch DEL001 (1,119,969,650, 4.88% of money). Call the 9 beyond twice their own gap first (C0225, C0154, C0428, C0116, C0221 have the largest ratios); the 11 others are within about their normal rhythm and need only a routine check. Make an account plan for the branch's 43 high-value customers (60.6% of its customers).
3. **Customer Analytics Lead:** re-run the segments every month with the current date and keep both views (the rule-based segment and the silence ratio), because only 31.0% of customers keep their segment after a year and the number quiet beyond twice their gap moves from 67 to 123 when the date moves 30 days.
4. **Commercial Director:** do not start paid loyalty or VIP programmes on past order counts alone, because customers cannot be told apart from chance (spread 6.04 against 6.01; 2023 and 2024 unrelated). Test any programme on a small group with a comparison group first.
5. **Customer Data Owner:** record the result of every follow-up call (reason, next order date) in the data and repair or retire customer_since, last_purchase_date and total_purchase_value, so that the next analysis can test whether outreach brings customers back (38 of the 75 customers At risk a year ago are Champions or Loyal today, with no data to explain why).

## Glossary

- **RFM:** Recency, Frequency, Monetary: how recently, how often and how much a customer buys.
- **Delivered order:** an order with status Delivered (counted as a sale). **Cancelled order:** an order with status Cancelled (left out).
- **Snapshot date:** the date recency is measured from (2025-01-01).
- **Recency:** days from the last Delivered order to the snapshot date. **Frequency:** number of Delivered orders. **Monetary:** sum of order values before GST.
- **Score (1 to 5):** the customer's group when customers are split into five groups of about equal size; 5 is best (for Recency, fewest days).
- **FM:** average of the frequency score and the money score.
- **Champions, Loyal, Recent but small, Average, Quiet and low value, At risk (valuable but quiet):** the six segments (see Methodology for the rules).
- **High value:** FM of 3.5 or more. **Quiet:** recency score 1 or 2 (more than 59 days since the last order).
- **Normal gap:** a customer's own average number of days between orders.
- **Silence ratio:** recency days divided by the normal gap; above 2 means quiet beyond twice their own gap.
- **Firm At risk group:** the 21 customers who are At risk under every cut point tested.
- **Chance test (binomial) and p-value:** a test of whether a group's count differs from what luck would give; a small p-value means luck is unlikely. **Strict line:** 0.05 divided by the number of groups tested.
- **Rank correlation:** how closely two rankings move together (1 together, 0 unrelated, -1 opposite). **Spread (std):** how much values differ from each other.
- **Same score / within one step:** customers whose score is identical, or differs by at most 1, between two dates.
- **Cut point:** a line in a segment rule, such as FM 3.5. **Sensitivity:** moving a cut point or date a little to see what changes.
- **Second method:** rebuilding the numbers with different code to confirm the first result. **Simulation:** inventing random customers with the same average behaviour to see what luck alone gives.
- **Header and lines:** the order header holds the order total; the lines hold each product line of the order.
