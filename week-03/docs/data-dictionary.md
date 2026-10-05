# Data Dictionary

**Project:** Heavy Supplier & Warehouse Analytics (CadetX) | **Week:** 3 | **Step:** 7 (data dictionary)

---

## 1. Purpose and how it was built

This dictionary describes every column in all 12 CSV files: its name, the kind of data it holds, how many values are empty, how many different values it has, its range or example values, and what it means.

CadetX's own field reference (Fields Documentation.docx) was not available to us, so this dictionary was built from the data itself.

- **Facts** (type, empty count, distinct count, range, sample values, row counts) come from a profile of the 12 files run in Colab. The profile covers 134 columns in 12 tables.
- **Meanings** were written from the column names and the values. They are **inferred from the data**, not taken from a CadetX document. Where a meaning could not be confirmed, the entry says so.
- **Findings** marked DO NOT USE or NOT UNIQUE come from the Week 3 checks recorded in `data-cleaning-log.md`.
- Which table links to which is in `table-join-map.md`. It is not repeated here.

**Reading the columns.** Type is one of: text, whole number, decimal number, or date (stored as text). In the raw files every date is stored as text and must be converted to a real date before use. "Empty" is the number of missing values. "Distinct" is the number of different values. "Range" is the smallest to the largest value for numbers, the earliest to the latest for dates, and the first to the last in alphabetical order for text.

---

## 2. The 12 tables at a glance

| Table | Rows | Columns | What it holds | Identifier |
|-------|------|---------|---------------|------------|
| `branches.csv` | 6 | 13 | The 6 branch warehouses: location, size, staff and money figures. | branch_id (unique) |
| `customers.csv` | 500 | 15 | The 500 customers: type, location, credit and payment terms. | customer_id (unique) |
| `inventory_master.csv` | 180 | 8 | Stock settings for every product in every branch. | product_id + branch_id (unique, 30 x 6 = 180) |
| `invoices.csv` | 18,033 | 10 | One invoice per Delivered sales order. | invoice_id is NOT unique (393 rows repeated). so_id is unique |
| `payments.csv` | 19,257 | 5 | Payments received against invoices. | payment_id is NOT unique (403 rows repeated) |
| `products.csv` | 30 | 23 | The 30 products: price, cost, size, tax and stock settings. | product_id (unique) |
| `purchase_orders_header.csv` | 24,000 | 10 | Purchase orders placed with suppliers (one row per order). | po_id (unique) |
| `purchase_orders_lines.csv` | 155,495 | 9 | The products and quantities on each purchase order. | po_id + line_number (unique) |
| `sales_orders_header.csv` | 20,000 | 11 | Sales orders placed by customers (one row per order). | so_id (unique) |
| `sales_orders_lines.csv` | 130,402 | 9 | The products and quantities on each sales order. | so_id + line_number (unique) |
| `stock_ledger.csv` | 237,230 | 9 | Every stock movement in and out of the branches. | movement_id (unique) |
| `suppliers.csv` | 8 | 12 | The 8 suppliers: location, lead time and reliability. | supplier_id (unique) |

Total: 12 tables, 134 columns.

---

## 3. Column reference

### 3.1 branches.csv

The 6 branch warehouses: location, size, staff and money figures. 6 rows, 13 columns. Identifier: branch_id (unique).

| Column | Type | Empty | Distinct | Range | Example values | Meaning (inferred from the data) |
|--------|------|-------|----------|-------|----------------|----------------------------------|
| `branch_id` | text | 0 | 6 | AHM001 to PUN001 | DEL001; PUN001; CHN001; HYD001 | Branch code (AHM001, CHN001, DEL001, HYD001, KOL001, PUN001). Key linking branches to every other table. |
| `branch_name` | text | 0 | 6 | Ahmedabad West Hub to Pune Distribution | Delhi Central; Pune Distribution; Chennai South Hub; Hyderabad Logistics | Full name of the branch warehouse. |
| `city` | text | 0 | 6 | Ahmedabad to Pune | Delhi; Pune; Chennai; Hyderabad | City where the branch is located. |
| `state` | text | 0 | 6 | Delhi to West Bengal | Delhi; Maharashtra; Tamil Nadu; Telangana | State where the branch is located. |
| `region` | text | 0 | 4 | East to West | North; West; South; East | Sales region of the branch (North, South, East, West). |
| `warehouse_type` | text | 0 | 4 | Branch to Service Hub | Central; Regional; Branch; Service Hub | Kind of warehouse (Central, Regional, Branch, Service Hub). |
| `warehouse_capacity` | text | 0 | 6 | 29880 sqft to 45230 sqft | 45230 sqft; 38750 sqft; 41890 sqft; 36210 sqft | Warehouse floor area, stored as text with the unit "sqft" (for example "45230 sqft"). Must have " sqft" removed before use. |
| `service_center_available` | text | 0 | 2 | No to Yes | Yes; No | Whether the branch has a service centre (Yes or No). |
| `manager_id` | whole number | 0 | 6 | 4390 to 9593 | 9593; 6114; 7922; 4390 | Number identifying the branch manager. No manager table exists in the data. |
| `total_employees` | whole number | 0 | 6 | 49 to 92 | 92; 74; 81; 57 | Number of employees at the branch. |
| `avg_monthly_revenue` | whole number | 0 | 6 | 2894770 to 5234890 | 5234890; 4389120; 4567780; 3102455 | Average monthly revenue of the branch, as a whole number. Currency is not stated in the file. |
| `monthly_operational_cost` | whole number | 0 | 6 | 1743000 to 3189000 | 3189000; 2654000; 2739000; 1887000 | Monthly running cost of the branch, as a whole number. Currency is not stated in the file. |
| `market_demand_index` | whole number | 0 | 4 | 6 to 9 | 9; 8; 7; 6 | Score of market demand for the branch (6 to 9). The scale is not stated in the file. |

