---
name: dynamic-linear-range-determination
description: Use when replicate concentration–response data from a targeted MS dilution
  series (e.g. SRM/MRM, PRM) must be reduced to a defensible linear range with LLOQ/ULOQ,
  instead of eyeballing response-factor plots.
when_to_use_negative:
- Data are single-point or lack replicate measurements per concentration level, so
  MeasCV and PB_av/PB_cv cannot be computed
- The goal is inspection of raw chromatograms or transition intensities (e.g. Skyline)
  rather than calibration-curve statistics
- The concentration–response relationship is expected to be nonlinear over the full
  span and no linear sub-range is of interest
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: CalibraCurve
  role: R package that performs the automated batch-mode determination of dynamic
    linear ranges and quantification limits via the MeasCV/PB_av algorithm
  repo: https://github.com/mpc-bioinformatics/CalibraCurve
- name: R
  role: Runtime environment; CalibraCurve is an R package using lm-based fitting
  repo: https://www.R-project.org/
- name: msqc1
  role: Source of the multi-instrument (Q Exactive, TSQ Vantage, QTRAP) dilution-series
    calibration benchmark data
- name: CalibraCurve_NF
  role: Nextflow wrapper for running the CalibraCurve linear-range workflow reproducibly
    at scale
  repo: https://github.com/mpc-bioinformatics/CalibraCurve_NF
provenance:
  source_task_ids:
  - task_004
  - task_006
  - task_003
  source_papers:
  - doi: 10.1002/pmic.201900143
    title: 'CalibraCurve: A Tool for Calibration of Targeted MS‐Based Measurements'
  source_content_hash: sha256:cfa63557b24316e42e706009462fe20103e191503933b88e425cea7bdc258401
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mpc-bioinformatics/CalibraCurve
  tools:
  - CalibraCurve
  - CalibraCurve_NF
  - R
  - msqc1
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# dynamic-linear-range-determination

## Summary

Automated determination of the dynamic linear range (lower and upper limits of quantification) for targeted-MS calibration data using CalibraCurve, which assesses calibration accuracy via precision (PB_av, PB_cv, MeasCV) and trueness measures.

## When to use

Use when replicate concentration–response data from a targeted MS dilution series (e.g. SRM/MRM, PRM) must be reduced to a defensible linear range with LLOQ/ULOQ, instead of eyeballing response-factor plots.

## When NOT to use

- Data are single-point or lack replicate measurements per concentration level, so MeasCV and PB_av/PB_cv cannot be computed
- The goal is inspection of raw chromatograms or transition intensities (e.g. Skyline) rather than calibration-curve statistics
- The concentration–response relationship is expected to be nonlinear over the full span and no linear sub-range is of interest

## Inputs

- Calibration concentration–response table with replicate measurements per concentration level (e.g. msqc1 dilution-series SummarizedExperiment with heavy/light peptide peak areas)
- User-defined thresholds TMeasCV and TPB_av
- Choice of weighted vs. unweighted linear regression model

## Outputs

- linear_range.csv containing the final linear range (Cl, Ch) and the ordered log of removed levels
- Per-level MeasCV, PB_av, and PB_cv values with pass/fail classification against TMeasCV and TPB_av
- Calibration curve and response factor plots
- R-squared values and linear ranges per analyte/instrument for summary tables

## How to apply

Load the calibration concentration–response table (replicate measurements per level) into R and fit weighted and unweighted linear models of instrument response versus concentration using CalibraCurve's lm-based routines. In process step 2 compute the per-level MeasCV (SD/mean of replicate intensities, %) and derive the preliminary linear range (PLR) as the longest run of consecutive levels with MeasCV below the user-defined threshold TMeasCV (default policy; alternatively allow levels above TMeasCV within the PLR). In process step 3 compute the precision-based bias (PB) per measurement against the fitted model, aggregate to PB_av and PB_cv per level, and iteratively remove boundary levels whose PB_av exceeds the target threshold TPB_av (Cl = lowest, Ch = highest), refitting after each removal with the documented tie-breaking rules until both boundary levels pass. Write the final linear range (Cl, Ch) plus the ordered log of removed levels to linear_range.csv.

## Related tools

- **CalibraCurve** (R package that performs the automated batch-mode determination of dynamic linear ranges and quantification limits via the MeasCV/PB_av algorithm) — https://github.com/mpc-bioinformatics/CalibraCurve
- **R** (Runtime environment; CalibraCurve is an R package using lm-based fitting) — https://www.R-project.org/
- **msqc1** (Source of the multi-instrument (Q Exactive, TSQ Vantage, QTRAP) dilution-series calibration benchmark data)
- **CalibraCurve_NF** (Nextflow wrapper for running the CalibraCurve linear-range workflow reproducibly at scale) — https://github.com/mpc-bioinformatics/CalibraCurve_NF

## Examples

```
BiocManager::install("CalibraCurve"); library(CalibraCurve)
```

## Evaluation signals

- linear_range.csv exists and contains the final Cl and Ch values plus the ordered removed-level log
- PB_av of both retained boundary levels (lowest Cl and highest Ch) is below TPB_av — this is the algorithm's own acceptance criterion
- Per-level MeasCV is reported for every concentration level present in the input, and the reported preliminary linear range equals the longest consecutive run of levels with MeasCV <= TMeasCV
- The reported R-squared per analyte/instrument matches the published benchmark pattern (Q Exactive worst overall R-squared; TSQ Vantage widest linear range on msqc1_dil)
- The linear span of the final range is plausible for targeted proteomics (up to 4–5 orders of magnitude)

## Limitations

- KNIME-workflow development was discontinued; current usage is via the R/Bioconductor package or the Nextflow workflow
- The default PLR selection takes the highest number of consecutive qualifying levels; choosing the alternative policy (allowing levels above TMeasCV within the PLR) yields different candidate ranges
- No discussion-section guidance on limitations, missing artifacts, or reproducibility gaps was available in the extracted article text

## Evidence

- [abstract] a software named CalibraCurve that enables an automated batch-mode determination of dynamic linear ranges and quantification limits: "a software named CalibraCurve that enables an automated batch-mode determination of dynamic linear ranges and quantification limits"
- [abstract] dynamic linear ranges that span up to 4-5 orders of magnitude: "dynamic linear ranges that span up to 4-5 orders of magnitude"
- [readme] linear range (lower and upper limits of quantification) using a well-designed algorithm: [evidence span withheld — anchor status: external_doc]
- [readme] Quality of the data and the accuracy of the produced calibration curve is assessed in several computational steps.: [evidence span withheld — anchor status: external_doc]
