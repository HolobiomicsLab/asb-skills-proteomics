---
name: calibration-model-summary-reporting
description: Use when a targeted proteomics (SRM/MRM or PRM) calibration experiment
  with concentration/dilution series data needs linear-range determination, precision/trueness
  result tables, and calibration-curve plus response-factor figures.
when_to_use_negative:
- The task is raw-data inspection or normalization/curve fitting of uncalibrated spectra
  (e.g. Skyline inspection) rather than reporting on already-fitted calibration models
- The data are global/shotgun proteomics measurements without a concentration or dilution
  series, so no linear-range or response-factor assessment applies
- Only the dynamic-range determination is needed without any result tables or customizable
  graph generation
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2269
tools:
- name: CalibraCurve
  role: R package that determines the linear range and provides the result-file writer
    and visualization backend for the calibration tables and the two customizable
    graph types
  repo: https://github.com/mpc-bioinformatics/CalibraCurve
- name: R
  role: Runtime environment in which CalibraCurve is installed and the result files
    and figures are generated
  repo: https://www.R-project.org/
- name: ggplot2
  role: Used to render the calibration curve and response factor plot figures as PNGs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/pmic.201900143
    title: 'CalibraCurve: A Tool for Calibration of Targeted MS‐Based Measurements'
  source_content_hash: sha256:cbb071d257e14e874322a60ca7139011ce84427d7c67444058fa1f38f5e27b24
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
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# calibration-model-summary-reporting

## Summary

Reconstructs CalibraCurve-style result-file generation for targeted MS calibration: writing MeasCV precision (CV) tables, PB/PB_av/PB_cv trueness-and-precision tables, and weighted/unweighted linear-model summaries, then rendering the two customizable graph types (calibration curves and response factor plots) as PNG figures.

## When to use

Use when a targeted proteomics (SRM/MRM or PRM) calibration experiment with concentration/dilution series data needs linear-range determination, precision/trueness result tables, and calibration-curve plus response-factor figures.

## When NOT to use

- The task is raw-data inspection or normalization/curve fitting of uncalibrated spectra (e.g. Skyline inspection) rather than reporting on already-fitted calibration models
- The data are global/shotgun proteomics measurements without a concentration or dilution series, so no linear-range or response-factor assessment applies
- Only the dynamic-range determination is needed without any result tables or customizable graph generation

## Inputs

- Calibration/dilution-series measurement data (known analyte amounts vs. measured intensities) from targeted MS assays
- Fitted weighted and unweighted calibration models
- Determined linear range (lower and upper limits of quantification, LLOQ/ULOQ)

## Outputs

- MeasCV result CSV with precision (CV) values per concentration level and per replicate
- PB, PB_av, and PB_cv result tables (back-calculated precision and trueness)
- Weighted and unweighted linear model summary files (coefficients, R-squared, residuals)
- Calibration curve PNG rendered with ggplot2
- Response factor plot PNG with 80%/120% RF_mean_total threshold lines

## How to apply

Load the upstream CalibraCurve results — fitted weighted/unweighted calibration models and the determined linear range (LLOQ/ULOQ) — in R. Compute and write the MeasCV table with precision (CV) values per concentration level and per replicate using CalibraCurve's result-file writer, then the PB tables (PB per replicate, PB_av per-level averages, PB_cv per-level coefficients of variation). Write the weighted and unweighted linear model summaries (coefficients, R-squared, residuals) to tabular result files. Render the calibration curve from the fitted models and linear range with ggplot2, and render the response factor plot with horizontal threshold lines at 80% and 120% of RF_mean_total marking the acceptable response-factor band. Rationale: CalibraCurve assesses calibration accuracy via precision and trueness, and these two customizable graph types are the tool's canonical visual outputs.

## Related tools

- **CalibraCurve** (R package that determines the linear range and provides the result-file writer and visualization backend for the calibration tables and the two customizable graph types) — https://github.com/mpc-bioinformatics/CalibraCurve
- **R** (Runtime environment in which CalibraCurve is installed and the result files and figures are generated) — https://www.R-project.org/
- **ggplot2** (Used to render the calibration curve and response factor plot figures as PNGs)

## Examples

```
BiocManager::install("CalibraCurve"); library(CalibraCurve); res <- calibrateConcentrationSeries(XLSXFile = "calibration_data.xlsx", outputDir = "results")
```

## Evaluation signals

- All result CSVs (MeasCV, PB, PB_av, PB_cv, linear model summaries) and both PNG figures exist and parse correctly
- The response factor plot encodes horizontal threshold lines at exactly 80% and 120% of RF_mean_total
- MeasCV table contains precision (CV) values for every concentration level and every replicate
- PB tables distinguish per-replicate values (PB), per-level averages (PB_av), and per-level coefficients of variation (PB_cv)
- Linear model summaries include coefficients, R-squared, and residuals for both weighted and unweighted fits

## Limitations

- CalibraCurve requires samples with known amounts of the analyte measured in replicates; the README notes data are usually measured 'in relicates' and linearity is only valid within the computed linear range
- The formerly available KNIME-workflow variant was discontinued and replaced by the R package and nextflow workflow, so older workflow-based reproductions are unsupported
- Terms of use restrict the software to academic research and training; results must be validated before further use and must not be used for clinical diagnosis

## Evidence

- [abstract] Two different kinds of customizable graphs are created (calibration curves and: "The software uses a variety of measures to assess the accuracy of the calibration, namely precision and trueness. Two different kinds of customizable graphs are created (calibration curves and"
- [abstract] enables an automated batch-mode determination of dynamic linear ranges and quantification limits for both targeted proteomics and similar assays: "enables an automated batch-mode determination of dynamic linear ranges and quantification limits for both targeted proteomics and similar assays"
- [readme] Visualizations of the calibration curves and the underlying data basis are generated and can be customized.: [evidence span withheld — anchor status: external_doc]
- [readme] CalibraCurve calculates the linear range (lower and upper limits of quantification) using a well-designed algorithm.: [evidence span withheld — anchor status: external_doc]
- [readme] a linear relationship between concentration and intensity is often limited to a certain range of concentrations, the so called linear range: [evidence span withheld — anchor status: external_doc]
