# KPI 12: Warehouse Space Utilisation

Project: Heavy Supplier & Warehouse Analytics | Week 6 | Notebook: Week6_KPI12_Warehouse_Space_Utilisation

## Summary

- **What it measures:** how crowded each of the 6 branch warehouses is. Headline = units of stock held per square foot (sqft) of warehouse. Supporting (comparison only) = estimated stock volume and the average pile height it would make on the floor.
- **End of data (28 January 2025):** 21,185,215 units of stock in 226,080 sqft. Units per sqft: KOL001 115.0, AHM001 101.8, HYD001 98.2, PUN001 92.3, CHN001 84.7, DEL001 79.5. The most crowded branch holds 1.45 times more units per sqft than the least crowded.
- **The order never changes:** KOL001, AHM001, HYD001, PUN001, CHN001, DEL001 at every year end 2019 to 2024 and at the end of data. The smallest warehouse (KOL001, 29,880 sqft) is the most crowded and the largest (DEL001, 45,230 sqft) the least.
- **Why the branches differ:** stock differs only 1.047 times between branches (3,435,814 to 3,597,678 units) while warehouse size differs 1.51 times (29,880 to 45,230 sqft). The crowding gap comes from the size of the warehouse, not from the stock.
- **Over time:** every branch became about 6 times denser between 2019 and 2024 (KOL001 18.8 to 113.8, DEL001 13.1 to 78.8 units per sqft), because stock grows 3.4 to 3.6 million units a year (units received are about 18 times units delivered) while the warehouse size is one current value.
- **Volume view (comparison only):** total stock volume 1,429,369.3 m3. If all stock stood in one layer on the floor, the average pile would be 56.6 m (DEL001) to 82.0 m (KOL001). Two products (P015 Engine Block, P014 Transmission Assembly) make 67.2% of the volume. Without them the heights are 19.2 to 27.7 m; without the top 5 products 8.7 to 12.4 m. The order and the gap (1.45, 1.44, 1.43 times) do not change.
- **Verification:** all 8 checks done with real results; every stock, volume and size total ties; three items rebuilt by hand from the raw tables match with difference 0.
- **Limits:** warehouse size is one current value per branch (no history); the size of one set or cartridge cannot be confirmed from the data; aisles, racking and building height are not in the data. Product-Warehouse Alignment: Not selected.

## 1 Definition

| Item | Definition |
|---|---|
| Stock (units) | Order-line stock per product and branch pair: opening stock (inventory master) + units on Received purchase order lines - units on Delivered sales order lines + signed stock-ledger adjustments (same figure as the KPI 8 headline: 21,185,215 at the end of data) |
| Warehouse size (sqft) | `warehouse_capacity` in the branches table (text such as "45230 sqft") turned into a number; one current value per branch |
| **Units per sqft (headline)** | Stock units at a date divided by the branch warehouse size in sqft, at each year end 2019 to 2024 and at the end of data |
| Stock volume (m3) | Sum over product-branch pairs of units x unit volume; unit volume = length x width x height in cm from `dimensions_cm` in the products table, divided by 1,000,000 to give m3 |
| Average pile height (m) | Stock volume (m3) divided by the floor area (m2 = sqft x 0.092903), as if all stock stood in one layer on the floor with no aisles or racking. Comparison only |
| End of data | 28 January 2025 (last receipt and last ledger date) |

**Business question:** do the 6 branch warehouses differ in how crowded they are, and how has that changed over time? (Agreed aim: either answer is a finding; the branches could also turn out to be about equally crowded.)

**Why this metric:** warehouse size differs a lot between branches (29,880 to 45,230 sqft, 1.51 times) while the earlier stock KPIs compare stock with demand or with money. This KPI compares the same stock with the space available, so it shows which branches are tightest on space.

**Scope:** 6 branches, 30 products, 180 product-branch pairs; year ends 2019 to 2024 plus the end of data (2025 contains only 28 days of January). Warehouse size is one current value, so the same size is used for every past year. Product-Warehouse Alignment (a possible section of this KPI) was considered and decided Not selected: no new number would be added, and product mix is already covered here through stock volume by product.

## 2 Methodology

### Sources and scoping

| File | Rows x columns | Used for |
|---|---|---|
| branches | 6 x 13 | warehouse type, `warehouse_capacity` (text), employees (not used here) |
| products | 30 x 23 | `dimensions_cm`, `weight_kg`, `uom` (unit of measure), unit volume |
| inventory_master (`inv`) | 180 x 8 | `opening_stock` per product-branch pair; `current_stock` (compared, not used) |
| stock_ledger (`ledger`) | 237,230 x 9 | signed adjustments, running balance |
| sales_orders_header (`so_h`) / lines (`so_l`) | 20,000 x 11 / 130,402 x 9 | Delivered lines: 117,656 lines, 1,234,939 units |
| purchase_orders_header (`po_h`) / lines (`po_l`) | 24,000 x 10 / 155,495 x 9 | Received lines: 140,077 lines, 22,387,123 units |

Order lines carry no branch; the branch comes from the order header (`branch_id`). Only Delivered sales lines and Received purchase lines are counted; Cancelled lines are left out (12,746 sales lines and 15,418 purchase lines, tied in Check 4). All counts equal the figures verified in Weeks 3 to 5.

### Step 1: data check
Drive mounted, 12 CSV files listed (branches, customers, inventory_master, invoices, payments, products, purchase_orders_header, purchase_orders_lines, sales_orders_header, sales_orders_lines, stock_ledger, suppliers). branches 6 x 13 and products 30 x 23. The products table has `dimensions_cm` (text length x width x height) and `weight_kg`, so an estimate from volume is possible. Example rows: P001 Hydraulic Pump HP-300, 40x25x25 cm, 38.5 kg, unit of measure piece; P002 Engine Oil Filter OF-90, 10x8x8 cm, 0.4 kg, piece.

### Step 2: product size check
| Check | Result |
|---|---|
| Products | 30 |
| `dimensions_cm` split into exactly 3 numbers | 30 of 30 |
| Empty `dimensions_cm` | 0 |
| Failed to convert | 0 |
| Zero or below 0 | 0 |
| Empty `weight_kg` | 0 |
| Unit of measure (`uom`) | piece 24, set 5, cartridge 1 |
| Smallest and largest unit volume | 256 cm3 (P027) and 728,000 cm3 (P015); 728,000 / 256 = about 2,844 times apart |

