---
name: uloq-out-of-range-prediction-flagging
description: Use when new intensity values from an SRM/MRM or PRM targeted MS assay
  (e.g. msqc1 dilution-series data) must be converted to concentrations with a fitted
  CalibraCurve model, and out-of-linear-range values must be flagged.
when_to_use_negative:
- The intensities lie within the established dynamic linear range and no out-of-range
  flagging is required — a plain curve inversion suffices
- No fitted calibration model or LLOQ/ULOQ determination exists yet — first run CalibraCurve's
  calibration on the dilution series to establish the linear range before predicting
  concentrations
- The data are uncalibrated raw traces that first need inspection/normalization in
  Skyline rather than inversion of an existing response curve
edam_operation: http://edamontology.org/operation_3630
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: CalibraCurve
  role: R package whose fitted calibration model is inverted to predict concentrations
    from new intensities; it also supplies the linear range and LLOQ/ULOQ limits used
    for out-of-range flagging
  repo: https://github.com/mpc-bioinformatics/CalibraCurve
- name: R
  role: Runtime environment in which the CalibraCurve package is loaded and the prediction
    table is assembled and exported to CSV
  repo: https://www.R-project.org/
- name: msqc1
  role: Bioconductor reference dataset providing the analyte concentrations and intensities
    underpinning the fitted calibration model
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/pmic.201900143
    title: 'CalibraCurve: A Tool for Calibration of Targeted MS‐Based Measurements'
  source_content_hash: sha256:0cb47a75a6ef5d0da3f1274528355285a8e876c2461b9cf78757f7204f9c038a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mpc-bioinformatics/CalibraCurve
  tools:
  - CalibraCurve
  - R
  - msqc1
  edam_operation: http://edamontology.org/operation_3630
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# uloq-out-of-range-prediction-flagging

## Summary

Applies a fitted CalibraCurve calibration model in R to predict analyte concentrations from new measured intensities (inverse of the fitted response curve), while flagging any prediction that falls above the ULOQ / outside the dynamic linear range as potentially unreliable.

## When to use

Use when new intensity values from an SRM/MRM or PRM targeted MS assay (e.g. msqc1 dilution-series data) must be converted to concentrations with a fitted CalibraCurve model, and out-of-linear-range values must be flagged.

## When NOT to use

- The intensities lie within the established dynamic linear range and no out-of-range flagging is required — a plain curve inversion suffices
- No fitted calibration model or LLOQ/ULOQ determination exists yet — first run CalibraCurve's calibration on the dilution series to establish the linear range before predicting concentrations
- The data are uncalibrated raw traces that first need inspection/normalization in Skyline rather than inversion of an existing response curve

## Inputs

- Fitted CalibraCurve calibration model (R object) for a chosen msqc1 analyte
- msqc1 reference dataset (R/Bioconductor package data)
- Vector of new measured intensity values to invert (e.g. 1e6, 1e7, 1e8)

## Outputs

- predicted_concentrations.csv with one row per queried intensity, a predicted concentration field, and an in-range/out-of-range (ULOQ warning) status column

## How to apply

Load the fitted CalibraCurve model for the chosen analyte (e.g. an msqc1 peptide) together with the msqc1 reference dataset in R. Apply CalibraCurve's concentration-prediction routine, i.e. the inverse of the fitted response curve, to each new intensity value (e.g. 1e6, 1e7, 1e8). Compare each queried intensity against the dynamic linear range and quantification limits (LLOQ/ULOQ) that CalibraCurve determined automatically during calibration. Mark any prediction above the ULOQ as unreliable, because linearity between concentration and intensity is limited to the linear range and product ion intensities can even be reduced at higher analyte concentrations. Assemble predicted concentrations with their in-range/out-of-range status into a table (one row per queried intensity) and write it to predicted_concentrations.csv, retaining the out-of-range warning on the offending row.

## Related tools

- **CalibraCurve** (R package whose fitted calibration model is inverted to predict concentrations from new intensities; it also supplies the linear range and LLOQ/ULOQ limits used for out-of-range flagging) — https://github.com/mpc-bioinformatics/CalibraCurve
- **R** (Runtime environment in which the CalibraCurve package is loaded and the prediction table is assembled and exported to CSV) — https://www.R-project.org/
- **msqc1** (Bioconductor reference dataset providing the analyte concentrations and intensities underpinning the fitted calibration model)

## Examples

```
BiocManager::install("CalibraCurve"); library(CalibraCurve); library(msqc1)
```

## Evaluation signals

- predicted_concentrations.csv exists and contains exactly one row per queried intensity (3 rows for 1e6, 1e7, 1e8)
- Every row carries a non-missing concentration field obtained by inverting the fitted response curve
- The 1e8 row carries an explicit out-of-linear-range warning, i.e. it is marked as above the ULOQ
- In-range rows (1e6, 1e7) are not flagged, matching their position within CalibraCurve's dynamic linear range
- The linear range and ULOQ used for flagging match those determined by CalibraCurve's automated batch-mode determination for the same analyte

## Limitations

- Predictions above the ULOQ are outside the linear range and are only flagged as potentially unreliable — the numeric prediction may be inaccurate because linearity breaks down at high analyte concentrations
- The linear range typically spans only 4-5 orders of magnitude in targeted proteomics, so intensities far outside it cannot be trusted for quantification
- Reduction of product ion intensities was observed at higher analyte concentrations, meaning out-of-range high signals may even decrease rather than increase

## Evidence

- [readme] readme: calibration curves as prediction basis: [evidence span withheld — anchor status: external_doc]
- [readme] readme: linear range limits quantification: [evidence span withheld — anchor status: external_doc]
- [readme] readme: LLOQ/ULOQ computation: [evidence span withheld — anchor status: external_doc]
- [abstract] abstract: automated dynamic linear ranges and quantification limits: "a software named CalibraCurve that enables an automated batch-mode determination of dynamic linear ranges and quantification limits"
- [intro] intro: signal reduction at high concentrations: "we observed reduction of product ion intensities at higher analyte concentrations (Figure 1)"
