---
name: precision-trueness-assessment
description: Use when evaluating linear calibration curves from MRM/PRM dilution series
  (e.g. transition-level intensities for a SIS peptide such as WTVFQK/MFAP4 from PASSEL
  PASS00653) and deciding whether a weighting scheme improves fit quality.
when_to_use_negative:
- The goal is ordinary concentration prediction from an already-validated assay rather
  than judging calibration fit quality or choosing a weighting scheme
- Input data are global (shotgun) proteomics results rather than targeted SRM/MRM
  or PRM dilution-series measurements
- Only a single regression metric such as R-squared is requested with no trueness
  assessment at low concentration levels — this skill exists precisely because R-squared
  alone is misleading
edam_operation: http://edamontology.org/operation_3630
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_2269
tools:
- name: CalibraCurve
  role: R/Bioconductor package that automates determination of dynamic linear ranges
    and quantification limits and assesses calibration accuracy via precision and
    trueness measures (PB_av) with calibration curve plots
  repo: https://github.com/mpc-bioinformatics/CalibraCurve
- name: R
  role: Statistical environment in which CalibraCurve runs the linear regressions
    (unweighted and 1/x^2-weighted) and from which metrics are extracted
  repo: https://www.R-project.org/
- name: CalibraCurve_NF
  role: Nextflow workflow wrapping the CalibraCurve R package for batch execution
    of the calibration assessment
  repo: https://github.com/mpc-bioinformatics/CalibraCurve_NF
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/pmic.201900143
    title: 'CalibraCurve: A Tool for Calibration of Targeted MS‐Based Measurements'
  source_content_hash: sha256:3afe621053e9eb4acaed9ec0599d45427add7e5c756fe3b0ff737df843f868b6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mpc-bioinformatics/CalibraCurve
  tools:
  - CalibraCurve
  - CalibraCurve_NF
  - R
  - ggplot2
  edam_operation: http://edamontology.org/operation_3630
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# precision-trueness-assessment

## Summary

Assess targeted MS (SRM/MRM) calibration quality by computing R-squared and the precision/trueness-based PB_av metric for unweighted and 1/x^2-weighted linear calibration models on the final linear range, using CalibraCurve, because R-squared alone is misleading when comparing calibration fits.

## When to use

Use when evaluating linear calibration curves from MRM/PRM dilution series (e.g. transition-level intensities for a SIS peptide such as WTVFQK/MFAP4 from PASSEL PASS00653) and deciding whether a weighting scheme improves fit quality.

## When NOT to use

- The goal is ordinary concentration prediction from an already-validated assay rather than judging calibration fit quality or choosing a weighting scheme
- Input data are global (shotgun) proteomics results rather than targeted SRM/MRM or PRM dilution-series measurements
- Only a single regression metric such as R-squared is requested with no trueness assessment at low concentration levels — this skill exists precisely because R-squared alone is misleading

## Inputs

- MRM/PRM dilution-series transition-level intensities for SIS peptides (e.g. Agilent 6490 triple quadrupole data from PASSEL PASS00653)
- CalibraCurve-compatible input table of concentration vs intensity measurements (replicates per calibration level)
- R environment with the CalibraCurve package installed

## Outputs

- Final linear range (LLOQ and ULOQ) per peptide/assay
- R-squared values for unweighted and 1/x^2-weighted linear calibration models
- PB_av (averaged precision-based accuracy/trueness) values overall and per calibration level
- Calibration curves and response factor plots; summary table (e.g. calibration_metrics.csv)

## How to apply

Format transition-level intensities of the dilution series as a CalibraCurve-compatible input table, then run CalibraCurve (R/Bioconductor) to automatically determine the final linear range (LLOQ/ULOQ) and fit an unweighted linear model, recording R-squared and PB_av. Re-run on the same linear range with 1/x^2 weighting enabled in the linear regression to obtain weighted R-squared and PB_av values. Extract PB_av at the lowest calibration level (e.g. 73.75% unweighted vs 14.65% weighted for WTVFQK) and at a mid-low level such as 0.25 fmol (72.4% vs 44.6%). Compare metrics: the unweighted model may attain the higher R-squared (~0.994 vs ~0.932 weighted) yet have substantially worse trueness at the lowest levels, so the decision must be based on PB_av in addition to R-squared, and the weighted model is preferred when it restores trueness at low concentrations.

## Related tools

- **CalibraCurve** (R/Bioconductor package that automates determination of dynamic linear ranges and quantification limits and assesses calibration accuracy via precision and trueness measures (PB_av) with calibration curve plots) — https://github.com/mpc-bioinformatics/CalibraCurve
- **R** (Statistical environment in which CalibraCurve runs the linear regressions (unweighted and 1/x^2-weighted) and from which metrics are extracted) — https://www.R-project.org/
- **CalibraCurve_NF** (Nextflow workflow wrapping the CalibraCurve R package for batch execution of the calibration assessment) — https://github.com/mpc-bioinformatics/CalibraCurve_NF

## Examples

```
if (!requireNamespace("BiocManager", quietly=TRUE)) install.packages("BiocManager"); BiocManager::install("CalibraCurve")
```

## Evaluation signals

- R-squared values fall near the reported range (~0.994 unweighted, ~0.932 weighted for WTVFQK/MFAP4); a much higher unweighted R-squared with worse trueness is the expected pattern, not an error
- PB_av at the lowest calibration level reproduces the reported contrast (73.75% unweighted vs 14.65% weighted), and 0.25 fmol values match (72.4% vs 44.6%)
- Both weighting schemes are evaluated on the identical final linear range determined by CalibraCurve, so LLOQ/ULOQ agree across runs
- Output summary (e.g. calibration_metrics.csv) contains both R-squared and PB_av per weighting scheme; absence of a trueness measure signals the workflow was applied incorrectly
- Calibration curves and response factor plots are produced for each peptide and are visually inspectable for accuracy at the lowest levels

## Limitations

- Considering R-squared alone is misleading for comparing calibration performances because it does not reflect trueness at the lowest concentration levels
- Linear relationship between concentration and intensity is limited to a certain range; metrics are only valid within the CalibraCurve-determined linear range
- No discussion-section limitations were extractable from the provided document text (missing changelog/limitations content)

## Evidence

- [abstract] a software named CalibraCurve that enables an automated batch-mode determination of dynamic linear ranges and quantification limits: "a software named CalibraCurve that enables an automated batch-mode determination of dynamic linear ranges and quantification limits"
- [abstract] The software uses a variety of measures to assess the accuracy of the calibration, namely precision and trueness.: "The software uses a variety of measures to assess the accuracy of the calibration, namely precision and trueness."
- [intro] in order to realize the full potential of targeted proteomics methods an impeccable statistical assessment: [evidence span withheld — anchor status: not_found]
- [readme] Quality of the data and the accuracy of the produced calibration curve is assessed in several computational steps.: [evidence span withheld — anchor status: external_doc]
- [readme] CalibraCurve calculates the linear range (lower and upper limits of quantification) using a well-designed algorithm.: [evidence span withheld — anchor status: external_doc]