### 3.2 customers.csv

The 500 customers: type, location, credit and payment terms. 500 rows, 15 columns. Identifier: customer_id (unique).

| Column | Type | Empty | Distinct | Range | Example values | Meaning (inferred from the data) |
|--------|------|-------|----------|-------|----------------|----------------------------------|
| `customer_id` | text | 0 | 500 | C0001 to C0500 | C0001; C0002; C0003; C0004 | Customer code (C0001 to C0500). Key linking customers to orders and invoices. |
| `customer_type` | text | 0 | 5 | Corporate to Retail | Retail; Fleet Owner; Government; Dealer | Kind of customer (Corporate, Dealer, Fleet Owner, Government, Retail). |
| `industry_segment` | text | 0 | 6 | Construction to Rental | Construction; Manufacturing; Logistics; Rental | Industry the customer works in (for example Construction, Manufacturing, Logistics, Rental, Infrastructure). |
| `city` | text | 0 | 21 | Ahmedabad to Visakhapatnam | Kolkata; Hyderabad; Surat; Chennai | Customer city. |
| `state` | text | 0 | 16 | Andhra Pradesh to West Bengal | West Bengal; Telangana; Gujarat; Tamil Nadu | Customer state. |
| `pincode` | whole number | 0 | 500 | 100171 to 999725 | 991035; 464589; 202475; 576213 | Postal code of the customer (6 digits). Every customer has a different value. |
| `region` | text | 0 | 6 | Central to West | East; South; West; North | Region of the customer (6 values, including Central). |
| `branch_id` | text | 0 | 6 | AHM001 to PUN001 | KOL001; HYD001; DEL001; CHN001 | Branch that serves the customer. Links to branches.branch_id. |
| `credit_limit` | whole number | 0 | 500 | 10552 to 1493816 | 45896; 531506; 1306933; 506336 | Credit limit given to the customer, as a whole number. Currency is not stated in the file. |
| `current_balance` | whole number | 0 | 499 | 63 to 1435105 | 43284; 426735; 733495; 250255 | Balance the customer currently carries, as a whole number. Not tested against invoices and payments. |
| `payment_terms` | text | 0 | 5 | Advance to Net 60 | Advance; Net 30; Net 60; Net 45 | Agreed payment terms (Advance, Net 15, Net 30, Net 45, Net 60). |
| `customer_since` | date (stored as text) | 0 | 467 | 2015-01-07 to 2024-12-15 | 2024-04-10; 2022-01-19; 2017-02-14; 2017-12-20 | Date the customer joined, stored as text. DO NOT USE: it does not match the real first order (293 of 500 customers have a first order earlier than this date). |
| `last_purchase_date` | date (stored as text) | 0 | 449 | 2015-04-30 to 2024-12-27 | 2024-04-25; 2024-12-04; 2019-06-11; 2021-12-26 | Date of the customer last purchase, stored as text. DO NOT USE: it does not match the real last order (467 of 500 are earlier than the real last order). |
| `total_purchase_value` | whole number | 0 | 500 | 5173 to 4998438 | 3892818; 3168730; 3584708; 3810635 | Total purchases by the customer. DO NOT USE: it differs from the real order total by more than 1% for all 500 customers. |
| `customer_rating` | whole number | 0 | 5 | 1 to 5 | 4; 2; 3; 1 | Rating given to the customer, 1 to 5. The meaning of the scale is not stated in the file. |

### 3.3 inventory_master.csv

Stock settings for every product in every branch. 180 rows, 8 columns. Identifier: product_id + branch_id (unique, 30 x 6 = 180).

