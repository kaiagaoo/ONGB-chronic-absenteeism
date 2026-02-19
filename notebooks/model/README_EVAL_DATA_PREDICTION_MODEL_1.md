# ONGB EvalData — Student Attendance Rate Prediction

## Overview

This notebook (`model_1ONGB_EvalData-1.ipynb`) builds machine learning models to predict student attendance rates and identify students at risk of chronic absenteeism. It uses historical attendance and student demographic data from `ONGB_EvalData_CLEANED.csv`.

## Goal

Predict each student's attendance rate for the 2023–24 school year (`AttRate_2324`) and flag those at risk of chronic absenteeism so that support interventions can be targeted early.

---

## Requirements

Install dependencies before running:

```bash
pip install pandas numpy scikit-learn xgboost matplotlib
```

---

## Input Data

| File | Description |
|---|---|
| `ONGB_EvalData_CLEANED.csv` | Cleaned student dataset with attendance rates, demographic fields, and an anonymized student ID (`ANON_ID`) |

**Key columns:**
- `ANON_ID` — anonymized student identifier
- `Birthdate` — dropped before modeling
- `AttRate_2324` — target variable (attendance rate for 2023–24, range 0–1)
- All other columns are used as features

---

## Workflow

### 1. Data Loading & Preprocessing
- Loads the CSV and identifies the target column (`AttRate_2324`)
- Splits students into those **with** and **without** 2023–24 attendance data
- Label-encodes categorical columns, coerces all features to numeric, and fills missing values with column medians

### 2. Model Training (80/20 train-test split)
Two models are trained and compared:

| Model | Notes |
|---|---|
| **Linear Regression** | Baseline; feature importance derived from absolute coefficients |
| **XGBoost Regressor** | 300 estimators, learning rate 0.05, max depth 6, subsample 0.8 |

Both models are evaluated on the test set using **MAE** (Mean Absolute Error) and **R²**.

### 3. Feature Importance
Top 10 most predictive features are printed for both models after training.

### 4. Threshold Optimization for Chronic Absenteeism
- Chronic absence is defined as attendance rate **< 0.90**
- Thresholds between 0.80 and 0.97 are swept to maximize **F1 score**
- A precision/recall/F1 curve is saved as `threshold_analysis.png`

### 5. Severity Band Classification
Students are classified into four severity bands:

| Band | Attendance Rate |
|---|---|
| On Track | ≥ 90% |
| Chronic | 80–89% |
| High Chronic | 70–79% |
| Severe | < 70% |

---

## Outputs

| File | Description |
|---|---|
| `predictions.csv` | Test set predictions from both models alongside actual values |
| `all_predictions.csv` | XGBoost predictions for **all** students (known and unknown 2324 data) |
| `future_predictions.csv` | XGBoost predicted attendance rate for all students (keyed by `ANON_ID`) |
| `threshold_analysis.png` | Precision / Recall / F1 chart across thresholds |

---

## Notes

- Students missing a `AttRate_2324` value are treated as the **prediction set** — they receive a predicted attendance rate but no actual value to compare against.
- The notebook uses `random_state=42` throughout for reproducibility.
- Model performance metrics are printed to the console during execution.