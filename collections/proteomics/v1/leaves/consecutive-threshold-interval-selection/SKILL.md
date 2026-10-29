---
name: consecutive-threshold-interval-selection
description: Use when replicate intensity measurements are grouped by concentration
  level and a contiguous interval of levels passing a CV threshold (MeasCV <= TMeasCV)
  must be extracted, e.g. to define the lower bound of a linear range for SRM/MRM
  or PRM calibration data.
when_to_use_negative:
- Input is a single measurement per concentration level (no replicates), so a per-level
  MeasCV cannot be computed
- The final linear range is already known or refined by precision/trueness criteria
  — this skill only yields the preliminary range of process step 2
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: CalibraCurve
  role: R package implementing the automated determination of dynamic linear ranges
    and quantification limits, including the MeasCV-based PLR selection
  repo: https://github.com/mpc-bioinformatics/CalibraCurve
- name: R
  role: Language/environment used to compute per-level MeasCV and run the consecutive-run
    selection
- name: CalibraCurve_NF
  role: Nextflow workflow packaging the CalibraCurve R script for batch execution
  repo: https://github.com/mpc-bioinformatics/CalibraCurve_NF
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.201900143
    title: 'CalibraCurve: A Tool for Calibration of Targeted MS‐Based Measurements'
  source_content_hash: sha256:7901cdd41a33fcdeb3f4fa921735c8c11b4f47db931bec3e7ef94b65a3d87407
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mpc-bioinformatics/CalibraCurve
  tools:
  - CalibraCurve
  - CalibraCurve_NF
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# consecutive-threshold-interval-selection

## Summary

Determines a preliminary linear range (PLR) by computing a per-level measurement CV (MeasCV) from replicate intensities of a dilution/concentration series, comparing it to a user-defined threshold TMeasCV, and selecting the longest run of consecutive concentration levels that pass. This is process step 2 of CalibraCurve's dynamic linear range determination for targeted MS assays.

## When to use

Use when replicate intensity measurements are grouped by concentration level and a contiguous interval of levels passing a CV threshold (MeasCV <= TMeasCV) must be extracted, e.g. to define the lower bound of a linear range for SRM/MRM or PRM calibration data.

## When NOT to use

- Input is a single measurement per concentration level (no replicates), so a per-level MeasCV cannot be computed
- The final linear range is already known or refined by precision/trueness criteria — this skill only yields the preliminary range of process step 2

## Inputs

- Replicate intensity table (e.g. msqc1 calibration dataset) with measurements grouped by concentration level
- User-defined threshold TMeasCV (maximum acceptable measurement CV in percent)

## Outputs

- CSV table listing concentration level, MeasCV value, pass/fail flag, and the selected preliminary linear range
- Preliminary linear range (longest consecutive run of levels with MeasCV <= TMeasCV)

## How to apply

Load the replicate intensity table (e.g. the msqc1 calibration dataset) into R and group measurements by concentration level. For each level compute MeasCV as the standard deviation of replicate intensities divided by their mean, expressed as a percentage, then classify each level as pass/fail against the user-defined threshold TMeasCV. Derive the preliminary linear range as the longest run of consecutive concentration levels whose MeasCV does not exceed TMeasCV; when several candidate ranges are separated by levels with MeasCV > TMeasCV, select the PLR with the highest number of consecutive levels by default, or optionally allow levels with MeasCV above TMeasCV inside the PLR. Record the per-level MeasCV, pass/fail flag, and the selected PLR in a CSV table, and optionally compute the alternative range based on overall span of qualifying levels. This step provides the preliminary range that later CalibraCurve steps (precision/trueness assessment, LLOQ/ULOQ determination) refine.

## Related tools

- **CalibraCurve** (R package implementing the automated determination of dynamic linear ranges and quantification limits, including the MeasCV-based PLR selection) — https://github.com/mpc-bioinformatics/CalibraCurve
- **R** (Language/environment used to compute per-level MeasCV and run the consecutive-run selection)
- **CalibraCurve_NF** (Nextflow workflow packaging the CalibraCurve R script for batch execution) — https://github.com/mpc-bioinformatics/CalibraCurve_NF

## Examples

```
BiocManager::install("CalibraCurve"); library(CalibraCurve)
```

## Evaluation signals

- Output CSV exists and contains a MeasCV value for every concentration level present in the input table
- Reported PLR corresponds exactly to the longest consecutive run of levels with MeasCV <= TMeasCV
- Every level flagged as passing satisfies MeasCV <= TMeasCV and every failing level satisfies MeasCV > TMeasCV
- When multiple candidate ranges exist, the selected PLR has the maximum number of consecutive levels (unless the allow-above-threshold option is set)
- MeasCV values are reproducible when recomputed as sd(replicates)/mean(replicates) x 100 per level

## Limitations

- PLR selection is a preliminary step; the final dynamic linear range and quantification limits require subsequent precision and trueness assessment by CalibraCurve
- By default levels with MeasCV above TMeasCV break the range; the alternative policy (allowing such levels within the PLR) must be explicitly chosen by the user
- CalibraCurve targets targeted MS assays (SRM/MRM, PRM) and similar concentration/dilution series experiments with replicates; product ion intensity reduction at high concentrations ( motivating ULOQ determination) is handled in later steps

## Evidence

- [readme] CalibraCurve calculates the linear range (lower and upper limits of quantification) using a well-designed algorithm.: [evidence span withheld — anchor status: external_doc]
- [abstract] enables an automated batch-mode determination of dynamic linear ranges and quantification limits: "a software named CalibraCurve that enables an automated batch-mode determination of dynamic linear ranges and quantification limits"
- [abstract] The software uses a variety of measures to assess the accuracy of the calibration, namely precision and trueness.: "The software uses a variety of measures to assess the accuracy of the calibration, namely precision and trueness."
- [readme] Here, samples with known amount of the analyte of interest are generated and measured by mass spectrometry, usually in relicates.: [evidence span withheld — anchor status: external_doc]
