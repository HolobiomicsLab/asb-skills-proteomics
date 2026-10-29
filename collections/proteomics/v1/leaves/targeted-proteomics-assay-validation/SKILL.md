---
name: targeted-proteomics-assay-validation
description: Use when a dilution/concentration series of replicate measurements (e.g.,
  transition-level intensities from an Agilent 6490 MRM run or msqc1 data) must be
  converted into a validated calibration curve with a defensible linear range and
  quantification limits.
when_to_use_negative:
- The data are not replicate measurements over a concentration/dilution series (e.g.,
  a single concentration or a global (shotgun) proteomics experiment), since no linear
  range or PB metrics can be determined
- You only need to report R² of a single fit without assessing trueness — the article
  explicitly shows R² alone is misleading for comparing unweighted vs weighted calibration
  models
- 'You require the legacy KNIME workflow: development of the KNIME-based version was
  discontinued and replaced by the R package and Nextflow workflow'
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: CalibraCurve
  role: R package that automates batch-mode determination of dynamic linear ranges
    and quantification limits, fits weighted/unweighted calibration models, and computes
    PB_av/PB_cv accuracy measures
  repo: https://github.com/mpc-bioinformatics/CalibraCurve
- name: CalibraCurve_NF
  role: Nextflow workflow wrapping the CalibraCurve R package for batch execution
    of the assay-validation analysis
  repo: https://github.com/mpc-bioinformatics/CalibraCurve_NF
- name: Skyline
  role: Used to inspect raw targeted-MS data prior to normalization and curve fitting
    in R
- name: R
  role: Runtime environment for the CalibraCurve package (linear model fitting, PB
    computation, plotting)
- name: msqc1
  role: R package/citation providing reference calibration datasets for targeted MS
    assay evaluation
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1002/pmic.201900143
    title: 'CalibraCurve: A Tool for Calibration of Targeted MS‐Based Measurements'
  source_content_hash: sha256:2faba0bbbe8b6063368d14224e803742dc1c6ccda0c8689d99d9ab2ef099839c
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# targeted-proteomics-assay-validation

## Summary

Validate targeted MS (SRM/MRM/PRM) quantitative assays by fitting weighted and unweighted calibration curves over dilution series with CalibraCurve, determining the final linear range (LLOQ/ULOQ) via PB_av/PB_cv thresholds, and judging fits by precision- and trueness-based measures rather than R² alone.

## When to use

Use when a dilution/concentration series of replicate measurements (e.g., transition-level intensities from an Agilent 6490 MRM run or msqc1 data) must be converted into a validated calibration curve with a defensible linear range and quantification limits.

## When NOT to use

- The data are not replicate measurements over a concentration/dilution series (e.g., a single concentration or a global (shotgun) proteomics experiment), since no linear range or PB metrics can be determined
- You only need to report R² of a single fit without assessing trueness — the article explicitly shows R² alone is misleading for comparing unweighted vs weighted calibration models
- You require the legacy KNIME workflow: development of the KNIME-based version was discontinued and replaced by the R package and Nextflow workflow

## Inputs

- Dilution/concentration series with replicate measurements per level (e.g., transition-level intensities from Agilent 6490 MRM, PASSEL PASS00653), formatted as a CalibraCurve-compatible input table
- Targeted MS raw data inspected/normalized via Skyline before curve fitting in R
- Reference datasets such as msqc1

## Outputs

- Final linear range (Cl, Ch) i.e., lower and upper limits of quantification, plus ordered log of removed levels (e.g., linear_range.csv)
- Per-level PB_av and PB_cv metrics and fitted R² for unweighted and 1/x²-weighted models (e.g., calibration_metrics.csv)
- Customizable calibration curve and response factor plots

## How to apply