My first expectation "only piece" was wrong (it came from the first two rows only); the real split is 24 / 5 / 1. New columns: `length_cm`, `width_cm`, `height_cm`, `unit_volume_cm3`.

### Step 3: non-piece products and size extremes
| Product | Unit of measure | Dimensions (cm) | Unit volume (cm3) | Weight (kg) | Max stock level |
|---|---|---|---|---|---|
| P004 Track Chain TC-45 | set | 180x40x35 | 252,000 | 210.0 | 20 |
| P012 Brake Pad BP-40 | set | 20x12x10 | 2,400 | 3.2 | 120 |
| P013 Seal Kit SK-15 | set | 15x10x6 | 900 | 0.6 | 200 |
| P014 Transmission Assembly TA-800 | set | 190x60x55 | 627,000 | 260.0 | 12 |
| P026 Grease Cartridge GC-5 | cartridge | 25x6x6 | 900 | 0.45 | 500 |
| P027 O-Ring Set OR-50 | set | 8x8x4 | 256 | 0.2 | 400 |

Five smallest unit volumes: P027 256, P008 Bucket Tooth BT-10 288, P029 Pressure Sensor PS-90 480, P002 Engine Oil Filter OF-90 640, P020 Pivot Pin PP-12 648. Five largest: P015 Engine Block EB-600 728,000 (320 kg, max stock level 8), P014 627,000, P004 252,000, P016 Radiator RD-250 72,000, P018 Wheel Rim WR-24 43,750. Two volumes checked by hand (160x70x65 = 728,000; 8x8x4 = 256). The sizes and weights of the 6 non-piece products are in a believable range for one whole counted unit, but the data cannot confirm what one set or cartridge contains. This is stated as a limit (no test is possible).

### Step 4: warehouse size from text to numbers
| Branch | Type | Size (sqft) |
|---|---|---|
| DEL001 | Central | 45,230 |
| PUN001 | Regional | 38,750 |
| CHN001 | Regional | 41,890 |
| HYD001 | Branch | 36,210 |
| KOL001 | Branch | 29,880 |
| AHM001 | Service Hub | 34,120 |

Failed to convert 0; zero or below 0: 0. Total 226,080 sqft (equals the hand total 45,230 + 38,750 + 41,890 + 36,210 + 29,880 + 34,120). Largest against smallest by hand: 45,230 / 29,880 = 1.51 times.

### Step 5: loading the order, ledger and inventory tables
so_h 20,000 x 11; so_l 130,402 x 9; po_h 24,000 x 10; po_l 155,495 x 9; ledger 237,230 x 9; inv 180 x 8 (all equal the earlier counts). Columns of the inventory master: product_id, branch_id, opening_stock, reorder_level, safety_stock, max_stock, current_stock, warehouse_bin. Ledger movement types: IN 127,611, OUT 107,165, ADJUSTMENT 2,454 (sum 237,230); reference types PO 127,611, SO 107,165, ADJ 2,454. The inventory master has 180 rows, 30 products and 6 branches. First rows (opening stock / reorder level / safety stock / max stock / `current_stock` / bin): P001 DEL001 196 / 59 / 46 / 340 / 115,928 / C15; P001 PUN001 198 / 70 / 43 / 349 / 109,389 / D16; P001 CHN001 187 / 62 / 44 / 311 / 99,373 / A11. Observation: `current_stock` (for example 115,928) is far above the opening stock and the max stock; this is not explained and it is not used as the stock figure.

### Step 6: stock per product-branch pair at the end of data
Per pair: stock at the end = opening stock + units received - units delivered + net adjustments (net adjustment per pair = last running balance - opening - ledger IN + ledger OUT). Ties to KPI 8:

| Item | Units |
|---|---|
| Opening stock | 32,969 |
| Received | 22,387,123 |
| Delivered | 1,234,939 |
| Net adjustments | 62 |
| **Stock at the end** | **21,185,215** (32,969 + 22,387,123 - 1,234,939 + 62) |

Pairs with no ledger balance: 0. Stock at the end per branch: AHM001 3,472,113; CHN001 3,548,773; DEL001 3,597,678; HYD001 3,555,196; KOL001 3,435,814; PUN001 3,575,641 (sum 21,185,215 by hand). Stock differs only 1.047 times between the largest (DEL001) and the smallest (KOL001) branch (3,597,678 / 3,435,814). The `current_stock` column of the inventory master totals 19,303,266, which is 1,881,949 units (about 8.9%) below the reconstructed stock; the cause is not known and the KPI 8 order-line stock stays the headline (Data Quality Note 4).

### Step 7: units per sqft at the end of data
| Branch | Type | Stock (units) | Size (sqft) | Units per sqft |
|---|---|---|---|---|
| KOL001 | Branch | 3,435,814 | 29,880 | 115.0 |
| AHM001 | Service Hub | 3,472,113 | 34,120 | 101.8 |
| HYD001 | Branch | 3,555,196 | 36,210 | 98.2 |
| PUN001 | Regional | 3,575,641 | 38,750 | 92.3 |
| CHN001 | Regional | 3,548,773 | 41,890 | 84.7 |
| DEL001 | Central | 3,597,678 | 45,230 | 79.5 |

Most crowded against least crowded: 115.0 / 79.5 = 1.45 times. All six values were worked out by hand before the run (KOL001 114.99, DEL001 79.54, AHM001 101.76, HYD001 98.18, PUN001 92.27, CHN001 84.72) and equal the code. The order is the exact reverse of warehouse size. Limit: units count a small seal and an engine block the same (the volume steps below address this).

### Step 8: ledger adjustments need a direction
| Item | Result |
|---|---|
| Adjustment rows | 2,454 |
| Quantity, smallest and largest | 5 and 50 |
| Negative, zero, positive quantity | 0, 0, 2,454 |
| Total of all adjustment quantities | 66,990 (not 62) |