| Column | Type | Empty | Distinct | Range | Example values | Meaning (inferred from the data) |
|--------|------|-------|----------|-------|----------------|----------------------------------|
| `product_id` | text | 0 | 30 | P001 to P030 | P001; P002; P003; P004 | Product code. Links to products.product_id. |
| `branch_id` | text | 0 | 6 | AHM001 to PUN001 | DEL001; PUN001; CHN001; HYD001 | Branch code. Links to branches.branch_id. Product and branch together identify one row (30 products x 6 branches = 180). |
| `opening_stock` | whole number | 0 | 127 | 80 to 300 | 196; 198; 187; 226 | Stock of this product in this branch at the start of the data. |
| `reorder_level` | whole number | 0 | 75 | 24 to 119 | 59; 70; 62; 71 | Stock level at which this product should be reordered in this branch. |
| `safety_stock` | whole number | 0 | 63 | 16 to 85 | 46; 43; 44; 65 | Buffer stock to keep in this branch for this product. |
| `max_stock` | whole number | 0 | 148 | 131 to 582 | 340; 349; 311; 417 | Maximum stock to hold in this branch for this product. |
| `current_stock` | whole number | 0 | 180 | 88853 to 121013 | 115928; 109389; 99373; 108772 | Stock level at the end of the data. DO NOT USE AS IT IS: it is 88,853 to 121,013 units while opening stock is 80 to 300. It equals the final running_balance in stock_ledger, so it is inflated by buying about 18 units for every 1 sold. KPI 8 rebuilds the true stock. |
| `warehouse_bin` | text | 0 | 95 | A01 to F20 | C15; D16; A11; A10 | Storage bin location inside the warehouse (letter and number, for example C15). |

### 3.4 invoices.csv

One invoice per Delivered sales order. 18,033 rows, 10 columns. Identifier: invoice_id is NOT unique (393 rows repeated). so_id is unique.

| Column | Type | Empty | Distinct | Range | Example values | Meaning (inferred from the data) |
|--------|------|-------|----------|-------|----------------|----------------------------------|
| `invoice_id` | text | 0 | 17,836 | INV-100074 to INV-999969 | INV-325361; INV-568476; INV-786913; INV-697615 | Invoice number (INV-xxxxxx). NOT UNIQUE: 393 rows share an ID with another invoice (17,836 distinct IDs in 18,033 rows). |
| `so_id` | text | 0 | 18,033 | SO-100070 to SO-999960 | SO-990591; SO-431510; SO-495859; SO-745068 | Sales order the invoice belongs to. Links to sales_orders_header.so_id. Unique: one invoice per Delivered order. |
| `customer_id` | text | 0 | 500 | C0001 to C0500 | C0070; C0031; C0013; C0382 | Customer billed. Links to customers.customer_id. |
| `branch_id` | text | 0 | 6 | AHM001 to PUN001 | AHM001; KOL001; HYD001; CHN001 | Branch that issued the invoice. Links to branches.branch_id. |
| `invoice_date` | date (stored as text) | 0 | 2,203 | 2019-01-03 to 2025-01-13 | 2022-05-28; 2022-05-08; 2023-10-02; 2021-10-09 | Date the invoice was issued, stored as text. |
| `due_date` | date (stored as text) | 0 | 2,235 | 2019-01-20 to 2025-03-14 | 2022-06-27; 2022-06-07; 2023-11-01; 2021-11-08 | Date the invoice is due for payment, stored as text. |
| `total_order_value` | whole number | 0 | 16,248 | 310 to 8169760 | 1345720; 2194850; 2620800; 1094950 | Value of the order before GST. |
| `total_gst_amount` | decimal number | 0 | 16,951 | 55.8 to 2282221.8 | 374439.6; 609423; 733824; 292961 | GST (tax) charged on the order. |
| `grand_total` | decimal number | 0 | 16,938 | 365.8 to 10451981.8 | 1720159.6; 2804273; 3354624; 1387911 | Total to pay: order value plus GST. |
| `payment_status` | text | 0 | 3 | Paid to Unpaid | Unpaid; Paid; Partially Paid | Payment state of the invoice: Paid, Partially Paid or Unpaid. |

### 3.5 payments.csv

Payments received against invoices. 19,257 rows, 5 columns. Identifier: payment_id is NOT unique (403 rows repeated).

