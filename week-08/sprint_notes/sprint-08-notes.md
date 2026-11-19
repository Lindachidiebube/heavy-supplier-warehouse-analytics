# Sprint 08 Notes

Project: Heavy Supplier & Warehouse Analytics

Sprint: Week 8 (Phase 3: Supplier, Customer & Predictive Analytics)

## Sprint Goal

- Answer four customer questions with four KPIs, each built on the same Delivered orders and invoice data and each verified with 8 checks before it was written up: which customers matter most and which are slipping away (KPI 14 RFM Segmentation), how much money each customer has brought in over the whole history (KPI 15 Historical Customer Value), which customers put cash flow at risk and how much money is involved (KPI 16 Customer Payment Risk), and whether the customers who go quiet are the ones who pay badly (KPI 17 Churn by Payment Behavior).

## Work Completed

### KPI 14 RFM Segmentation

- Scored all 500 customers on recency, frequency and money from 18,033 Delivered orders (snapshot date 1 Jan 2025, the day after the last order), using five equal groups for each score, and sorted them into six segments by rules: Champions 66, At risk 70, Loyal 71, Quiet and low value 127, Recent but small 114, Average 52.
- The At risk segment (valuable but quiet) holds 3,859,948,290 of spend, 16.8% of revenue. Measured against each customer's own normal gap between orders, 67 customers are quiet for more than twice their own gap (13.13% of revenue), and a firm group of 21 customers stays At risk under every rule tested (5.07% of revenue, 19 of the 21 above twice their own gap). That firm group is the first call list.
- The size of the At risk group depends on the cut points chosen: moving each cut point one at a time gave a range of 33 to 92 customers around the 70.
- Branch DEL001 (Delhi) was the one tested difference: 60.6% of its customers are high value against 41.4% overall (p 0.0015) and its At risk share is 28.2% against 14.0% (p 0.0017). Delhi customers are not later than their own rhythm (14.1% quiet against 13.4% overall, p 0.86), and a higher quiet share among its valuable customers (46.5% against 33.8%) is possible but not proven (p 0.1055). No other customer type, region, payment term or industry showed a tested difference.
- Frequency and money move together (rank correlation 0.70) while recency is nearly independent of both. Customers do not differ in how often or how much they order beyond what chance produces (order counts in 2023 and 2024 are unrelated: correlations -0.04 and 0.04), and 67 quiet customers sit inside the range that randomly ordering customers give (51 to 93, average 72.0). Segments are therefore a point-in-time view: 155 of 500 customers (31.0%) kept the same segment a year later.
- All 8 checks agree: blank, zero and negative values, edge cases, repeated orders, totals (22,958,987,430 equals the line revenue), a second method with 0 differences for all 500 customers, outliers (no single order decides a customer's segment), history and dates (72 months), and a hand check of C0001, C0002 and C0003 from the raw files.

### KPI 15 Historical Customer Value

- Ranked all 500 customers by money brought in over 2019 to 2024 (22,958,987,430 before GST). Customer money runs from 17,721,530 (C0347) to 84,966,970 (C0153), with an average of 45,917,975.
- Money is spread out: the top 10 customers hold 3.23%, the top 50 hold 14.33% and the top 100 hold 26.63%, against an even spread of 2%, 10% and 20%.
- Money by customer type, branch, region and industry segment was measured for who, where and how much. Government has the most customers (114) and none in the top 20, Dealer has the highest average per customer, DEL001 has the highest value per customer (49,296,215), and no branch or industry dominates. Central and North East have only 11 customers each, so their averages are less reliable.
- Money per year is flat between 3.68 and 3.92 billion, and 2024 is the lowest year (3.84% below 2023). All 500 customers bought in 2024.
- All 8 checks agree: the second method differs by 0, 19 of the top 20 customers stay in the top 20 when each customer's largest order is removed (C0359 falls from rank 17 to 26 and C0490 enters), all 72 months are present, and a hand check of C0153, C0347 and C0436 matches the raw orders file. The ranking describes the past; KPI 14 found customer money in 2023 and 2024 to be unrelated.

### KPI 16 Customer Payment Risk

- Billed 28,599,637,860.60 (including GST), paid 22,795,333,903.87, outstanding 5,804,303,956.73, which is about 20.3% of billed money. The money figures use 17,640 clean invoices (393 invoices left out for repeated invoice IDs, about 2.2% of money) and 18,840 payments (417 left out because they point to a repeated invoice ID).
- A first run showed 306 Paid invoices with money outstanding and 88 odd Partially Paid invoices. The cause was tested: all of them had one of 395 payments that had been left out only because their payment ID is repeated. With those payments included, every mismatch count is 0. The late payment rate of KPI 4 stands (41.50% on the KPI 4 payments, 41.51% on all 18,840).
- The late share depends on the due period: 78.57% for 15-day invoices, 57.40% for 30 days, 34.04% for 45 days and 7.16% for 60 days. Customers pay at the same pace whatever the due period (median 34 to 36 days, p 0.3861), so a short due period brings cash in no faster.
- Inside each due period group, customers do not differ in lateness beyond luck (25 customers below p 0.05 against about 25 expected), and the unpaid invoice share is also within luck. No group differs in outstanding money across due period, customer type, branch, region or industry (27 groups, smallest p 0.0944).
- The 100 most valuable customers (KPI 15 ranking) hold 26.03% of the outstanding money against 26.74% of the billing, so they are not riskier, and value rank and outstanding share are unrelated (correlation 0.042, p 0.3444). A call list of the 20 valuable customers with the biggest balances holds 461,987,109.14 (7.96% of the outstanding money).
- The unpaid money is old: 99.34% of it is older than 90 days and 88.01% is over a year old, and no payment in the data arrived later than 90 days after its invoice. The leak is steady, between 19.17% and 21.52% of billing in every invoice year 2019 to 2024.
- Check 3 showed that repeated payment IDs are unrelated payments, not copies (193 shared IDs, all pointing to different invoices, 192 of them across different customers). A rise in the late rate by payment date in 2025 (58.19% for 342 payments against 39.78% to 43.20% in 2019 to 2024) comes from the end of the data and not from customers paying slower. A hand check of C0297 showed all 14 of its Paid invoices were paid late, 1 to 45 days after the due date.

### KPI 17 Churn by Payment Behavior

- Our data has no field that says a customer has left (checked on all four tables: customers 15 columns, orders 11, invoices 10, payments 5; the only status columns describe an order and an invoice), so churn is measured by silence. A customer is quiet when it has been silent for more than twice its own normal gap: 67 of 500 customers (13.4%), and all 23 customers whose last order is on or before 30 Jun 2024 are inside the 67.
- No link was found between payment behaviour and going quiet. Quiet customers paid 45.53% of invoices late against 44.90% for the others (p 0.9290), left 32.01% of invoices not fully paid against 30.37% (p 0.2787), and owe 21.45% of their billing against 20.23% (p 0.3554). Before they went quiet (invoices before 1 Jul 2024) they did not pay worse either (late 45.35% against 44.69%, p 0.8945). Bad payers are not more often quiet: the quiet share is between 12.0% and 13.6% whether a bad payer is the top 10%, 25% or half of customers by late rate (all p 1.0). Quiet and other customers have the same due period mix (p 0.9511), so their late rates can be compared fairly.
- The quiet customers owe 804,728,429 (13.86% of all unpaid money) and the 23 owe 266,055,010 (4.58%). Random groups of the same size reach their owed share in 8.3% and 7.5% of draws, so the higher share is not proven.
- Every customer placed its last order in 2024, but 25 customers have been silent for more than 180 days and the longest silence is 359 days. The quiet count depends on the snapshot date: 42 at 1 Dec 2024, 67 at 1 Jan 2025, 125 at 1 Feb 2025, 201 at 1 Mar 2025 and 494 at 1 Jun 2025.
- The result held when the quiet line was moved to 1.5 and 2.5, when the borderline customer C0299 was removed, when each customer's largest invoice was removed, and when the 136 invoices dated after 20 Dec 2024 (less than 90 days to be paid) were removed. The smallest p value in 15 sensitivity comparisons was 0.0766.
- All 8 checks agree: no blank or negative values, no duplicates, billed, paid and outstanding money equal the KPI 16 totals with difference 0, a plain-loop second method gave the same 67 quiet customers, and a hand check of C0299, C0002 and C0029 from the raw files matches the tables.
- The business answer: use silence, and not late or unpaid invoices, to find customers who may be slipping away; the 25 customers silent for more than 180 days are the first to contact.

## Blockers & Resolutions

- KPI 14: early readings of the Delhi branch were corrected by tests. Delhi is not quieter than its own rhythm, and its At risk count is about half more high-value customers and half a possibly higher quiet share that is not proven. Remarks about Advance payers, Dealers and Retail customers were dropped because the tests did not support them.
- KPI 15: after the wrong cell was deleted, the second-method test failed with a missing variable. It was fixed by replacing the cell with one complete second-method cell and running the notebook again from the top.
- KPI 16: the 306 Paid invoices that still showed money outstanding were traced to payments left out for repeated payment IDs, and the money rules were rebuilt so that status and payment sums agree for every clean invoice.
- KPI 17: rounding the normal gap before dividing gave 66 quiet customers instead of 67 and lost C0299 (exact ratio 2.00098). The exact ratio is used from then on.

## Next Sprint Focus

- Week 9 (Forecasting): KPI 18 Product Demand Forecast, built from units sold on Delivered sales lines, and KPI 19 Stockout Risk and Reorder Point (average daily demand multiplied by supplier lead time, plus safety stock), built on the reconstructed stock balance of KPI 8.
