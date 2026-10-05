# Monday Audit: Research Question, Study Design, & Data Audit

**Author:** [PagielCecil]\
**Date:** [05-10-2026]\
**Project:** eLife COVID-19 & HIV Cohort Reanalysis

------------------------------------------------------------------------

## 1. Research Question (P-E-C-O-T Framework)

## 2. Study Design Identification & Justification

## 3. First Data Audit & Row Count Reconciliation

## 4. Identified Data Quality Issues

## 5. Unresolved Questions & Methodological Uncertainties

## 1. Research Question (P-E-C-O-T Framework)

> **Primary Question:** Among adult participants presenting with confirmed SARS-CoV-2 infection or as close contacts at hospital facilities in Durban, South Africa during infection waves 1 and 2 (June 2020 – May 2021) (**P**), does living with HIV (**E**) compared to being HIV-negative (**C**) increase the risk of developing severe COVID-19 requiring supplemental oxygen therapy (**O**) over the course of clinical follow-up (**T**)?

- **Outcome Definition & Dataset Variable:**
  - *Definition:* Experiencing severe COVID-19 (defined as requirement for supplemental oxygen or WHO Ordinal Scale ≥ 4).
  - *Variables:* Captured by `Disease_Severity` (`Asymptomatic`, `Mild`, `Moderate`, `Severe`) and supported by `WHO_Ordinal_Scale`.
- **Unit of Analysis & Rationale:**
  - *Unit:* **Person** ($n = 236$).
  - *Rationale:* Disease severity represents a cumulative, individual-level clinical outcome rather than an independent event evaluated separately at each longitudinal visit.

## 2. Study Design Identification & Justification

**Identified Design:** Prospective Observational Cohort Study

**Justification:** 1. **Selection Rule:** Participants were enrolled based on inclusion criteria (SARS-CoV-2 diagnostic swab or contact exposure) and categorized by exposure status (PLWH vs. HIV-negative), rather than sampled based on final outcome status. 2. **Timing:** Exposure status (HIV infection and baseline ART) was determined at enrollment prior to longitudinal observation of COVID-19 progression. 3. **Unit of Analysis:** Participants were followed prospectively across up to 8 longitudinal timepoints to track disease progression. 4. **Author Agreement:** I agree with the paper's classification of a "longitudinal observational cohort study" because the study follows exposed and unexposed cohorts forward through time to measure disease severity.

## 3. First Data Audit & Row Count Reconciliation

### Audit Script (`R/01_import.R`)

``` r

# 0. Install and load required packages
if (!requireNamespace("readxl", quietly = TRUE)) install.packages("readxl")
if (!requireNamespace("dplyr", quietly = TRUE)) install.packages("dplyr")

library(readxl)
library(dplyr)

# Define the file path using forward slashes
data_path <- "C:/Users/user/OneDrive/Documents/EpidemiologistBiostatistician  28-Week Practical Curriculum/epi-data-portfolio/project-02/data/raw/elife-67397-data1-v3.xlsx"

# Read the raw data
raw_data <- read_excel(data_path)

# Verify dimensions and unique subjects
cat("Total Rows:", nrow(raw_data), "\n")
cat("Unique Subjects:", n_distinct(raw_data$Deidentified_PID), "\n")

# Investigate 1,087 vs 986 discrepancy
raw_data %>% 
  count(Participant_Type, Telephonic_Follow_Up)
```
### Explaining the Row Gap (1,087 Rows vs. 986 Visits)

The raw Excel file contains **1,087 rows**, whereas the manuscript reports **986 visits**.

- **In-Person Visits (`Telephonic_Follow_Up == "No"`):** 984 rows
- **Telephonic Follow-ups (`Telephonic_Follow_Up == "Yes"`):** 103 rows
- **Total Dataset Rows:** 984 + 103 = 1,087 rows

**Conclusion:** The paper's figure of 986 visits refers specifically to physical in-person clinical visits (984 rows in the raw data, representing a minor 0.2% variance due to post-publication data cleaning). The raw file includes an additional 103 remote telephonic checks.

### Visit Distribution per Participant (n = 236)
- **Range:** 1 to 8 visits per participant (Mean: 4.61, Median: 5)

| Visits | Participants |
| :--- | :--- |
| 1 | 16 |
| 2 | 28 |
| 3 | 15 |
| 4 | 37 |
| 5 | 66 |
| 6 | 34 |
| 7 | 32 |
| 8 | 8 |


---

### Section 4: Documenting Data Quality Issues
List at least 5 issues using a clear Markdown table or itemized list stating the variable name and why it impacts analysis.


## 4. Identified Data Quality Issues

| # | Variable Name | Data Issue | Impact on Analysis |
|---|---|---|---|
| 1 | `qPCR_RawAverageSARSCt` | Stored as `character` containing `"."` for missing values and string bounds like `">35.77"`. | Direct numeric conversion fails or silently generates NAs, breaking viral load models. |
| 2 | `EFV`, `TFV`, `FTC` | Mixed numeric values with string inequality expressions (e.g., `"<150"`, `">5000"`). | Prevents quantitative ARV concentration summary without boundary processing. |
| 3 | `Date_Diagnostic_Swab`, `Enrolment_Date` | Non-standard text strings (e.g., `"jun2020"`) rather than `Date` objects. | Prevents elapsed time calculations (e.g., days from swab to symptom onset). |
| 4 | `Viral_Load` | High missingness (724 NAs) and explicit detection limits (`40.0`). | Requires lower-limit-of-detection (LLOD) handling; treating NAs as zero biases results. |
| 5 | `Experienced_Cough?` | Text strings (`"Yes"`, `"No"`, `NA`) with special characters (`?`) in column name. | Invalid syntax in R formulas and risks incorrect default reference levels in regressions. |