Quantity is unsigned, so it does not say whether stock was added or removed. Net adjustments of 62 are tiny against 66,990 units, so adjustments go both ways. Adjustment rows per year (count / units): 2019 384 / 10,590; 2020 391 / 10,509; 2021 420 / 11,598; 2022 440 / 11,940; 2023 421 / 11,744; 2024 398 / 10,609 (counts sum to 2,454, units to 66,990).

**Adjustment direction test:** for every ledger row the change in running balance against the previous row (first row against opening stock) was compared with the quantity:
| Movement | Result |
|---|---|
| ADJUSTMENT, balance went down by the quantity | 1,227 |
| ADJUSTMENT, balance went up by the quantity | 1,227 (total 2,454) |
| IN, balance went up by the quantity | 127,611 (went down: 0) |
| OUT, balance went down by the quantity | 107,165 (went up: 0) |
| Rows in "something else" | 0 |
| Signed total of adjustment changes | 62 (ties to the net adjustments) |

The direction of an adjustment is the sign of the change in running balance. The chain is consistent for every row. Exactly 1,227 up and 1,227 down is noted; no reason is claimed.

### Step 9: stock at each year end per branch
Stock = opening + received (by received date year) - delivered (by delivery date year) + signed adjustments (by movement date year), running total. Column 2025 = end of data (28 January 2025).

| Branch | 2019 | 2020 | 2021 | 2022 | 2023 | 2024 | End of data |
|---|---|---|---|---|---|---|---|
| AHM001 | 552,716 | 1,142,747 | 1,764,206 | 2,297,090 | 2,858,247 | 3,442,316 | 3,472,113 |
| CHN001 | 564,933 | 1,200,597 | 1,727,213 | 2,322,200 | 2,914,591 | 3,516,571 | 3,548,773 |
| DEL001 | 593,228 | 1,151,252 | 1,732,555 | 2,362,474 | 2,971,460 | 3,566,310 | 3,597,678 |
| HYD001 | 549,018 | 1,141,894 | 1,798,295 | 2,367,793 | 2,968,447 | 3,517,178 | 3,555,196 |
| KOL001 | 561,067 | 1,134,619 | 1,682,062 | 2,281,961 | 2,868,931 | 3,400,815 | 3,435,814 |
| PUN001 | 563,698 | 1,189,138 | 1,825,612 | 2,377,887 | 2,975,711 | 3,550,443 | 3,575,641 |
| **Total** | 3,384,660 | 6,960,247 | 10,529,943 | 14,009,405 | 17,557,387 | 20,993,633 | 21,185,215 |

The six year totals equal the KPI 9 and KPI 10 totals exactly (difference 0); the end-of-data column equals the pair table per branch (largest difference 0). The 2019 column summed by hand is 3,384,660. Stock rises every year in every branch (AHM001 2019 to 2024: 3,442,316 / 552,716 = 6.2 times).

### Step 10: units per sqft per year
Same size used for every year (limit).

| Branch | 2019 | 2020 | 2021 | 2022 | 2023 | 2024 | End of data |
|---|---|---|---|---|---|---|---|
| KOL001 | 18.8 | 38.0 | 56.3 | 76.4 | 96.0 | 113.8 | 115.0 |
| AHM001 | 16.2 | 33.5 | 51.7 | 67.3 | 83.8 | 100.9 | 101.8 |
| HYD001 | 15.2 | 31.5 | 49.7 | 65.4 | 82.0 | 97.1 | 98.2 |
| PUN001 | 14.5 | 30.7 | 47.1 | 61.4 | 76.8 | 91.6 | 92.3 |
| CHN001 | 13.5 | 28.7 | 41.2 | 55.4 | 69.6 | 83.9 | 84.7 |
| DEL001 | 13.1 | 25.5 | 38.3 | 52.2 | 65.7 | 78.8 | 79.5 |
| Most / least crowded | 1.44 | 1.49 | 1.47 | 1.46 | 1.46 | 1.44 | 1.45 |

The ranking, most to least crowded, is identical every year: KOL001, AHM001, HYD001, PUN001, CHN001, DEL001. Units per sqft rise about 14 to 20 in each year in every branch, about 6 times from 2019 to 2024 (KOL001 113.8 / 18.8 = 6.05; DEL001 78.8 / 13.1 = 6.0 by hand); the gap between branches stays about 1.45 times. The end-of-data column equals Step 7.

### Step 11: stock volume and average pile height
For each pair: stock at the end x unit volume (cm3 to m3). Floor area m2 = sqft x 0.092903; average pile height = volume / floor area (all stock in one layer on the floor, no aisles or racking). Pairs with no unit volume: 0.

| Branch | Stock volume (m3) | Size (sqft) | Floor area (m2) | Average pile height (m) |
|---|---|---|---|---|
| KOL001 | 227,728.2 | 29,880 | 2,775.9 | 82.0 |
| AHM001 | 233,904.3 | 34,120 | 3,169.9 | 73.8 |
| HYD001 | 242,707.2 | 36,210 | 3,364.0 | 72.1 |
| PUN001 | 239,881.1 | 38,750 | 3,600.0 | 66.6 |
| CHN001 | 247,324.9 | 41,890 | 3,891.7 | 63.6 |
| DEL001 | 237,823.5 | 45,230 | 4,202.0 | 56.6 |

Most piled against least piled: 82.0 / 56.6 = 1.45 times. Two values worked out by hand: KOL001 29,880 x 0.092903 = 2,775.9 and 227,728.2 / 2,775.9 = 82.0; DEL001 237,823.5 / 4,202.0 = 56.6. The pile-height order equals the units-per-sqft order. The volume itself is not in the units order (CHN001 has the largest volume, 247,324.9, KOL001 the smallest, 227,728.2; 1.086 times apart), so product mix matters. Finding: even the least crowded branch would have an average pile of 56.6 m if all stock stood in one layer on the floor. The data has no building height or racking, so no exact limit can be claimed, but the reconstructed stock does not look physically placeable in the stated floor space; the cause is not claimed.

### Step 12: which products make up the volume
All branches together, stock at the end of data:

