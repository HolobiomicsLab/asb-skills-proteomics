---
name: targeted-proteomics-calibration-curve-fitting
description: Use when concentration/dilution-series data from targeted MS (SRM/MRM
  or PRM, e.g., msqc1 heavy/light peptide peak areas) must be converted into calibration
  curves with LLOQ/ULOQ and linear-range estimates, or compared across instruments.
when_to_use_negative:
- Data are not from a dilution/concentration series with known analyte amounts (CalibraCurve
  relies on known concentrations to fit calibration curves)
- The goal is global/shotgun proteomics quantification rather than targeted (SRM/MRM
  or PRM) assay calibration
- Only raw instrument files need inspection without curve fitting — raw data inspection
  is done with Skyline, not CalibraCurve
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: CalibraCurve
  role: R package that performs automated batch-mode determination of dynamic linear
    ranges and quantification limits from dilution-series data, creating calibration
    curves and response factor plots
  repo: https://github.com/mpc-bioinformatics/CalibraCurve
- name: R
  role: Statistical computing environment in which CalibraCurve runs and in which
    the msqc1 SummarizedExperiment data is loaded and the per-instrument summary table
    is aggregated
- name: msqc1
  role: Source package providing the msqc1_dil SummarizedExperiment dilution-series
    dataset (heavy/light peptide peak areas) used as calibration input
- name: Skyline
  role: Used to inspect raw data prior to normalization and curve fitting in R
- name: CalibraCurve_NF
  role: Nextflow workflow wrapping the CalibraCurve R package for batch execution
  repo: https://github.com/mpc-bioinformatics/CalibraCurve_NF
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/pmic.201900143
    title: 'CalibraCurve: A Tool for Calibration of Targeted MS‐Based Measurements'
  source_content_hash: sha256:cb2cab4029d8b41baf4566bfa7837f355c1405ca766e4f5bd3de72275071f6ee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mpc-bioinformatics/CalibraCurve
  tools:
  - CalibraCurve
  - CalibraCurve_NF
  - R
  - Skyline
  - msqc1
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# targeted-proteomics-calibration-curve-fitting

## Summary

Fits calibration curves to targeted-MS dilution-series data (e.g., msqc1 heavy/light peptide peak areas) using CalibraCurve to determine dynamic linear ranges, quantification limits, and R-squared quality of fit per instrument, enabling cross-instrument benchmarking.

## When to use

Use when concentration/dilution-series data from targeted MS (SRM/MRM or PRM, e.g., msqc1 heavy/light peptide peak areas) must be converted into calibration curves with LLOQ/ULOQ and linear-range estimates, or compared across instruments.

## When NOT to use

- Data are not from a dilution/concentration series with known analyte amounts (CalibraCurve relies on known concentrations to fit calibration curves)
- The goal is global/shotgun proteomics quantification rather than targeted (SRM/MRM or PRM) assay calibration
- Only raw instrument files need inspection without curve fitting — raw data inspection is done with Skyline, not CalibraCurve

## Inputs

- msqc1 SummarizedExperiment dilution-series object (heavy/light peptide peak areas) from the msqc1 R package
- Concentration/dilution-series measurements of known analyte amounts, measured in replicates by targeted MS (SRM/MRM or PRM)

## Outputs

- Calibration curves and response factor plots per peptide-instrument combination
- Dynamic linear ranges with lower and upper limits of quantification (LLOQ/ULOQ)
- Per-instrument summary table of R-squared values and linear ranges (calibracurve_instrument_summary.csv)

## How to apply

Load the msqc1 SummarizedExperiment dilution-series object (heavy/light peptide peak areas) from the msqc1 package in R and extract the heavy-peptide dilution data. Install CalibraCurve from Bioconductor (BiocManager::install) and run it in automated batch mode separately for each instrument's measurements (e.g., QTRAP, TSQ Vantage, Q Exactive). CalibraCurve assesses calibration accuracy using precision and trueness measures to determine the dynamic linear range and the lower/upper limits of quantification (LLOQ/ULOQ), since linearity is often limited to a certain concentration range. Aggregate the fitted R-squared values and linear ranges per instrument into a summary table. In the msqc1 benchmark this pattern is expected: Q Exactive (PRM) yields the worst overall R-squared while TSQ Vantage gives the widest linear range; write the table to calibracurve_instrument_summary.csv.

## Related tools

- **CalibraCurve** (R package that performs automated batch-mode determination of dynamic linear ranges and quantification limits from dilution-series data, creating calibration curves and response factor plots) — https://github.com/mpc-bioinformatics/CalibraCurve
- **R** (Statistical computing environment in which CalibraCurve runs and in which the msqc1 SummarizedExperiment data is loaded and the per-instrument summary table is aggregated)
- **msqc1** (Source package providing the msqc1_dil SummarizedExperiment dilution-series dataset (heavy/light peptide peak areas) used as calibration input)
- **Skyline** (Used to inspect raw data prior to normalization and curve fitting in R)
- **CalibraCurve_NF** (Nextflow workflow wrapping the CalibraCurve R package for batch execution) — https://github.com/mpc-bioinformatics/CalibraCurve_NF

## Examples

```
BiocManager::install("CalibraCurve"); library(CalibraCurve)
```

## Evaluation signals

- The per-instrument summary table reproduces the published benchmark pattern: Q Exactive (PRM) shows the lowest R-squared overall and TSQ Vantage the widest linear range across the msqc1 dilution series
- Each peptide-instrument combination yields a defined dynamic linear range with LLOQ and ULOQ derived from precision and trueness measures
- Calibration curves and response factor plots are generated for every assessed analyte
- The summary CSV (calibracurve_instrument_summary.csv) contains one row per instrument with R-squared and linear-range columns populated
- Linear ranges fall within the expected magnitude for targeted proteomics (up to 4-5 orders of magnitude)

## Limitations

- A linear relationship between concentration and intensity is often limited to a certain range; reduction of product ion intensities was observed at higher analyte concentrations, indicating the need of ULOQ determination
- Development of the former KNIME-workflow was discontinued and replaced by the R package and nextflow workflow, so legacy workflow versions may not be maintained
- Results should be appropriately validated before further use; the software is intended exclusively for academic research and must not be used for clinical diagnosis

## Evidence

- [abstract] a software named CalibraCurve that enables an automated batch-mode determination of dynamic linear ranges and quantification limits: "a software named CalibraCurve that enables an automated batch-mode determination of dynamic linear ranges and quantification limits"
- [abstract] The software uses a variety of measures to assess the accuracy of the calibration, namely precision and trueness.: "The software uses a variety of measures to assess the accuracy of the calibration, namely precision and trueness."
- [intro] In comparison with classical shotgun techniques, targeted proteomics approaches (SRM/MRM and PRM) show higher sensitivity, reliability, and reproducibility.: "In comparison with classical shotgun techniques, targeted proteomics approaches (SRM/MRM and PRM) show higher sensitivity, reliability, and reproducibility."
- [readme] CalibraCurve calculates the linear range (lower and upper limits of quantification) using a well-designed algorithm.: [evidence span withheld — anchor status: external_doc]
- [readme] However, a linear relationship between concentration and intensity is often limited to a certain range of concentrations, the so called linear range.: [evidence span withheld — anchor status: external_doc]
