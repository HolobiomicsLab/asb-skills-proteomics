---
name: targeted-proteomics-dilution-series-analysis
description: Use when a replicate intensity table from a concentration or dilution
  series (e.g. msqc1 calibration data) must be reduced to a linear quantification
  range via per-level replicate CV against a threshold TMeasCV.
when_to_use_negative:
- Input is already a fitted calibration curve or a summary statistics table — MeasCV
  must be recomputed from replicate intensities per concentration level
- Data are shotgun (DDA) proteomics measurements rather than targeted SRM/MRM/PRM
  dilution-series replicates, for which the paper's dynamic-range framing does not
  apply
- The goal is ULOQ determination from signal reduction at high analyte concentrations
  without replicate-level CV computation
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: CalibraCurve
  role: R package that automates per-level MeasCV computation, PLR selection, and
    dynamic linear range / quantification limit determination in batch mode
  repo: https://github.com/mpc-bioinformatics/CalibraCurve
- name: R
  role: Execution environment for loading the replicate intensity table, computing
    per-level CVs, and running the CalibraCurve package
  repo: https://www.R-project.org/
- name: msqc1
  role: Source of the calibration (dilution series) dataset used as the exemplar input
    for the PLR reconstruction
- name: CalibraCurve_NF
  role: Nextflow workflow wrapper for running the CalibraCurve analysis reproducibly
    at scale
  repo: https://github.com/mpc-bioinformatics/CalibraCurve_NF
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.201900143
    title: 'CalibraCurve: A Tool for Calibration of Targeted MS‐Based Measurements'
  source_content_hash: sha256:862bb3bceb132025210ce74729260779eb2ec0110288271069e0bfce675be373
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mpc-bioinformatics/CalibraCurve
  tools:
  - CalibraCurve
  - CalibraCurve_NF
  - R
  - msqc1
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# targeted-proteomics-dilution-series-analysis

## Summary

Reconstructs CalibraCurve's preliminary linear range (PLR) computation from per-concentration-level MeasCV values and a user-defined threshold TMeasCV, enabling determination of dynamic linear ranges and quantification limits for targeted MS (SRM/MRM, PRM) dilution-series data.

## When to use

Use when a replicate intensity table from a concentration or dilution series (e.g. msqc1 calibration data) must be reduced to a linear quantification range via per-level replicate CV against a threshold TMeasCV.

## When NOT to use

- Input is already a fitted calibration curve or a summary statistics table — MeasCV must be recomputed from replicate intensities per concentration level
- Data are shotgun (DDA) proteomics measurements rather than targeted SRM/MRM/PRM dilution-series replicates, for which the paper's dynamic-range framing does not apply
- The goal is ULOQ determination from signal reduction at high analyte concentrations without replicate-level CV computation

## Inputs

- Replicate intensity table from a targeted MS concentration/dilution series (e.g. msqc1 calibration dataset), grouped by concentration level
- User-defined threshold TMeasCV on the per-level coefficient of variation
- Selection policy choice: longest consecutive run (default) vs. allowing levels with MeasCV > TMeasCV within the PLR

## Outputs

- CSV table with columns: concentration level, MeasCV (%), pass/fail flag, selected preliminary linear range
- Preliminary linear range (lower/upper limits of quantification, LLOQ/ULOQ)
- Calibration curve and response factor visualizations

## How to apply

Load the replicate intensity table (e.g. from the msqc1 calibration dataset) into R and group measurements by concentration level. For each level compute MeasCV as the standard deviation of replicate intensities divided by their mean, expressed as a percentage. Classify each level as passing or failing by comparison against the user-defined threshold TMeasCV, then derive the preliminary linear range as the longest run of consecutive levels that pass; when several candidate PLRs exist, CalibraCurve selects the one with the highest number of consecutive levels by default, with an alternative policy allowing levels above TMeasCV inside the PLR. Write a CSV listing concentration level, MeasCV, pass/fail flag, and the selected PLR, and validate that every input level appears and that the reported PLR matches exactly the longest consecutive passing run.

## Related tools

- **CalibraCurve** (R package that automates per-level MeasCV computation, PLR selection, and dynamic linear range / quantification limit determination in batch mode) — https://github.com/mpc-bioinformatics/CalibraCurve
- **R** (Execution environment for loading the replicate intensity table, computing per-level CVs, and running the CalibraCurve package) — https://www.R-project.org/
- **msqc1** (Source of the calibration (dilution series) dataset used as the exemplar input for the PLR reconstruction)
- **CalibraCurve_NF** (Nextflow workflow wrapper for running the CalibraCurve analysis reproducibly at scale) — https://github.com/mpc-bioinformatics/CalibraCurve_NF

## Examples

```
BiocManager::install("CalibraCurve"); library(CalibraCurve)
```

## Evaluation signals

- The output CSV exists and contains a MeasCV value for every concentration level present in the input replicate table
- The reported preliminary linear range corresponds exactly to the longest consecutive run of levels with MeasCV <= TMeasCV
- When both selection policies are applied, both candidate ranges are recorded and the default range is the one with the highest number of consecutive qualifying levels
- Reported linear range limits (LLOQ/ULOQ) span a plausible dynamic range (targeted proteomics ranges span up to 4-5 orders of magnitude)
- Per-level MeasCV equals sd(replicate intensities)/mean(replicate intensities) * 100 for each level

## Limitations

- Linear relationship between concentration and intensity is often limited to a certain concentration range, so the PLR is preliminary and may exclude levels depending on the consecutive-run policy
- Reduction of product ion intensities at higher analyte concentrations requires ULOQ determination beyond the PLR step
- Results must be appropriately validated before further use; the tool is intended exclusively for academic research, not clinical diagnosis
- No discussion-section limitations were extractable from the provided article text

## Evidence

- [readme] CalibraCurve calculates the linear range (lower and upper limits of quantification) using a well-designed algorithm.: [evidence span withheld — anchor status: external_doc]
- [abstract] a software named CalibraCurve that enables an automated batch-mode determination of dynamic linear ranges and quantification limits: "a software named CalibraCurve that enables an automated batch-mode determination of dynamic linear ranges and quantification limits"
- [intro] we observed reduction of product ion intensities at higher analyte concentrations (Figure 1): "we observed reduction of product ion intensities at higher analyte concentrations (Figure 1)"
- [readme] samples with known amount of the analyte of interest are generated and measured by mass spectrometry, usually in relicates.: [evidence span withheld — anchor status: external_doc]
