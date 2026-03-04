# Data Merge Pipeline README

## Overview

This README documents the process of building `ONGB_final_clean.csv` — a merged, feature-enriched longitudinal student dataset combining OUSD student records, geocoded home-to-school distances, and neighborhood socioeconomic data from the US Census.

---

## Input Files

| File | Description | Shape |
|---|---|---|
| `ONGB_with_distances.csv` | Base student dataset with year-by-year records and geocoded distances | 79,460 × 178 |
| `oakland_socioeconomic_by_zip.csv` | ACS census data by Oakland zip code and year | 120 × 18 |

### Why not `abs_with_new_features.csv`?

An earlier dataset (`abs_with_new_features.csv`) contained additional features including census data and a real student ID (`ID`). We attempted to merge this into ONGB but **could not establish a reliable join key** because:

- `abs_with_new_features` uses real student IDs (e.g. `443282`)
- `ONGB_with_distances` uses anonymous sequential IDs (`ANON_ID`: 1, 2, 3...)
- There is **zero overlap** between the two ID columns
- Fuzzy matching on `Birthdate + Gender + Zip + Ethnicity` produced too many duplicates to be reliable (23,669 duplicate keys in ONGB)

**Decision:** Rather than risk incorrect student matches, all census and socioeconomic features were pulled fresh from `oakland_socioeconomic_by_zip.csv` and joined directly to each student-year record using home zip code + academic year.

---

## Merge Steps

### Step 1 — Build Long-Format Address Table

Each student has up to 7 home zip codes (one per academic year). We unpivot the wide-format ONGB data into a long table with one row per student per year, extracting the residential zip code (`Zip_{y}.1`) for each year.

```python
# Academic year labels mapped to census calendar years
year_map = {
    '1718': 2017, '1819': 2018, '1920': 2019,
    '2021': 2020, '2122': 2021, '2223': 2022, '2324': 2023
}
```

Zip codes are cleaned to remove malformed values (e.g. `'9460i'` → extracted with regex, zero-padded to 5 digits).

### Step 2 — Join Census Data by Zip + Year

The socioeconomic file is joined to the long address table on `zip_code` and `year`. This gives each student-year record the neighborhood characteristics for where they lived that year.

**15 census features joined:**
- `total_population`, `poverty_universe`, `below_poverty`, `poverty_rate_pct`
- `median_household_income`, `labor_force`, `unemployed`, `unemployment_rate_pct`
- `education_universe_25plus`, `high_school_plus_rate_pct`, `college_degree_rate_pct`
- `median_gross_rent`, `median_home_value`
- `health_insurance_universe`, `uninsured_rate_pct`

### Step 3 — Pivot Back to Wide Format

The long-format census data is pivoted back to one row per student, creating year-suffixed columns (e.g. `poverty_rate_pct_1718`, `poverty_rate_pct_1819`, ...).

All columns are added using `pd.concat` rather than iterative `df[col] =` assignment to avoid DataFrame fragmentation.

### Step 4 — Engineer Prior-Year Features

16 features are computed from each student's historical year-by-year data. All features use only data **prior to 2023-24** to avoid data leakage in modeling.

| Feature | Description |
|---|---|
| `has_prior_data` | Whether student has any data before 2023-24 |
| `prior_years_enrolled` | Number of years enrolled before 2023-24 |
| `prev_days_enr` | Days enrolled in most recent prior year |
| `prev_days_abs` | Days absent in most recent prior year |
| `prev_att_rate` | Attendance rate in most recent prior year |
| `prev_chronic` | Chronic absence flag in most recent prior year |
| `prior_mean_att_rate` | Mean attendance rate across all prior years |
| `prior_min_att_rate` | Worst attendance rate across all prior years |
| `prior_chronic_count` | Number of prior years with chronic absence |
| `school_mean_att` | Mean attendance rate at student's 2023-24 school |
| `school_chronic_rate` | Chronic absence rate at student's 2023-24 school |
| `SED_SED` | Binary: SED in most recent year |
| `SED_Not SED` | Binary: Not SED in most recent year |
| `SED_Unknown` | Binary: SED status missing |
| `Eth_Asian` | Binary: ethnicity is Asian in most recent year |
| `Fluency_EO` | Binary: English Only in most recent year |

### Step 5 — Clean Up Columns

- Removed 28 **latitude/longitude columns** (`school_lat_*`, `school_lon_*`, `home_lat_*`, `home_lon_*`) — distances already computed
- Converted 7 **distance columns from km to miles** (`dist_km_*` → `dist_miles_*`, multiplied by 0.621371)

---

## Output File

**`ONGB_final_clean.csv`**

| Attribute | Value |
|---|---|
| Rows | 79,460 |
| Columns | 271 |
| Student identifier | `ANON_ID` (anonymous sequential integer) |

---

## Census Match Rates

Coverage is limited to Oakland zip codes in the source file. Students with addresses outside Oakland or with unclean zip codes will have null census values.

| Academic Year | Students Matched | Match Rate |
|---|---|---|
| 2017-18 | 39,717 | 50.0% |
| 2018-19 | 39,308 | 49.5% |
| 2019-20 | 38,752 | 48.8% |
| 2020-21 | 37,187 | 46.8% |
| 2021-22 | 37,236 | 46.9% |
| 2022-23 | 36,892 | 46.4% |
| 2023-24 | 36,885 | 46.4% |

---

## Known Issues & Decisions

| Issue | Decision |
|---|---|
| `ANON_ID` in ONGB does not match real student IDs in `abs_with_new_features` | Could not merge — census features re-pulled from source file instead |
| Some zip codes malformed (e.g. `'9460i'`) | Cleaned with regex extraction before joining |
| Students outside Oakland zip codes have no census match | Left as null — do not impute, flag during modeling |
| `DtypeWarning` on column 102 of ONGB | Load with `low_memory=False` to suppress |
| DataFrame fragmentation warning during pivot | Fixed by using `pd.concat` instead of iterative column assignment |

---

## How to Run

```python
# Dependencies
import pandas as pd
import numpy as np

# Input files needed in working directory:
# - ONGB_with_distances.csv
# - oakland_socioeconomic_by_zip.csv

# Run the full pipeline
# Output: ONGB_final_clean.csv
```

See `01_merge_pipeline.ipynb` for the full annotated code.

---

## Requirements

```
pandas
numpy
openpyxl
```

Install with:
```bash
pip install pandas numpy openpyxl
```
