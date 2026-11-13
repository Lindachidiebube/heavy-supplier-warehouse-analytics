# KPI 15: Historical Customer Value

Project: Heavy Supplier & Warehouse Analytics | Week 8 | Notebook: Week8_KPI15_Historical_Customer_Value

---

## Summary

- KPI 15 measures how much money each of the 500 customers has brought in over the whole history (1 Jan 2019 to 31 Dec 2024), ranks them from biggest to smallest, and shows how spread out the money is.
- Money means the order value before GST on the 18,033 Delivered orders: 22,958,987,430 in total. The 1,967 Cancelled orders are left out on purpose.
- Customer values run from 17,721,530 to 84,966,970 (the biggest is about 4.8 times the smallest). The average is 45,917,975 and the middle customer has 45,321,980. The biggest customer is C0153 (Retail, branch PUN001) with 84,966,970 from 56 orders.
- The money is spread out. The top 10 customers hold 3.23% of the money, the top 50 hold 14.33% and the top 100 hold 26.63%. An even spread would give 2%, 10% and 20%. The business does not depend on a few customers.
- Customer type, branch, region and industry all show similar value per customer. Branch DEL001 (Delhi) has the highest value per customer (49,296,215) with the second smallest number of customers (71).
- Money per year is flat, between 3.68 and 3.92 billion. 2024 is the lowest year, 3.84% below 2023 and 6.11% below the best year (2020).
- All 8 verification checks agree (see Verification Log). Two ways of calculating give identical numbers, every cut adds back to the total, and three customers rebuilt from the raw orders file match the table.
- The ranking describes the past six years. KPI 14 tested this: a customer's money in 2023 and in 2024 are unrelated (rank correlation 0.04), so the ranking is a record of history and not a prediction.

---

## 1. KPI Definition

### Metrics

| Metric | Definition | Unit |
|---|---|---|
| Historical customer value | Sum of total_order_value (order value before GST) over the customer's Delivered orders, 1 Jan 2019 to 31 Dec 2024 | money |
| Customer rank | Position when all 500 customers are sorted by historical customer value (1 = most money, 500 = least) | rank |
| Orders per customer | Number of Delivered orders of the customer | orders |
| Average order value per customer | Historical customer value divided by orders per customer | money per order |
| Concentration | Share of the total money held by the top 10, top 50 and top 100 customers, next to the share an even spread would give (10 / 500 = 2%, 50 / 500 = 10%, 100 / 500 = 20%) | percent |
| Money by customer group | Customers, orders, money, share of money and average money per customer for each customer type, branch, region and industry segment | money, percent |
| Money per year | Orders, customers buying, money, average order value, share of money and change against the previous year, for 2019 to 2024 | money, percent |

### Business question

Which customers have brought in the most money over the whole history, and how dependent is the business on the biggest ones?

### Why this metric

KPI 14 sorted customers by how recently, how often and how much they buy, and showed that the money per customer is one of its three inputs. KPI 15 answers a different question: who are the customers behind the money, how is the money spread across them, and across customer types, branches, regions, industries and years. It tells the business how exposed it is if one or a few customers leave, and where the valuable customers sit.

### Scope

- Included: the 18,033 Delivered orders from the sales orders header, all 500 customers, 1 Jan 2019 to 31 Dec 2024.
- Not included: Cancelled orders (1,967), GST, any prediction of future value. "Historical" means this KPI describes the past only.

---

## 2. Methodology

### Sources and scoping

| Source | What is used | What is not used |
|---|---|---|
| Sales orders header (loaded as so_h, 20,000 rows, 11 columns) | Delivered rows only (18,033): so_id, customer_id, order_date, total_order_value, order_status, branch_id | Cancelled rows (1,967); GST amounts |
| Customers table (500 rows, 15 columns) | customer_id, customer_type, branch_id, region, industry_segment | customer_since, last_purchase_date and total_purchase_value (Week 3 found they do not match the transactions) |

Money is taken from the order header (total_order_value, before GST), the same money as KPI 7 and KPI 14. Dates and money come from the orders, never from the customers table's own summary columns.

### What KPI 15 reuses from KPI 14 and what is new

