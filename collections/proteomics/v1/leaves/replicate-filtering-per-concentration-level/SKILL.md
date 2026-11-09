---
name: replicate-filtering-per-concentration-level
description: Use when a concentration-series table (e.g. .xlsx/.csv/.txt with Concentration
  and Measurement columns, such as msqc1 GGPFSDSYR_QExactive_y5.xlsx) must be cleaned
  of zero/NA measurements and under-replicated levels before curve fitting.
when_to_use_negative:
- The dataset is not a concentration/dilution series with a Concentration column —
  CalibraCurve expects samples with known amounts of the analyte measured by mass
  spectrometry
- 'Rows are already curated: no zero/NA Measurement values remain and every concentration
  level already has sufficient replicates'
- The goal is inspecting raw data upstream (e.g. Skyline inspection of raw files)
  rather than preparing a table for curve fitting
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: CalibraCurve
  role: R package whose preprocessing stage (readDataTable) reads the calibration
    table and whose downstream algorithm computes the linear range after replicate
    filtering
  repo: github.com/mpc-bioinformatics/CalibraCurve
- name: R
  role: Runtime environment for loading, filtering and writing the cleaned concentration-measurement
    table
  repo: https://www.R-project.org/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pmic.201900143
    title: 'CalibraCurve: A Tool for Calibration of Targeted MS‐Based Measurements'
  source_content_hash: sha256:3b1bc382091a7f0363a872dba6c48b7675f68d43bc5769f27394716f51e3f51c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CalibraCurve
  - R
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# replicate-filtering-per-concentration-level

## Summary

A preprocessing skill from the CalibraCurve calibration workflow: remove zero/NA measurements and concentration levels lacking sufficient replicate support before fitting calibration curves, so that the accuracy assessment (precision/trueness) is not skewed by unsupported levels.

## When to use

Use when a concentration-series table (e.g. .xlsx/.csv/.txt with Concentration and Measurement columns, such as msqc1 GGPFSDSYR_QExactive_y5.xlsx) must be cleaned of zero/NA measurements and under-replicated levels before curve fitting.

## When NOT to use

- The dataset is not a concentration/dilution series with a Concentration column — CalibraCurve expects samples with known amounts of the analyte measured by mass spectrometry
- Rows are already curated: no zero/NA Measurement values remain and every concentration level already has sufficient replicates
- The goal is inspecting raw data upstream (e.g. Skyline inspection of raw files) rather than preparing a table for curve fitting

## Inputs

- Single .xlsx, .csv, or .txt calibration file with at least a Concentration column and a Measurement column (e.g. msqc1 GGPFSDSYR_QExactive_y5.xlsx from the CalibraCurve package)

## Outputs

- Cleaned concentration-measurement table with zero/NA rows and insufficient-replicate levels removed (e.g. cleaned_calibration_data.csv)

## How to apply

Load the calibration file with CalibraCurve's readDataTable function, which imports single .xlsx, .csv or .txt files containing at least a Concentration column and a Measurement column. Filter out rows in which the Measurement value is zero or NA, since these cannot contribute to calibration accuracy assessment. Count the replicate measurements per concentration level and drop any concentration level that does not provide sufficient replicates for downstream calibration fitting (decision point: a level with too few replicates cannot support precision/trueness assessment). Write the cleaned concentration-measurement table (e.g. to cleaned_calibration_data.csv) before invoking CalibraCurve's linear-range algorithm that computes LLOQ/ULOQ.

## Related tools

- **CalibraCurve** (R package whose preprocessing stage (readDataTable) reads the calibration table and whose downstream algorithm computes the linear range after replicate filtering) — github.com/mpc-bioinformatics/CalibraCurve
- **R** (Runtime environment for loading, filtering and writing the cleaned concentration-measurement table) — https://www.R-project.org/

## Examples

```
d <- CalibraCurve::readDataTable("GGPFSDSYR_QExactive_y5.xlsx"); d <- d[!is.na(d$Measurement) & d$Measurement != 0, ]; write.csv(d, "cleaned_calibration_data.csv")
```

## Evaluation signals

- Output table contains no rows with Measurement == 0 or Measurement == NA
- Every retained Concentration level has at least the required number of replicate measurements; levels below the threshold are absent
- Row count of cleaned_calibration_data.csv equals original rows minus zero/NA measurements and minus all rows of dropped concentration levels
- Cleaned table still contains both a Concentration column and a Measurement column and can be consumed by CalibraCurve's readDataTable/fitting step
- Downstream CalibraCurve run produces calibration curves and LLOQ/ULOQ limits without errors attributable to empty or singleton concentration levels

## Limitations

- Replicate counts per level are limited by the assay design; dilution/concentration series measured 'usually in relicates' may leave singleton levels that must be dropped, reducing the usable linear range
- Preprocessing applies to single files read by readDataTable; batch handling requires the nextflow workflow (CalibraCurve_NF) rather than this per-file step
- Zero and NA measurements are unrecoverable at this stage and cannot contribute to the precision/trueness accuracy assessment

## Evidence

- [readme] Samples with known amount of the analyte of interest are generated and measured by mass spectrometry, usually in relicates.: [evidence span withheld — anchor status: external_doc]
- [readme] CalibraCurve calculates the linear range using a well-designed algorithm.: [evidence span withheld — anchor status: external_doc]
- [abstract] dynamic linear ranges that span up to 4-5 orders of magnitude: "dynamic linear ranges that span up to 4-5 orders of magnitude"
- [abstract] uses a variety of measures to assess the accuracy of the calibration, namely precision and trueness: "The software uses a variety of measures to assess the accuracy of the calibration, namely precision and trueness."
