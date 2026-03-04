# Chronic Absenteeism Prediction Model

## Overview

This model predicts which Oakland Unified School District (OUSD) students are at risk of chronic absenteeism in the **next school year**, enabling proactive interventions before attendance problems become severe.

Rather than flagging students after they are already chronically absent, this model uses longitudinal attendance history, socioeconomic context, and school-level data to predict future risk — giving counselors, teachers, and administrators a head start.

---

## Problem Framing

### Target Variable
Students are classified into three tiers based on predicted attendance rate for the upcoming school year:

| Tier | Attendance Rate | Action |
|---|---|---|
| ✅ On Track | ≥ 90% | No intervention needed |
| ⚠️ Chronic | 80–89% | Monitor + light outreach |
| 🚨 Critical | < 80% | Immediate intensive intervention |

> **Note:** The Critical tier combines what were previously "High Chronic" (70–79%) and "Severe" (<70%) categories. Both require immediate action and combining them improved model performance on minority classes.

### Why Predict Next Year?
Chronic absenteeism compounds over time. A student who misses 15% of school in 5th grade is significantly more likely to drop out of high school. Early identification allows schools to intervene during the summer or first weeks of school — when intervention is most effective.

---

## Training Strategy

### Stacked Year Transitions
Rather than only using the most recent year, we stack all year-over-year transitions as independent training examples:

```
2017-18 features → predict 2018-19 tier
2018-19 features → predict 2019-20 tier
2019-20 features → predict 2020-21 tier
2020-21 features → predict 2021-22 tier
2021-22 features → predict 2022-23 tier
2022-23 features → predict 2023-24 tier
```

This gives us **225,376 training examples** from 79,460 students across 7 years, capturing temporal patterns and maximizing training data.

### Train/Test Split
- 80% training (180,300 rows)
- 20% test (45,076 rows)
- Stratified by tier to preserve class balance

---

## Features (35 total)

### Attendance History (strongest predictors)
| Feature | Description |
|---|---|
| `prior_mean_att` | Average attendance rate across all prior years |
| `prior_min_att` | Worst attendance rate across all prior years |
| `curr_att_rate` | Current year attendance rate |
| `curr_days_abs` | Total days absent this year |
| `prior_total_abs` | Cumulative absences across all prior years |
| `curr_days_enr` | Days enrolled this year |
| `prior_chronic_count` | Number of prior years with chronic absence |
| `curr_tier` | Current year attendance tier |
| `prior_years_enrolled` | Number of years enrolled before current year |

### School Context
| Feature | Description |
|---|---|
| `school_chronic_rate` | Chronic absence rate at student's school |
| `school_mean_att` | Mean attendance rate at student's school |
| `curr_grade` | Current grade level |
| `curr_gpa` | Current weighted GPA (null for K-5) |
| `curr_susp` | Number of suspensions this year |

### Distance Features
| Feature | Description |
|---|---|
| `curr_dist_miles` | Miles from home to school |
| `curr_dist_bucket` | Distance category (Walking/Near/Far/Very Far) |
| `curr_transport_burden` | Flag: student lives >3 miles from school |
| `total_significant_moves` | Number of times student moved >2 miles between years |
| `most_recent_dist_bucket` | Most recent distance category |
| `avg_dist_change` | Average year-over-year distance change |
| `max_dist_change` | Largest single-year distance change |

### Socioeconomic (neighborhood-level, by zip code)
| Feature | Description |
|---|---|
| `curr_poverty_rate` | % below poverty line in student's zip |
| `curr_median_income` | Median household income in student's zip |
| `curr_unemployment` | Unemployment rate in student's zip |
| `curr_rent` | Median gross rent in student's zip |
| `curr_home_value` | Median home value in student's zip |
| `curr_uninsured` | % uninsured in student's zip |
| `curr_hs_rate` | % adults with HS diploma in student's zip |
| `curr_college_rate` | % adults with college degree in student's zip |

### Demographics
| Feature | Description |
|---|---|
| `age` | Student age |
| `gen` | Gender |
| `eth` | Ethnicity |
| `fluency` | English language fluency status |
| `curr_sed_binary` | Socioeconomically disadvantaged (1/0) |
| `school_level` | Elementary / Middle / High / PreK |

---

## Model Selection

We trained and compared 6 models:

