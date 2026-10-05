# Monday Audit (v2): Research Question, Study Design, & Data Audit

**Author:** Pagiel Cecil\
**Project:** eLife COVID-19 & HIV Cohort Analysis

------------------------------------------------------------------------

## 1. Research Question (P-E-C-O-T Framework) & Population Decisions

### P-E-C-O-T Specification

- **Population (P):** Adult participants ($\ge 18$ years) presenting with confirmed SARS-CoV-2 infection at hospital facilities in Durban, South Africa during infection Waves 1 and 2 (June 2020 – May 2021).
  - **PUI Eligibility Decision:** Persons Under Investigation (`Participant_Type == "PUI"`; $n = 10$ participants, 34 total rows) are **excluded** from the primary analytical cohort. Empirical audit shows PUIs have unconfirmed SARS-CoV-2 status or negative RT-PCR tests (`qPCR_Result_` recorded as "Not Detected" or "Inconclusive"). Restricting the cohort to confirmed cases (`Participant_Type == "Case"`) yields $N = 226$ participants (1,053 rows).
- **Exposure (E):** Living with HIV (`HIV_Status == "POSITIVE"`).
- **Comparator (C):** Being HIV-negative (`HIV_Status == "NEGATIVE"`).
- **Outcome (O):** Developing severe COVID-19 requiring supplemental oxygen therapy or resulting in death ($\text{WHO\_Ordinal\_Scale} \ge 4$, corresponding to `Disease_Severity == "Supplemental Oxygen or Death"`).
- **Timeframe (T):** From hospital presentation ($T_0$) through longitudinal clinical follow-up (up to 8 visits).

### Rewritten Research Question (Associational Wording)

> Among hospitalized and ambulatory adults with confirmed SARS-CoV-2 infection, is living with HIV associated with an increased risk of severe COVID-19 outcome (WHO Ordinal Scale $\ge 4$ or death) during clinical follow-up compared with being HIV-negative?

------------------------------------------------------------------------

## 2. Corrected Outcome Definition, Death Decision, & Collapsing Rule

### Empirical Dataset Categories (`Disease_Severity` vs. `WHO_Ordinal_Scale`)

Cross-tabulation of `Disease_Severity` and `WHO_Ordinal_Scale` in the raw dataset reveals the exact empirical mapping:

| `Disease_Severity` Level | `WHO_Ordinal_Scale` | Row Count ($n$) | Clinical Description |
|:-----------------|:-----------------:|:-----------------:|:-----------------|
| **Asymptomatic** | 1 | 475 | Asymptomatic / Uninfected state |
| **Mild** | 2 | 330 | Symptomatic independent, no oxygen |
| **Mild** | 3 | 164 | Symptomatic assistance needed, no oxygen |
| **Supplemental Oxygen or Death** | 4 | 102 | Hospitalized, non-ICU (Supplemental oxygen / low-flow) |
| **Supplemental Oxygen or Death** | 5 | 3 | Hospitalized, severe (High-flow nasal cannula / NIV) |
| **Supplemental Oxygen or Death** | 8 | 13 | Death |

*Note:* Levels WHO 6 and WHO 7 (invasive mechanical ventilation / ECMO) do not appear in this dataset.

### Answers to Methodological Outcome Questions

1.  **Top Severity Category:** The highest severity score in the dataset is WHO Code 8 (Death; $n = 13$ rows across 13 unique participants).
2.  **Inclusion of Death in "WHO** $\ge 4$ / Supplemental Oxygen": Yes. In this dataset, WHO Code 8 is explicitly grouped under `Disease_Severity == "Supplemental Oxygen or Death"`. Because $8 \ge 4$, any threshold definition of $\text{WHO} \ge 4$ includes fatal outcomes.
3.  **Target Estimand & Death Separation:** Including WHO 8 in the primary binary outcome defines a **composite endpoint of severe disease progression OR all-cause mortality**. To align with Table 1 of the published paper (which presents oxygen requirement and death in separate rows), we establish three primary outcome definitions:
    - **Primary Outcome (Composite):** Severe COVID-19 or Death ($\text{WHO} \ge 4$).
    - **Secondary Endpoint 1 (Supplemental Oxygen in Survivors):** $\text{WHO} \in \{4, 5\}$.
    - **Secondary Endpoint 2 (In-Hospital Mortality):** $\text{WHO} = 8$.