| Product | Unit of measure | Units | Unit volume (cm3) | Volume (m3) | Share of volume (%) |
|---|---|---|---|---|---|
| P015 Engine Block EB-600 | piece | 694,538 | 728,000 | 505,623.7 | 35.4 |
| P014 Transmission Assembly TA-800 | set | 724,530 | 627,000 | 454,280.3 | 31.8 |
| P004 Track Chain TC-45 | set | 700,277 | 252,000 | 176,469.8 | 12.3 |
| P016 Radiator RD-250 | piece | 693,262 | 72,000 | 49,914.9 | 3.5 |
| P018 Wheel Rim WR-24 | piece | 709,520 | 43,750 | 31,041.5 | 2.2 |
| P021 Boom Cylinder BC-400 | piece | 703,315 | 42,120 | 29,623.6 | 2.1 |
| P011 Hydraulic Cylinder HC-200 | piece | 713,667 | 38,880 | 27,747.4 | 1.9 |
| P023 Idler Wheel IW-55 | piece | 718,664 | 37,500 | 26,949.9 | 1.9 |

Top 2 products = 67.2% of the volume (35.4 + 31.8); top 5 = 85.2%. Total volume 1,429,369.3 m3 (Check 4). Hand check for P015: 694,538 x 0.728 = about 505,624 m3. Units per product are nearly equal (about 694,000 to 725,000 in the rows shown; average 21,185,215 / 30 = 706,174), so big products dominate the volume only because of their size. The pile height therefore depends strongly on the size values of P015, P014 (set, 190x60x55 cm, meaning of one set not confirmable) and P004 (set); this is stated as a limit. Observation (not tested, no claim): in the products table the max stock level of P015 is 8, while its stock at the end is 694,538 units across 6 branches.

### Step 13: sensitivity of the pile height to the biggest products
Average pile height (m):

| Branch | All products | Without top 2 (P015, P014) | Without top 5 (P015, P014, P004, P016, P018) |
|---|---|---|---|
| AHM001 | 73.8 | 24.2 | 11.1 |
| CHN001 | 63.6 | 20.1 | 8.9 |
| DEL001 | 56.6 | 19.2 | 8.7 |
| HYD001 | 72.1 | 23.3 | 10.3 |
| KOL001 | 82.0 | 27.7 | 12.4 |
| PUN001 | 66.6 | 21.8 | 10.1 |
| Most / least | 1.45 | 1.44 | 1.43 |

The order (KOL001, AHM001, HYD001, PUN001, CHN001, DEL001) is the same in all three columns (27.7 / 19.2 = 1.44 and 12.4 / 8.7 = 1.43 by hand). My hand preview (multiplying by 0.328 and 0.148 using all-branch shares, e.g. KOL001 26.9 and 12.1) was close but not exact because each branch has its own product mix; the code figures are the ones used. Result: the crowding order and the gap (about 1.45 times) do not depend on the biggest products; the absolute height does (56.6 to 82.0 m down to 8.7 to 12.4 m).

### Step 14: headline decision
Agreed with the user: headline = units per sqft at each year end 2019 to 2024 and at the end of data; supporting = stock volume and average pile height (comparison only); limits stated once. Product-Warehouse Alignment = Not selected (user decision).

### Alternatives considered

| Alternative | Decision and reason |
|---|---|
| Units per sqft as headline | Chosen: needs only data that is complete and verified (stock, size) |
| Volume or pile height as headline | Supporting only: depends on sizes of 30 products, 6 of them sets or cartridges whose contents cannot be confirmed, and 2 products make 67.2% of the volume |
| `current_stock` of the inventory master as stock | Not used: total 19,303,266 is 1,881,949 (about 8.9%) below the reconstructed stock, cause unknown |
| Ledger running balance as stock | Not used as the stock source: order lines are complete; in 2025 the ledger lacks rows for 135 counted lines (Check 7) |
| Cancelled orders counted in stock | Not used: Cancelled lines are not delivered or received (Check 4) |
| Weight (`weight_kg`) as the load measure | Not used: no floor load limit in the data |
| Warehouse size history | Not possible: branches table holds one current value per branch |
| Product-Warehouse Alignment section | Not selected (user decision): no new number would be added; product mix is covered by volume by product |

## 3 Verification Log

### Summary of the 8 checks

| # | Check | Method | Result in plain words |
|---|---|---|---|
| 1 | Missing, zero, negative values and date failures | Counted in every column used (inventory master, ledger, delivered and received lines, products sizes, warehouse size text) | None found: all counts 0 |
| 2 | Edge cases | 180 unique pairs, stock at 7 year ends per pair (1,260 values), none at or below 0; pair sums equal the branch table | No pair ever at 0 or below; smallest value 13,616 (P029 HYD001 end of 2019); difference 0 |
| 3 | Duplicates | Repeated ids and whole identical rows; exact copies of repeated lines (extra-units test and reconciliation) | No repeated ids; 684 sales pairs with an exact copy = 7,604 units (0.62% of delivered units, 0.036% of stock); purchases 0; copies at chance level, none removed |
| 4 | Totals reconcile | Lines and units by status; stock four ways; volume two routes; warehouse size total | All differences 0; stock 21,185,215 four ways; volume 1,429,369.3 m3 two ways; size 226,080 sqft |
| 5 | Second method | Dictionary map and isin (no merge), size text parsed again with a regular expression, units per sqft at 2019, 2024, end of data | Largest difference 0.0 |
| 6 | Outliers | Stock per pair (smallest, mean, largest, shares), line size (cited from KPI 11), product volume dominance and sensitivity | No pair above 3.74% of its branch; volume dominated by 2 products, order and gap stable |
| 7 | History | Dates, partial 2025, ledger against order lines in 2025, what drives yearly stock growth | Data ends 28 Jan 2025; 120-row ledger difference fully explained; growth comes from units IN being about 18 times units OUT |
| 8 | Manual spot-check | 3 items rebuilt by hand from the raw tables (2 pairs and 1 branch) | All three equal the saved figures; difference 0 |