| Column | Type | Empty | Distinct | Range | Example values | Meaning (inferred from the data) |
|--------|------|-------|----------|-------|----------------|----------------------------------|
| `payment_id` | text | 0 | 19,055 | PAY-100018 to PAY-999938 | PAY-207775; PAY-942474; PAY-657121; PAY-684019 | Payment number (PAY-xxxxxx). NOT UNIQUE: 403 rows share an ID with another payment (19,055 distinct IDs in 19,257 rows). |
| `invoice_id` | text | 0 | 16,036 | INV-100074 to INV-999969 | INV-786913; INV-697615; INV-747829; INV-154762 | Invoice the payment is for. Links to invoices.invoice_id. One invoice can have several payments. |
| `payment_date` | date (stored as text) | 0 | 2,243 | 2019-01-09 to 2025-03-20 | 2023-10-07; 2021-11-23; 2024-12-26; 2022-04-07 | Date the payment was made, stored as text. |
| `payment_amount` | decimal number | 0 | 18,778 | 19.13 to 9970301 | 3354624; 1387911; 1475; 1050598 | Amount paid. |
| `payment_method` | text | 0 | 5 | Bank Transfer to UPI | Cheque; Credit Card; Bank Transfer; Cash | How the customer paid (Bank Transfer, Cash, Cheque, Credit Card, UPI). |

### 3.6 products.csv

The 30 products: price, cost, size, tax and stock settings. 30 rows, 23 columns. Identifier: product_id (unique).

| Column | Type | Empty | Distinct | Range | Example values | Meaning (inferred from the data) |
|--------|------|-------|----------|-------|----------------|----------------------------------|
| `product_id` | text | 0 | 30 | P001 to P030 | P001; P002; P003; P004 | Product code (P001 to P030). Key linking products to order lines, inventory and the ledger. |
| `product_name` | text | 0 | 30 | Air Filter AF-120 to Wheel Rim WR-24 | Hydraulic Pump HP-300; Engine Oil Filter OF-90; Air Filter AF-120; Track Chain TC-45 | Name of the product, including its model number. |
| `category` | text | 0 | 15 | Attachments to Undercarriage | Hydraulic; Filters; Undercarriage; Engine | Product category (15 values, for example Hydraulic, Filters, Engine, Electrical, Undercarriage). |
| `machine_type` | text | 0 | 6 | All to Loader | Excavator; Loader; Bulldozer; Crane | Type of machine the part fits (Excavator, Loader, Bulldozer, Crane, Dumper, or All). |
| `brand` | text | 0 | 6 | CAT to Volvo | CAT; JCB; Komatsu; Volvo | Machine brand the part fits (CAT, JCB, Komatsu, Volvo, Tata Hitachi and others). |
| `model_compatibility` | text | 0 | 18 | All Models to Volvo EC210 | CAT 320D; JCB 3DX; Komatsu PC210; Volvo D6 | Machine model the part fits (18 values, including All Models). |
| `unit_cost` | whole number | 0 | 30 | 180 to 95500 | 18950; 450; 820; 32500 | Standard cost of one unit. |
| `unit_price` | whole number | 0 | 28 | 310 to 119000 | 24500; 720; 1250; 39800 | Standard selling price of one unit. |
| `margin_percentage` | decimal number | 0 | 30 | 20.8 to 84.6 | 29.2; 37.3; 52.4; 22.5 | Profit percentage given in the file. DO NOT USE: it mixes two definitions. 29 of 30 products show markup on cost and P002 shows margin on price. Recompute as (unit_price - unit_cost) / unit_price. |
| `gst_rate` | whole number | 0 | 2 | 18 to 28 | 28; 18 | GST (tax) rate in percent (18 or 28). |
| `weight_kg` | decimal number | 0 | 29 | 0.2 to 320 | 38.5; 0.4; 0.8; 210 | Weight of one unit in kilograms. |
| `dimensions_cm` | text | 0 | 29 | 10x8x6 to 90x4x4 | 40x25x25; 10x8x8; 18x12x12; 180x40x35 | Size of one unit in centimetres, written as length x width x height (text). |
| `material_type` | text | 0 | 10 | alloy to synthetic | steel; paper; synthetic; alloy | Main material of the product (10 values, for example steel, rubber, alloy, paper, synthetic). |
| `warranty_months` | whole number | 0 | 5 | 3 to 24 | 12; 6; 18; 24 | Warranty length in months (3 to 24). |
| `reorder_level` | whole number | 0 | 20 | 3 to 200 | 15; 80; 60; 8 | Product-level reorder level. The branch-level values are in inventory_master. |
| `safety_stock` | whole number | 0 | 17 | 2 to 100 | 10; 40; 30; 4 | Product-level safety stock. The branch-level values are in inventory_master. |
| `max_stock_level` | whole number | 0 | 19 | 8 to 500 | 40; 200; 150; 20 | Product-level maximum stock level. The branch-level values are in inventory_master. |
| `lead_time_days` | whole number | 0 | 19 | 4 to 40 | 21; 7; 30; 18 | Lead time given in the product file (4 to 40 days). DO NOT USE: the real lead time is about 18 days for every product. Use suppliers.lead_time_days or the real purchase order dates. |
| `criticality_level` | text | 0 | 3 | High to Medium | High; Medium; Low | How important the part is (High, Medium, Low). |
| `usage_frequency` | text | 0 | 3 | High to Medium | Medium; High; Low | How often the part is used (High, Medium, Low). |
| `uom` | text | 0 | 3 | cartridge to set | piece; set; cartridge | Unit of measure: piece (24 products), set (5) or cartridge (1). Do not add quantities across products. |
| `last_purchase_price` | whole number | 0 | 30 | 170 to 94000 | 18500; 430; 800; 32000 | Price paid at the last purchase of this product. |
| `last_purchase_date` | date (stored as text) | 0 | 30 | 2019-03-15 to 2025-04-27 | 2019-03-15; 2019-07-22; 2020-01-10; 2020-06-18 | Date of the last purchase, stored as text. DO NOT USE: it runs to 2025-04-27, after the end of the order data. |

