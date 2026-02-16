# Data Cleaning Pipelines

## Notebooks
- `ONGB_Data_Cleaning.ipynb` - OUSD evaluation data pipeline
- `Chronic_Data_Cleaning.ipynb` - Chronic absenteeism cleaning and pivoting

## Data Files
Data files are NOT included in this repository (see `.gitignore`).

To run these notebooks, you need:
1. `RAWChronic Absent Students with DOB_2025-05-29.csv`
2. `ONGB_EvalData_Complete_Anonymized.xlsx`

Place these files in the project root directory before running notebooks.

## What Each Notebook Does

### ONGB_Data_Cleaning.ipynb
- Pivots wide format → long format
- Removes duplicate demographic columns
- Converts numeric variables to proper data types
- Output: `ONGB_EvalData_Cleaned_Pivoted.csv`

### Chronic_Data_Cleaning.ipynb
- Converts AttRate from % to decimal
- Converts Birthdate to datetime
- Removes PII (keeps address for distance calculations)
- Fills suspension NaN with 0
- Pivots to long format
- Outputs: `RAWChronic_Cleaned.csv` and `RAWChronic_Cleaned_Pivoted.csv`

## Ready for Modeling
Both datasets are pivoted and cleaned, ready to feed into ML models to test which format works best.