| Reused from KPI 14 (shown only with its tie-out, not re-tested from scratch) | New in KPI 15 |
|---|---|
| The Delivered orders table: 18,033 orders, 500 customers, 2019-01-01 to 2024-12-31 | Named ranking of all 500 customers (top 20 list) |
| Money per customer (Monetary): total 22,958,987,430, range 17,721,530 to 84,966,970, average 45,917,975 | Concentration: share of money held by the top 10, 50 and 100 customers |
| Overall average order value 1,273,165 | Money by customer type, branch, region and industry segment |
| | Money, orders and customers buying per year, 2019 to 2024 |
| | Orders and average order value per customer, and the customers with the biggest and smallest average order |

### Steps

| Step name | What it does |
|---|---|
| Load tables | Mounts the drive, loads the orders header and customers, builds the Delivered orders table (sales_h) and prints row counts and totals |
| Ranking | Adds the money of each customer, sorts from biggest to smallest, adds the rank, ties the total back to the orders, prints the top 20 with type and branch |
| Concentration | Adds the money of the top 10, 50 and 100 customers and shows each as a share of the total next to the even share |
| Money by customer type, branch, region, industry | Attaches the customer details to the ranking and groups: customers, orders, money, average per customer, share of money; each group table is tied back to the total |
| Money per year | Groups the Delivered orders by order year: orders, customers buying, money, average order value, share, change against the previous year; tied back to the total |
| Orders and order size per customer | Describes the spread of orders, average order value and money across the 500 customers; lists the five biggest and five smallest average orders |
| Verification checks | The 8 verification checks and their supporting tests (see Verification Log) |

### Alternatives considered

| Alternative | Decision and reason |
|---|---|
| Use customers.total_purchase_value for customer money | Rejected. Week 3 found it differs by more than 1% from the real orders for 500 of 500 customers |
| Include Cancelled orders in customer money | Rejected. Cancelled orders have no invoice and brought no money; they are 1,967 orders, kept out of every total |
| Money including GST | Rejected. Money before GST is the number used in KPI 7 and KPI 14, so KPIs stay comparable |
| Rank customers by number of orders | Rejected. The question is about money; orders are shown next to money in the ranking |
| Money from invoices or payments | Rejected. Invoice IDs have 393 repeated rows and payment IDs 403 (Week 3); the Delivered orders are complete (one per invoice) and need no repair |
| Predict future customer value (lifetime value model) | Rejected. KPI 15 describes the past; KPI 14 found no evidence that a customer's past money predicts next year |
| Only report the 80/20 rule | Rejected as the only view. The top 20% of customers (100 customers) hold 26.63% of the money, so 80/20 does not apply; top 10, 50 and 100 shares are reported next to an even spread |

---

## 3. Verification Log

### Summary

| # | Check | Method | Result |
|---|---|---|---|
| 1 | Blank, zero and negative values | Count empty cells, blank text, zero or negative values in every column used | No empty cells, no blank text, no zero or negative values |
| 2 | Edge cases | Orders without a known customer, customers without orders, customers with a year without orders | No orphan orders, every customer has orders; 4 empty customer-year spots out of 3,000, all explained |
| 3 | Duplicates | Repeated order IDs, repeated customer IDs, orders with the same customer, date and value | No repeats anywhere |
| 4 | Totals reconcile | Delivered orders and money against the orders file; every cut added back to the total | Everything adds to 18,033 orders and 22,958,987,430, difference 0 in every cut |
| 5 | Second-method recompute | Ranking by pivot table, groups by attaching details to each order first, years by date ranges, shares by largest-N, average order value against KPI 14 | Identical in every comparison, difference 0 |
| 6 | Outliers | Largest order against the customer's average, share of money from the largest order, ranking without each customer's largest order | No extreme orders; 19 of the top 20 stay in the top 20 |
| 7 | History and date coverage | 72 months, first and last month complete, first and last order of each customer, late starters and early enders | All 72 months present and complete; every customer active in 2019 and 2024; history length tested |
| 8 | Manual spot-check of 3 customers | Rebuild C0153, C0347 and the customer at rank 250 (C0436) from the raw orders file with a plain loop | Identical to the table on every field for all three |

### Check 1: Blank, zero and negative values

- Delivered orders table (so_id, customer_id, order_date, total_order_value): 0 empty cells in each column.
- Customers table (customer_id, customer_type, branch_id, region, industry_segment): 0 empty cells and 0 blank text in each column.
- Orders with a value of zero or less: 0. Customers with money of zero or less: 0. Customers with zero orders: 0.
- Result: no empty cells, no blank text, no zero or negative numbers. No order or customer is wrongly counted or left out because of bad values.

