---
name: mrm-dilution-series-analysis
description: Use when analyzing dilution/concentration-series replicate measurements
  from targeted MS assays (e.g. MRM transition-level intensities for a peptide like
  WTVFQK/MFAP4) to determine the linear range, LLOQ/ULOQ, and compare weighted vs
  unweighted calibration models.
when_to_use_negative:
- The data are classical shotgun (DDA) proteomics measurements rather than targeted
  SRM/MRM or PRM dilution series with known analyte concentrations
- The calibration model comparison relies on R-squared alone, without the precision/trueness
  (PB_av) assessment at low concentration levels
- A single concentration level without replicate dilution points is all that is available
  — the linear range algorithm needs a multi-level dilution series
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: CalibraCurve
  role: R package that runs the automated batch-mode determination of dynamic linear
    ranges and quantification limits and computes precision/trueness-based calibration
    metrics
  repo: https://github.com/mpc-bioinformatics/CalibraCurve
- name: CalibraCurve_NF
  role: Nextflow workflow wrapping the CalibraCurve R package for pipeline execution
  repo: https://github.com/mpc-bioinformatics/CalibraCurve_NF
- name: R
  role: Language/environment in which CalibraCurve is implemented and calibration
    regression is performed
  repo: https://www.R-project.org/
- name: msqc1
  role: Source of the msqc1_dil dilution-series SummarizedExperiment benchmark data
    used for multi-instrument (QTRAP, TSQ Vantage, Q Exactive) calibration comparisons
- name: Skyline
  role: Used for inspecting raw targeted-MS data prior to calibration curve fitting
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.1002/pmic.201900143
    title: 'CalibraCurve: A Tool for Calibration of Targeted MS‐Based Measurements'
  source_content_hash: sha256:b60c8c69058334373cf73ec44b2616f556c5d583ebe771e9e617154801a0f355
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mpc-bioinformatics/CalibraCurve
  tools:
  - CalibraCurve
  - CalibraCurve_NF
  - R
  - Skyline
  - ggplot2
  - msqc1
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# mrm-dilution-series-analysis

## Summary

Determines the dynamic linear range and quantification limits (LLOQ/ULOQ) of targeted MS (SRM/MRM/PRM) dilution-series calibration data using CalibraCurve, assessing calibration accuracy with precision- and trueness-based measures (PB_av) rather than R² alone.

## When to use

Use when analyzing dilution/concentration-series replicate measurements from targeted MS assays (e.g. MRM transition-level intensities for a peptide like WTVFQK/MFAP4) to determine the linear range, LLOQ/ULOQ, and compare weighted vs unweighted calibration models.

## When NOT to use

- The data are classical shotgun (DDA) proteomics measurements rather than targeted SRM/MRM or PRM dilution series with known analyte concentrations
- The calibration model comparison relies on R-squared alone, without the precision/trueness (PB_av) assessment at low concentration levels
- A single concentration level without replicate dilution points is all that is available — the linear range algorithm needs a multi-level dilution series

## Inputs

- Transition-level peak-area/intensity table from an SRM/MRM or PRM dilution series (e.g. Agilent 6490 MRM data from PASSEL accession PASS00653), formatted for CalibraCurve
- msqc1_dil SummarizedExperiment object (heavy/light peptide peak areas) from the msqc1 R package
- Concentration/dilution series measured in replicates with known analyte amounts

## Outputs

- Determined linear range with lower and upper limits of quantification (LLOQ/ULOQ) per peptide/analyte
- Per-model calibration metrics (R-squared, PB_av at lowest level and at specific levels such as 0.25 fmol) for unweighted vs 1/x^2-weighted fits
- Calibration curves and response factor plots
- Summary tables such as calibration_metrics.csv or calibracurve_instrument_summary.csv comparing R-squared and linear range across instruments

## How to apply

