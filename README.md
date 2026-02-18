# ONGB-chronic-absenteeism

A dual-scale data analysis project integrating national education datasets and Oakland student-level data to identify early signals of chronic absenteeism and inform data-driven intervention strategies.

## Group Member

Quiana Anderson, Kaia Gao, Yucheng Lu

## 1. Introduction

### 1.1 Project Support

This project is supported by Oakland Natives Give Back (ONGB), a community-based organization founded in 2008 by Oakland-native Black women to empower local youth through innovative, community-first programs, and Wizearly, which provides an AI-driven early warning solution for chronic absenteeism.

### 1.2 Project Overview

 Conduct a comprehensive landscape analysis of chronic absenteeism trends at both national and local scales, specifically in Oakland Unified School District (OUSD). This dual-scale approach will establish both national evidence and local validation for understanding systemic absenteeism challenges, positioning ONGB's programs and Wizearly's early-warning platform for local impact and national scalability.

### 1.3 Data Sources

1. ONGB will provide a dataset containing 7 years of Oakland Unified School District (OUSD) attendance records with 80,000 records (NDA required). 

2. Clinical records (Care Notes) from Wizearly platform containing student mental health and social indicators.

3. Expecting to incorporate additional factors such as poverty levels, weather, and food accessibility data. 

### 1.4 Tools

1. Python or R programming

2. Data cleaning and manipulation (Pandas, NumPy, dplyr)

3. Statistical modeling and machine learning

4. Data visualization (Matplotlib, Seaborn, Plotly, Tableau, PowerBI)

5. SQL or database management

Communication through Slack and email.

### 1.5 Deliverables  

Part 1. National & Local Early-Signal Landscape Report 

Clean the data and decide how to handle the many missing components.

A comprehensive analysis identifying early indicators that consistently precede chronic absenteeism at both national and Oakland levels finalizing with a detailed report.

Part 2. Predictive Feature Library for Wizearly

A structured catalog of nationally relevant features for risk modeling, validated with OUSD data, including feature definitions, expected correlations, preliminary importance rankings, and performance metrics from Oakland student-level predictions

### 1.6 Project Timeline

Phase 1 (Weeks 1-3): Data Acquisition & Exploration

Access and download national datasets (EdFacts/NCES, CRDC, Census ACS, state dashboards)
Receive and explore OUSD student-level dataset (thousands of records)
Establish representative district comparison set (100-300 districts across 10 states)
Initial exploratory data analysis of national trends (2018-2024) and Oakland-specific patterns

Phase 2 (Weeks 4-6): Data Integration & Feature Engineering

Clean and harmonize multi-source national datasets
Process and integrate OUSD granular student data
Develop early-signal taxonomy using Wizearly's IPIR framework (attendance patterns, academic dips, behavioral incidents, family engagement, basic needs, community factors)
Create intervention typology from documented district plans
Compare Oakland features with national data patterns

Phase 3 (Weeks 7-9): Statistical Modeling & Pattern Analysis

Conduct national-level regression and clustering analyses
Build predictive models using OUSD student-level data
Validate national patterns against Oakland findings
Identify geographic, demographic, and socioeconomic variation patterns
Perform geospatial analysis of district characteristics
Preliminary causal inference on intervention effectiveness

Phase 4 (Weeks 10-12): Synthesis & Deliverable Production

Build interactive dashboard prototype (national + Oakland drill-down)
Compile predictive feature library validated with local data
Write comprehensive landscape report synthesizing national and local findings
Develop strategic recommendations for ONGB/Wizearly
Document Oakland-specific insights for OUSD partnership

Phase 5 (Weeks 13-15): Refinement & Final Presentation

Incorporate feedback from ONGB, Wizearly, and OUSD stakeholders
Finalize all deliverables
Present findings to ONGB/Wizearly leadership and OUSD partners

## 2. Data Cleaning

Two separate datasets were cleaned, each targeting a different analytical lens: a 7-year longitudinal panel for trend analysis, and a single-year cross-sectional snapshot for intervention targeting.

---

### 2.1 Longitudinal OUSD Dataset (`anonymousonbgclean-1.ipynb`)

**Source:** `ONGB_EvalData_Complete_Anonymized.xlsx` — 79,460 students × 122 columns in wide format, covering school years 2017–18 through 2023–24. Each year contributes its own block of columns (e.g., `AttRate_1718`, `Grade_2324`). NaN in any year's columns means the student was not enrolled that year — this is structural, not missing data.

**Steps:**

1. **Gender standardization** — Normalized two records where gender was coded as lowercase `'m'` to `'M'`.

2. **Chronic absenteeism flags** — Created `ChronicAbs_YYYY` for each of the 7 school years: `1` if `AttRate < 0.90` (federal threshold of missing 10%+ of enrolled days), `0` otherwise, `NaN` if not enrolled.

3. **Age calculation** — Created `Age_YYYY` as the student's age on September 1 of each school year (1 decimal place), derived from `Birthdate`.

4. **Suspension imputation** — Suspension records are only logged when an incident occurs, so ~96–97% of enrolled students had `NaN`. For enrolled students only, `NaN` was filled with `0` (Missing Not at Random). Non-enrolled students retain `NaN`.

