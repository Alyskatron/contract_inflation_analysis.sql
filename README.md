# Contract Inflation Analysis (Sanitized)

**Purpose:**  
This repository contains a sanitized BigQuery SQL script (`contract_inflation_analysis.sql`) that computes contract-level price inflation metrics between two years (2023 and 2024). The original script has been generalized to remove proprietary names and dataset references so it is safe to publish on GitHub.

---

## Contents

- `contract_inflation_analysis.sql` — main BigQuery SQL script (parameterized).
- `.gitignore` — recommended ignore file for Git.
- `README.md` — this file.

---

## High-level approach

1. Identify contracts that have purchases in both years (2023 & 2024).
2. Compute SKU-level average prices, quantities, and spend per year.
3. Pair 2023 vs 2024 SKU prices within each contract and compute percentage price change.
4. Aggregate SKU price changes to contract-level metrics:
   - weighted inflation (spend-weighted),
   - median inflation,
   - calibrated inflation (benchmarked to 5%),
   - confidence scoring,
   - descriptive flags (e.g., "High Inflation").
5. Output contract-level results ordered by total base spend.

---

## Required input schema (column names used by the script)

**purchase_table** (example columns used in the query)
- `product_id` (string/int) - product identifier
- `sku` (string) - product SKU
- `category_name` (string) - category
- `invoice_date` (DATE or TIMESTAMP) - invoice / transaction date
- `price_each` (numeric/float) - price (or extended price if already per unit)
- `unit_quantity` (numeric/float) - quantity purchased
- `data_source` (string) - used to filter rows (script expects `'APFeed File'`)
- `facility_id` (string) - facility identifier

**itemprice_table** (example columns used in the query)
- `product_id` (string/int) - product identifier (join key)
- `contract_number` (string) - contract identifier
- `tier_name` (string) - tier or pricing tier metadata

> The script assumes `product_id` is the join key between purchase and itemprice tables.

---

## How to configure before running

1. Replace the placeholders in `contract_inflation_analysis.sql`:
   - `{{PROJECT}}` → your GCP project id
   - `{{DATASET}}` → dataset name
   - `purchase_table`, `itemprice_table` → actual table names

2. Verify column names in your tables match the script. If your column names differ, rename them in the script (e.g., `invoice_date` → `po_date`).

3. (Optional) Adjust filters/thresholds:
   - `p.data_source = 'APFeed File'` — change if your source label differs.
   - `ABS(cp.price_change_pct) < 100` — outlier filter.
   - `cp.spend_2023 > 100` and `HAVING COUNT(*) >= 3` — inclusion thresholds.

---

## Running the query

**BigQuery UI (console):**
1. Open BigQuery in the Google Cloud Console.
2. Copy the contents of `contract_inflation_analysis.sql`.
3. Replace placeholders with real values.
4. Run the query.

**bq CLI:**
```bash
bq query --use_legacy_sql=false "$(cat contract_inflation_analysis.sql)"
