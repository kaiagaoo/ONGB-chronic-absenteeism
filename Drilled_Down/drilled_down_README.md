Drilled Down EDA

A multi-step analytical pipeline examining chronic absenteeism patterns in Oakland USD student data, developed for thesis research. The notebook (Drilleddown.ipynb) walks through data loading, school-level aggregation, and logistic regression modeling to identify predictors of chronic absenteeism.

Overview
This analysis investigates what factors predict chronic absenteeism (attendance rate below 90%) among Oakland USD students in the 2023–24 school year, with a particular focus on gender, socioeconomic status, special education status, and demographics.

Data
Input file: ONGB_EvalData_CLEANED.csv
Key variables include:
VariableDescriptionAttRate_XXXXAttendance rate by school yearChronicAbs_XXXXChronic absenteeism flagDaysAbs_XXXXDays absentSusp_XXXXSuspension countSED_Binary_XXXXSocioeconomic disadvantage indicatorSpEd_XXXXSpecial education statusFluency_XXXXEnglish learner statusEth_XXXXEthnicity/raceSiteName_XXXXSchool name

Column suffixes correspond to school year (e.g., 2324 = 2023–24, 2021 = 2020–21).


Notebook Structure
Step 1 — Load and Explore Data
Basic data loading, column inspection, and summary statistics.
Step 2 — School-Level Aggregations
Aggregates student-level records to the school level, computing:

Average attendance rate and chronic absenteeism rate
Suspension and absence averages
Demographic compositions (ethnicity, SED, SpEd, English Learner percentages)
Grade level distributions and school type classification

Uses Berkeley branding (Berkeley Blue #003262, Cal Gold #FDB515) and Times New Roman formatting for all visualizations.
Step 3 — Logistic Regression Modeling
Runs a series of nested logistic regression models predicting chronic absenteeism:

Model 1: Gender only
Model 2: Gender + demographic controls (age, ethnicity, grade level, EL status, SpEd)
Model 3: Full model (+ socioeconomic disadvantage)

Outputs odds ratios with 95% confidence intervals and significance flags.
Step 4 — Visualization & Publication Tables
Generates a forest plot of odds ratios for key predictors and exports publication-ready CSV tables.
Step 5 — Interpretation & Policy Implications
Summarizes key findings and their implications for thesis write-up and district policy.

Key Findings
PredictorOdds RatioInterpretationSocioeconomic Disadvantage3.99Strongest predictor — nearly 4× higher oddsSpecial Education Status1.3939% higher oddsStudent Age1.098.7% higher odds per year of ageGender (Male vs. Female)0.93Males have 7.4% lower odds
Socioeconomic disadvantage is the dominant driver of chronic absenteeism; gender remains a statistically significant but smaller factor even after full controls.

Output Files
FileDescriptionodds_ratios_chronic_absenteeism.csvFull logistic regression resultsregression_comparison_table.txtModel fit comparison across nested modelsforest_plot_chronic_absenteeism_predictors.pngForest plot visualization (300 dpi)regression_summary_for_publication.csvClean, publication-ready summary table

Requirements
pandas
numpy
matplotlib
seaborn
scipy
statsmodels

Usage

Place ONGB_EvalData_CLEANED.csv in the same directory as the notebook.
Run all cells in order (Steps 1–5).
Output files will be saved to the working directory.


Context
This analysis was developed as part of a graduate thesis examining attendance equity in Oakland USD. The focus year is 2023–24 (2324 suffix in column names). Historical comparisons use earlier year suffixes (e.g., 1920, 2021).