### Check 1: missing, zero, negative values and date failures
| Table | Result |
|---|---|
| Inventory master, 180 rows | Opening stock empty 0, zero or below 0: 0; product or branch empty 0 |
| Ledger, 237,230 rows | Quantity empty 0, zero or below 0: 0; running balance empty 0, below 0: 0; dates failing to convert 0 |
| Delivered sales lines, 117,656 rows | Delivery dates failing to convert 0 |
| Received purchase lines, 140,077 rows | Received dates failing to convert 0 |
| Products, 30 rows | `dimensions_cm` clean 30 of 30; `weight_kg` empty 0 (Step 2) |
| Branches, 6 rows | Warehouse size failed to convert 0, zero or below 0: 0 (Step 4) |

### Check 2: edge cases
Duplicate product and branch pairs in the inventory master: 0. Pairs 180; year ends 7 (2019 to 2024 plus end of data); values checked 1,260 (180 x 7); values below 0: 0; values at 0: 0; smallest stock value 13,616 at P029 (Pressure Sensor) HYD001 at the end of 2019; largest difference between the pair sums and the branch table: 0. Limit: year ends only, not every day; the ledger running balance was never below 0 at any ledger row (Check 1).

### Check 3: duplicates
First run: ledger movement ids repeated 0; sales order ids repeated in the header 0 of 20,000; purchase order ids repeated in the header 0 of 24,000; identical whole rows in sales order lines 0 of 130,402; in purchase order lines 0 of 155,495. Sales line columns: so_id, line_number, product_id, quantity, unit_price, gst_rate, line_total, gst_amount, line_grand_total; purchase line columns: po_id, line_number, product_id, quantity, unit_cost, gst_rate, line_total, gst_amount, line_grand_total. Limit: `line_number` is part of every row, so a whole-row test cannot catch a copied line with a new line number; the relevant test ignores `line_number`.

**Check 3 extra A (extra-units test)**, key = order id + product + quantity + price: sales groups with an exact copy 686; extra lines 688; extra units in the copies 7,604 = 0.62% of delivered units (7,604 / 1,234,939 = 0.616 by hand) and 0.036% of stock at the end (7,604 / 21,185,215 = 0.0359). Purchases: 0 groups. Worst case by hand: if all 7,604 units sat in one branch, KOL001 would move by 7,604 / 29,880 = 0.25 units per sqft of 115.0.

**Check 3 extra B (pair and group reconciliation)**: KPI 11 counted 684 order-product pairs with an exact copy; extra A counted 686 groups. Test: order and product pairs with an exact copy 684 (equals KPI 11); pairs holding 2 or more different copy groups 2; groups made of 3 or more identical lines 2. Arithmetic: 686 - 684 = 2 pairs with a second group; 688 extra lines - 686 groups = 2 groups of 3 lines. The repeated-line test itself (KPI 11 Check 3, same lines) found sales exact copies at chance level (5.61% of repeated pairs, mixed 2-line and 3+-line) and purchases 0 exact copies because every repeated purchase line has a different unit cost. Copies are at chance level, so none were removed.

### Check 4: totals reconcile
| Item | Lines | Units |
|---|---|---|
| Sales: Cancelled | 12,746 | 133,595 |
| Sales: Delivered | 117,656 | 1,234,939 |
| Sales: all | 130,402 | 1,368,534 |
| Purchases: Cancelled | 15,418 | 2,480,593 |
| Purchases: Received | 140,077 | 22,387,123 |
| Purchases: all | 155,495 | 24,867,716 |

Counted + left out = all for lines and units on both sides, difference 0. Only two statuses exist on each side. Left-out share by hand: lines 9.77% (sales) and 9.92% (purchases); units 9.76% and 9.98%.

**Check 4 extra (stock, volume and size totals):** stock the long way (opening + received - delivered + adjustments) 21,185,215; pair table 21,185,215; year-end table at the end of data 21,185,215; units-per-sqft table 21,185,215. Volume: pair table 1,429,369.3 m3 and a second route (units per product x unit volume) 1,429,369.3 m3 (the earlier 1,429,369.5 was the sum of 30 per-product values each rounded to 1 decimal; the sum of the six rounded branch figures is 1,429,369.2). Warehouse size total 226,080 sqft in the branches table and in the units-per-sqft table.

### Check 5: second method
Rebuilt without merges: dictionary map of order id to branch, isin on the Delivered and Received ids, adjustments by direction from the ledger, and the size text parsed again with a regular expression. Units per sqft at 2019 / 2024 / end of data:

| Branch | 2019 | 2024 | End of data |
|---|---|---|---|
| AHM001 | 16.2 | 100.9 | 101.8 |
| CHN001 | 13.5 | 83.9 | 84.7 |
| DEL001 | 13.1 | 78.8 | 79.5 |
| HYD001 | 15.2 | 97.1 | 98.2 |
| KOL001 | 18.8 | 113.8 | 115.0 |
| PUN001 | 14.5 | 91.6 | 92.3 |

Largest difference against the main table: 0.0. Limit: the second method rebuilds orders, branch mapping and the size parse independently but uses the same adjustment direction rule (that rule was tested separately in Step 8).

### Check 6: outliers
| Stock per pair at the end | Value |
|---|---|
| Pairs | 180 |
| Smallest / mean / largest | 98,642 / 117,696 / 132,967 |
| Largest against mean | 1.13 |
| Smallest against mean | 0.84 |

Mean by hand: 21,185,215 / 180 = 117,695.6. Three largest pairs and their share of branch stock: P002 HYD001 132,967 = 3.74%; P012 DEL001 129,402 = 3.60%; P003 PUN001 129,186 = 3.61% (by hand 132,967 / 3,555,196 = 3.74%, 129,402 / 3,597,678 = 3.60%, 129,186 / 3,575,641 = 3.61%). Largest single pair share per branch: AHM001 3.60, CHN001 3.54, DEL001 3.60, HYD001 3.74, KOL001 3.55, PUN001 3.61 (an equal share would be 1 / 30 = 3.33%, so the biggest pair is at most 1.12 times an equal share). The three largest pairs are each the largest of their own branch. No pair is near 10%.