### Explicit Person-Level Collapsing Rule

Because participants completed between 1 and 8 visits, participants with longer follow-up have greater opportunity to be observed as severe. To collapse repeated visit rows to one record per participant ($N = 226$ confirmed cases):

- **"Ever / Max Severity" Rule:** A participant is classified as experiencing severe outcome if their maximum WHO score across all completed visits satisfies: $$\text{Severe\_Outcome}_i = \mathbb{I}\left(\max_{j \in V_i} (\text{WHO\_Ordinal\_Scale}_{i,j}) \ge 4\right)$$
- **Follow-up Opportunity Bias Handling:** Because 68 of 75 severe participants (90.7%) reached WHO $\ge 4$ at Visit 1 ($T_0$), differential visit counts contribute minimally to outcome misclassification. However, sensitivity models will evaluate outcomes restricted strictly to baseline ($T_0$) and within a fixed 14-day post-enrolment window.

------------------------------------------------------------------------

## 3. Redesigned Study Design Justification: Time Zero & Selection

### Time Zero ($T_0$), Prevalent vs. Incident Outcomes

- **Time Zero Definition:** $T_0$ is established at baseline hospital presentation/enrolment (`Timepoint == 1`), which occurred at a median of 11 days post-symptom onset (`DaysSymptomToVisit`).
- **Prevalent vs. Incident Severity:**
  - **Prevalent Severity at Entry:** 68 participants presented with $\text{WHO} \ge 4$ at Visit 1 ($T_0$). This represents **prevalent severe disease at presentation**.
  - **Incident Severity Progression:** Only 7 participants progressed to $\text{WHO} \ge 4$ at subsequent visits ($V_2$–$V_7$) after presenting as non-severe at $T_0$.
- **Analytical Consequence:** The primary estimand evaluates the association between HIV status and **overall odds of severe disease course** (prevalent + incident). Secondary incidence models will isolate progression specifically among participants non-severe at entry ($\text{WHO}_{T_0} < 4$).

### Selection & Generalizability

- **Survivorship / Left-Truncation Selection:** Enrolling participants at hospital presentation (\~11 days post-symptom onset) excludes individuals who died rapidly in the community before reaching clinical care, as well as infected individuals who never sought hospital evaluation.
- **Target Target Population:** Findings generalize specifically to **adults presenting to hospital facilities for clinical management of COVID-19**, rather than general community infections.

### Exposure Refinement

- **Primary Exposure:** HIV status (`HIV_Status`, binary: POSITIVE vs. NEGATIVE), determined at baseline.
- **Role of ART:** Antiretroviral therapy (regimen, duration, suppression) is an internal exposure modifier applicable strictly within the HIV-positive subset, not part of the primary binary cohort comparator.

------------------------------------------------------------------------

## 4. Row Reconciliation & Empirical Telephonic Audit

### Testing the Telephonic Follow-up Hypothesis

To evaluate whether telephonic rows (`Telephonic_Follow_Up == "Yes"`) represent non-clinical contact logs, we measured laboratory missingness across interaction modes:

| Variable Name | In-Person Visits (`Telephonic == "No"`) % Missing | Telephonic Visits (`Telephonic == "Yes"`) % Missing |
|:--------------------|:------------------------:|:------------------------:|
| `CD4_Abs` | 3.6% | **100.0%** |
| `Neutrophils_Abs` | 2.0% | **100.0%** |
| `Viral_Load` | 63.1% | **100.0%** |
| `qPCR_RawAverageSARSCt` | 83.0% | **97.1%** |

**Findings:** Laboratory biomarkers are **100% missing** in telephonic rows, confirming that telephonic entries represent remote symptom check-ins rather than physical clinical visits.

### Row Gap Breakdown & Unexplained Remainder