5. **SED binary flags** — Created `SED_Binary_YYYY`: `1` (Socioeconomically Disadvantaged), `0` (Not SED), `NaN` (Unknown or not enrolled). Note: SED data is only reliable from 2019–20 onward; all 2017–18 and 2018–19 records show `'Unknown'`.

**Data quality confirmed:** No duplicate students, all birthdates valid, all attendance rates within [0, 1], no cases where `DaysAbs > DaysEnr`.

**Output:** `ONGB_EvalData_CLEANED.csv` — 79,460 rows × 143 columns (+21 derived columns).

---

### 2.2 Cross-Sectional Chronic Absence Dataset (`Clean_Chronic_Absent_ONGB.ipynb`)

**Source:** `RAWChronic Absent Students with DOB_2025-05-29.xlsx` — 18,638 rows × 28 columns. A single-year snapshot of students flagged as at-risk or chronically absent, including contact information, GPA, suspension records, and residential address.

**Steps:**

1. **Duplicate removal** — 982 duplicate student IDs arose from the data being pulled on two consecutive days (May 28 and May 29, 2025). Duplicates were confirmed to be the same student at the same school and grade with negligible metric differences. The most recent record (May 29) was kept for each student, yielding **17,656 unique students**.

2. **Missing value handling:**
   - *Suspensions:* `NumSusp` and `NumDaysSusp` NaN → `0` (MNAR — blank means no suspension recorded).
   - *GPA:* Retained as `NaN` for elementary students, who do not receive GPA. Any GPA analysis should filter to secondary grades only.
   - *Parent contact:* Rather than dropping records, boolean flags `Missing_Email` and `Missing_Phone` were added. This preserves all student records while creating a to-do list for outreach staff.

3. **Grade label standardization** — Created human-readable `Grade_Label` column: `'Pre-K'` (grade −2), `'TK'` (grade −1), `'Grade 0'`–`'Grade 12'`, and `'Post-Secondary'` (grade 15+).

4. **Derived columns:**
   - `Is_Chronic` — boolean flag for `AttRate < 0.90`.
   - `Absence_Severity` — 4-tier severity scale for intervention targeting:
     - **Severe** (< 70%): Crisis intervention needed
     - **High Chronic** (70–79%): Intensive support needed
     - **Moderate Chronic** (80–89%): Active monitoring needed
     - **Low Risk** (90–100%): Minimal intervention needed
   - `School_Level` — grouped grade ranges: Early Childhood (pre-K/TK), Elementary (K–5), Middle (6–8), High School (9–12), Post-Secondary.

5. **Validation** — Verified that `DaysPresent / DaysEnr ≈ AttRate` and `DaysPresent + DaysAbs = DaysEnr` for all 17,656 records (100% match).

**Output:** `cleaned_chronic_absence_data.csv` / `.xlsx` — 17,656 rows × 32 columns.

---

### 2.3 Feature Engineering: Contextual Enrichment (`add_feature.ipynb`)

**Source data appended:**
- `oakland_crime_by_zip.csv` — crime counts by Oakland zip code and year (2017–2024)
- `oakland_socioeconomic_by_zip.csv` — socioeconomic indicators by zip code and year (2017–2024), sourced from ACS

**Steps:**

1. **Year filtering** — Both datasets span 2017–2024. The 2024 records were extracted as the most recent year available, matching the 2024–25 academic year of the absence snapshot.

2. **Residence-based merge** — Crime and socioeconomic features were joined to each student record on `ZipResidence` (student's home zip code). All appended columns carry a `_res` suffix to distinguish them from future school-based features.

**New columns added (20 total, all `_res` suffix):**

*Crime (5):* `total_crimes_res`, `violent_crimes_res`, `property_crimes_res`, `drug_crimes_res`, `other_crimes_res`

*Socioeconomic (15):* `total_population_res`, `poverty_universe_res`, `below_poverty_res`, `poverty_rate_pct_res`, `median_household_income_res`, `labor_force_res`, `unemployed_res`, `unemployment_rate_pct_res`, `education_universe_25plus_res`, `high_school_plus_rate_pct_res`, `college_degree_rate_pct_res`, `median_gross_rent_res`, `median_home_value_res`, `health_insurance_universe_res`, `uninsured_rate_pct_res`

> **Planned:** School-based contextual features (joined on school zip code with `_sch` suffix) are not yet implemented. The mapping from `SiteName` to `ZipSchool` requires deriving school zip codes from the eval dataset (`ONGB_EvalData_CLEANED.csv`, `SiteName_2324 → Zip_2324`), with manual corrections for three schools not present in that lookup.

**Output:** `abs_with_new_features.csv` — 17,656 rows × 52 columns.

---

### Cleaned Dataset Snapshot (2024–25)

| Metric | Value |
|---|---|
| Total students (chronic absence dataset) | 17,656 |
| Chronic absenteeism rate (< 90%) | 51.5% |
| Severe (< 70%) | 10.0% (1,766 students) |
| High Chronic (70–79%) | 10.5% (1,851 students) |
| Moderate Chronic (80–89%) | 34.2% (6,043 students) |
| Socioeconomically Disadvantaged | 89.6% |
| Special Education | 21.7% |
| Latino / African American combined | 76.6% |

## 3. Exploratory Data Analysis