**Check 6 extra A (volume dominance)**: Step 12 (top 2 = 67.2%, top 5 = 85.2%). **Check 6 extra B (sensitivity)**: Step 13 (order and gap stable: 1.45, 1.44, 1.43). Line-size outliers are cited from KPI 11 Check 6, which used the same 117,656 delivered and 140,077 received lines: largest line against mean 1.9 times on both sides (20 / 10.5 and 300 / 160), the largest line equal to the top allowed value in every branch.

### Check 7: history
**Check 7 extra A (dates and 2025 lines).** Delivered lines: delivery dates 2019-01-03 to 2025-01-13; received lines: received dates 2019-01-11 to 2025-01-28; stock ledger: movement dates 2019-01-01 to 2025-01-28; ledger rows after 2025: 0. Data therefore ends on 28 January 2025 and 2025 is not a full year. Lines dated 2025: delivered 340 lines / 3,372 units; received 1,246 lines / 194,954 units (equal to KPI 11). By hand 194,954 - 3,372 = 191,582 = 21,185,215 - 20,993,633.

**Check 7 extra B (ledger rows 2025 by type).** Ledger rows dated 2025: 1,466 = IN 1,143 + OUT 323. There are no ADJUSTMENT rows dated 2025, so the 2025 adjustment total is 0 and the 2025 stock change (191,582) is explained by receipts minus deliveries. The 2025 order lines are 340 + 1,246 = 1,586, which is 120 more than the 1,466 ledger rows. This difference was tested (extras C to F):

| Test | Received (IN) | Delivered (OUT) |
|---|---|---|
| Extra C: lines dated 2025 | 1,246 | 340 |
| Extra C: lines whose order and product pair has a ledger row | 1,151 | 320 |
| Extra C: lines whose pair has no ledger row | 95 | 20 |
| Extra C: ledger rows dated 2025 with no 2025 line | 9 | 6 |
| Extra D: distinct order and product pairs among lines with a ledger row | 1,000 | 283 |
| Extra D: of those pairs, ledger row dated 2025 / another year | 1,000 / 0 | 283 / 0 |
| Extra D: ledger rows dated 2025 that match a 2025 line | 1,134 | 317 |
| Extra E: pairs with 2025 lines | 1,095 | 303 |
| Extra E: pairs with no ledger row (lines in them) | 95 (95) | 20 (20) |
| Extra E: pairs with fewer ledger rows than lines (lines short) | 17 (17) | 3 (3) |
| Extra E: pairs with the same number | 983 | 280 |
| Extra E: pairs with more ledger rows than lines | 0 | 0 |

Reading (arithmetic): repeated product lines in one order share a pair (1,151 lines sit on 1,000 pairs), so a line counted as "having a ledger row" is not the same as having its own row. Lines without a ledger row of their own = 95 + 17 = 112 received and 20 + 3 = 23 delivered, 135 in total (8.5% of 1,586). Every other line has its own row; no pair has more ledger rows than lines; the ledger date never falls in a different year from the line (0 pairs).

**Check 7 extra F (the 15 ledger rows with no 2025 line).** All 15 were found in the order tables and all have status Cancelled:

| Side | Order | Product | Ledger date |
|---|---|---|---|
| Purchase | PO-448619 | P003 | 2025-01-14 |
| Purchase | PO-190792 | P005 | 2025-01-22 |
| Purchase | PO-968989 | P018 | 2025-01-08 |
| Purchase | PO-850081 | P021 | 2025-01-01 |
| Purchase | PO-217565 | P022 | 2025-01-06 |
| Purchase | PO-667371 | P023 | 2025-01-17 |
| Purchase | PO-482299 | P027 | 2025-01-17 |
| Purchase | PO-618520 | P028 | 2025-01-02 |
| Purchase | PO-926660 | P029 | 2025-01-16 |
| Sales | SO-802105 | P004 | 2025-01-02 |
| Sales | SO-975400 | P005 | 2025-01-06 |
| Sales | SO-724047 | P007 | 2025-01-07 |
| Sales | SO-724047 | P009 | 2025-01-07 |
| Sales | SO-804205 | P022 | 2025-01-05 |
| Sales | SO-506518 | P023 | 2025-01-06 |

The 9 purchase orders have an empty received date; the 6 sales rows sit on 5 orders (SO-724047 has two products) and their delivery date equals the ledger date. They are left out of the counted lines because only Delivered and Received lines are counted (same rule as Check 4). Final reconciliation: 1,586 order lines - 135 lines without their own ledger row + 15 ledger rows on cancelled orders = 1,466 ledger rows; 135 - 15 = 120. Why the ledger lacks rows for 135 counted lines and holds rows for 15 cancelled lines is not in the data, and no reason is claimed. Only 2025 was tested. The stock figure uses Delivered and Received order lines only.

**Check 7 extra G (what drives the yearly stock growth).** Rebuilt per year from the order lines and the ledger adjustments (signed change in running balance):

| Year | Units IN | Units OUT | Net adjustments | Change in stock | Stock at year end | Units IN per 1 OUT |
|---|---|---|---|---|---|---|
| 2019 | 3,556,945 | 204,586 | -668 | 3,351,691 | 3,384,660 | 17.4 |
| 2020 | 3,786,822 | 211,190 | -45 | 3,575,587 | 6,960,247 | 17.9 |
| 2021 | 3,776,259 | 206,935 | 372 | 3,569,696 | 10,529,943 | 18.2 |
| 2022 | 3,682,130 | 202,716 | 48 | 3,479,462 | 14,009,405 | 18.2 |
| 2023 | 3,752,938 | 205,578 | 622 | 3,547,982 | 17,557,387 | 18.3 |
| 2024 | 3,637,075 | 200,562 | -267 | 3,436,246 | 20,993,633 | 18.1 |
| 2025 (to 28 Jan) | 194,954 | 3,372 | 0 | 191,582 | 21,185,215 | 57.8 |

Every year-end total equals the known total; the net adjustments equal my hand figures (derived from the known year-end totals) and add up to 62. Hand checks: 32,969 + 3,351,691 = 3,384,660; 3,556,945 / 204,586 = 17.4; 3,637,075 / 200,562 = 18.1; whole period 22,387,123 / 1,234,939 = 18.1. Adjustments are under 0.02% of each year's stock change (668 / 3,351,691 = 0.0199%). Result: stock grows by about 3.4 to 3.6 million units in every full year because units IN are about 18 times units OUT. The 2025 ratio (57.8) is not comparable: the data ends on 28 January 2025, the last delivery is dated 13 January and the last receipt 28 January (extra A); it is not used. The limit that warehouse size is one current value used for every year has no test (the data holds no history); it is stated in Data Quality Note 1.

