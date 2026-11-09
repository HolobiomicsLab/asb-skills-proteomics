---
name: surface-plot-visualization-of-curve-differences
description: Use when raw and smoothed melting curves (e.g. from SimpleDSFviewer's
  built-in MC smoothing in Matlab) must be compared point-wise to quantify and display
  how much smoothing distorts the MC surface.
when_to_use_negative:
- Input is a single melting curve without a paired smoothed version — there is no
  second surface to difference against
- Goal is Tm determination or normalization rather than assessing smoothing artifacts;
  use SimpleDSFviewer's Tm analysis workflow instead
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SimpleDSFviewer
  role: Provides the raw DSF melting-curve data and the built-in MC smoothing function
    whose output is differenced and surface-plotted
  repo: github.com/hscsun/SimpleDSFviewer-5.0
- name: Matlab
  role: Runtime environment in which the smoothing is applied, the per-point differences
    are computed, and the Figure 4c surface plot is rendered (Matplotlib acceptable
    as an equivalent)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pro.3703
    title: 'SimpleDSFviewer: A tool to analyze and view differential scanning fluorimetry
      data for characterizing protein thermal stability and interactions'
  source_content_hash: sha256:5b726ea0726f3f5f1e7d78178168874605fd01c952237100c6c6543ce2e6f253
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Matlab
  - SimpleDSFviewer
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# Surface plot visualization of curve differences

## Summary

Compute point-wise differences between raw and smoothed differential scanning fluorimetry (DSF) melting curves on a shared temperature axis and visualize them as a 2D surface plot (Figure 4c-style), confirming that SimpleDSFviewer's MC smoothing preserves curve shape in flat (folded) regions while admitting larger differences at high-peak (denatured) transitions.

## When to use

Use when raw and smoothed melting curves (e.g. from SimpleDSFviewer's built-in MC smoothing in Matlab) must be compared point-wise to quantify and display how much smoothing distorts the MC surface.

## When NOT to use

- Input is a single melting curve without a paired smoothed version — there is no second surface to difference against
- Goal is Tm determination or normalization rather than assessing smoothing artifacts; use SimpleDSFviewer's Tm analysis workflow instead

## Inputs

- Raw DSF melting-curve dataset (e.g. His-FGF7 + 50 µM heparin exemplar from SimpleDSFviewer 5.0)
- Smoothed melting curves generated with SimpleDSFviewer's built-in MC smoothing function in Matlab
- Shared temperature axis for raw and smoothed curves

## Outputs

- mc_differences.csv with one raw-minus-smoothed difference value per temperature point
- Surface plot of the per-point differences reproducing Figure 4c
- Maximum differences reported separately for the folded-protein flat area and the denatured-protein high-peak region

## How to apply

Load the exemplar raw DSF melting-curve dataset (e.g. His-FGF7 + 50 µM heparin from the SimpleDSFviewer 5.0 package) and the corresponding curves smoothed with SimpleDSFviewer's built-in MC smoothing function in Matlab. Align raw and smoothed curves on the shared temperature axis and compute the per-point difference (raw minus smoothed) at every data point. Scan the difference profile to locate the maximum within the folded-protein flat area and within the denatured-protein high-peak region — exemplar extremes are 0.0392 (folded) and 2.517 (denatured). Export the per-point differences to mc_differences.csv and render a surface plot of the differences (Matlab plotting or an equivalent Matplotlib surface) reproducing the Figure 4c layout. The rationale is that small differences in flat areas versus localized peaks at the transition demonstrate smoothing preserves MC shape while removing noise.

## Related tools

- **SimpleDSFviewer** (Provides the raw DSF melting-curve data and the built-in MC smoothing function whose output is differenced and surface-plotted) — github.com/hscsun/SimpleDSFviewer-5.0
- **Matlab** (Runtime environment in which the smoothing is applied, the per-point differences are computed, and the Figure 4c surface plot is rendered (Matplotlib acceptable as an equivalent))

## Evaluation signals

- Computed maximum difference in the folded-protein flat area equals the reported value 0.0392
- Computed maximum difference in the denatured-protein high-peak region equals the reported value 2.517
- mc_differences.csv exists and contains exactly one difference value per temperature point
- Difference profile shows near-zero values across the flat folded region and localized high peaks only in the denatured transition region
- Surface plot is visually consistent with the Figure 4c surface (flat area of the surface plot for folded, high peaks for denatured)

## Limitations

- The 0.0392 and 2.517 maxima are exemplar-specific to the His-FGF7 + 50 µM heparin dataset; other conditions will yield different extremes
- Requires raw and smoothed curves already aligned on a shared temperature axis; misalignment inflates differences
- No changelog found for the SimpleDSFviewer repository, so version drift in the smoothing implementation is not tracked

## Evidence
