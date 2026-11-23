# Cafe Sales Data Cleaning Pipeline

An end-to-end data cleaning and preprocessing pipeline built in Python and pandas to transform a messy 10,000-row cafe POS dataset (`dirty_cafe_sales.csv`) into an analysis-ready dataset (`clean_coffee_sales.csv`).

Instead of blindly dropping incomplete rows or imputing column averages, this workflow uses **business logic**, **relational lookups**, and **algebraic recovery** to retain **99.74%** of the original records without introducing fabricated noise.

---

## Cleaning Methodology

1. **Standardized Ingestion & Schema Normalization**
   - Parsed placeholder error strings (`UNKNOWN`, `ERROR`, `nan`, `NaN`, empty strings) directly into proper `NaN` values upon load.
   - Standardized column headers to `snake_case` and verified zero duplicate records.

2. **Relational & Algebraic Recovery**
   - **Item $\rightarrow$ Price Lookup:** Mapped known menu items to their exact unit prices to recover missing `price_per_unit` entries.
   - **Math Identity Recovery:** Applied $\text{Total Spent} = \text{Quantity} \times \text{Price Per Unit}$ across three passes to recover missing totals, quantities, and unit prices.
   - **1-to-1 Reverse Price Lookup:** Built a strict scalar mapping from `price_per_unit` back to `item` *only* for prices that uniquely identify a single menu item (recovering 495 missing item names without mislabeling overlapping price tiers).

3. **Integrity Checks & Strategic Missing-Value Handling**
   - **Math Audit:** Verified zero discrepancies (`|quantity * price_per_unit - total_spent| <= 1e-5`) across all recovered records.
   - **High-Missingness Categoricals:** Replaced missing `location` (39.61%) and `payment_method` (31.78%) values with `'Not Recorded'`, and remaining ambiguous items with `'Unknown Item'`, preserving missingness as an informative category rather than discarding ~40% of the dataset.
   - **Unrecoverable Records:** Dropped the 26 rows (0.26%) where both `quantity` and `total_spent` (or `price_per_unit`) were simultaneously missing and impossible to calculate.
   - **Timeline Preservation:** Confirmed `transaction_id` values are randomly distributed rather than chronological; left the 460 missing `transaction_date` entries as `NaT` instead of forward-filling inaccurate dates.

4. **Strict Type Casting**
   - Stripped the `TXN_` prefix and cast `transaction_id` to `int64`.
   - Cast `quantity` to `int64`, categorical text columns to pandas' native `string` dtype, and `transaction_date` to `datetime64[ns]`.

---

## Before vs. After Summary

| Column | Raw Missing Count | Raw Missing % | Cleaned Missing Count | Cleaned Dtype | Handling Strategy |
| :--- | :---: | :---: | :---: | :--- | :--- |
| `transaction_id` | 0 | 0.00% | 0 | `int64` | Stripped `TXN_` prefix & cast to integer |
| `item` | 969 | 9.69% | 0 | `string` | 1-to-1 price lookup + `'Unknown Item'` (474 rows) |
| `quantity` | 479 | 4.79% | 0 | `int64` | Derived via `total / price`; dropped 26 unrecoverable rows |
| `price_per_unit` | 533 | 5.33% | 0 | `float64` | Item price lookup + `total / quantity` |
| `total_spent` | 502 | 5.02% | 0 | `float64` | Derived via `quantity * price`; dropped 26 unrecoverable rows |
| `payment_method` | 3,178 | 31.78% | 0 | `string` | Imputed as `'Not Recorded'` (3,168 remaining rows) |
| `location` | 3,961 | 39.61% | 0 | `string` | Imputed as `'Not Recorded'` (3,952 remaining rows) |
| `transaction_date` | 460 | 4.60% | 460 (`NaT`) | `datetime64[ns]` | Preserved as `NaT` (non-sequential IDs prevent `ffill`) |

* **Total Rows:** `10,000` raw $\rightarrow$ `9,974` cleaned (**99.74% data retention**)

---

## How to Run

1. Clone the repository:
   ```bash
   git clone [https://github.com/](https://github.com/)<your-username>/cafe-sales-data-cleaning.git
   cd cafe-sales-data-cleaning
   ```
2. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```
3. Open `cafe_sales_cleaning.ipynb` in Jupyter Notebook or VS Code and run all cells to generate `clean_coffee_sales.csv`.