### Check 8: manual spot-check from the raw tables
Three items were rebuilt from the raw tables (`so_h`, `so_l`, `po_h`, `po_l`, `ledger`, `inv`, `branches`), not from the helper tables: stock = opening + units on Received purchase lines up to the date - units on Delivered sales lines up to the date + signed ledger adjustments up to the date.

| Item | Opening | Received | Delivered | Net adjustments | Stock | Saved figure | Difference |
|---|---|---|---|---|---|---|---|
| P002 at HYD001, end of data (largest pair) | 219 | 138,227 | 5,413 | -66 | 132,967 | 132,967 | 0 |
| P029 at HYD001, end of 2019 (smallest year-end value) | 192 | 14,461 | 1,031 | -6 | 13,616 | 13,616 | 0 |
| KOL001, all 30 products, end of data | | | | | 3,435,814 | 3,435,814 | 0 |

By hand: 219 + 138,227 - 5,413 - 66 = 132,967; 192 + 14,461 - 1,031 - 6 = 13,616. For KOL001 the warehouse size was read from the raw text "29880 sqft": 3,435,814 / 29,880 = 114.99 = 115.0 units per sqft, equal to the saved figure.

### Test labels

| Label | Plain name | Question | Result | Where |
|---|---|---|---|---|
| Product size check | Step 2 | Are all 30 sizes complete and readable? | 30 of 30 clean; unit of measure 24 / 5 / 1 | Step 2 |
| Adjustment direction test | Step 8 | Is the direction of an adjustment the sign of the change in running balance? | Yes for all 2,454 rows (1,227 up, 1,227 down), signed total 62 | Step 8 |
| Check 3 extra A | Extra-units test | How many units sit in exact copies? | 7,604 units, 0.62% of delivered units | Check 3 |
| Check 3 extra B | Pair and group reconciliation | Why 686 groups against 684 pairs? | 2 pairs with a second group, 2 groups of 3 lines | Check 3 |
| Check 4 extra | Stock, volume and size totals | Do stock, volume and size agree by different routes? | Yes, all differences 0 | Check 4 |
| Check 6 extra A | Volume dominance | Do a few products make most of the volume? | Top 2 = 67.2%, top 5 = 85.2% | Check 6, Step 12 |
| Check 6 extra B | Sensitivity | Does the crowding order depend on the biggest products? | No: order the same, gap 1.45 / 1.44 / 1.43 | Check 6, Step 13 |
| Check 7 extra A | Dates and 2025 lines | What dates does the data cover and how much is 2025? | 2019-01-01 to 2025-01-28; 2025 = 28 days | Check 7 |
| Check 7 extra B | Ledger rows 2025 by type | How many ledger rows in 2025 and of which type? | 1,466 = IN 1,143 + OUT 323; no adjustments | Check 7 |
| Check 7 extra C | Lines with a ledger row | How many 2025 lines have a ledger row? | 1,151 of 1,246 received; 320 of 340 delivered | Check 7 |
| Check 7 extra D | Distinct pairs | Why do lines with a row outnumber the matching rows? | 1,151 lines on 1,000 pairs; 1,134 rows | Check 7 |
| Check 7 extra E | Lines against ledger rows per pair | How many lines lack their own row? | 112 received + 23 delivered = 135 | Check 7 |
| Check 7 extra F | Cancelled-order rows | What are the 15 ledger rows with no 2025 line? | All on Cancelled orders | Check 7 |
| Check 7 extra G | Growth driver | What makes stock grow each year? | Units IN about 18 times units OUT | Check 7 |

## 4 Data Quality Notes

1. **Warehouse size is one current value per branch.** The branches table holds one value per branch and no history, so the same size is used for every year from 2019. Units per sqft in past years describe the current size.
2. **Unit sizes of sets and cartridge cannot be confirmed.** P004, P012, P013, P014, P027 (sets) and P026 (cartridge) have a size, but the data does not say what one counted unit contains. All volume figures assume the size describes one counted unit.
3. **No aisles, racking or building height in the data.** The average pile height assumes one layer on the floor with no aisles, so it is a comparison between branches, not a real height.
4. **`current_stock` in the inventory master is lower than the reconstructed stock.** It totals 19,303,266, which is 1,881,949 units (about 8.9%) below the order-line stock of 21,185,215. The cause is not known. The KPI 8 order-line stock stays the headline.
5. **Ledger adjustment quantity is unsigned.** 2,454 rows, quantity 5 to 50, 66,990 units in total. The direction comes from the change in running balance (1,227 up, 1,227 down, net 62).
6. **Exact copies of sales lines.** 684 order-product pairs (686 groups, 688 extra lines, 7,604 units) are at chance level; none removed. Purchases have 0 exact copies because every repeated purchase line has a different unit cost.
7. **The ledger does not match the order lines one for one in 2025.** 135 of the 1,586 counted lines (8.5%) have no ledger row of their own and 15 ledger rows belong to cancelled orders. The cause is not in the data; only 2025 was tested. The stock figure does not use the ledger rows (only the signed adjustments).
8. **2025 is not a full year.** The data ends on 28 January 2025; the last delivery is dated 13 January. The end-of-data figures are not a year-end and the 2025 ratio of units IN to units OUT (57.8) is not used.
9. **Units of measure differ.** piece 24, set 5, cartridge 1 products. Units per sqft count every unit the same, whatever its size, which is why the volume view was added.
10. **Product max stock level against stock.** In the products table P015 has a max stock level of 8 while its stock at the end is 694,538 units across 6 branches. Observed, not tested, no claim.
11. **Second method is not fully independent.** Check 5 rebuilds orders, branch mapping and the size parse independently but uses the same adjustment direction rule (tested separately in Step 8).
12. **Rounding.** Total volume is 1,429,369.3 m3 (exact); the sum of the six rounded branch figures is 1,429,369.2 and the sum of 30 rounded product figures is 1,429,369.5. The exact figure is used.