- **Total Raw Dataset Rows:** 1,087 rows
- **In-Person Visits (`Telephonic_Follow_Up == "No"`):** 984 rows (958 Case, 26 PUI)
- **Telephonic Follow-ups (`Telephonic_Follow_Up == "Yes"`):** 103 rows (95 Case, 8 PUI)
- **Reported Paper Count:** 986 visits

**Unexplained Remainder:** An unexplained discrepancy of **2 rows** remains between in-person dataset rows (984) and published visit counts (986).

### Competing Hypotheses for the 2-Row Discrepancy

1.  **Post-Publication Cleaning:** Minor reconciliation or removal of 2 duplicate screening logs prior to manuscript release.
2.  **De-duplication in Analysis Code:** The authors' original script may have filtered 2 edge-case records (e.g. repeated same-day swab entries).
3.  **Database Export Mismatch:** Discrepancy between central laboratory logs and hospital clinical report forms (CRFs).

------------------------------------------------------------------------

## 5. Prioritized Data Quality Issues Table

Issues are ordered strictly by **analytic impact** on inference, parameter estimation, and bias:

| Rank | Variable Name | Data Quality Issue | Specific Analytic Impact | Corrective Rule / Handling Strategy |
|:-------------:|:--------------|:--------------|:--------------|:--------------|
| **1** | `Viral_Load` | **Structural Missingness & Left-Censoring:** 665 of 724 NAs belong to HIV-negative participants (`HIV_Status == "NEGATIVE"`). 259 HIV+ rows equal 40. | Global listwise deletion or standard imputation will corrupt models; HIV- NAs represent non-applicable metrics, while 40 represents lower limit of detection (LOD). | Code HIV- negative rows as `"Not Applicable"`. Model values of 40 as left-censored ($<40$ copies/mL) or binary suppressed/unsuppressed. |
| **2** | `qPCR_RawAverageSARSCt` | **Inverse Metric Scale & High Missingness:** Higher Ct indicates *lower* viral RNA load. Missing in 83.0% of in-person visits. | Direct linear entry flips sign of viral load effect; extreme missingness limits multivariable adjustment. | Model Ct inversely ($-\text{Ct}$) or recode to binary detected/undetected. Avoid continuous inclusion in primary outcome models. |
| **3** | `Date_` columns, `DaysSwabToVisit` | **Coarse Date Granularity & Negative Counters:** Dates are YYYY-MM strings (`"jun2020"`). `DaysSwabToVisit` contains 4 negative values; `DaysBetweenSwabAndSympt` contains 14 negative values. | Month-only dates prevent calendar survival modeling; negative day counts indicate diagnostic swab dates recorded after visit dates. | Rely on relative day variables rather than string dates; flag and set negative relative day counters to `NA` or audit source logs. |
| **4** | `ARV_` columns / Concentration | **Left- & Right-Censored Strings:** Concentrations and durations contain inequality symbols (`"<150"`, `">5000"`, `"<1 year"`). | Prevents direct quantitative continuous modeling in HIV-positive subgroup models. | Convert to ordinal factors (e.g. $<150$, $150-5000$, $>5000$) with explicit missingness indicator levels. |
| **5** | `Pregnancy_` | **Structural Missingness by Sex:** 386 NA values occur exclusively in male participants (`Sex_ == "M"`). | Complete-case analysis without structural awareness will drop all male participants from the dataset. | Recode male NAs to `"Not Applicable (Male)"` to preserve full cohort sample size ($N = 226$). |
| **6** | `Age_Bin` vs. `Age` | **Binned Categorization:** Binned into 10-year intervals (`"18-29"`, `"30-39"`), masking continuous variation. | Inhibits flexible non-linear adjustment (e.g. continuous age splines) and prevents exact replication of continuous Table 1 metrics. | Use raw continuous `Age` for multivariable adjustment; reserve `Age_Bin` solely for table formatting. |
| **7** | `qPCR_Result_` | **Category Inconsistency & Missingness:** Contains 118 NAs, plus inconsistent categories (`"Detected"`, `"2Targets"`, `"Inconclusive"`, `"Not Detected"`). | Split string categories misclassify infection status and reduce sample size if left uncleaned. | Standardize to factor levels: `"Detected"` (combining 2Targets), `"Inconclusive"`, `"Not Detected"`, and `"Missing"`. |
| **8** | Character Columns (`Site`, `Ethnicity`) | **Unstandardized Text & Mixed Missingness:** Mixed empty strings `""`, `"Unknown"`, whitespace padding, and trailing special characters. | Generates spurious duplicate factor levels in regression output and distorts summary table denominators. | Trim whitespace and harmonize all blank/unknown entries to standard `NA` or explicit `"Unknown"` levels. |

