 ONGB Evaul Data Cleaning

This repository contains the data cleaning pipeline for the ONGB (anonymized) student evaluation dataset, spanning school years 2017–18 through 2023–24. The goal is to prepare a clean, analysis-ready dataset for exploratory data analysis (EDA) and predictive modeling of chronic absenteeism.

## Source Data

**Input:** `ONGB_EvalData_Complete_Anonymized.xlsx`  
**Output:** `ONGB_EvalData_CLEANED.csv`

The dataset contains longitudinal student-level records with yearly variables covering attendance, grades, ethnicity, special education status, English fluency, socioeconomic disadvantage (SED), suspensions, and school placement. Students are identified by an anonymized ID (`ANON_ID`).

## Notebook: `anonymousonbgclean-1.ipynb`

### Workflow

**1. Initial Exploration**
- Load and inspect the dataset shape, column names, and data types
- Review grade distributions and attendance rate summaries across all years

**2. Data Quality Checks**
- Confirm no duplicate students
- Validate birthdates (range and nulls)
- Verify attendance rates fall within the valid 0–1 range
- Confirm no cases where days absent exceed days enrolled
- Check consistency of categorical variables: grades, ethnicity, gender, SpEd, Fluency, SED
- Confirm missing data is enrollment-based (NaN = not enrolled that year), not data loss

**3. Cleaning Steps**

| Step | Description |
|------|-------------|
| Gender standardization | Normalized lowercase `'m'` to `'M'` |
| Chronic absenteeism flags | Created `ChronicAbs_[year]` = `1` if attendance rate < 90%, `0` otherwise, `NaN` if not enrolled |
| Age calculation | Created `Age_[year]` = age (in years, 1 decimal) as of September 1 of each school year |
| Suspension imputation | Filled `NaN` with `0` for enrolled students; retained `NaN` for non-enrolled students |
| SED binary flags | Created `SED_Binary_[year]` = `1` (SED), `0` (Not SED), `NaN` (Unknown or not enrolled) |

**New columns added:** 28 total (7 per new variable × 4 variables: ChronicAbs, Age, SED_Binary, plus Susp imputation)

## Key Decisions

- **NaN = Not Enrolled.** Missing values across yearly variables are preserved as `NaN` to indicate a student was not enrolled that year, not missing data. Enrolled students with complete records had no unexpected missingness.
- **Suspension NaN → 0.** For enrolled students, missing suspension records are treated as no suspension (0 days). NaN is retained for non-enrolled students.
- **SED binary encoding.** The original categorical SED variable (`SED` / `Not SED` / `Unknown`) is converted to a binary flag for use in machine learning models.

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

## Usage

1. Place `ONGB_EvalData_Complete_Anonymized.xlsx` in the project directory
2. Run `anonymousonbgclean-1.ipynb` top to bottom
3. The cleaned file `ONGB_EvalData_CLEANED.csv` will be saved in the working directory

> **Note:** Data files (`.csv`, `.xlsx`) are excluded from version control via `.gitignore`.

## Data Quality Confirmed

- No duplicate students
- All birthdates valid
- All attendance rates within 0–1 range
- No cases where days absent exceed days enrolled
- Missing data reflects non-enrollment, not data errors

## Next Steps

- Exploratory Data Analysis (EDA)
- Feature engineering for chronic absenteeism prediction
- Model training and evaluation