Format the replicate intensities per concentration level into a CalibraCurve-compatible input table, then run the CalibraCurve R package (or its Nextflow workflow) to fit unweighted and 1/x^2-weighted linear models of response versus concentration. CalibraCurve computes precision-based bias (PB) per measurement from the residuals, aggregates it per level into PB_av (mean bias) and PB_cv (bias variability), and iteratively removes boundary levels whose PB_av exceeds the threshold TPB_av until the lowest (Cl) and highest (Ch) retained levels both satisfy TPB_av — this defines the final linear range and the quantification limits. Always compare both weighting schemes on the final range using R² AND PB_av: the MFAP4 WTVFQK example shows the unweighted model attains the higher R² (~0.994 vs ~0.932) yet far worse trueness at low levels (PB_av 73.75% vs 14.65% at the lowest level; 72.4% vs 44.6% at 0.25 fmol), so R² alone is misleading. Output the final Cl/Ch, removed-level log, calibration curves, and response factor plots, and check that boundary PB_av values are below TPB_av before accepting the result.

## Related tools

- **CalibraCurve** (R package that automates batch-mode determination of dynamic linear ranges and quantification limits, fits weighted/unweighted calibration models, and computes PB_av/PB_cv accuracy measures) — https://github.com/mpc-bioinformatics/CalibraCurve
- **CalibraCurve_NF** (Nextflow workflow wrapping the CalibraCurve R package for batch execution of the assay-validation analysis) — https://github.com/mpc-bioinformatics/CalibraCurve_NF
- **Skyline** (Used to inspect raw targeted-MS data prior to normalization and curve fitting in R)
- **R** (Runtime environment for the CalibraCurve package (linear model fitting, PB computation, plotting))
- **msqc1** (R package/citation providing reference calibration datasets for targeted MS assay evaluation)

## Examples

```
if (!requireNamespace("BiocManager", quietly=TRUE)) install.packages("BiocManager"); BiocManager::install("CalibraCurve")
```

## Evaluation signals

- Acceptance criterion: PB_av of both boundary levels (lowest Cl and highest Ch) is below TPB_av in the final iteration, and linear_range.csv contains the final Cl, Ch and the removed-level log
- Reproduction of the MFAP4 WTVFQK (PASS00653) reference metrics: R² ~0.994 unweighted vs ~0.932 (1/x²-weighted); PB_av 73.75% vs 14.65% at the lowest level and 72.4% vs 44.6% at 0.25 fmol
- Both weighting schemes are reported on the same final linear range; the weighted model is NOT rejected merely because its R² is lower
- Calibration curve and response factor plots are generated and match the input concentration–intensity table
- The fit was run on the final range only (boundary levels with PB_av > TPB_av were removed and refit iteratively)

## Limitations

- R² alone is an unreliable criterion for comparing unweighted vs 1/x²-weighted calibration fits; PB_av trueness must be checked, especially at the lowest concentration levels
- Product ion intensities can be reduced at higher analyte concentrations, so the upper end of the curve must be assessed to determine the ULOQ
- The former KNIME-workflow distribution of CalibraCurve is discontinued; use the R package (Bioconductor) or the Nextflow workflow instead

## Evidence

- [abstract] a software named CalibraCurve that enables an automated batch-mode determination of dynamic linear ranges and quantification limits: "a software named CalibraCurve that enables an automated batch-mode determination of dynamic linear ranges and quantification limits"
- [abstract] The software uses a variety of measures to assess the accuracy of the calibration, namely precision and trueness.: "The software uses a variety of measures to assess the accuracy of the calibration, namely precision and trueness."
- [intro] we observed reduction of product ion intensities at higher analyte concentrations (Figure 1): "we observed reduction of product ion intensities at higher analyte concentrations (Figure 1)"
- [methods] msqc1 = citation("msqc1")[1]: "msqc1 = citation("msqc1")[1]"
- [readme] CalibraCurve calculates the linear range (lower and upper limits of quantification) using a well-designed algorithm.: [evidence span withheld — anchor status: external_doc]
- [intro] Raw data was inspected with the Skyline software, followingnormalizationandcurvefittinginR.: "Raw data was inspected with the Skyline software, followingnormalizationandcurvefittinginR."
