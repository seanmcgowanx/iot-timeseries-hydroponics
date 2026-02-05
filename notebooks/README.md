# ASFP Hydroponics IoT — EDA + Cleaned Feature Dataset

This repo contains exploratory data analysis (EDA) and a cleaned/feature-engineered dataset for downstream modeling on the ASFP hydroponics IoT sensor data.

## Steps
- Loaded raw IoT CSV
- Standardized column names and parsed timestamps (UTC-aware)
- Coerced sensor columns to numeric (non-numeric → NaN)
- Computed time gaps (`dt_seconds`) within each sensor stream
- Split the data into continuous **segments** so we don’t “leak” values across outages
- Cleaned sensor artifacts (notably placeholder zeros in some sensors)
- Filled **short** missing stretches within a segment only (limited forward-fill)
- Added basic time features + 1-hour rolling means for modeling

---

## Files
- `Group_Project_EDA_Refined (3).ipynb`  
  The Colab notebook that performs the EDA + cleaning + feature engineering.
- `asfp_cleaned_features.csv`  
  
## Cleaning / processing logic (details)

### 1) Timestamp parsing
- Column `time` is parsed with:
  - `pd.to_datetime(..., utc=True, errors="coerce")`
- Rows with invalid timestamps are dropped.

### 2) Numeric coercion
- `water_temp`, `ph`, `ec`, `do` are forced numeric using `errors="coerce"`
- Any non-numeric values become `NaN`.

### 3) Gaps + segmentation (prevents leakage across outages)
- Data is sorted by `sensor`, then `time`.
- `dt_seconds` = time difference between consecutive readings **within each sensor**.
- A new `segment` begins when `dt_seconds > 600` seconds (10 minutes).
  - Rationale: expected cadence is ~5 minutes (~300s). 10 minutes is a clear break.

### 4) Placeholder zero cleanup
- For `ph`, `ec`, and `do`: exact zeros are treated as invalid and converted to `NaN`.

### 5) Short-gap filling (within segment only)
- Forward-fill is applied **within (`sensor`, `segment`) groups only**.
- Fill limit: `12` rows (~1 hour at 5-minute sampling).
- This fills brief dropouts but avoids “smearing” values across outages.

---

## Feature engineering

### Time features
- `hour` = hour of day (0–23)
- `dayofweek` = day of week (Mon=0 … Sun=6)
- `month` = month number (1–12)

### Rolling mean features (1-hour window)
- Window size: `12` samples (≈ 1 hour at 5-min cadence)
- Computed within (`sensor`, `segment`) so it never crosses outages.
- Uses `min_periods=3` (first few rows in a segment may have NaNs in rolling columns).

---

## Column dictionary (asfp_cleaned_features.csv)

| Column | Type | Meaning |
|---|---:|---|
| `time` | string (UTC timestamp) | Timestamp of the reading (UTC). Parsed as datetime in notebook, saved as string in CSV export. |
| `sensor` | int | Sensor ID / source stream. |
| `water_temp` | float | Water temperature measurement. |
| `ph` | float | pH measurement (zeros converted to NaN before fill). |
| `ec` | float | Electrical conductivity measurement (zeros converted to NaN before fill). |
| `do` | float | Dissolved oxygen measurement (zeros converted to NaN before fill). |
| `dt_seconds` | float | Time gap (seconds) from previous reading within the same sensor. First row per sensor is NaN. |
| `segment` | int | Continuous-run ID within each sensor; increments when a gap > 10 minutes occurs. |
| `hour` | int | Hour extracted from `time` (0–23). |
| `dayofweek` | int | Day of week extracted from `time` (Mon=0 … Sun=6). |
| `month` | int | Month extracted from `time` (1–12). |
| `water_temp_roll1h_mean` | float | 1-hour rolling mean of `water_temp` (12 samples), within sensor+segment. |
| `ph_roll1h_mean` | float | 1-hour rolling mean of `ph`, within sensor+segment. |
| `ec_roll1h_mean` | float | 1-hour rolling mean of `ec`, within sensor+segment. |
| `do_roll1h_mean` | float | 1-hour rolling mean of `do`, within sensor+segment. |

---

## Notes / caveats
- Missing values can remain after cleaning if:
  - the value was truly missing,
  - the dropout exceeded the 1-hour fill limit, or
  - rolling means don’t have enough history early in a segment.
- If modeling, you may still need:
  - explicit imputation strategy,
  - scaling/normalization,
  - train/test split that respects time and sensor segmentation.

---

## Quick usage example (Python)
```python
import pandas as pd

df = pd.read_csv("asfp_cleaned_features.csv")
df["time"] = pd.to_datetime(df["time"], utc=True)

# Example: filter one sensor
s1 = df[df["sensor"] == 1].sort_values("time")
