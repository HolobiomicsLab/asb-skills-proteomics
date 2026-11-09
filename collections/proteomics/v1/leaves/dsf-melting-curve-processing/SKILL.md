---
name: dsf-melting-curve-processing
description: Use when raw DSF fluorescence-versus-temperature melting curves (e.g.
  a 96-well plate) must be normalized, smoothed, and converted to per-well Tm values,
  or when checking whether smoothing perturbs FD-method Tm values.
when_to_use_negative:
- Input fluorescence curves are already min-max normalized to the 0-1 range, so Equation
  1 would be applied redundantly
- The analysis goal is not thermal stability (e.g. non-thermal interaction or kinetics
  modeling), which the MC normalization/smoothing/Tm pipeline is not designed for
tools:
- name: SimpleDSFviewer
  role: Matlab GUI implementing MC normalization (Equation 1), curve smoothing, and
    Tm analysis of DSF melting curves; used to load raw data and preview analyzed
    results
  repo: github.com/hscsun/SimpleDSFviewer-5.0
- name: Matlab
  role: Runtime environment in which SimpleDSFviewer executes the normalization, smoothing,
    and FDC-based Tm computation
- name: Matlab Runtime
  role: Alternative execution route for running SimpleDSFviewer without a full Matlab
    installation
provenance:
  source_task_ids:
  - task_005
  - task_001
  source_papers:
  - doi: 10.1002/pro.3703
    title: 'SimpleDSFviewer: A tool to analyze and view differential scanning fluorimetry
      data for characterizing protein thermal stability and interactions'
  source_content_hash: sha256:df575a88509888ed9e9539b72e939a61dbe68da4810f6e3a37bd84815bc2c053
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Matlab
  - Matlab Runtime
  - SimpleDSFviewer
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# dsf-melting-curve-processing

## Summary

Implements the SimpleDSFviewer processing chain for differential scanning fluorimetry (DSF): per-well melting curve (MC) normalization (Equation 1), curve smoothing, and melting temperature (Tm) extraction via the first derivative curve (FDC). It turns raw fluorescence-versus-temperature plate data into normalized curves and per-well Tm tables.

## When to use

Use when raw DSF fluorescence-versus-temperature melting curves (e.g. a 96-well plate) must be normalized, smoothed, and converted to per-well Tm values, or when checking whether smoothing perturbs FD-method Tm values.

## When NOT to use

- Input fluorescence curves are already min-max normalized to the 0-1 range, so Equation 1 would be applied redundantly
- The analysis goal is not thermal stability (e.g. non-thermal interaction or kinetics modeling), which the MC normalization/smoothing/Tm pipeline is not designed for

## Inputs

- Raw DSF fluorescence-versus-temperature melting curve data per well (e.g. 96-well exemplar plate)
- Exemplar wells for validation (His-FGF7 and His-FGF10 DSF data)

## Outputs

- normalized_melting_curves.csv — per-well MC curves scaled to the 0-1 range per Equation 1
- tm_unsmoothed_vs_smoothed.csv — paired unsmoothed and smoothed Tm values per well (Tm = temperature at max(FDC), Equation 3)

## How to apply

Load the raw DSF fluorescence-versus-temperature data (e.g. the 96-well exemplar plate, or His-FGF7/His-FGF10 exemplar wells) into Matlab via SimpleDSFviewer's data import. Normalize each well's melting curve by Equation 1: subtract the curve's minimum fluorescence from every point and divide by the (max - min) range, mapping each curve to the 0-1 interval. Optionally apply SimpleDSFviewer's melting curve smoothing routine. Compute the first derivative curve (FDC) and take Tm as the temperature at max(FDC) per Equation 3; run this on both unsmoothed and smoothed curves when assessing whether smoothing changes Tm. Write per-well outputs (normalized curves to normalized_melting_curves.csv; paired Tm values to tm_unsmoothed_vs_smoothed.csv) and verify normalized values lie in [0,1] and that smoothed vs unsmoothed Tm are very similar for folded-protein wells.

## Related tools

- **SimpleDSFviewer** (Matlab GUI implementing MC normalization (Equation 1), curve smoothing, and Tm analysis of DSF melting curves; used to load raw data and preview analyzed results) — github.com/hscsun/SimpleDSFviewer-5.0
- **Matlab** (Runtime environment in which SimpleDSFviewer executes the normalization, smoothing, and FDC-based Tm computation)
- **Matlab Runtime** (Alternative execution route for running SimpleDSFviewer without a full Matlab installation)

## Evaluation signals

- Every normalized fluorescence value in normalized_melting_curves.csv lies within [0, 1], confirming correct min-subtraction/max-division per curve
- The output file exists and tabulates all 96 wells of the exemplar plate
- Tm per well equals the temperature at the maximum of the FDC for both unsmoothed and smoothed curves
- For wells with folded proteins, smoothed-data Tm values are very similar to unsmoothed-data Tm values, showing smoothing does not change Tm
- Normalized and Tm analysis results preview correctly inside the SimpleDSFviewer interface

## Limitations

- DSF data analysis remains a challenge, requiring improved methods — SimpleDSFviewer is one approach among evolving ones
- No changelog found for the SimpleDSFviewer-5.0 repository, so version-specific behavior changes may be undocumented
- Requires Matlab or Matlab Runtime to execute, since SimpleDSFviewer is developed in Matlab

## Evidence