### 3.7 purchase_orders_header.csv

Purchase orders placed with suppliers (one row per order). 24,000 rows, 10 columns. Identifier: po_id (unique).

| Column | Type | Empty | Distinct | Range | Example values | Meaning (inferred from the data) |
|--------|------|-------|----------|-------|----------------|----------------------------------|
| `po_id` | text | 0 | 24,000 | PO-100001 to PO-999978 | PO-141186; PO-215234; PO-296826; PO-700652 | Purchase order number (PO-xxxxxx). Unique. Links to purchase_orders_lines.po_id. |
| `supplier_id` | text | 0 | 8 | SUP0001 to SUP0008 | SUP0001; SUP0005; SUP0008; SUP0004 | Supplier the order was placed with. Links to suppliers.supplier_id. |
| `branch_id` | text | 0 | 6 | AHM001 to PUN001 | PUN001; DEL001; AHM001; CHN001 | Branch the order is for. Links to branches.branch_id. |
| `order_date` | date (stored as text) | 0 | 2,192 | 2019-01-01 to 2024-12-31 | 2019-04-10; 2020-12-30; 2022-04-05; 2019-07-29 | Date the order was placed, stored as text. |
| `expected_delivery_date` | date (stored as text) | 0 | 2,207 | 2019-01-11 to 2025-01-28 | 2019-04-29; 2021-01-27; 2022-05-01; 2019-08-16 | Date delivery was promised, stored as text. |
| `received_date` | date (stored as text) | 2,370 | 2,207 | 2019-01-11 to 2025-01-28 | 2019-04-29; 2021-01-27; 2022-05-01; 2019-08-17 | Date the goods arrived, stored as text. Empty (2,370 rows) exactly for Cancelled orders. |
| `po_status` | text | 0 | 2 | Cancelled to Received | Received; Cancelled | Status of the order: Received or Cancelled. |
| `total_cost` | decimal number | 0 | 24,000 | 5675.01 to 87121612.67 | 22955232.11; 19406544.39; 1295425.79; 38753295.34 | Cost of the order before GST. |
| `total_gst_amount` | decimal number | 0 | 24,000 | 1021.5 to 24394051.55 | 6427464.99; 5428358.44; 362719.22; 10803118.14 | GST on the order. |
| `grand_total` | decimal number | 0 | 24,000 | 6696.51 to 111515664.22 | 29382697.1; 24834902.83; 1658145.01; 49556413.48 | Total of the order: cost plus GST. |

### 3.8 purchase_orders_lines.csv

The products and quantities on each purchase order. 155,495 rows, 9 columns. Identifier: po_id + line_number (unique).

| Column | Type | Empty | Distinct | Range | Example values | Meaning (inferred from the data) |
|--------|------|-------|----------|-------|----------------|----------------------------------|
| `po_id` | text | 0 | 24,000 | PO-100001 to PO-999978 | PO-141186; PO-215234; PO-296826; PO-700652 | Purchase order the line belongs to. Links to purchase_orders_header.po_id. |
| `line_number` | whole number | 0 | 12 | 1 to 12 | 1; 2; 3; 4 | Position of the line in the order (1 to 12). With po_id it identifies one line. |
| `product_id` | text | 0 | 30 | P001 to P030 | P004; P022; P030; P001 | Product bought. Links to products.product_id. |
| `quantity` | whole number | 0 | 281 | 20 to 300 | 234; 79; 46; 293 | Units bought on the line (20 to 300). |
| `unit_cost` | decimal number | 0 | 142,292 | 186.05 to 107090.25 | 34709.54; 12317.84; 13840.45; 16988.41 | Price paid for one unit on this line. It changes from line to line, unlike products.unit_cost. |
| `gst_rate` | whole number | 0 | 2 | 18 to 28 | 28; 18 | GST rate in percent on the line (18 or 28). |
| `line_total` | decimal number | 0 | 155,224 | 3763.6 to 32102145 | 8122032.36; 973109.36; 636660.7; 4977604.13 | Line value before GST: quantity x unit cost. |
| `gst_amount` | decimal number | 0 | 155,254 | 677.45 to 8988600.6 | 2274169.06; 272470.62; 178265; 1393729.16 | GST on the line. |
| `line_grand_total` | decimal number | 0 | 155,269 | 4441.05 to 41090745.6 | 10396201.42; 1245579.98; 814925.7; 6371333.29 | Line total plus GST. |