### Check 2: Edge cases

- Orders whose customer is not in the customers table: 0. Customers with no Delivered order: 0.
- Customer-and-year spots with no order: 4 out of 3,000 (500 customers times 6 years). This matches the year table (498 customers bought in 2021 and in 2022).

| Customer | 2019 | 2020 | 2021 | 2022 | 2023 | 2024 |
|---|---|---|---|---|---|---|
| C0133 | 3 | 5 | 0 | 9 | 9 | 3 |
| C0143 | 7 | 5 | 0 | 5 | 5 | 5 |
| C0344 | 6 | 8 | 4 | 0 | 3 | 7 |
| C0482 | 1 | 6 | 8 | 0 | 7 | 9 |

- Result: all four bought in every other year, including 2024, so none has stopped buying. A year with no order is very rare (4 of 3,000).

### Check 3: Duplicates

- Rows in the Delivered orders table: 18,033. Unique order IDs: 18,033. Repeated order IDs: 0.
- Repeated customer IDs in the customers table: 0. Repeated customer IDs in the ranking: 0.
- Orders sharing the same customer, date and value with another order: 0.
- Result: nothing is counted twice, and no sign of an order entered twice under different IDs.

### Check 4: Totals reconcile

- Orders in the orders file by status: Delivered 18,033, Cancelled 1,967, together 20,000 (all rows).
- Delivered orders straight from the orders file: 18,033. Delivered money straight from the orders file: 22,958,987,430. Orders and money used in this KPI: 18,033 and 22,958,987,430. Difference: 0.
- Every cut adds to 22,958,987,430 with a difference of 0: customer ranking (500 customers), customer type, branch, region, industry segment, year (orders over the six years add to 18,033).
- KPI 14 showed that the header value equals the sum of the order lines for every Delivered order (difference 0), so this money agrees with line revenue (KPI 7).
- Result: the same money appears in every table; nothing was lost or added when the data was sliced.

### Check 5: Second-method recompute

- Money per customer by pivot table (instead of group-by): 500 customers, largest difference 0, the same customer (C0153) first.
- Money by customer type, branch, region and industry segment, calculated by attaching the customer details to each order first and then adding: largest difference 0 in all four. The customer's branch column was renamed before attaching, so it does not clash with the order's own branch column.
- Orders whose own branch differs from the customer's branch: 0 (see Data Quality Notes).
- Money per year by date ranges (1 Jan to 31 Dec of each year) instead of the order year: 2019 3,870,016,600; 2020 3,917,532,120; 2021 3,852,479,620; 2022 3,816,288,220; 2023 3,824,675,310; 2024 3,677,995,560; difference to the year table 0 for every year.
- Top 10, 50 and 100 shares by largest-N: 3.23%, 14.33% and 26.63%, the same as before.
- Overall average order value: 1,273,165, the same as KPI 14.
- Result: the two ways agree exactly everywhere.

### Check 6: Outliers

- Largest single order in the table: 8,169,760, about 6.4 times the overall average order value (by hand: 8,169,760 / 1,273,165 = 6.42).
- A customer's largest order compared with their own average order: middle 3.23 times, highest 5.86 times (C0225).
- Share of a customer's money that comes from their largest order: middle 9.08%, highest 17.38% (C0349). An ordinary order would be about 2.8% of a customer's money (about 36 orders), but the largest of about 36 orders is naturally several times bigger.
- Top 20 customers: the highest share of money from one order is 11.53%.
- Ranking without each customer's largest order: 19 of the top 20 stay in the top 20.

| Customer | Original rank | Largest order | Share from largest order | Rank without it |
|---|---|---|---|---|
| C0359 (dropped out of the top 20) | 17 | 7,752,220 | 11.53% | 26 |
| C0490 (came in) | 21 | 4,257,600 | 6.40% | 20 |

- Result: no extreme orders distort the results. C0359 is the one top 20 customer whose position leans most on a single order; it falls 9 places and stays within the top 30. C0490 moves up from rank 21 to 20, so the change is a close swap at the edge of the top 20.

### Check 7: History and date coverage

