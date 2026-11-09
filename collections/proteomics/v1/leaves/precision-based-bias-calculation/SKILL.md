---
name: precision-based-bias-calculation
description: Use when building targeted MS (SRM/MRM or PRM) calibration curves from
  replicate concentration–response data and you must determine the linear range by
  checking whether boundary levels Cl and Ch satisfy PB_av below TPB_av.
when_to_use_negative:
- Data are not a calibration/dilution series with known analyte concentrations and
  replicates — the PB metrics require replicate measurements per level
- The goal is assessing instrument sensitivity or ULOQ from signal drop-off (e.g.
  reduced product ion intensities at high concentration) rather than computing precision-based
  bias boundaries
- The calibration curve is already final and you only need visualization; use the
  CalibraCurve plotting routines instead of re-running the boundary-removal iteration
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: CalibraCurve
  role: R package implementing the PB/PB_av/PB_cv computation, TPB_av thresholding,
    boundary-level removal iteration, and linear_range.csv output
  repo: https://github.com/mpc-bioinformatics/CalibraCurve
- name: R
  role: Runtime environment; the linear fitting and bias aggregation are carried out
    with R's lm-based routines inside the package
  repo: https://www.R-project.org/
- name: msqc1
  role: Example calibration dataset package used with CalibraCurve for calibration
    data
- name: Skyline
  role: Upstream inspection of raw targeted MS data before curve fitting in R
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pmic.201900143
    title: 'CalibraCurve: A Tool for Calibration of Targeted MS‐Based Measurements'
  source_content_hash: sha256:ff9c2cf0e8fbdb1b51712f3ffcc0801fad26d580fad4e460b8868012a1d69989
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mpc-bioinformatics/CalibraCurve
  tools:
  - CalibraCurve
  - R
  - Skyline
  - msqc1
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# precision-based-bias-calculation

## Summary

Computes the precision-based bias (PB) of individual calibration measurements in CalibraCurve, aggregates it per concentration level into PB_av (mean bias) and PB_cv (bias variability), and uses the TPB_av threshold to iteratively define the final linear range.

## When to use

Use when building targeted MS (SRM/MRM or PRM) calibration curves from replicate concentration–response data and you must determine the linear range by checking whether boundary levels Cl and Ch satisfy PB_av below TPB_av.

## When NOT to use

- Data are not a calibration/dilution series with known analyte concentrations and replicates — the PB metrics require replicate measurements per level
- The goal is assessing instrument sensitivity or ULOQ from signal drop-off (e.g. reduced product ion intensities at high concentration) rather than computing precision-based bias boundaries
- The calibration curve is already final and you only need visualization; use the CalibraCurve plotting routines instead of re-running the boundary-removal iteration

## Inputs

- Calibration concentration–response table with replicate measurements per concentration level (targeted MS data, e.g. SRM/MRM or PRM peak areas from Skyline inspection)
- Threshold parameter TPB_av (target value for average precision-based bias per level)
- Weighting choice for the linear model (weighted and unweighted lm fits)

## Outputs

- linear_range.csv containing the final Cl and Ch concentration values of the linear range and the ordered log of removed levels
- Per-level PB_av and PB_cv values for the calibration levels

## How to apply

Load the calibration concentration–response table (replicate measurements per level) into R with the CalibraCurve package, then fit weighted and unweighted linear models of instrument response versus concentration across all levels using CalibraCurve's lm-based routines. From the residuals against the fitted model, compute the precision-based bias (PB) for each individual measurement and aggregate to PB_av (mean) and PB_cv (variability) per concentration level. Compare the PB_av of the lowest boundary level (Cl) and the highest boundary level (Ch) against the target threshold TPB_av; if a boundary level exceeds TPB_av, remove it using the documented highest/lowest tie-breaking rules, refit the model, and repeat until both boundary levels satisfy the threshold. The iteration terminates when the remaining dataset is 'final', i.e. PB_av of both Cl and Ch is below TPB_av, at which point the linear range and the ordered log of removed levels are written to linear_range.csv.

## Related tools

- **CalibraCurve** (R package implementing the PB/PB_av/PB_cv computation, TPB_av thresholding, boundary-level removal iteration, and linear_range.csv output) — https://github.com/mpc-bioinformatics/CalibraCurve
- **R** (Runtime environment; the linear fitting and bias aggregation are carried out with R's lm-based routines inside the package) — https://www.R-project.org/
- **msqc1** (Example calibration dataset package used with CalibraCurve for calibration data)
- **Skyline** (Upstream inspection of raw targeted MS data before curve fitting in R)

## Examples

```
if (!requireNamespace("BiocManager", quietly=TRUE)) install.packages("BiocManager"); BiocManager::install("CalibraCurve"); library(CalibraCurve)
```

## Evaluation signals

- Termination invariant: after iteration stops, PB_av of the retained lowest level (Cl) and highest level (Ch) are both strictly below TPB_av
- linear_range.csv contains the final Cl and Ch values plus the ordered log of removed levels; the removed-level log is consistent with the per-level PB_av history
- Every level removed from the linear range had PB_av exceeding TPB_av at the time of removal under the fitted model
- Refitting occurs after each level removal: final PB_av/PB_cv values must be computed from the residuals of the model refit on the retained levels, not the original all-level fit
- Both weighted and unweighted lm fits are available and the reported linear range must match the weighting scheme actually used

## Limitations

- The KNIME-workflow implementation was discontinued and replaced by the R package and nextflow workflow, so legacy documentation may not match current behavior
- Linear relationship between concentration and intensity is only valid within the determined linear range; extrapolating outside Cl–Ch is not supported by the calibration curve
- Results must be validated by the user before further use; the software is intended for academic research purposes only, not clinical diagnosis

## Evidence

- [abstract] The software uses a variety of measures to assess the accuracy of the calibration, namely precision and trueness.: "The software uses a variety of measures to assess the accuracy of the calibration, namely precision and trueness."
- [readme] CalibraCurve calculates the linear range (lower and upper limits of quantification) using a well-designed algorithm.: [evidence span withheld — anchor status: external_doc]
- [readme] Quality of the data and the accuracy of the produced calibration curve is assessed in several computational steps.: [evidence span withheld — anchor status: external_doc]
- [intro] we observed reduction of product ion intensities at higher analyte concentrations (Figure 1): "we observed reduction of product ion intensities at higher analyte concentrations (Figure 1)"
