---
name: calibration-curve-visualization
description: Use when you have fitted calibration models (weighted/unweighted) and
  a determined linear range from a targeted mass-spectrometry concentration or dilution
  series and need to assess calibration accuracy visually.
when_to_use_negative:
- Input is raw, unprocessed transition-level intensities that have not yet been curve-fitted
  or assigned a linear range by CalibraCurve
- The goal is de novo determination of dynamic linear ranges or quantification limits
  rather than visualization of already-fitted calibration models
- Data are from classical shotgun (DDA) proteomics rather than targeted SRM/MRM or
  PRM calibration experiments
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: CalibraCurve
  role: R package that fits the calibration models, determines the linear range, and
    creates the calibration curves and response factor plots
  repo: github.com/mpc-bioinformatics/CalibraCurve
- name: R
  role: Runtime environment in which the CalibraCurve package renders the figures
    and writes result files
  repo: https://www.R-project.org/
- name: ggplot2
  role: Graphics layer used to render the calibration curve and response factor plot
    PNGs
- name: CalibraCurve_NF
  role: Nextflow workflow that runs the CalibraCurve pipeline producing the visual
    outputs
  repo: mpc-bioinformatics/CalibraCurve_NF
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/pmic.201900143
    title: 'CalibraCurve: A Tool for Calibration of Targeted MS‐Based Measurements'
  source_content_hash: sha256:b43ebe42411204ea85aa516d76fd3b1e765625e4f6bc442c0efddbd275a9bca9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CalibraCurve
  - CalibraCurve_NF
  - R
  - ggplot2
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# calibration-curve-visualization

## Summary

Renders the two kinds of customizable graphs produced by the CalibraCurve workflow for targeted MS-based (SRM/MRM, PRM) assays: calibration curves from fitted weighted/unweighted linear models and response factor plots marking the acceptable RF band.

## When to use

Use when you have fitted calibration models (weighted/unweighted) and a determined linear range from a targeted mass-spectrometry concentration or dilution series and need to assess calibration accuracy visually.

## When NOT to use

- Input is raw, unprocessed transition-level intensities that have not yet been curve-fitted or assigned a linear range by CalibraCurve
- The goal is de novo determination of dynamic linear ranges or quantification limits rather than visualization of already-fitted calibration models
- Data are from classical shotgun (DDA) proteomics rather than targeted SRM/MRM or PRM calibration experiments

## Inputs

- Fitted weighted and unweighted calibration linear models from CalibraCurve
- Determined linear range (lower and upper limits of quantification, LLOQ/ULOQ)
- Concentration/dilution series intensity measurements from targeted MS (e.g. PRM on Q Exactive)
- Response factor summary (RF_mean_total) for threshold computation

## Outputs

- Calibration curve figure (PNG)
- Response factor plot (PNG) with 80% and 120% RF_mean_total threshold lines
- Accompanying tabular result files (MeasCV, PB, PB_av, PB_cv, linear model summaries)

## How to apply

Load the upstream CalibraCurve results in R — the fitted weighted/unweighted calibration models and the determined linear range (LLOQ/ULOQ) — then render the calibration curve figure with ggplot2, overlaying measured points and the fitted model restricted to the linear range. Next, render the response factor plot, drawing horizontal threshold lines at 80% and 120% of RF_mean_total to delimit the acceptable response-factor band; points outside this band indicate concentrations where back-calculated response deviates from trueness. Save both figures as PNG result files alongside the tabular outputs (MeasCV precision table, PB/PB_av/PB_cv tables, and linear model summaries with coefficients, R-squared, residuals). CalibraCurve automates batch-mode determination of dynamic linear ranges and quantification limits, and these visualizations support judging precision and trueness of the calibration. Validate that both PNGs exist, parse correctly, and that the response factor plot encodes the 80%/120% RF_mean_total thresholds.

## Related tools

- **CalibraCurve** (R package that fits the calibration models, determines the linear range, and creates the calibration curves and response factor plots) — github.com/mpc-bioinformatics/CalibraCurve
- **R** (Runtime environment in which the CalibraCurve package renders the figures and writes result files) — https://www.R-project.org/
- **ggplot2** (Graphics layer used to render the calibration curve and response factor plot PNGs)
- **CalibraCurve_NF** (Nextflow workflow that runs the CalibraCurve pipeline producing the visual outputs) — mpc-bioinformatics/CalibraCurve_NF

## Examples

```
devtools::install_github("mpc-bioinformatics/CalibraCurve"); library(CalibraCurve)
```

## Evaluation signals

- Both PNG figures (calibration curve and response factor plot) exist and parse correctly as images
- Response factor plot encodes horizontal threshold lines at 80% and 120% of RF_mean_total
- Calibration curve restricts the fitted model to the determined linear range (LLOQ–ULOQ)
- Result CSVs (MeasCV, PB, PB_av, PB_cv, model summaries) parse and align with the plotted levels
- Plotted replicate points match the back-calculated concentrations in the PB table

## Limitations

- Development of the former KNIME-workflow version of CalibraCurve was discontinued and replaced by the R package and Nextflow workflow
- The linear relationship between concentration and intensity is often limited to a certain concentration range, so plots are only meaningful relative to the determined linear range
- No discussion-section limitations were extractable from the provided article text

## Evidence

- [abstract] Two different kinds of customizable graphs are created (calibration curves and: "The software uses a variety of measures to assess the accuracy of the calibration, namely precision and trueness. Two different kinds of customizable graphs are created (calibration curves and"
- [readme] Visualizations of the calibration curves and the underlying data basis are generated and can be customized.: [evidence span withheld — anchor status: external_doc]
- [readme] a linear relationship between concentration and intensity is often limited to a certain range of concentrations, the so called linear range: [evidence span withheld — anchor status: external_doc]
- [abstract] enables an automated batch-mode determination of dynamic linear ranges and quantification limits: "a software named CalibraCurve that enables an automated batch-mode determination of dynamic linear ranges and quantification limits"