- Months with orders: 72 of 72. Fewest orders in a month 209 (May 2024), most 312 (July 2020), average 250.5 (18,033 / 72 by hand).
- First month (January 2019): 254 orders. Last month (December 2024): 223 orders. Days with orders: 31 in the first month, 31 in the last month, 31 in the middle month. Both months are complete.
- Earliest first order among customers: 1 Jan 2019. Latest first order: 18 Dec 2019. Earliest last order: 8 Jan 2024. Latest last order: 31 Dec 2024. Every customer started buying in 2019 and has a last order in 2024.
- Customers whose first order is after 30 Jun 2019: 31. Customers whose last order is before 1 Jul 2024: 23. Two customers are in both groups (by hand: 448 customers are in neither group, so 500 - 448 = 52 customers are in at least one group, and 31 + 23 - 52 = 2).

Supporting test: does a shorter history make customers look worse?

| Group | Customers | Average orders | Average money | Average order value | In the bottom 50 customers |
|---|---|---|---|---|---|
| First order after 30 Jun 2019 | 31 | 35.1 | 46,313,703 | 1,327,312 | 3 |
| Last order before 1 Jul 2024 | 23 | 31.9 | 41,017,466 | 1,284,819 | 4 |
| Neither | 448 | 36.3 | 46,118,212 | 1,270,065 | 43 |

- Late starters look like everyone else: 3 of 31 are in the bottom 50 (about 10%, which is what chance gives: 50 of 500).
- Early enders have about 11% less money, from fewer orders (31.9 against 36.3), not from smaller orders (similar average order value). 4 of 23 (17%) are in the bottom 50. It is a small group.
- Result: the six years are fully covered; a late start does not hurt a customer's total; the 23 early enders rank a little lower because they stopped ordering sooner, which is correct for a historical ranking.

### Check 8: Manual spot-check of 3 customers

Three customers rebuilt from the orders table as loaded straight from the raw file, with the money added by a plain loop instead of the pandas sum: the biggest customer (rank 1), the smallest (rank 500) and the customer in the middle (rank 250).

| Field | C0153 (rank 1) | C0347 (rank 500) | C0436 (rank 250) |
|---|---|---|---|
| Orders: raw file / table | 56 / 56 | 24 / 24 | 34 / 34 |
| Money: raw file / table | 84,966,970 / 84,966,970 | 17,721,530 / 17,721,530 | 45,357,190 / 45,357,190 |
| Largest order: raw file / table | 6,716,250 / 6,716,250 | 1,928,300 / 1,928,300 | 2,984,530 / 2,984,530 |
| First and last order: raw file | 2019-01-17 and 2024-12-12 | 2019-03-04 and 2024-12-03 | 2019-02-01 and 2024-12-15 |
| First and last order: table | 2019-01-17 and 2024-12-12 | 2019-03-04 and 2024-12-03 | 2019-02-01 and 2024-12-15 |
| Type and branch (customers file) | Retail, PUN001 | Retail, DEL001 | Fleet Owner, KOL001 |
| Cancelled orders left out | 3, worth 910,050 | 3, worth 5,130,870 | 4, worth 4,367,080 |

- Result: the raw file and the table are identical on every field for all three customers.

### Supporting tests

| Supporting test | Belongs to | Result |
|---|---|---|
| Loading tables (row counts, columns, Delivered orders) | Methodology | Orders table 20,000 rows by 11 columns; customers 500 by 15; Delivered orders 18,033; Delivered money 22,958,987,430; customers 500 (all match Week 3 and KPI 14) |
| Concentration against an even spread | Business Interpretation | Top 10 3.23% (even 2.00%), top 50 14.33% (even 10.00%), top 100 26.63% (even 20.00%) |
| Ranking without each customer's largest order | Check 6 | 19 of the top 20 stay; C0359 and C0490 swap |
| Does a short history make customers look worse | Check 7 | Late starters no; early enders about 11% less money from fewer orders |

---

## 4. Data Quality Notes