## 5 Business Interpretation

**Business question:** do the 6 branch warehouses differ in how crowded they are, and how has that changed over time?

**Answer:** yes. At the end of data the branches hold between 79.5 (DEL001) and 115.0 (KOL001) units per sqft, a gap of 1.45 times. The order is the same at every year end, with the smallest warehouses the most crowded. The gap comes from warehouse size, not from stock: every branch holds about the same stock (1.047 times apart) in warehouses that differ 1.51 times in size. Every branch has also become about 6 times denser since 2019.

**Findings**

1. KOL001 (29,880 sqft) is the most crowded with 115.0 units per sqft; DEL001 (45,230 sqft) is the least with 79.5. In between: AHM001 101.8, HYD001 98.2, PUN001 92.3, CHN001 84.7.
2. The ranking is identical in every year from 2019 to 2024 and at the end of data; the gap between most and least crowded stays between 1.44 and 1.49 times.
3. Stock per branch is almost equal (3,435,814 to 3,597,678 units), so the crowding gap follows the size of the warehouse (largest against smallest 1.51 times).
4. Every branch holds about 6 times more units per sqft in 2024 than in 2019 (KOL001 18.8 to 113.8, DEL001 13.1 to 78.8), because stock grows 3.4 to 3.6 million units a year: units received are about 18 times units delivered (17.4 to 18.3 each year) and adjustments are under 0.02% of the yearly change.
5. In volume terms the stock is about 1.43 million m3. If it all stood in one layer on the floor, the average pile would be 56.6 m (DEL001) to 82.0 m (KOL001); the same order as units per sqft. The data does not hold building height or racking, so no exact limit can be claimed, but the reconstructed stock does not look physically placeable in the stated floor space.
6. Two products (P015 Engine Block, P014 Transmission Assembly) make 67.2% of the stock volume and the top five 85.2%, although every product has about the same number of units (about 694,000 to 725,000). Without them the pile heights fall to 19.2 to 27.7 m (top 2 removed) and 8.7 to 12.4 m (top 5 removed) and the crowding order stays the same.
7. Stock is spread evenly over the 180 pairs: the largest pair is 1.13 times the mean and at most 3.74% of its branch (equal share 3.33%).
8. The result rests on one current warehouse size per branch, on unconfirmed sizes for 6 set and cartridge products and on no aisle or racking information (Data Quality Notes 1 to 3).

**Recommended actions for the business owners**

1. **Warehouse Operations Manager:** review space first at KOL001 (115.0 units per sqft) and AHM001 (101.8), the two most crowded branches, and compare their handling and layout with DEL001 (79.5).
2. **Procurement Manager:** compare the quantities received at each branch with its sales and with its warehouse size before placing further purchase orders, because units received are about 18 times units delivered and stock grows 3.4 to 3.6 million units a year.
3. **Branch Managers (all 6 branches):** report the usable floor area, racking and building height of each warehouse, so the pile-height comparison can become a real space-use figure.
4. **Warehouse Operations Manager and Procurement Manager:** set a limit of units or volume per sqft for each branch, using the branch size, and track it at each year end next to this KPI.
5. **Data Owner (the team that keeps the stock records):** record the history of each warehouse size, confirm what one set or cartridge contains for P004, P012, P013, P014, P027 and P026, and explain the gaps between the inventory master `current_stock`, the ledger and the order lines (1,881,949 units; 135 counted lines without a ledger row; 15 ledger rows on cancelled orders).

## Glossary

| Label | Meaning |
|---|---|
| Pair | One product at one branch (30 products x 6 branches = 180 pairs); table `pair` holds opening, received, delivered, adjustments and the stock at the end per pair |
| Units per sqft | Stock units divided by warehouse size in sqft (table `dens` at the end of data, `dens_y` per year) |
| `capacity_sqft` | Warehouse size as a number, read from the text in `warehouse_capacity` |
| `unit_volume_cm3` | Length x width x height of one unit in cm3, from `dimensions_cm` (with `length_cm`, `width_cm`, `height_cm`) |
| Average pile height | Stock volume divided by floor area, one layer, no aisles (table `vb`; `sens` for the sensitivity) |
| `vol` | The pair table with unit volume and volume in m3 added |
| `pv` | Volume by product across all branches |
| `inv` | Inventory master (180 rows) |
| `ledger` | Stock ledger (237,230 rows) |
| `led` | The ledger with the previous balance, the change and what happened (up, down) per row |
| `sd`, `pr` | Delivered sales lines and Received purchase lines with their branch and dates |
| `ad` | Adjustment rows of the ledger with their year and signed change |
| `stock_ye` | Stock per branch at each year end 2019 to 2024 and at the end of data (built with `in_y`, `out_y`, `adj_y` and the opening stock per branch) |
| `in_y`, `out_y`, `adj_y` | Units received, units delivered and net adjustments per branch and year |
| `stock_p`, `in_p`, `out_p`, `adj_p` | Stock, units received, units delivered and net adjustments per pair at each year end (Check 2) |
| `pl`, `sl` | Received purchase lines and Delivered sales lines with the date (`d`) of receipt or delivery (Check 7) |
| `rec25`, `del25` | The 2025 received and delivered lines (Check 7) |
| `lg_in`, `lg_out` | Ledger rows dated 2025 that refer to a purchase order (IN) or a sales order (OUT) (Check 7) |
| `g` | Table of units IN, units OUT, net adjustments and stock per year (Check 7 extra G) |
| `stock_of` | Function that rebuilds the stock of one pair up to a date from the raw tables (Check 8) |
| IN, OUT, ADJUSTMENT | Ledger movement types: stock received, stock delivered, stock correction |
| Delivered, Received, Cancelled | Order statuses; only Delivered sales lines and Received purchase lines are counted |
| `uom` | Unit of measure of a product: piece, set or cartridge |
| sqft, m2, m3 | Square feet; square metres (sqft x 0.092903); cubic metres (cm3 / 1,000,000) |
| End of data | 28 January 2025 |
| Exact copy | Two lines of the same order and product with the same quantity and the same price or cost |
