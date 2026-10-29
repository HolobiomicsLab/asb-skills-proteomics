---
name: weighted-least-squares-calibration-fitting
description: Use when building calibration curves from targeted proteomics dilution
  series (e.g. Agilent 6490 MRM data in PASSEL) and comparing unweighted vs. weighted
  (1/x^2) linear fits on the final linear range.
when_to_use_negative:
- When you only need to select a model by goodness-of-fit for data spanning the full
  dynamic range without regard to low-concentration trueness — R-squared alone will
  mislead, as shown by the WTVFQK example
- When raw vendor files have not yet been inspected/normalized (e.g. with Skyline)
  and formatted into the CalibraCurve input table
- When the analyte's response is not yet known to be linear over any subrange — the
  linear range must first be determined by CalibraCurve's algorithm before fitting
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: CalibraCurve
  role: R package used to determine the dynamic linear range (LLOQ/ULOQ), fit unweighted
    and 1/x^2-weighted linear calibration models, and compute R-squared and PB_av
  repo: https://github.com/mpc-bioinformatics/CalibraCurve
- name: R
  role: Statistical environment in which CalibraCurve is implemented and the linear
    regression (weighted/unweighted) is executed
  repo: https://www.R-project.org/
- name: Skyline
  role: Used to inspect raw MRM data prior to calibration fitting
- name: CalibraCurve_NF
  role: Nextflow workflow wrapping the CalibraCurve R package for batch execution
  repo: https://github.com/mpc-bioinformatics/CalibraCurve_NF
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/pmic.201900143
    title: 'CalibraCurve: A Tool for Calibration of Targeted MS‐Based Measurements'
  source_content_hash: sha256:afaf766bf219f3f14c2be48ac6fa0e24784d351ae5abb1152f3b151a0ef3d4e9
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
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# weighted-least-squares-calibration-fitting

## Summary

Fits linear calibration models to targeted MS (SRM/MRM/PRM) dilution-series data with and without 1/x^2 weighting, and judges fit quality using trueness (PB_av) in addition to R-squared, since R-squared alone is misleading for comparing calibration performance.

## When to use

Use when building calibration curves from targeted proteomics dilution series (e.g. Agilent 6490 MRM data in PASSEL) and comparing unweighted vs. weighted (1/x^2) linear fits on the final linear range.

## When NOT to use

- When you only need to select a model by goodness-of-fit for data spanning the full dynamic range without regard to low-concentration trueness — R-squared alone will mislead, as shown by the WTVFQK example
- When raw vendor files have not yet been inspected/normalized (e.g. with Skyline) and formatted into the CalibraCurve input table
- When the analyte's response is not yet known to be linear over any subrange — the linear range must first be determined by CalibraCurve's algorithm before fitting

## Inputs

- MRM/PRM dilution-series measurements (transition-level peak-area intensities), e.g. Agilent 6490 triple quadrupole data from PASSEL accession PASS00653
- CalibraCurve-compatible input table mapping concentration levels to replicate intensities for a target peptide (e.g. WTVFQK / MFAP4)

## Outputs

- Determined linear range (LLOQ and ULOQ) per analyte
- R-squared values for unweighted and 1/x^2-weighted linear calibration models
- PB_av (precision-based accuracy / trueness averaged over levels) values per weighting scheme and per concentration level
- Calibration curves and response factor plots; summary table (e.g. calibration_metrics.csv)

## How to apply

Format the transition-level intensities (e.g. the SIS peptide WTVFQK from MFAP4, PASSEL PASS00653) as a CalibraCurve-compatible input table. Run CalibraCurve (R package) to automatically determine the final linear range, i.e. the LLOQ and ULOQ, and fit an unweighted linear regression, recording R-squared and the PB_av trueness measure. Re-run the same linear range with 1/x^2 weighting enabled to obtain weighted R-squared and PB_av values, including PB_av at the lowest calibration level and at the 0.25 fmol level. Compare both weighting schemes on both metrics: in the MFAP4 example the unweighted model attained the higher R-squared (~0.994 vs ~0.932) yet showed substantially worse trueness at low levels (PB_av 73.75% vs 14.65% at the lowest level; 72.4% vs 44.6% at 0.25 fmol), demonstrating that R-squared alone is not a reliable comparison criterion. Rationale: the weighted fit compensates for heteroscedasticity at low concentrations, where absolute errors matter most for quantification, and CalibraCurve's precision- and trueness-based assessment exposes this better than R-squared.

## Related tools

- **CalibraCurve** (R package used to determine the dynamic linear range (LLOQ/ULOQ), fit unweighted and 1/x^2-weighted linear calibration models, and compute R-squared and PB_av) — https://github.com/mpc-bioinformatics/CalibraCurve
- **R** (Statistical environment in which CalibraCurve is implemented and the linear regression (weighted/unweighted) is executed) — https://www.R-project.org/
- **Skyline** (Used to inspect raw MRM data prior to calibration fitting)
- **CalibraCurve_NF** (Nextflow workflow wrapping the CalibraCurve R package for batch execution) — https://github.com/mpc-bioinformatics/CalibraCurve_NF

## Examples

```
BiocManager::install("CalibraCurve"); library(CalibraCurve)  # then run CalibraCurve on the WTVFQK dilution table twice: unweighted and with 1/x^2 weighting
```

## Evaluation signals

- Reported R-squared values should match the reference values (~0.994 unweighted vs ~0.932 with 1/x^2 weighting) on the WTVFQK/MFAP4 final linear range
- PB_av at the lowest calibration level should improve markedly with weighting: ~73.75% (unweighted) vs ~14.65% (1/x^2)
- PB_av at 0.25 fmol should be ~72.4% (unweighted) vs ~44.6% (1/x^2)
- Both weighting schemes must be fit on the identical linear range determined by CalibraCurve so the comparison is fair
- CalibraCurve output should include the required accuracy measures (precision and trueness) and the two plot types (calibration curves and response factor plots)

## Limitations

- Considering R-squared alone is misleading for comparing calibration performances; the unweighted model attains the higher R-squared while its trueness at the lowest concentrations is substantially worse
- The linear relationship between concentration and intensity is limited to a certain range, so fits must be restricted to the calculated linear range (LLOQ–ULOQ); product ion intensity reduction occurs at higher analyte concentrations
- Development of the former KNIME workflow was discontinued and replaced by the R package and Nextflow workflow; the software is intended for academic research and must not be used for clinical decision-making without appropriate validation

## Evidence

- [abstract] a software named CalibraCurve that enables an automated batch-mode determination of dynamic linear ranges and quantification limits: "a software named CalibraCurve that enables an automated batch-mode determination of dynamic linear ranges and quantification limits"
- [abstract] The software uses a variety of measures to assess the accuracy of the calibration, namely precision and trueness.: "The software uses a variety of measures to assess the accuracy of the calibration, namely precision and trueness."
- [results] considering R² alone is misleading for comparison of calibration performances: the unweighted model attains the higher R², but the model fits must additionally be judged by the PB_av trueness measure: [evidence span withheld — anchor status: not_found]
- [intro] Raw data was inspected with the Skyline software, followingnormalizationandcurvefittinginR.: "Raw data was inspected with the Skyline software, followingnormalizationandcurvefittinginR."
- [readme] However, a linear relationship between concentration and intensity is often limited to a certain range of concentrations, the so called linear range.: [evidence span withheld — anchor status: external_doc]
- [readme] CalibraCurve calculates the linear range (lower and upper limits of quantification) using a well-designed algorithm.: [evidence span withheld — anchor status: external_doc]