1. Only Delivered orders count as money. The 1,967 Cancelled orders have no invoice and are left out of every total. For the three customers checked by hand, the cancelled orders left out were 3 (worth 910,050), 3 (worth 5,130,870) and 4 (worth 4,367,080). For C0347 the cancelled orders equal about 29% of its Delivered money (5,130,870 / 17,721,530, by hand).
2. Money is the order value before GST (total_order_value), the same money as KPI 7 and KPI 14.
3. The customers table's own summary columns (customer_since, last_purchase_date, total_purchase_value) are not used. Week 3 found they do not match the transactions. All dates and money come from the Delivered orders.
4. The money per customer is the same number as the Monetary value in KPI 14: total 22,958,987,430, smallest 17,721,530, biggest 84,966,970, average 45,917,975. The overall average order value 1,273,165 is the same as in KPI 14.
5. There are five customer types (Government, Corporate, Dealer, Fleet Owner, Retail), not four.
6. Every Delivered order was placed at the same branch as its customer's branch (0 orders differ), so the branch view is the same whether the order's or the customer's branch is used. Branch and region are different columns: a customer can be in branch DEL001 and region West.
7. Two regions are very small: Central and North East have 11 customers each (2.19% and 2.07% of the money). Their averages are less reliable and should not be compared with the larger regions.
8. Four customer-and-year spots have no order (C0133 and C0143 in 2021, C0344 and C0482 in 2022); each of these customers bought in every other year. 4 of 3,000 spots is very rare.
9. History length: every customer started buying in 2019 (latest first order 18 Dec 2019) and has a last order in 2024 (earliest last order 8 Jan 2024). 31 customers started after 30 Jun 2019, 23 stopped before 1 Jul 2024, 2 are in both groups. A late start does not lower a customer's total; the 23 early enders have about 11% less money from fewer orders.
10. The average of the 500 customers' own average order values is 1,273,690. The overall average order value (all money divided by all orders) is 1,273,165. The two differ slightly because they are calculated differently, not because of an error.
11. 2024 is the lowest year (3,677,995,560), 3.84% below 2023 and 6.11% below 2020 (by hand: 239,536,560 / 3,917,532,120). It comes from slightly fewer orders (2,956 against 3,022, 2.2% fewer) and a smaller average order (1,244,247 against 1,265,611, 1.7% smaller). One lower year is not a trend.
12. The ranking describes the past six years only. KPI 14 found that a customer's money in 2023 and in 2024 are unrelated (rank correlation 0.04) and that the number of orders per customer varies only as much as chance produces (spread 6.04 against 6.01 expected from chance alone). The ranking should not be read as a prediction of next year.

---

## 5. Business Interpretation

### Business question and answer

Question: Which customers have brought in the most money over the whole history, and how dependent is the business on the biggest ones?

Answer: The biggest customer is C0153 (Retail, branch PUN001) with 84,966,970 from 56 orders. The business is not dependent on a few customers: the top 10 hold 3.23% of the money and the top 100 (a fifth of all customers) hold 26.63%. Value is spread across customer types, branches, regions and industries, and the money has been flat over the six years, with 2024 the lowest year.

### Ranking: top 20 customers

| Rank | Customer | Type | Branch | Orders | Money |
|---|---|---|---|---|---|
| 1 | C0153 | Retail | PUN001 | 56 | 84,966,970 |
| 2 | C0288 | Dealer | DEL001 | 55 | 76,291,160 |
| 3 | C0483 | Corporate | DEL001 | 47 | 75,352,940 |
| 4 | C0475 | Fleet Owner | KOL001 | 55 | 74,442,140 |
| 5 | C0495 | Fleet Owner | AHM001 | 48 | 74,165,320 |
| 6 | C0240 | Corporate | HYD001 | 46 | 72,778,580 |
| 7 | C0420 | Retail | CHN001 | 46 | 72,545,910 |
| 8 | C0110 | Dealer | CHN001 | 42 | 71,230,450 |
| 9 | C0083 | Dealer | DEL001 | 45 | 70,378,740 |
| 10 | C0206 | Retail | PUN001 | 40 | 70,279,540 |
| 11 | C0178 | Dealer | KOL001 | 54 | 70,236,970 |
| 12 | C0254 | Dealer | AHM001 | 48 | 70,227,620 |
| 13 | C0466 | Retail | DEL001 | 47 | 69,276,730 |
| 14 | C0207 | Corporate | KOL001 | 45 | 68,312,200 |
| 15 | C0046 | Retail | AHM001 | 46 | 68,236,200 |
| 16 | C0304 | Dealer | DEL001 | 48 | 67,903,880 |
| 17 | C0359 | Fleet Owner | HYD001 | 46 | 67,221,300 |
| 18 | C0134 | Fleet Owner | AHM001 | 48 | 66,943,890 |
| 19 | C0107 | Fleet Owner | DEL001 | 49 | 66,825,070 |
| 20 | C0142 | Retail | AHM001 | 45 | 66,580,100 |

