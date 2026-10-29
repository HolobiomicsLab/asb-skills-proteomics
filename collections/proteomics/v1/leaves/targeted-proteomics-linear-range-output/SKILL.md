---
name: targeted-proteomics-linear-range-output
description: Use when targeted MS (SRM/MRM or PRM) calibration data from concentration
  or dilution series with known analyte amounts must yield an automated linear range
  (LLOQ/ULOQ) and fitted calibration curves for downstream concentration prediction.
when_to_use_negative:
- Data are from classical shotgun (DDA) proteomics rather than targeted SRM/MRM or
  PRM assays, where the calibration-based linear range concept does not apply the
  same way
- The calibration relationship must hold across the full concentration span with no
  linear range restriction — CalibraCurve explicitly models the region where linearity
  breaks down (e.g. reduced product ion intensities at higher analyte concentrations)
- Raw MS data inspection is the goal (use Skyline); CalibraCurve operates on curve-fitting/calibration
  data, not raw spectra
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: CalibraCurve
  role: Automated batch-mode determination of dynamic linear ranges and quantification
    limits; generates calibration curves and response factor plots
  repo: https://github.com/mpc-bioinformatics/CalibraCurve
- name: R
  role: Runtime environment and language in which CalibraCurve is implemented; used
    to fit weighted/unweighted linear models and render ggplot2 figures
- name: ggplot2
  role: Renders the calibration curve and response factor plot PNGs with threshold
    lines
- name: CalibraCurve_NF
  role: Nextflow workflow wrapping the CalibraCurve R package for batch execution
  repo: https://github.com/mpc-bioinformatics/CalibraCurve_NF
- name: msqc1
  role: Reference dataset providing calibration data for targeted MS analytes
provenance:
  source_task_ids:
  - task_005
  - task_007
  source_papers:
  - doi: 10.1002/pmic.201900143
    title: 'CalibraCurve: A Tool for Calibration of Targeted MS‐Based Measurements'
  source_content_hash: sha256:49db358a64e98cb563c66d153789850919730d129f51213d3da39279863a8f82
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mpc-bioinformatics/CalibraCurve
  tools:
  - CalibraCurve
  - CalibraCurve_NF
  - R
  - ggplot2
  - msqc1
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# targeted-proteomics-linear-range-output

## Summary

Determines the dynamic linear range (lower and upper limits of quantification) of targeted MS-based (SRM/MRM, PRM) assays from dilution-series calibration data using CalibraCurve, and emits fitted weighted/unweighted calibration models plus calibration curve and response factor plot outputs for assessing precision and trueness.

## When to use

Use when targeted MS (SRM/MRM or PRM) calibration data from concentration or dilution series with known analyte amounts must yield an automated linear range (LLOQ/ULOQ) and fitted calibration curves for downstream concentration prediction.

## When NOT to use

- Data are from classical shotgun (DDA) proteomics rather than targeted SRM/MRM or PRM assays, where the calibration-based linear range concept does not apply the same way
- The calibration relationship must hold across the full concentration span with no linear range restriction — CalibraCurve explicitly models the region where linearity breaks down (e.g. reduced product ion intensities at higher analyte concentrations)
- Raw MS data inspection is the goal (use Skyline); CalibraCurve operates on curve-fitting/calibration data, not raw spectra

## Inputs

- Replicate measurements of a concentration/dilution series (known analyte amounts, e.g. msqc1 dataset intensities per concentration level)
- Fitted weighted/unweighted calibration models and determined linear range from a CalibraCurve run

## Outputs

- Determined dynamic linear range with lower and upper limits of quantification (LLOQ/ULOQ)
- Weighted and unweighted linear model summaries (coefficients, R-squared, residuals) as tabular result files
- MeasCV precision (CV) tables per concentration level and replicate
- PB, PB_av, and PB_cv back-calculated precision/trueness tables
- Calibration curve PNG and response factor plot PNG with 80%/120% RF_mean_total threshold lines

## How to apply

Load the replicate measurements from the dilution series (e.g. from the msqc1 reference dataset) into R and run CalibraCurve, which uses precision and trueness measures to assess calibration accuracy and computes the linear range via its dedicated algorithm. Fit weighted and unweighted linear models (retaining coefficients, R-squared, and residuals) and record the determined lower and upper limits of quantification, since SRM/MRM dynamic linear ranges can span 4-5 orders of magnitude. Generate the two customizable graph types CalibraCurve creates: a calibration curve figure and a response factor plot with horizontal threshold lines at 80% and 120% of RF_mean_total marking the acceptable response-factor band. Write tabular result files (MeasCV precision/CV tables, PB back-calculated precision/trueness tables, linear model summaries) alongside the PNG figures. Validate that all result CSVs and both PNGs parse correctly and that the response factor plot encodes the 80%/120% thresholds.

## Related tools

- **CalibraCurve** (Automated batch-mode determination of dynamic linear ranges and quantification limits; generates calibration curves and response factor plots) — https://github.com/mpc-bioinformatics/CalibraCurve
- **R** (Runtime environment and language in which CalibraCurve is implemented; used to fit weighted/unweighted linear models and render ggplot2 figures)
- **ggplot2** (Renders the calibration curve and response factor plot PNGs with threshold lines)
- **CalibraCurve_NF** (Nextflow workflow wrapping the CalibraCurve R package for batch execution) — https://github.com/mpc-bioinformatics/CalibraCurve_NF
- **msqc1** (Reference dataset providing calibration data for targeted MS analytes)

## Examples

```
BiocManager::install("CalibraCurve"); library(CalibraCurve)
```

## Evaluation signals

- All result CSVs (MeasCV, PB, PB_av, PB_cv, linear model summaries) and both PNG figures exist and parse correctly
- The response factor plot encodes horizontal threshold lines at exactly 80% and 120% of RF_mean_total
- Determined linear range limits (LLOQ/ULOQ) are consistent with the reported 4-5 orders of magnitude span for SRM/MRM dynamic linear ranges
- Predicted concentrations from the fitted model are flagged with an out-of-linear-range warning when the queried intensity exceeds the ULOQ (e.g. intensity 1e8)

## Limitations

- A linear relationship between concentration and intensity is often limited to a certain range; predictions outside the linear range are flagged as potentially unreliable rather than corrected
- Product ion intensity reduction was observed at higher analyte concentrations, which is why ULOQ determination is needed
- The former KNIME-workflow distribution was discontinued and replaced by the R package and nextflow workflow

## Evidence

- [abstract] a software named CalibraCurve that enables an automated batch-mode determination of dynamic linear ranges and quantification limits: "a software named CalibraCurve that enables an automated batch-mode determination of dynamic linear ranges and quantification limits"
- [abstract] Two different kinds of customizable graphs are created (calibration curves and: "The software uses a variety of measures to assess the accuracy of the calibration, namely precision and trueness. Two different kinds of customizable graphs are created (calibration curves and"
- [abstract] dynamic linear ranges that span up to 4-5 orders of magnitude: "dynamic linear ranges that span up to 4-5 orders of magnitude"
- [readme] CalibraCurve calculates the linear range (lower and upper limits of quantification) using a well-designed algorithm.: [evidence span withheld — anchor status: external_doc]
- [readme] This kind of data is used by CalibraCurve to generate calibration curves, which can be the basis for predicting concentrations from intensities in new data.: [evidence span withheld — anchor status: external_doc]
