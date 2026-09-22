# KPI 1 — Stock Accumulation Ratio

**Sprint:** 2
**Status:** Fully verified

## 1. Definition

**KPI Name:** Stock Accumulation Ratio

**Formula:**
```
Stock Accumulation Ratio = Total IN quantity ÷ Total OUT quantity
(calculated per product, per branch, over a given period)
```

**Data source:** `stock_ledger` — using `movement_type`, `quantity`, `product_id`, `branch_id`

**How to read it:**
- Ratio ≈ 1.0 → stock coming in roughly matches stock going out (healthy)
- Ratio significantly above 1.0 → stock accumulating faster than it sells (overstock risk)
- Ratio below 1.0 → stock depleting faster than it's replenished (stockout risk)

**Business question it answers:** Which products are accumulating excess stock, and which are at risk of running out — using trustworthy movement data, since `inventory_master.current_stock` itself is known to be unreliable (see Week 1 findings).

**Maps to CadetX's stated business value:** "Detect stock risks and operational anomalies early" and "Optimise inventory levels and reduce capital locked in stock."

## 2. Methodology Note

The ratio was calculated using two independent pandas methods — `groupby` aggregation and `pivot_table` — to confirm the calculation logic itself was correct. Both methods produced identical results across all 30 products (max difference: 0.0), confirming the formula is implemented correctly.

## 3. Verification Log

All 8 checks were run against the full `stock_ledger` dataset (237,230 rows, 2019-01-01 to 2025-01-28, 30 products, 6 branches) before this KPI was trusted for reporting.

| # | Check | Method | Result |
|---|---|---|---|
| 1 | Missing values | `.isnull().sum()` on product_id, branch_id, movement_type, quantity | 0 nulls found |
| 2 | Zero-OUT edge case | Compared product-months with IN but no OUT | 0 cases (lifetime and monthly) |
| 3 | Zero-IN edge case | Compared product-months with OUT but no IN | 0 cases (lifetime and monthly) |
| 4 | Totals reconciliation | Row counts by movement_type vs. total table rows | Reconciled only after identifying a third category, ADJUSTMENT (see Data Quality Notes) |
| 5 | Formula cross-check | groupby vs. pivot_table | Identical results, 0.0 difference |
| 6 | Outlier check | max ÷ mean delivery size per product | Max delivery never more than ~1.9x average — no bulk-order outliers |
| 7 | History length per product | min/max movement_date span per product | All 30 products span 2,205–2,219 days (~6 years), negligible variation |
| 8 | Manual spot-check | Hand-verified 3 products spanning the full ratio range | P029 (17.42), P003 (18.11), P024 (19.22) — all confirmed correct by hand, each backed by 3,400+ to 4,300+ real transactions |

## 4. Data Quality Notes

**Note 1 — Unusually clean activity coverage:** No product-month in the 6-year history has zero IN or zero OUT activity. This is unusually consistent for operational data and most likely reflects that this is a synthetically generated training dataset rather than real-world data, where such gaps are normal. This assumption should be re-verified before applying this methodology to live production data.

**Note 2 — ADJUSTMENT movements excluded:** `stock_ledger.movement_type` contains a third category beyond IN/OUT: `ADJUSTMENT` (2,454 rows, ~1% of the dataset). All ADJUSTMENT rows have `reference_type = 'ADJ'` — not tied to any purchase order or sales order — confirming they represent stock corrections (e.g. stocktake adjustments, damage write-offs) rather than genuine deliveries or sales. These rows are deliberately excluded from the ratio, since including them would introduce noise unrelated to actual supply-vs-demand flow. This means the KPI does not capture the complete picture of stock changes (e.g. write-offs) and should not substitute for a full stock reconciliation if one is needed elsewhere in the project.

**Note 3 — No bulk-order outliers:** The largest single delivery for any product was never more than ~1.9x that product's average delivery size. This confirms that high ratios reflect a consistent, ongoing over-delivery pattern rather than being caused by rare, one-off bulk orders — meaning the accumulation problem is structural and recurring, not a historical accident.

**Note 4 — Uniform product history:** All 30 products have almost identical history lengths (~6 years each, std ≈ 3.5 days), so no product's baseline is built on insufficient data. Combined with Note 1, this reinforces that the dataset is likely synthetic.

## 5. Business Interpretation

Across all 30 products, only roughly 5–6% of everything ever delivered into the warehouse has actually been sold. For example:
- **P029** (lowest ratio in the dataset): 673,125 units delivered vs. 38,648 sold — a 5.7% sell-through rate
- **P024** (highest ratio in the dataset): 690,571 units delivered vs. 35,924 sold — roughly 95% of everything delivered has never sold

This pattern holds consistently across thousands of individual transactions per product (4,000+ deliveries, 3,400+ sales each) over the full 6-year history — not a one-time event or a small sample. Every one of the 30 products shows a ratio between 17.42 and 19.22, an extremely narrow and consistent range.

This is direct, verified evidence of the exact problem described in the CadetX project brief: *"Warehouses and supply-chain teams often operate reactively... leading to excess inventory."* The consistency across every product and thousands of transactions indicates a structural, ongoing disconnect between ordering decisions and actual customer demand — not an isolated incident.