- All 500 customers are ranked. Money added over the customers is 22,958,987,430, the same as the money in the Delivered orders (difference 0). Smallest customer 17,721,530, biggest 84,966,970, average per customer 45,917,975.
- The top 20 contain four of the five customer types (Retail, Dealer, Corporate, Fleet Owner); no type owns the top, and no Government customer is in the top 20 even though Government has the most customers (114). All twenty are between 66.6 million and 85 million, not far above the average. Branch DEL001 has 6 of the top 20 (C0288, C0483, C0083, C0466, C0304, C0107).

### Concentration

| Customers | Share of the money | Share if money were evenly spread | Times the even share (by hand) |
|---|---|---|---|
| Top 10 | 3.23% | 2.00% | 1.6 |
| Top 50 | 14.33% | 10.00% | 1.4 |
| Top 100 | 26.63% | 20.00% | 1.3 |

The money is spread out. The biggest customers hold a little more than an even share, but nothing like the 80/20 rule: the top 20% of customers (100 customers) hold 26.63%, not 80%. Losing a few big customers would not hurt badly.

### Spread of orders, order size and money across the 500 customers

| | Mean | Spread | Smallest | 25% | Middle | 75% | Biggest |
|---|---|---|---|---|---|---|---|
| Orders per customer | 36 | 6 | 21 | 32 | 36 | 40 | 56 |
| Average order value per customer | 1,273,690 | 197,849 | 738,397 | 1,131,625 | 1,269,734 | 1,395,526 | 1,955,791 |
| Money per customer | 45,917,975 | 10,298,204 | 17,721,530 | 38,446,610 | 45,321,980 | 52,438,192 | 84,966,970 |

The mean and the middle are close in all three rows, so a few giant customers are not pulling the averages. A customer's value comes from two things together: how many orders they place and how big each order is.

| Biggest average order | Orders | Average order value | Money | Smallest average order | Orders | Average order value | Money |
|---|---|---|---|---|---|---|---|
| C0234 | 29 | 1,955,791 | 56,717,930 | C0347 | 24 | 738,397 | 17,721,530 |
| C0491 | 27 | 1,894,105 | 51,140,830 | C0174 | 29 | 806,422 | 23,386,250 |
| C0482 | 31 | 1,866,024 | 57,846,740 | C0145 | 33 | 820,578 | 27,079,080 |
| C0498 | 28 | 1,815,909 | 50,845,460 | C0125 | 30 | 835,927 | 25,077,820 |
| C0191 | 29 | 1,778,374 | 51,572,860 | C0180 | 27 | 876,943 | 23,677,460 |

- The five customers with the biggest average order placed only 27 to 31 orders (fewer than the typical 36); their money (50.8 to 57.8 million) is above average but none is in the top 20.
- The top spender C0153 wins through volume (56 orders, the most of any customer). The smallest customer C0347 has both smaller orders and fewer orders (24).

### Money by customer type

| Customer type | Customers | Share of customers | Orders | Money | Average per customer | Share of money |
|---|---|---|---|---|---|---|
| Government | 114 | 22.8% | 4,075 | 5,049,342,920 | 44,292,482 | 21.99% |
| Corporate | 104 | 20.8% | 3,672 | 4,717,364,970 | 45,359,279 | 20.55% |
| Dealer | 99 | 19.8% | 3,633 | 4,683,087,090 | 47,303,910 | 20.40% |
| Fleet Owner | 95 | 19.0% | 3,457 | 4,411,907,790 | 46,441,135 | 19.22% |
| Retail | 88 | 17.6% | 3,196 | 4,097,284,660 | 46,560,053 | 17.85% |

Customers with no type: 0. Customers add to 500, orders to 18,033 (by hand: 4,075 + 3,672 + 3,633 + 3,457 + 3,196) and money to 22,958,987,430 (difference 0). Government is first in total only because it has the most customers; a Government customer is worth slightly less than the others. Dealer has the highest average and Government the lowest; the gap is about 7% (by hand: 47,303,910 / 44,292,482 = 1.068). No customer type is clearly more valuable than another.

### Money by branch

| Branch | Customers | Share of customers | Orders | Money | Average per customer | Share of money |
|---|---|---|---|---|---|---|
| CHN001 | 102 | 20.4% | 3,677 | 4,682,202,010 | 45,903,941 | 20.39% |
| KOL001 | 96 | 19.2% | 3,413 | 4,339,861,320 | 45,206,889 | 18.90% |
| AHM001 | 88 | 17.6% | 3,115 | 3,986,314,570 | 45,299,029 | 17.36% |
| DEL001 | 71 | 14.2% | 2,682 | 3,500,031,300 | 49,296,215 | 15.24% |
| PUN001 | 75 | 15.0% | 2,668 | 3,373,530,010 | 44,980,400 | 14.69% |
| HYD001 | 68 | 13.6% | 2,478 | 3,077,048,220 | 45,250,709 | 13.40% |

