---
name: precision-trueness-table-generation
description: Use when you have fitted calibration models and a determined linear range
  for a dilution-series assay and need tabular precision (CV) and trueness summaries
  per concentration level and replicate for assay validation.
when_to_use_negative:
- Input is raw Skyline transition intensities that have not yet been fitted to weighted/unweighted
  calibration models — run the CalibraCurve calibration first
- The goal is only visual QC of the calibration — use the calibration curve and response
  factor plot rendering directly instead of table generation
- The data are not from a concentration/dilution series with known analyte amounts
  (e.g. global shotgun proteomics data)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3368
- http://edamontology.org/topic_0091
tools:
- name: CalibraCurve
  role: R package providing the calibration fit, linear-range determination, and the
    precision/trueness assessment from which the MeasCV and PB result tables are computed
  repo: https://github.com/mpc-bioinformatics/CalibraCurve
- name: R
  role: Runtime environment in which the result-file writer produces the MeasCV, PB,
    PB_av, and PB_cv tables from the fitted models
  repo: https://www.R-project.org/
- name: ggplot2
  role: Renders the accompanying calibration curve and response factor plot PNGs that
    validate the tabulated precision/trueness results
- name: Skyline
  role: Used upstream to inspect raw PRM data before curve fitting and table generation
    in R
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/pmic.201900143
    title: 'CalibraCurve: A Tool for Calibration of Targeted MS‐Based Measurements'
  source_content_hash: sha256:9b010b3534f80bd26cd8d9bf8e5f829c34f682e1977e1852d107183bdd9cadf6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mpc-bioinformatics/CalibraCurve
  tools:
  - CalibraCurve
  - R
  - Skyline
  - ggplot2
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3368
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# precision-trueness-table-generation

## Summary

Generates tabular result files quantifying calibration accuracy for targeted MS-based (SRM/MRM/PRM) measurements: per-level precision (CV) tables and back-calculated precision and trueness tables derived from weighted/unweighted CalibraCurve calibration models.

## When to use

Use when you have fitted calibration models and a determined linear range for a dilution-series assay and need tabular precision (CV) and trueness summaries per concentration level and replicate for assay validation.

## When NOT to use

- Input is raw Skyline transition intensities that have not yet been fitted to weighted/unweighted calibration models — run the CalibraCurve calibration first
- The goal is only visual QC of the calibration — use the calibration curve and response factor plot rendering directly instead of table generation
- The data are not from a concentration/dilution series with known analyte amounts (e.g. global shotgun proteomics data)

## Inputs

- Fitted weighted and unweighted calibration models (linear model objects) from CalibraCurve
- Determined linear range with lower and upper limits of quantification (LLOQ/ULOQ)
- Raw replicate intensity/concentration data from concentration or dilution series measured by targeted MS (e.g. Q Exactive PRM data, Skyline-inspected)

## Outputs

- MeasCV result CSV with precision (CV) values per concentration level and per replicate
- PB result CSV with back-calculated precision and trueness per replicate
- PB_av result CSV with per-level averages
- PB_cv result CSV with per-level coefficients of variation
- Weighted and unweighted linear model summary tables (coefficients, R-squared, residuals)
- Calibration curve PNG rendered with ggplot2
- Response factor plot PNG with horizontal threshold lines at 80% and 120% of RF_mean_total

## How to apply

Load the upstream CalibraCurve results (fitted weighted and unweighted linear calibration models plus the determined linear range with LLOQ/ULOQ) in R. Compute and write the MeasCV table containing precision (coefficient of variation) values per concentration level and per replicate. Compute and write the PB tables: PB (back-calculated precision and trueness per replicate), PB_av (per-level averages), and PB_cv (per-level coefficients of variation). Write the weighted and unweighted linear model summaries (coefficients, R-squared, residuals) to tabular result files. The rationale is that CalibraCurve assesses calibration accuracy through precision and trueness measures, so tabulating these metrics per level makes the dynamic linear range determination auditable alongside the rendered calibration curve and response factor plots.

## Related tools

- **CalibraCurve** (R package providing the calibration fit, linear-range determination, and the precision/trueness assessment from which the MeasCV and PB result tables are computed) — https://github.com/mpc-bioinformatics/CalibraCurve
- **R** (Runtime environment in which the result-file writer produces the MeasCV, PB, PB_av, and PB_cv tables from the fitted models) — https://www.R-project.org/
- **ggplot2** (Renders the accompanying calibration curve and response factor plot PNGs that validate the tabulated precision/trueness results)
- **Skyline** (Used upstream to inspect raw PRM data before curve fitting and table generation in R)

## Examples

```
BiocManager::install("CalibraCurve"); library(CalibraCurve)
```

## Evaluation signals

- All result CSVs (MeasCV, PB, PB_av, PB_cv, linear model summaries) exist and parse correctly as tables
- MeasCV rows cover every concentration level in the dilution series with a CV value per replicate
- PB table reports back-calculated precision and trueness per replicate, and PB_av/PB_cv aggregate these per level
- Weighted and unweighted model summaries both contain coefficients, R-squared, and residuals
- Companion calibration curve and response factor PNGs exist and the response factor plot encodes threshold lines at 80% and 120% of RF_mean_total

## Limitations

- No discussion-section content was available in the provided document, so reproducibility gaps and edge-case caveats could not be extracted
- The linear relationship between concentration and intensity is limited to the linear range; values outside LLOQ/ULOQ (e.g. reduced product ion intensities at higher concentrations) fall outside the calibration's validity
- README terms of use restrict the software to academic research and training purposes and require appropriate validation of results before further use

## Evidence

- [abstract] CalibraCurve assesses calibration accuracy using precision and trueness measures: "The software uses a variety of measures to assess the accuracy of the calibration, namely precision and trueness."
- [readme] Quality assessment in several computational steps: [evidence span withheld — anchor status: external_doc]
- [abstract] Two customizable graph types: "Two different kinds of customizable graphs are created (calibration curves and"