Format the transition-level intensities as a CalibraCurve-compatible input table (e.g. from PASSEL data such as PASS00653 acquired on an Agilent 6490 triple quadrupole). Run CalibraCurve in automated batch mode to determine the final linear range and fit an unweighted linear model, recording R-squared and the precision/trueness-based PB_av. Re-run on the same linear range with 1/x^2 weighting in the linear regression and compare both models. Judge model quality by PB_av at the lowest calibration levels (e.g. 73.75% unweighted vs 14.65% weighted at the lowest level; 72.4% vs 44.6% at 0.25 fmol), since R² alone is misleading — the unweighted model may attain the higher R² (~0.994 vs ~0.932) yet be substantially less true at low concentrations. Aggregate metrics per peptide–instrument combination (e.g. QTRAP, TSQ Vantage, Q Exactive on msqc1_dil data) when benchmarking across instruments.

## Related tools

- **CalibraCurve** (R package that runs the automated batch-mode determination of dynamic linear ranges and quantification limits and computes precision/trueness-based calibration metrics) — https://github.com/mpc-bioinformatics/CalibraCurve
- **CalibraCurve_NF** (Nextflow workflow wrapping the CalibraCurve R package for pipeline execution) — https://github.com/mpc-bioinformatics/CalibraCurve_NF
- **R** (Language/environment in which CalibraCurve is implemented and calibration regression is performed) — https://www.R-project.org/
- **msqc1** (Source of the msqc1_dil dilution-series SummarizedExperiment benchmark data used for multi-instrument (QTRAP, TSQ Vantage, Q Exactive) calibration comparisons)
- **Skyline** (Used for inspecting raw targeted-MS data prior to calibration curve fitting)

## Examples

```
if (!requireNamespace("BiocManager", quietly=TRUE)) { install.packages("BiocManager") }; BiocManager::install("CalibraCurve")
```

## Evaluation signals

- The per-instrument summary matches the published benchmark: Q Exactive (PRM) yields the worst overall R-squared while TSQ Vantage gives the widest linear range on the msqc1 dilution series
- Reproduced WTVFQK (MFAP4) metrics match the paper: R² ≈ 0.994 unweighted vs ≈ 0.932 with 1/x² weighting
- PB_av values at the lowest level reproduce ~73.75% (unweighted) vs ~14.65% (weighted), and ~72.4% vs ~44.6% at the 0.25 fmol level
- Model selection is justified by PB_av trueness at low concentrations, not by R² alone (unweighted model has higher R² but worse PB_av)
- The output tables (calibration_metrics.csv, calibracurve_instrument_summary.csv) contain per-peptide/per-instrument R-squared and linear range entries for every level set analyzed

## Limitations

- R² alone is misleading for comparing calibration performance: the unweighted model can have the higher R² yet substantially worse trueness (PB_av) at the lowest concentration levels
- Linear behavior is limited to a certain concentration range, so calibration outside the determined linear range (e.g. reduced product ion intensities at higher analyte concentrations near ULOQ) is not valid
- The formerly available KNIME workflow was discontinued and replaced by the R package and nextflow workflow; software is BSD 3-Clause licensed for academic research only and not for clinical diagnosis

## Evidence

- [abstract] a software named CalibraCurve that enables an automated batch-mode determination of dynamic linear ranges and quantification limits: "a software named CalibraCurve that enables an automated batch-mode determination of dynamic linear ranges and quantification limits"
- [abstract] The software uses a variety of measures to assess the accuracy of the calibration, namely precision and trueness. Two different kinds of customizable graphs are created: "The software uses a variety of measures to assess the accuracy of the calibration, namely precision and trueness. Two different kinds of customizable graphs are created"
- [intro] SRM/MRM has a dynamic linear range that span up to 4–5 orders of magnitude.: [evidence span withheld — anchor status: not_found]
- [intro] we observed reduction of product ion intensities at higher analyte concentrations (Figure 1): "we observed reduction of product ion intensities at higher analyte concentrations (Figure 1)"
- [readme] However, a linear relationship between concentration and intensity is often limited to a certain range of concentrations, the so called linear range.: [evidence span withheld — anchor status: external_doc]
- [intro] Raw data was inspected with the Skyline software, followingnormalizationandcurvefittinginR.: "Raw data was inspected with the Skyline software, followingnormalizationandcurvefittinginR."