Customers with no branch: 0. Customers add to 500 and money to 22,958,987,430 (difference 0). No branch holds an unhealthy share (the biggest, CHN001, holds about 20%). CHN001 is first mainly because it has the most customers. DEL001 stands out: it has fewer customers (71) but the highest value per customer, about 9.6% above PUN001, the lowest (by hand: 49,296,215 / 44,980,400 = 1.096). The other four branches are close together (45.2 to 45.9 million per customer).

### Money by region

| Region | Customers | Share of customers | Orders | Money | Average per customer | Share of money |
|---|---|---|---|---|---|---|
| South | 158 | 31.6% | 5,722 | 7,261,744,460 | 45,960,408 | 31.63% |
| West | 151 | 30.2% | 5,335 | 6,730,779,040 | 44,574,696 | 29.32% |
| East | 87 | 17.4% | 3,161 | 4,051,950,040 | 46,574,138 | 17.65% |
| North | 82 | 16.4% | 3,029 | 3,938,165,590 | 48,026,410 | 17.15% |
| Central | 11 | 2.2% | 406 | 501,842,570 | 45,622,052 | 2.19% |
| North East | 11 | 2.2% | 380 | 474,505,730 | 43,136,885 | 2.07% |

Customers with no region: 0. Customers add to 500 and money to 22,958,987,430 (difference 0). South and West together hold about 61% of the money (60.95%) and 309 of the 500 customers, so the money follows the customers. Central and North East are tiny (11 customers each), so their averages move a lot with one or two customers and should not be compared. North has the highest value per customer and North East the lowest, a gap of about 11% (by hand: 48,026,410 / 43,136,885 = 1.113).

### Money by industry segment

| Industry segment | Customers | Share of customers | Orders | Money | Average per customer | Share of money |
|---|---|---|---|---|---|---|
| Logistics | 96 | 19.2% | 3,461 | 4,392,333,800 | 45,753,477 | 19.13% |
| Mining | 98 | 19.6% | 3,394 | 4,249,857,340 | 43,365,891 | 18.51% |
| Construction | 90 | 18.0% | 3,214 | 4,028,671,970 | 44,763,022 | 17.55% |
| Infrastructure | 79 | 15.8% | 2,864 | 3,695,216,850 | 46,774,897 | 16.09% |
| Manufacturing | 70 | 14.0% | 2,649 | 3,401,175,350 | 48,588,219 | 14.81% |
| Rental | 67 | 13.4% | 2,451 | 3,191,732,120 | 47,637,793 | 13.90% |

Customers with no industry segment: 0. Customers add to 500 and money to 22,958,987,430 (difference 0). No single industry carries the business (the biggest holds 19.13%). Logistics and Mining bring the most money mainly through the number of customers. Mining has the most customers (98) but the lowest average (43,365,891). Manufacturing has the highest average (48,588,219), about 12% above Mining (by hand: 48,588,219 / 43,365,891 = 1.120), and Rental is also high.

### Money per year

| Year | Orders | Customers buying | Money | Average order value | Share of money | Change against previous year |
|---|---|---|---|---|---|---|
| 2019 | 3,031 | 500 | 3,870,016,600 | 1,276,812 | 16.86% | not applicable |
| 2020 | 3,038 | 500 | 3,917,532,120 | 1,289,510 | 17.06% | +1.23% |
| 2021 | 3,038 | 498 | 3,852,479,620 | 1,268,097 | 16.78% | -1.66% |
| 2022 | 2,948 | 498 | 3,816,288,220 | 1,294,535 | 16.62% | -0.94% |
| 2023 | 3,022 | 500 | 3,824,675,310 | 1,265,611 | 16.66% | +0.22% |
| 2024 | 2,956 | 500 | 3,677,995,560 | 1,244,247 | 16.02% | -3.84% |

Orders with no date: 0. First order 2019-01-01, last order 2024-12-31, so all six years are full years. Orders added over the years 18,033 and money 22,958,987,430 (difference 0). The business is flat: it did not grow and it did not collapse. 2024 is the lowest year, from a few fewer orders and a smaller average order (see Data Quality Notes). All 500 customers bought in 2024, so no customer has fully stopped buying.