### 3.9 sales_orders_header.csv

Sales orders placed by customers (one row per order). 20,000 rows, 11 columns. Identifier: so_id (unique).

| Column | Type | Empty | Distinct | Range | Example values | Meaning (inferred from the data) |
|--------|------|-------|----------|-------|----------------|----------------------------------|
| `so_id` | text | 0 | 20,000 | SO-100070 to SO-999995 | SO-990591; SO-431510; SO-495859; SO-745068 | Sales order number (SO-xxxxxx). Unique. Links to sales_orders_lines.so_id and invoices.so_id. |
| `customer_id` | text | 0 | 500 | C0001 to C0500 | C0070; C0031; C0013; C0382 | Customer who placed the order. Links to customers.customer_id. |
| `branch_id` | text | 0 | 6 | AHM001 to PUN001 | AHM001; KOL001; HYD001; CHN001 | Branch that handled the order. Links to branches.branch_id. |
| `order_date` | date (stored as text) | 0 | 2,192 | 2019-01-01 to 2024-12-31 | 2022-05-24; 2022-05-01; 2023-10-01; 2021-10-04 | Date the order was placed, stored as text. |
| `delivery_date` | date (stored as text) | 0 | 2,204 | 2019-01-02 to 2025-01-13 | 2022-05-28; 2022-05-08; 2023-10-02; 2021-10-09 | Date the order was delivered, stored as text. Filled for every order, including Cancelled ones. |
| `order_status` | text | 0 | 2 | Cancelled to Delivered | Delivered; Cancelled | Status of the order: Delivered or Cancelled. |
| `payment_terms` | text | 0 | 5 | Advance to Net 60 | Net 30; Advance; Net 45; Net 15 | Payment terms of the order (Advance, Net 15, Net 30, Net 45, Net 60). |
| `total_order_value` | whole number | 0 | 17,905 | 310 to 8169760 | 1345720; 2194850; 2620800; 1094950 | Value of the order before GST. |
| `total_gst_amount` | decimal number | 0 | 18,749 | 55.8 to 2282221.8 | 374439.6; 609423; 733824; 292961 | GST on the order. |
| `grand_total` | decimal number | 0 | 18,735 | 365.8 to 10451981.8 | 1720159.6; 2804273; 3354624; 1387911 | Total of the order: value plus GST. |
| `sales_channel` | text | 0 | 4 | Counter Sale to Online | Counter Sale; Dealer Network; Online; Field Sales | Channel the order came through (Counter Sale, Dealer Network, Field Sales, Online). |

### 3.10 sales_orders_lines.csv

The products and quantities on each sales order. 130,402 rows, 9 columns. Identifier: so_id + line_number (unique).

| Column | Type | Empty | Distinct | Range | Example values | Meaning (inferred from the data) |
|--------|------|-------|----------|-------|----------------|----------------------------------|
| `so_id` | text | 0 | 20,000 | SO-100070 to SO-999995 | SO-990591; SO-431510; SO-495859; SO-745068 | Sales order the line belongs to. Links to sales_orders_header.so_id. |
| `line_number` | whole number | 0 | 12 | 1 to 12 | 1; 2; 3; 4 | Position of the line in the order (1 to 12). With so_id it identifies one line. |
| `product_id` | text | 0 | 30 | P001 to P030 | P018; P026; P013; P004 | Product sold. Links to products.product_id. |
| `quantity` | whole number | 0 | 20 | 1 to 20 | 19; 2; 3; 9 | Units sold on the line (1 to 20). |
| `unit_price` | whole number | 0 | 28 | 310 to 119000 | 11300; 310; 1450; 39800 | Selling price of one unit on this line. Same range as products.unit_price. |
| `gst_rate` | whole number | 0 | 2 | 18 to 28 | 28; 18 | GST rate in percent on the line (18 or 28). |
| `line_total` | whole number | 0 | 534 | 310 to 2380000 | 214700; 620; 930; 13050 | Line value before GST: quantity x unit price. |
| `gst_amount` | decimal number | 0 | 534 | 55.8 to 666400 | 60116; 111.6; 167.4; 2349 | GST on the line. |
| `line_grand_total` | decimal number | 0 | 534 | 365.8 to 3046400 | 274816; 731.6; 1097.4; 15399 | Line total plus GST. |

