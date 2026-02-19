# Chronic Absenteeism Data Cleaning Pipeline

## Overview

This notebook (`pivot_chronic_clean.ipynb`) cleans and reshapes a raw chronic absenteeism dataset from Oakland Unified School District (OUSD) into analysis- and ML-ready formats.

**Input:** `RAWChronic Absent Students with DOB_2025-05-29.csv`  
**Outputs:** `RAWChronic_Cleaned.csv` and `RAWChronic_Cleaned_Pivoted.csv`

---

## Source Data

| Property | Value |
|---|---|
| Rows | 18,638 students |
| Columns | 28 |
| Date pulled | 2025-05-29 |

**Key fields:** Student ID, name, birthdate, school site, grade, attendance metrics (days enrolled/absent/present, attendance rate, attendance group), demographics (gender, ethnicity, English fluency, home language, Special Ed status, SED status), GPA (cumulative and current), address, parent contact info, and suspension data.

---

## What the Notebook Does

### Step-by-Step Pipeline

1. **Load & Inspect** — Read the raw CSV, review shape, column names, data types, and missing values.

2. **Convert AttRate** — Strip the `%` symbol and convert attendance rate from string (e.g., `"82.8%"`) to a float decimal (e.g., `0.828`).

3. **Parse Birthdate** — Convert `Birthdate` from string to `datetime` for age calculations.

4. **Remove PII** — Drop columns with personally identifiable information that aren't needed for analysis: `LastName`, `FirstName`, `ParentName`, `Telephone`, `PG_Email_1`. *(Address fields are retained for potential distance-to-school calculations.)*

5. **Fill Suspension NaNs** — Replace missing values in `NumSusp` and `NumDaysSusp` with `0` (no suspensions recorded = 0).

6. **Standardize Attendance Group labels** — Clean up any inconsistencies in the `AttGrp` categorical field.

7. **Save cleaned wide-format file** — `RAWChronic_Cleaned.csv` (18,638 rows × 23 columns).

8. **Pivot to long format** — Use `pd.melt()` to unpivot numeric metrics (`DaysEnr`, `DaysAbs`, `DaysPresent`, `AttRate`, GPA fields, suspension fields) into a `Metric` / `Value` column pair, keeping demographic and identifier columns as `id_vars`. Produces `RAWChronic_Cleaned_Pivoted.csv` (167,742 rows × 16 columns).

---

## Output Files

| File | Rows | Columns | Description |
|---|---|---|---|
| `RAWChronic_Cleaned.csv` | 18,638 | 23 | One row per student, PII removed, types corrected |
| `RAWChronic_Cleaned_Pivoted.csv` | 167,742 | 16 | Long format — one row per student per metric |

---

## Attendance Groups

Students are classified into the following attendance groups based on their attendance rate:

- **Satisfactory** — ≥ 96%
- **At Risk** — 90–95.9%
- **Chronic Absent** — 80–89.9%
- **Severe Chronic Absent** — < 80%

---

## Related Notebooks

This notebook is part of a broader data cleaning suite:

- `ONGB_Data_Cleaning.ipynb` — Cleans and pivots OUSD program evaluation data (`ONGB_EvalData_Complete_Anonymized.xlsx`)

Both datasets are prepared in long format and intended as inputs for machine learning models or statistical analysis.

---

## Setup & Usage

### Requirements

```
pandas
```

### Running the Notebook

1. Place the raw data file in the project root directory:
   ```
   RAWChronic Absent Students with DOB_2025-05-29.csv
   ```

2. Open and run `pivot_chronic_clean.ipynb` top to bottom.

3. Outputs will be saved to the project root.

---

## Data Privacy

Raw data files are excluded from version control via `.gitignore`. Do **not** commit any `.csv` or `.xlsx` files containing student data to this repository.