| Model | Accuracy | Critical Recall | Chronic Recall | Macro F1 |
|---|---|---|---|---|
| Decision Tree (shallow) | 61.1% | 62.6% | 46.7% | 0.538 |
| Decision Tree (deep) | 60.9% | 64.1% | 53.5% | 0.548 |
| Random Forest (baseline) | 63.9% | 66.1% | 51.0% | 0.548 |
| Random Forest (tuned) | 65.0% | 66.8% | 52.8% | 0.565 |
| RF + Distance Features | 65.0% | 66.9% | 52.5% | 0.580 |
| **XGBoost (final)** | **66.2%** | **65.9%** | **56.2%** | **0.595** |

### Why XGBoost?
- Best Macro F1 (0.595) — most balanced performance across all three tiers
- Best Chronic recall (56.2%) — catches the most middle-tier students
- Strong Critical recall (65.9%) — identifies 2 in 3 at-risk students
- Sample weighting used to counteract class imbalance

### Final Model Parameters
```python
XGBClassifier(
    n_estimators=400,
    max_depth=10,
    learning_rate=0.03,
    subsample=0.7,
    colsample_bytree=0.7,
    min_child_weight=3,
    gamma=0.5,
    random_state=42,
    n_jobs=-1
)
```

---

## Results

### Test Set Performance
```
              precision    recall  f1-score   support

     Chronic      0.390     0.562     0.457      9240
    Critical      0.439     0.659     0.531      6259
    On Track      0.899     0.680     0.774     29577

    accuracy                          0.662     45076
   macro avg      0.576     0.634     0.587     45076
weighted avg      0.731     0.662     0.678     45076
```

### 2024-25 Predictions (36,695 students)

| Tier | Students | Rate |
|---|---|---|
| 🚨 Critical — Immediate Intervention | 5,667 | 15.4% |
| ⚠️ Chronic — Monitor & Outreach | 7,909 | 21.6% |
| ✅ On Track | 23,119 | 63.0% |

### Critical Rate by School Level
| School Level | Predicted Critical Rate |
|---|---|
| High School | 31.5% |
| Middle School | 14.3% |
| PreK | 10.3% |
| Elementary | 6.6% |

---

## Top Feature Importances

1. `prior_mean_att` — 12.3%
2. `prior_min_att` — 11.9%
3. `curr_att_rate` — 9.4%
4. `curr_days_abs` — 6.9%
5. `prior_total_abs` — 4.7%
6. `curr_days_enr` — 4.6%
7. `age` — 4.2%
8. `curr_gpa` — 4.0%
9. `prior_chronic_count` — 3.8%
10. `school_chronic_rate` — 3.4%

**Key insight:** A student's attendance history completely dominates predictions. The single strongest signal is their worst ever attendance year (`prior_min_att`), followed by their historical average. Socioeconomic and distance features contribute but are secondary.

---

## Output File

**`FINAL_student_predictions_2425.csv`**

| Column | Description |
|---|---|
| `ANON_ID` | Anonymous student identifier |
| `predicted_tier` | On Track / Chronic / Critical |
| `risk_score` | Weighted risk score (0–1), higher = more at risk |
| `prob_on_track` | Model probability of being On Track |
| `prob_chronic` | Model probability of being Chronic |
| `prob_critical` | Model probability of being Critical |
| `immediate_action` | 1 if Critical, 0 otherwise |
| `monitor` | 1 if Chronic, 0 otherwise |
| `action_label` | Human-readable action recommendation |

---

## Limitations

- **~54% of students missing distance data** — students without geocodable addresses have null distance features
- **Census data limited to Oakland zip codes** — students living outside Oakland have null socioeconomic features
- **GPA only available for grades 6-12** — elementary students have null GPA, handled via school level flag
- **COVID years (2020-21, 2021-22)** may introduce noise — attendance patterns were abnormal during remote learning
- **Model predicts tiers, not exact rates** — use risk scores for finer-grained prioritization within tiers

---

## How to Run

```python
# Dependencies
pip install pandas numpy scikit-learn xgboost matplotlib

# Run the full modeling pipeline
# Input:  ONGB_model_ready.csv
# Output: FINAL_student_predictions_2425.csv
```

See `02_model_training.ipynb` for the full annotated pipeline.

---

## Requirements

```
pandas
numpy
scikit-learn
xgboost
matplotlib
```
