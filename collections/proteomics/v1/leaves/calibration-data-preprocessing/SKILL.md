---
name: calibration-data-preprocessing
description: Use when raw targeted-MS calibration data from a concentration or dilution
  series (e.g. msqc1 xlsx files such as GGPFSDSYR_QExactive_y5.xlsx) must be loaded
  and cleaned before determining dynamic linear ranges and quantification limits.
when_to_use_negative:
- The input already contains no zero/NA measurements or the replicate counts per concentration
  level have not been assessed — reapplying the filter would needlessly alter the
  data
- The data is untargeted shotgun proteomics output (e.g. raw mzML/Skyline export)
  rather than a concentration-dilution series table used for SRM/MRM or PRM assay
  calibration
- The goal is to fit the calibration curve or determine LLOQ/ULOQ — that is the downstream
  CalibraCurve fitting step, not preprocessing
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: CalibraCurve
  role: R package whose readDataTable function imports the calibration table; its
    calibration-fitting step consumes the cleaned data
  repo: https://github.com/mpc-bioinformatics/CalibraCurve
- name: R
  role: Runtime environment for executing the CalibraCurve preprocessing and filtering
    steps
  repo: https://www.R-project.org/
- name: msqc1
  role: Source of the shipped example xlsx calibration files used as test input for
    preprocessing
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pmic.201900143
    title: 'CalibraCurve: A Tool for Calibration of Targeted MS‐Based Measurements'
  source_content_hash: sha256:a559cc698158b300ed99b2106178bac64d0923a1851ec6d0acd0d09234a5c23d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mpc-bioinformatics/CalibraCurve
  tools:
  - CalibraCurve
  - R
  - msqc1
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# calibration-data-preprocessing

## Summary

Reconstructs the preprocessing stage of CalibraCurve, which reads single .xlsx, .csv or .txt calibration files containing Concentration and Measurement columns via readDataTable and cleans them (zero/NA removal, replicate-count filtering) before linear-range fitting.

## When to use

Use when raw targeted-MS calibration data from a concentration or dilution series (e.g. msqc1 xlsx files such as GGPFSDSYR_QExactive_y5.xlsx) must be loaded and cleaned before determining dynamic linear ranges and quantification limits.

## When NOT to use

- The input already contains no zero/NA measurements or the replicate counts per concentration level have not been assessed — reapplying the filter would needlessly alter the data
- The data is untargeted shotgun proteomics output (e.g. raw mzML/Skyline export) rather than a concentration-dilution series table used for SRM/MRM or PRM assay calibration
- The goal is to fit the calibration curve or determine LLOQ/ULOQ — that is the downstream CalibraCurve fitting step, not preprocessing

## Inputs

- Single calibration data file (.xlsx, .csv or .txt) with at least a Concentration column and a Measurement column
- msqc1 package example file such as GGPFSDSYR_QExactive_y5.xlsx

## Outputs

- Cleaned concentration-measurement table written to cleaned_calibration_data.csv

## How to apply

Import a single .xlsx, .csv or .txt file via CalibraCurve's readDataTable function, which requires at least a Concentration column and a Measurement column. In R, load the calibration file (e.g. the msqc1 example GGPFSDSYR_QExactive_y5.xlsx) and subset to these two columns. Filter out rows where the Measurement value is zero or NA, since zero/missing measurements cannot contribute to calibration accuracy assessment. Count replicate measurements per concentration level and drop concentration levels with insufficient replicates for downstream calibration fitting. The rationale is that CalibraCurve assesses calibration accuracy using precision and trueness, so unreliable measurement rows would corrupt the precision-based evaluation. Write the cleaned concentration-measurement table to cleaned_calibration_data.csv for the subsequent curve-fitting step.

## Related tools

- **CalibraCurve** (R package whose readDataTable function imports the calibration table; its calibration-fitting step consumes the cleaned data) — https://github.com/mpc-bioinformatics/CalibraCurve
- **R** (Runtime environment for executing the CalibraCurve preprocessing and filtering steps) — https://www.R-project.org/
- **msqc1** (Source of the shipped example xlsx calibration files used as test input for preprocessing)

## Examples

```
library(CalibraCurve); data <- readDataTable("GGPFSDSYR_QExactive_y5.xlsx"); clean <- data[!is.na(data$Measurement) & data$Measurement != 0, ]; write.csv(clean, "cleaned_calibration_data.csv")
```

## Evaluation signals

- The loaded table contains both a Concentration column and a Measurement column; loading fails or errors otherwise
- After filtering, no row has a zero or NA Measurement value
- Every retained concentration level has a sufficient number of replicate measurements for calibration fitting; levels with too few replicates are absent
- The cleaned table is written as cleaned_calibration_data.csv and its row count equals the input row count minus zero/NA and low-replicate removals
- Downstream CalibraCurve calibration fitting succeeds on the cleaned table, producing precision- and trueness-based accuracy assessment

## Limitations

- Preprocessing only accepts single .xlsx, .csv or .txt files; other formats are not supported by readDataTable
- Rows with zero or missing Measurement values must be excluded, so very sparse datasets may lose many rows
- Concentration levels with insufficient replicates are dropped, which may remove legitimately measured points near the quantification limits
- Development of the former KNIME-workflow was discontinued and replaced by the R package and a nextflow workflow (CalibraCurve_NF)

## Evidence

- [abstract] enables an automated batch-mode determination of dynamic linear ranges and quantification limits for both targeted proteomics and similar assays: "enables an automated batch-mode determination of dynamic linear ranges and quantification limits for both targeted proteomics and similar assays"
- [abstract] The software uses a variety of measures to assess the accuracy of the calibration, namely precision and trueness.: "The software uses a variety of measures to assess the accuracy of the calibration, namely precision and trueness."
- [readme] This kind of data is used by CalibraCurve to generate calibration curves, which can be the basis for predicting concentrations from intensities in new data.: [evidence span withheld — anchor status: external_doc]
- [readme] During assay development, often experiments using concentration or dilution series are conducted.: [evidence span withheld — anchor status: external_doc]