### 3.11 stock_ledger.csv

Every stock movement in and out of the branches. 237,230 rows, 9 columns. Identifier: movement_id (unique).

| Column | Type | Empty | Distinct | Range | Example values | Meaning (inferred from the data) |
|--------|------|-------|----------|-------|----------------|----------------------------------|
| `movement_id` | text | 0 | 237,230 | MOV-ADJ-0 to MOV-OUT-258012 | MOV-OUT-184742; MOV-OUT-208422; MOV-IN-111128; MOV-OUT-159140 | Unique number of the stock movement (MOV-IN-, MOV-OUT- or MOV-ADJ- followed by a number). |
| `product_id` | text | 0 | 30 | P001 to P030 | P001; P002; P003; P004 | Product that moved. Links to products.product_id. |
| `branch_id` | text | 0 | 6 | AHM001 to PUN001 | AHM001; CHN001; DEL001; HYD001 | Branch where the stock moved. Links to branches.branch_id. |
| `movement_type` | text | 0 | 3 | ADJUSTMENT to OUT | OUT; IN; ADJUSTMENT | Direction of the movement: IN (received), OUT (sold) or ADJUSTMENT (stock correction, direction read from the change in running_balance). |
| `movement_date` | date (stored as text) | 0 | 2,218 | 2019-01-01 to 2025-01-28 | 2019-01-03; 2019-01-05; 2019-01-23; 2019-01-25 | Date of the movement, stored as text. |
| `quantity` | whole number | 0 | 300 | 1 to 300 | 8; 12; 174; 11 | Units moved (1 to 300). Always positive: the direction comes from movement_type. |
| `reference_type` | text | 0 | 3 | ADJ to SO | SO; PO; ADJ | Document the movement came from: PO (purchase order), SO (sales order) or ADJ (adjustment). |
| `reference_id` | text | 0 | 43,709 | ADJ-0 to SO-999995 | SO-157767; SO-168735; PO-181094; SO-947377 | Number of that document (PO-xxxxxx, SO-xxxxxx or ADJ-n). Links to purchase_orders_header.po_id or sales_orders_header.so_id. ADJ rows match no order, by design. |
| `running_balance` | whole number | 0 | 98,171 | 14 to 121013 | 228; 216; 390; 378 | Stock balance of the product in the branch after this movement. It changes by exactly the quantity on every entry. |

### 3.12 suppliers.csv

The 8 suppliers: location, lead time and reliability. 8 rows, 12 columns. Identifier: supplier_id (unique).

| Column | Type | Empty | Distinct | Range | Example values | Meaning (inferred from the data) |
|--------|------|-------|----------|-------|----------------|----------------------------------|
| `supplier_id` | text | 0 | 8 | SUP0001 to SUP0008 | SUP0001; SUP0002; SUP0003; SUP0004 | Supplier code (SUP0001 to SUP0008). Key linking suppliers to purchase orders. |
| `supplier_name` | text | 0 | 8 | Guangzhou Local Vendor Supplies to Tianjin OEM Supplies | Shenzhen OEM Supplies; Guangzhou Local Vendor Supplies; Shanghai Distributor Supplies; Ningbo Distributor Supplies | Name of the supplier. |
| `supplier_type` | text | 0 | 3 | Distributor to OEM | OEM; Local Vendor; Distributor | Kind of supplier (OEM, Distributor, Local Vendor). |
| `product_category` | text | 0 | 3 | Large Parts to Small Parts | Medium Parts; Small Parts; Large Parts | Size class of parts the supplier provides (Small Parts, Medium Parts, Large Parts). |
| `city` | text | 0 | 8 | Guangzhou to Tianjin | Shenzhen; Guangzhou; Shanghai; Ningbo | Supplier city. |
| `province` | text | 0 | 6 | Guangdong to Zhejiang | Guangdong; Shanghai; Zhejiang; Tianjin | Supplier province. |
| `region` | text | 0 | 3 | East China to South China | South China; East China; North China | Supplier region (East China, North China, South China). |
| `pincode` | whole number | 0 | 8 | 202099 to 966203 | 586808; 966203; 655757; 536477 | Postal code of the supplier. |
| `lead_time_days` | whole number | 0 | 7 | 10 to 28 | 19; 10; 16; 18 | Promised delivery time in days (10 to 28). Matches the promised days on the purchase orders for all 8 suppliers. |
| `reliability_score` | whole number | 0 | 3 | 3 to 5 | 5; 4; 3 | Reliability score of the supplier (3 to 5). The scale is not stated in the file. |
| `import_duty_rate` | whole number | 0 | 4 | 5 to 15 | 8; 12; 5; 15 | Import duty rate in percent (5 to 15). |
| `china_tax_id` | text | 0 | 8 | 39GUANGZ946640 to 96SHENZH983302 | 90SUZHOU398241; 89TIANJI357980; 96SHENZH983302; 39GUANGZ946640 | Supplier tax identification code (text). |

