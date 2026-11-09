---
name: missing-measurement-exclusion
description: Use when a calibration table (e.g. .xlsx/.csv/.txt with Concentration
  and Measurement columns) contains zero or NA measurements, or concentration levels
  with too few replicate measurements, prior to linear-range or LLOQ/ULOQ estimation.
when_to_use_negative:
- The input is raw instrument output (e.g. Skyline-exported transition intensities)
  that has not yet been assembled into a Concentration/Measurement table
- The goal is exploratory inspection of measurement completeness rather than preparation
  for calibration curve fitting
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: CalibraCurve
  role: R package whose readDataTable function and preprocessing stage perform the
    missing/zero measurement exclusion and replicate-count filtering before curve
    fitting
  repo: https://github.com/mpc-bioinformatics/CalibraCurve
- name: R
  role: Runtime environment for executing the CalibraCurve preprocessing and filtering
    steps
  repo: https://www.R-project.org/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pmic.201900143
    title: 'CalibraCurve: A Tool for Calibration of Targeted MS‐Based Measurements'
  source_content_hash: sha256:8a7701844ae6ca5dfa60c192d6c5c3e3e8ff47d938306573ec96a9775d57ed14
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mpc-bioinformatics/CalibraCurve
  tools:
  - CalibraCurve
  - R
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# missing-measurement-exclusion

## Summary

A preprocessing filter from CalibraCurve's calibration workflow that removes rows with zero or missing (NA) measurement values and drops concentration levels lacking sufficient replicates before fitting calibration curves. This prevents non-informative measurements from corrupting precision and trueness-based assessment of the linear range.

## When to use

Use when a calibration table (e.g. .xlsx/.csv/.txt with Concentration and Measurement columns) contains zero or NA measurements, or concentration levels with too few replicate measurements, prior to linear-range or LLOQ/ULOQ estimation.

## When NOT to use

- The input is raw instrument output (e.g. Skyline-exported transition intensities) that has not yet been assembled into a Concentration/Measurement table
- The goal is exploratory inspection of measurement completeness rather than preparation for calibration curve fitting

## Inputs

- Single .xlsx, .csv or .txt calibration file (e.g. msqc1 GGPFSDSYR_QExactive_y5.xlsx) with at least Concentration and Measurement columns
- Replicate measurements per concentration level from dilution/concentration series

## Outputs

- Cleaned concentration–measurement table with zero/NA rows and under-replicated concentration levels removed (e.g. cleaned_calibration_data.csv)

## How to apply

Load the calibration file with CalibraCurve's readDataTable, which accepts single .xlsx, .csv or .txt files containing at least a Concentration column and a Measurement column. Remove rows where the Measurement value is zero or NA, since these cannot contribute to calibration accuracy assessment (precision and trueness are the metrics CalibraCurve uses). Then count replicate measurements per concentration level and drop any concentration level that does not provide sufficient replicates for downstream fitting. The rationale is that CalibraCurve's algorithm computes the linear range (LLOQ and ULOQ) from the concentration–intensity relationship, so undefined or non-measured intensities would distort dynamic-range limits. Write the cleaned concentration–measurement table (e.g. to cleaned_calibration_data.csv) so the filtered basis of the fit is auditable.

## Related tools

- **CalibraCurve** (R package whose readDataTable function and preprocessing stage perform the missing/zero measurement exclusion and replicate-count filtering before curve fitting) — https://github.com/mpc-bioinformatics/CalibraCurve
- **R** (Runtime environment for executing the CalibraCurve preprocessing and filtering steps) — https://www.R-project.org/

## Examples

```
data <- CalibraCurve::readDataTable("GGPFSDSYR_QExactive_y5.xlsx"); data <- data[!is.na(data$Measurement) & data$Measurement != 0, ]
```

## Evaluation signals

- Cleaned table contains no rows with Measurement == 0 or Measurement == NA
- Every remaining concentration level retains at least the minimum number of replicate measurements required for fitting
- Concentration and Measurement columns are both present and parseable by readDataTable after cleaning
- Row counts decrease monotonically relative to the raw input file, and dropped levels are traceable to replicate-count deficits

## Limitations

- Only single-file inputs (.xlsx, .csv, .txt) with the expected Concentration/Measurement structure are supported by the preprocessing stage
- Dropping under-replicated concentration levels can shorten the usable dynamic range if replicates are missing at the extremes of the dilution series

## Evidence

- [readme] Quality of the data and the accuracy of the produced calibration curve is assessed in several computational steps.: [evidence span withheld — anchor status: external_doc]
- [abstract] CalibraCurve assesses calibration accuracy using precision and trueness measures: "The software uses a variety of measures to assess the accuracy of the calibration, namely precision and trueness."
- [other] readDataTable accepts xlsx/csv/txt with Concentration and Measurement columns: [evidence span withheld — anchor status: not_found]
- [other] Filter out zero/NA measurements workflow step: [evidence span withheld — anchor status: not_found]
