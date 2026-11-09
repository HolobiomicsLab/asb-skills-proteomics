---
name: replicate-cv-precision-calculation
description: Use when replicate-grouped calibration data from a targeted MS (SRM/MRM/PRM)
  dilution series must be screened per concentration level for precision (MeasCV vs
  TMeasCV) in order to derive the preliminary linear range.
when_to_use_negative:
- Input is a single measurement per concentration level (no replicates), since MeasCV
  cannot be computed
- The goal is the final dynamic linear range and quantification limits (LLOQ/ULOQ)
  from curve fitting rather than the preliminary per-level precision screen
- Data are not from a calibration/dilution series (e.g. already-normalized abundance
  matrices)
tools:
- name: CalibraCurve
  role: 'Defines the PLR algorithm: per-level MeasCV vs TMeasCV filtering and selection
    of the candidate linear range'
  repo: github.com/mpc-bioinformatics/CalibraCurve
- name: R
  role: Implementation environment for grouping replicates, computing MeasCV, and
    exporting the CSV output
  repo: https://www.R-project.org/
- name: msqc1
  role: Source of the replicate intensity calibration data (e.g. msqc1 dataset cited
    in the paper's methods)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.201900143
    title: 'CalibraCurve: A Tool for Calibration of Targeted MS‐Based Measurements'
  source_content_hash: sha256:2e82faecca6d70284bd4f894164ba0dcecf36938b9daf1e0c58dd8c15e8d82d8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CalibraCurve
  - R
  - msqc1
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# replicate-cv-precision-calculation

## Summary

Computes per-level replicate precision (MeasCV: SD/mean of replicate intensities as a percentage) for each concentration level of a targeted MS calibration dilution series and classifies levels against the user-defined threshold TMeasCV to reconstruct CalibraCurve's preliminary linear range (PLR).

## When to use

Use when replicate-grouped calibration data from a targeted MS (SRM/MRM/PRM) dilution series must be screened per concentration level for precision (MeasCV vs TMeasCV) in order to derive the preliminary linear range.

## When NOT to use

- Input is a single measurement per concentration level (no replicates), since MeasCV cannot be computed
- The goal is the final dynamic linear range and quantification limits (LLOQ/ULOQ) from curve fitting rather than the preliminary per-level precision screen
- Data are not from a calibration/dilution series (e.g. already-normalized abundance matrices)

## Inputs

- Replicate intensity table from a targeted MS calibration/dilution series (e.g. msqc1 dataset), grouped by concentration level
- User-defined threshold TMeasCV (maximum acceptable replicate CV in percent)

## Outputs

- CSV table with concentration level, MeasCV value, pass/fail flag, and the selected preliminary linear range
- Candidate PLR ranges (longest consecutive run and, optionally, span-based alternative)

## How to apply

Load the replicate intensity table (e.g. from the msqc1 calibration dataset) into R and group measurements by concentration level. For each level, compute MeasCV as the standard deviation of replicate intensities divided by their mean, expressed as a percentage. Compare each level's MeasCV against the user-defined threshold TMeasCV to assign a pass/fail flag. Derive the preliminary linear range as the longest run of consecutive concentration levels passing the TMeasCV criterion; when several candidate PLRs are separated by levels exceeding TMeasCV, CalibraCurve selects the one with the highest number of consecutive levels by default. Optionally, also compute the alternative selection based on overall span of qualifying levels, and support the user option that allows levels with MeasCV above TMeasCV within the PLR. Write a CSV listing concentration level, MeasCV, pass/fail flag, and the selected PLR.

## Related tools

- **CalibraCurve** (Defines the PLR algorithm: per-level MeasCV vs TMeasCV filtering and selection of the candidate linear range) — github.com/mpc-bioinformatics/CalibraCurve
- **R** (Implementation environment for grouping replicates, computing MeasCV, and exporting the CSV output) — https://www.R-project.org/
- **msqc1** (Source of the replicate intensity calibration data (e.g. msqc1 dataset cited in the paper's methods))

## Examples

```
BiocManager::install("CalibraCurve"); library(CalibraCurve)
```

## Evaluation signals

- Output CSV exists and contains a MeasCV value for every concentration level present in the input
- Pass/fail flags are exactly consistent with the comparison of each MeasCV against the supplied TMeasCV
- Reported preliminary linear range corresponds exactly to the longest consecutive run of levels with MeasCV <= TMeasCV
- When multiple candidate ranges exist, the selected PLR is the one with the highest number of consecutive levels (default policy)
- Both the consecutive-run and span-based candidate ranges are recorded when the alternative selection policy is exercised

## Limitations

- By default levels whose MeasCV exceeds TMeasCV break the PLR; allowing such levels inside the PLR is an explicit alternative user option, not the default behavior
- This is the preliminary linear range (process step 2), not the final dynamic linear range with quantification limits, which CalibraCurve determines via additional precision and trueness measures
- The article's discussion section was not available in the provided document, so reproducibility caveats and edge-case limits are not documented there

## Evidence

- [other] In process step 2, when several candidate preliminary linear ranges are separated by concentration levels whose MeasCV exceeds TMeasCV: [evidence span withheld — anchor status: not_found]
- [other] compute the MeasCV as the standard deviation of replicate intensities divided by their mean, expressed as a percentage: [evidence span withheld — anchor status: not_found]
- [readme] CalibraCurve calculates the linear range using a well-designed algorithm: [evidence span withheld — anchor status: external_doc]
- [readme] CalibraCurve is written in R and available as an R package: [evidence span withheld — anchor status: external_doc]
