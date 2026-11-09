---
name: spreadsheet-data-import
description: Use when the input is a single .xlsx, .csv or .txt file containing at
  least a Concentration column and a Measurement column from a targeted MS (SRM/MRM/PRM)
  dilution series, and the goal is to prepare data for calibration-curve fitting with
  CalibraCurve.
when_to_use_negative:
- The input is already a cleaned concentration-measurement table with zero/NA rows
  removed — re-importing and re-filtering is unnecessary.
- The data are raw instrument outputs (e.g. Skyline raw data) rather than a spreadsheet
  with Concentration and Measurement columns; those must first be exported/normalized
  to a tabular format.
- The task is calibration fitting itself (determining LLOQ/ULOQ or linear range) rather
  than reading and preprocessing the input data.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: CalibraCurve
  role: R package whose readDataTable function imports the .xlsx/.csv/.txt calibration
    file and drives the preprocessing (process step 1)
  repo: https://github.com/mpc-bioinformatics/CalibraCurve
- name: R
  role: Language environment in which the import, filtering of zero/NA measurements,
    and replicate counting are performed
  repo: https://www.R-project.org/
- name: msqc1
  role: R package supplying the example calibration data (e.g. GGPFSDSYR_QExactive_y5.xlsx)
    used to exercise the import step
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pmic.201900143
    title: 'CalibraCurve: A Tool for Calibration of Targeted MS‐Based Measurements'
  source_content_hash: sha256:2c6dba78cbdc97e30f34c7dd454efd9000f4f33758e9cad061f73ee34cc5e8c0
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
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# spreadsheet-data-import

## Summary

Reads spreadsheet-style calibration input files (.xlsx, .csv, .txt) into R as a concentration-measurement table so that targeted MS calibration data can be cleaned and fitted. This is the entry point of CalibraCurve's preprocessing (process step 1) before linear-range determination.

## When to use

Use when the input is a single .xlsx, .csv or .txt file containing at least a Concentration column and a Measurement column from a targeted MS (SRM/MRM/PRM) dilution series, and the goal is to prepare data for calibration-curve fitting with CalibraCurve.

## When NOT to use

- The input is already a cleaned concentration-measurement table with zero/NA rows removed — re-importing and re-filtering is unnecessary.
- The data are raw instrument outputs (e.g. Skyline raw data) rather than a spreadsheet with Concentration and Measurement columns; those must first be exported/normalized to a tabular format.
- The task is calibration fitting itself (determining LLOQ/ULOQ or linear range) rather than reading and preprocessing the input data.

## Inputs

- single .xlsx file with Concentration and Measurement columns (e.g. GGPFSDSYR_QExactive_y5.xlsx from the msqc1 package)
- single .csv or .txt file with Concentration and Measurement columns from a dilution series

## Outputs

- cleaned concentration-measurement table (cleaned_calibration_data.csv) with zero/NA measurements removed and under-replicated concentration levels dropped

## How to apply

Load the calibration file with CalibraCurve's readDataTable function, which accepts single .xlsx, .csv or .txt files and expects at least a Concentration column and a Measurement column. Use a shipped example dataset such as the msqc1 file GGPFSDSYR_QExactive_y5.xlsx (acquired on a Q Exactive instrument) to validate the import path. After reading, filter out rows where the Measurement value is zero or missing (NA), since these cannot contribute to calibration accuracy assessment. Then count replicate measurements per concentration level and drop any concentration level lacking sufficient replicates for downstream fitting. Write the cleaned concentration-measurement table to cleaned_calibration_data.csv so the preprocessing output can be diffed against the original file.

## Related tools

- **CalibraCurve** (R package whose readDataTable function imports the .xlsx/.csv/.txt calibration file and drives the preprocessing (process step 1)) — https://github.com/mpc-bioinformatics/CalibraCurve
- **R** (Language environment in which the import, filtering of zero/NA measurements, and replicate counting are performed) — https://www.R-project.org/
- **msqc1** (R package supplying the example calibration data (e.g. GGPFSDSYR_QExactive_y5.xlsx) used to exercise the import step)

## Examples

```
Rscript -e 'library(CalibraCurve); d <- readDataTable("GGPFSDSYR_QExactive_y5.xlsx"); d <- d[!is.na(d$Measurement) & d$Measurement != 0, ]; write.csv(d, "cleaned_calibration_data.csv", row.names=FALSE)'
```

## Evaluation signals

- The loaded table contains both a Concentration column and a Measurement column; import fails or is invalid if either is missing from the .xlsx/.csv/.txt source.
- No rows remain in the cleaned table with Measurement == 0 or Measurement == NA after filtering.
- Every retained concentration level has a sufficient number of replicate measurements for downstream calibration fitting; under-replicated levels are dropped and the drop is documented.
- cleaned_calibration_data.csv row counts equal the raw file row counts minus removed zero/NA rows minus dropped concentration levels (a verifiable diff against the source file).

## Limitations

- Only single .xlsx, .csv or .txt files with the expected Concentration/Measurement column structure are supported; other formats or schemas are not handled by readDataTable.
- Measurements that are zero or NA are discarded, so genuinely zero-intensity observations cannot be retained for calibration.
- CalibraCurve assumes concentration/dilution-series experiments measured in replicates; data without replicate structure at concentration levels cannot be validated for sufficiency.

## Evidence

- [other] Preprocessing accepts xlsx/csv/txt with Concentration and Measurement columns: [evidence span withheld — anchor status: not_found]
- [readme] README: CalibraCurve generates calibration curves from dilution-series data: [evidence span withheld — anchor status: external_doc]
- [readme] README: installation of CalibraCurve R package from Bioconductor: "BiocManager::install("CalibraCurve")"
- [abstract] Article abstract: automated determination of dynamic ranges and quantification limits: "a software named CalibraCurve that enables an automated batch-mode determination of dynamic linear ranges and quantification limits"
- [methods] Methods: msqc1 dataset citation: "msqc1 = citation("msqc1")[1]"
