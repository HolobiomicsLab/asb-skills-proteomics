---
name: protein-thermal-stability-data-analysis
description: Use when raw DSF fluorescence-versus-temperature data (e.g. His-FGF7
  ± heparin wells) must be normalized per curve to the 0–1 range, smoothed, and checked
  that smoothing preserves MC shape before Tm analysis.
when_to_use_negative:
- Input curves are already min-max normalized fluorescence values (re-normalization
  would be redundant and alter relative curve positions).
- The goal is Tm determination on already-processed curves rather than the normalization/smoothing
  module itself.
- Data are not fluorescence-versus-temperature DSF curves (e.g. isothermal endpoint
  reads or CD/DSC thermograms).
edam_operation: http://edamontology.org/operation_2409
tools:
- name: SimpleDSFviewer
  role: Provides the MC normalization (Equation 1), MC smoothing, and Tm analysis
    pipeline and previews the analyzed DSF data
  repo: github.com/hscsun/SimpleDSFviewer-5.0
- name: Matlab
  role: Environment in which raw curves are loaded, Equation 1 normalization and smoothing
    are applied, and per-point differences and surface plots are computed/exported
  repo: github.com/hscsun/SimpleDSFviewer-5.0
- name: Matlab Runtime
  role: Alternative execution route for running SimpleDSFviewer's analysis functions
    without a full Matlab installation
  repo: github.com/hscsun/SimpleDSFviewer-5.0
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1002/pro.3703
    title: 'SimpleDSFviewer: A tool to analyze and view differential scanning fluorimetry
      data for characterizing protein thermal stability and interactions'
  source_content_hash: sha256:99ca08077ee4b80626f722266c5f31ea421d450a2141600346ceda651a989679
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Matlab
  - Matlab Runtime
  - SimpleDSFviewer
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# protein-thermal-stability-data-analysis

## Summary

Normalizes and smooths differential scanning fluorimetry (DSF) melting curves per well — min-subtraction with max-division scaling to 0–1 followed by MC smoothing — using SimpleDSFviewer/Matlab to characterize protein thermal stability without distorting the melting-curve shape.

## When to use

Use when raw DSF fluorescence-versus-temperature data (e.g. His-FGF7 ± heparin wells) must be normalized per curve to the 0–1 range, smoothed, and checked that smoothing preserves MC shape before Tm analysis.

## When NOT to use

- Input curves are already min-max normalized fluorescence values (re-normalization would be redundant and alter relative curve positions).
- The goal is Tm determination on already-processed curves rather than the normalization/smoothing module itself.
- Data are not fluorescence-versus-temperature DSF curves (e.g. isothermal endpoint reads or CD/DSC thermograms).

## Inputs

- Raw DSF fluorescence-versus-temperature melting curves per well (e.g. His-FGF7 and His-FGF10 exemplar wells from the SimpleDSFviewer 5.0 package, github.com/hscsun/SimpleDSFviewer-5.0)
- Smoothed melting curves generated with SimpleDSFviewer's built-in MC smoothing function in Matlab (for raw-vs-smoothed validation)

## Outputs

- normalized_melting_curves.csv — per-well melting curves scaled to [0, 1]
- mc_differences.csv — one raw-minus-smoothed difference value per temperature point
- Surface plot of per-point differences reproducing Figure 4c (Matlab or equivalent Matplotlib)

## How to apply

Load the raw fluorescence-versus-temperature data per well into Matlab as handled by SimpleDSFviewer's data import, then apply Equation 1: subtract each curve's minimum fluorescence value from every point and divide by the (max − min) range, mapping each melting curve to the 0–1 interval. Next apply SimpleDSFviewer's built-in MC smoothing function to the normalized curves. Quantify smoothing fidelity by aligning raw and smoothed curves on the shared temperature axis and computing the per-point difference (raw minus smoothed); on the exemplar His-FGF7 + 50 µM heparin data the maximum difference is 0.0392 in the folded-protein flat area and 2.517 in the denatured-protein high-peak region (Figure 4c surface plot), confirming the MC shape is preserved. Export normalized curves and per-point differences, then proceed to melting temperature (Tm) analysis, which SimpleDSFviewer integrates alongside normalization and smoothing.

## Related tools

- **SimpleDSFviewer** (Provides the MC normalization (Equation 1), MC smoothing, and Tm analysis pipeline and previews the analyzed DSF data) — github.com/hscsun/SimpleDSFviewer-5.0
- **Matlab** (Environment in which raw curves are loaded, Equation 1 normalization and smoothing are applied, and per-point differences and surface plots are computed/exported) — github.com/hscsun/SimpleDSFviewer-5.0
- **Matlab Runtime** (Alternative execution route for running SimpleDSFviewer's analysis functions without a full Matlab installation) — github.com/hscsun/SimpleDSFviewer-5.0

## Examples

```
mc_norm = (mc - min(mc)) / (max(mc) - min(mc)); d = mc_raw - mc_smooth; writematrix(d, 'mc_differences.csv');
```

## Evaluation signals

- Every normalized fluorescence value lies within the [0, 1] range, consistent with successful min-max normalization per curve.
- Maximum raw-vs-smoothed difference equals 0.0392 in the folded-protein flat area and 2.517 in the denatured-protein high-peak region on the exemplar His-FGF7 + 50 µM heparin data.
- mc_differences.csv exists with exactly one difference value per temperature point, aligned to the shared temperature axis.
- Difference surface plot visually reproduces Figure 4c: flat surface over the folded region, high peaks over the denatured region.
- normalized_melting_curves.csv exists with one normalized curve per well.

## Limitations

- Smoothing introduces larger point-wise deviations in the denatured-protein high-peak region (up to 2.517) than in the flat folded region (0.0392), so peak-level smoothed values should not be read as exact raw values.
- Normalization is per-curve min-max scaling, so absolute fluorescence intensities across wells are not comparable after normalization.
- No changelog found for the SimpleDSFviewer-5.0 repository, so version drift in analysis behavior is not documented.

## Evidence
