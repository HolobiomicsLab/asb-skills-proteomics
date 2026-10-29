---
name: response-factor-threshold-plotting
description: Use when targeted MS (SRM/MRM or PRM) calibration data has been fitted
  (weighted/unweighted linear models, linear range determined) and a visual check
  of response-factor behavior across concentration levels is needed.
when_to_use_negative:
- Input is raw Skyline-exported transition intensities that have not yet been fitted
  to weighted/unweighted calibration models — determine the linear range first
- Data are shotgun/global proteomics intensity distributions rather than concentration-series
  calibration measurements from targeted assays
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: CalibraCurve
  role: Produces the fitted calibration models, linear range, and accuracy measures
    (precision and trueness) from which the response factor plot is derived
  repo: https://github.com/mpc-bioinformatics/CalibraCurve
- name: R
  role: Runtime environment for the CalibraCurve package; used to compute response
    factors and render the plot
  repo: https://www.R-project.org/
- name: ggplot2
  role: Graphics package used to render the response factor plot with horizontal threshold
    lines and save it as PNG
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/pmic.201900143
    title: 'CalibraCurve: A Tool for Calibration of Targeted MS‐Based Measurements'
  source_content_hash: sha256:f1f19e8cf8df603984f39d2d6180c393587754d7530b87ef66435d481b889f2c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mpc-bioinformatics/CalibraCurve
  tools:
  - CalibraCurve
  - R
  - ggplot2
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# response-factor-threshold-plotting

## Summary

Render a response factor plot from fitted CalibraCurve calibration data, overlaying horizontal threshold lines that define the acceptable response-factor band, to visually support assessment of calibration accuracy alongside the calibration curve.

## When to use

Use when targeted MS (SRM/MRM or PRM) calibration data has been fitted (weighted/unweighted linear models, linear range determined) and a visual check of response-factor behavior across concentration levels is needed.

## When NOT to use

- Input is raw Skyline-exported transition intensities that have not yet been fitted to weighted/unweighted calibration models — determine the linear range first
- Data are shotgun/global proteomics intensity distributions rather than concentration-series calibration measurements from targeted assays

## Inputs

- Fitted weighted and unweighted calibration models from CalibraCurve (R linear model objects)
- Determined linear range (lower and upper limits of quantification) from CalibraCurve
- Calibration measurement data from concentration or dilution series (e.g. replicate intensities per concentration level, as in the msqc1 dataset)

## Outputs

- Response factor plot (PNG) with horizontal threshold lines at 80% and 120% of RF_mean_total
- Calibration curve figure (PNG) rendered from the fitted models and linear range

## How to apply

Load the upstream CalibraCurve results — the fitted weighted and unweighted calibration models and the determined linear range (LLOQ/ULOQ) — in R. Compute the response factor (intensity ratio per concentration level) from the calibration data, then render the response factor plot with ggplot2, drawing horizontal threshold lines at 80% and 120% of RF_mean_total to mark the acceptable response-factor band. Levels whose response factors fall outside this band indicate concentrations at which the linear relationship breaks down, consistent with CalibraCurve's algorithm for determining the linear range. Save the figure as a PNG alongside the calibration curve figure, both of which are the two kinds of customizable graphs CalibraCurve is reported to create.

## Related tools

- **CalibraCurve** (Produces the fitted calibration models, linear range, and accuracy measures (precision and trueness) from which the response factor plot is derived) — https://github.com/mpc-bioinformatics/CalibraCurve
- **R** (Runtime environment for the CalibraCurve package; used to compute response factors and render the plot) — https://www.R-project.org/
- **ggplot2** (Graphics package used to render the response factor plot with horizontal threshold lines and save it as PNG)

## Evaluation signals

- Both PNG figures (calibration curve and response factor plot) exist and parse as valid images
- The response factor plot encodes horizontal threshold lines at 80% and 120% of RF_mean_total
- Response factors plotted per concentration level are consistent with the fitted calibration models and the reported linear range
- Levels outside the 80–120% band correspond to concentrations excluded from the determined linear range (LLOQ–ULOQ)

## Limitations

- The linear relationship between concentration and intensity is often limited to a certain concentration range, so the plot is only meaningful within/near the linear range
- Reduction of product ion intensities was observed at higher analyte concentrations, so response factors above the upper threshold signal the need for ULOQ determination

## Evidence

- [abstract] Two different kinds of customizable graphs are created (calibration curves and: "Two different kinds of customizable graphs are created (calibration curves and"
- [readme] readme: customizable visualizations: [evidence span withheld — anchor status: external_doc]
- [intro] intro: high-concentration signal reduction: "we observed reduction of product ion intensities at higher analyte concentrations (Figure 1)"