------------------------------------------------------------------------

## 6. Updated R Import & Audit Script (`R/01_import.R`)

``` r
# ==============================================================================
# Script: R/01_import.R
# Purpose: Reproducible import, audit, and structural filtering for eLife COVID-19/HIV data
# Note: Uses relative paths via here::here(). Package management handled via renv.
# ==============================================================================

install.packages(c("janitor", "here"))
library(here)
library(readxl)
library(dplyr)
library(janitor)

# --- 1. Load Raw Dataset via Relative Path ---
data_path <- here::here("data", "raw", "elife-67397-data1-v3.xlsx")

if (!file.exists(data_path)) {
  stop("Dataset missing at: ", data_path, ". Ensure file is placed in data/raw/")
}

raw_data <- read_excel(data_path) %>%
  clean_names()

# --- 2. Audit Disease Severity and WHO Scale Mapping ---
cat("=== Disease Severity x WHO Ordinal Scale ===\n")
raw_data %>%
  count(disease_severity, who_ordinal_scale) %>%
  print()

# --- 3. Telephonic Visit & Laboratory Missingness Audit ---
cat("\n=== Laboratory Missingness by Interaction Mode ===\n")
raw_data %>%
  group_by(telephonic_follow_up) %>%
  summarise(
    total_rows = n(),
    cd4_missing_pct = mean(is.na(cd4_abs)) * 100,
    neutrophil_missing_pct = mean(is.na(neutrophils_abs)) * 100,
    viral_load_missing_pct = mean(is.na(viral_load)) * 100,
    ct_missing_pct = mean(is.na(q_pcr_raw_average_sarsc_t)) * 100
  ) %>%
  print()

# --- 4. Structural Missingness Verification (Viral Load & HIV Status) ---
cat("\n=== Viral Load Counts by HIV Status ===\n")
raw_data %>%
  group_by(hiv_status) %>%
  summarise(
    total_rows = n(),
    na_count = sum(is.na(viral_load)),
    equals_40_count = sum(!is.na(viral_load) & viral_load == 40),
    valid_numeric_count = sum(!is.na(viral_load) & viral_load != 40)
  ) %>%
  print()

# --- 5. Cohort Filtering: Exclude PUIs & Summary ---
confirmed_cohort <- raw_data %>%
  filter(participant_type == "Case")

cat("\n=== Cohort Filtering Summary ===\n")
cat("Raw Rows:", nrow(raw_data), "\n")
cat("Confirmed Case Rows:", nrow(confirmed_cohort), "\n")

# Safely extract unique participant count using "Deidentified_PID" regardless of janitor renaming
pid_column <- grep("deidentified", names(confirmed_cohort), value = TRUE, ignore.case = TRUE)[1]

cat("Unique Confirmed Participants:", n_distinct(confirmed_cohort[[pid_column]]), "\n")
```
---

## 7. Methodological Uncertainties & Unsure Points

1. **Prevalent Outcome Adjustment Strategy:** Given that 90.7% of severe cases were already severe at Visit 1 ($T_0$), should primary logistic models estimate overall odds of severe disease course across the full cohort ($N = 226$) while adjusting for baseline covariates, or should primary inference be restricted to an incidence model among participants non-severe at entry ($n = 158$)?

2. **Handling Left-Censored Viral Loads:** For HIV-positive participants with `Viral_Load == 40` (the lower limit of detection), is Tobit/left-censored regression preferred for log-viral load modeling, or is binary classification (Suppressed $<40$ vs. Unsuppressed $\ge 40$) methodologically superior given clinical interest in viral suppression?