### History length of customers

See Check 7 for the table. In short: every customer started in 2019 and was active in 2024; the 23 customers whose last order was before 1 Jul 2024 have about 11% less money, from fewer orders; the 31 late starters look like everyone else.

### What the numbers can and cannot say

- The ranking is a record of the past six years. KPI 14 found that a customer's money in 2023 and in 2024 are unrelated, so the top 20 should not be treated as a guaranteed top 20 next year.
- The tables show that customer type, branch, region and industry have similar value per customer (gaps of about 7% to 12%). Where a group is clearly bigger in total (South, West, CHN001, Government), it is because it has more customers.
- No customer type, branch, region or industry was tested for a difference beyond chance in KPI 15; the gaps reported are descriptions of the past.

## Recommended actions for the business owners
1. Head of Sales: give each of the top 20 customers (each brought between 66,580,100 and 84,966,970) a named account owner, and refresh the ranking every quarter. Use the list as a record of history, not as a promise of next year, because a customer's money in one year does not predict the next. Keep an eye on C0359 (rank 17), whose position leans most on one order (11.53% of its money from a single 7,752,220 order).
2. Delhi Branch Manager (branch DEL001): make sure every Delhi account has a named owner. Delhi has the fewest customers after HYD001 (71) but the highest value per customer (49,296,215, about 9.6% above Pune), and 6 of the top 20 customers.
3. Commercial Director: watch the first months of 2025 against 2024 (3,677,995,560 from 2,956 orders at an average of 1,244,247). 2024 is the lowest year; decide on any action only after more months show whether orders and order size keep falling.
4. Customer Success Lead: call the 23 customers whose last order was before 1 Jul 2024 (about 11% less money than other customers, from fewer orders). It is a cheap follow-up list and not proof that they are leaving.
5. Customer Data Owner: stop using customer_since, last_purchase_date and total_purchase_value in the customers table for reporting, and take customer money and dates from the Delivered orders, because the table's own summary columns do not match the real orders.

---

## Glossary

| Term | Meaning |
|---|---|
| Delivered order | An order with status Delivered (18,033). Only these count as money |
| Cancelled order | An order with status Cancelled (1,967). Left out of every total |
| Money | Order value before GST (total_order_value) |
| Historical customer value | The money a customer has brought in over 1 Jan 2019 to 31 Dec 2024 |
| Rank | Position of a customer when sorted by money (1 = most) |
| Average order value | Money divided by number of orders; overall = all money / all orders (1,273,165) |
| Top 10, top 50, top 100 | The 10, 50 or 100 customers with the most money |
| Even spread | The share the top customers would hold if every customer brought the same money (n / 500) |
| Concentration | How much of the total money sits with the top customers |
| Middle | The value in the middle when customers are sorted (also called median) |
| Mean | The average |
| Spread | How far customers are from the average on a typical basis (standard deviation) |
| Customer-and-year spot | One customer in one year; 500 times 6 = 3,000 spots |
| Late starter | A customer whose first order is after 30 Jun 2019 |
| Early ender | A customer whose last order is before 1 Jul 2024 |
| Second method | The same numbers calculated a different way (pivot table, attaching details to each order first, date ranges, largest-N) and compared |
| Largest order share | The share of a customer's money that comes from their single largest order |
| Without the largest order | Each customer's money after removing their single largest order, then re-ranked |
| Customer details | customer_type, branch_id, region and industry_segment from the customers table |
| so_h | The sales orders header as loaded from the raw file (20,000 rows) |
| customers | The customers table (500 rows) |
| sales_h | The Delivered orders (18,033 rows) with the order date as a date |
| cust_value | One row per customer: orders, money, rank, average order value |
| money_pivot | Money per customer calculated by pivot table (second method) |
| by_type, by_branch, by_region, by_industry, by_year | The result tables of money by customer type, branch, region, industry segment and year |
| orders_with_customer | The Delivered orders with the customer details attached to every order |
| cust_check | cust_value with each customer's largest order and its share added |
| first_and_last_order, customer_history | Each customer's first and last order date, and the same joined to cust_value |
| top20_ids | The 20 customers with the most money |
| money_without_largest | Each customer's money minus their largest order, sorted |
| middle_customer | The customer at rank 250 (C0436) used in the manual check |