---

## 4. Columns that need care

These findings come from the Week 3 checks. The rule for the project is: the transaction tables (orders, order lines, invoices, payments) agree with each other and are the source of truth. Summary columns in the master tables are not used.

### 4.1 Do not use

| Column | Why | What to use instead |
|--------|-----|---------------------|
| `customers.customer_since` | 293 of 500 customers have a real first order earlier than this date | First order date from `sales_orders_header` |
| `customers.last_purchase_date` | 467 of 500 are earlier than the real last order | Last order date from `sales_orders_header` |
| `customers.total_purchase_value` | Differs from the real order total by more than 1% for all 500 customers | Total from `sales_orders_header` and `invoices` |
| `products.margin_percentage` | Mixes two definitions (29 of 30 are markup on cost, P002 is margin on price) | Recompute: (unit_price - unit_cost) / unit_price |
| `products.lead_time_days` | Real lead time is about 18 days for every product, not 4 to 40 | `suppliers.lead_time_days` or real purchase order dates |
| `products.last_purchase_date` | Runs to 2025-04-27, after the end of the order data | Purchase order dates |
| `inventory_master.current_stock` | 88,853 to 121,013 units against an opening stock of 80 to 300. It equals the last `running_balance` in the ledger | Reconstructed stock balance (KPI 8) |

### 4.2 Not unique

| Column | Finding |
|--------|---------|
| `invoices.invoice_id` | 18,033 rows but 17,836 distinct IDs. 393 rows in 196 groups share an ID with an unrelated invoice |
| `payments.payment_id` | 19,257 rows but 19,055 distinct IDs. 403 rows in 201 groups share an ID |

These rows are not repaired. They are left out only where invoices are joined to payments.

### 4.3 Format to fix before use

| Column | Finding |
|--------|---------|
| `branches.warehouse_capacity` | Text such as "45230 sqft". Remove " sqft" and convert to a number |
| All date columns | Stored as text. All convert to real dates with 0 failures |
| `products.uom` | Mixes piece (24 products), set (5) and cartridge (1). Do not add quantities across products |

### 4.4 Empty values

Only one column in the 12 files has empty values: `purchase_orders_header.received_date` has 2,370 empty values. They are exactly the 2,370 Cancelled purchase orders, so they are expected. Every other column has 0 empty values.

### 4.5 Cancelled orders keep their lines

Cancelled sales orders (1,967) keep 12,746 lines in `sales_orders_lines`, and cancelled purchase orders (2,370) keep 15,418 lines in `purchase_orders_lines`. Filter to Delivered sales orders and Received purchase orders before counting.

---

## 5. Rules checked against the data

These rules were tested in Week 3 and found to hold (details in `data-cleaning-log.md`):

- Every other identifier is unique with no empty IDs. Each product and branch pair appears once in `inventory_master` (180 = 30 x 6).
- Links between tables: all 17 direct links have 0 empty values and 0 orphans. In `stock_ledger`, the PO and SO references have 0 orphans. The 2,454 ADJ rows match no order, by design.
- Every Delivered sales order has one invoice, and no invoice belongs to a non-Delivered order.
- Date-order rules have 0 violations in the Week 3 checks.
- Maths rules hold with 0 errors: order lines (quantity x price), GST, order headers against lines, invoices against orders, and purchase order totals.
- Invoice and order customer and branch agree. Order payment terms and branch match the customer record. The ledger branch matches the order branch.
- `payment_status` agrees with the sum of payments for all 17,640 invoices with a unique ID. There are no overpayments.
- In `stock_ledger`, `running_balance` changes by exactly the quantity on every entry, and the file is in date order.
- About 10% of order lines never reached the ledger (at random), so order lines are the complete source for demand and purchases.

---

## 6. What this dictionary does not claim

- Meanings are inferred from names and values. Where the profile cannot show the scale or the currency, the entry says so (for example `market_demand_index`, `customer_rating`, `reliability_score`, and the money columns in `branches`).
- `customers.current_balance` was not tested against invoices and payments.
- Range and example values come from the profile. Example values are the first values found, not a ranking.
- Order data runs from 2019-01-01 to 2024-12-31. Some dates run later because they follow the order (invoice dates to 2025-01-13, payments to 2025-03-20, ledger to 2025-01-28). The only column that runs past the end of the orders without a reason is `products.last_purchase_date` (2025-04-27).
