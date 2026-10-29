---
name: dsf-melting-curve-smoothing-comparison
description: Use when you have raw DSF melting curves plus SimpleDSFviewer-smoothed
  counterparts (e.g. His-FGF7 + 50 µM heparin exemplar) and must verify smoothing
  preserves MC shape via per-point difference analysis.
when_to_use_negative:
- You only need melting temperature (Tm) values or MC normalization, not shape-preservation
  checks on smoothing — SimpleDSFviewer handles Tm analysis directly
- You lack both the raw and the smoothed MC versions of the same condition; difference
  analysis requires the paired curves on a shared temperature axis
- Input data come from a DSF analysis pipeline other than SimpleDSFviewer's built-in
  smoothing, so the reported 0.0392 / 2.517 benchmarks do not apply
edam_topics:
- http://edamontology.org/topic_3334
- http://edamontology.org/topic_3391
tools:
- name: SimpleDSFviewer
  role: Provides the built-in MC smoothing function applied to the raw melting curves
    and supplies the exemplar His-FGF7 + 50 µM heparin dataset for the raw-vs-smoothed
    comparison
  repo: github.com/hscsun/SimpleDSFviewer-5.0
- name: Matlab
  role: Environment in which SimpleDSFviewer's smoothing runs and in which per-point
    differences are computed, exported to CSV, and plotted as the Figure 4c surface
    plot
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pro.3703
    title: 'SimpleDSFviewer: A tool to analyze and view differential scanning fluorimetry
      data for characterizing protein thermal stability and interactions'
  source_content_hash: sha256:e138239084e2fec97e8e3a4fb9fa0b50a8419fcc36900ebc2db240faad2eb25f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Matlab
  - SimpleDSFviewer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3334
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# dsf-melting-curve-smoothing-comparison

## Summary

Quantifies point-wise differences between raw and smoothed differential scanning fluorimetry (DSF) melting curves to confirm that SimpleDSFviewer's MC smoothing preserves curve shape, benchmarking against reported extremes (0.0392 for folded protein flat areas; 2.517 for denatured-protein high peaks).

## When to use

Use when you have raw DSF melting curves plus SimpleDSFviewer-smoothed counterparts (e.g. His-FGF7 + 50 µM heparin exemplar) and must verify smoothing preserves MC shape via per-point difference analysis.

## When NOT to use

- You only need melting temperature (Tm) values or MC normalization, not shape-preservation checks on smoothing — SimpleDSFviewer handles Tm analysis directly
- You lack both the raw and the smoothed MC versions of the same condition; difference analysis requires the paired curves on a shared temperature axis
- Input data come from a DSF analysis pipeline other than SimpleDSFviewer's built-in smoothing, so the reported 0.0392 / 2.517 benchmarks do not apply

## Inputs

- Raw DSF melting-curve dataset for His-FGF7 + 50 µM heparin from the SimpleDSFviewer 5.0 package (github.com/hscsun/SimpleDSFviewer-5.0)
- Smoothed melting curves generated with SimpleDSFviewer's built-in MC smoothing function in Matlab
- Shared temperature axis for raw and smoothed curves

## Outputs

- mc_differences.csv with one raw-minus-smoothed difference value per temperature point
- Surface plot of per-point MC differences reproducing Figure 4c
- Maximum differences: 0.0392 (folded, flat area) and 2.517 (denatured, high peaks)

## How to apply

Load the exemplar raw melting-curve dataset for His-FGF7 + 50 µM heparin shipped with the SimpleDSFviewer 5.0 package, plus the smoothed MCs produced with SimpleDSFviewer's built-in MC smoothing function in Matlab. Align raw and smoothed curves on the shared temperature axis and compute the per-point difference (raw minus smoothed) for every temperature point. Scan the difference profile to locate the maximum difference in the folded-protein flat area (expected 0.0392) and the denatured-protein high-peak region (expected 2.517) of the Figure 4c surface plot. Export one difference value per temperature point to mc_differences.csv and render a surface plot of the differences (Matlab plotting or an equivalent Matplotlib surface) to reproduce Figure 4c.

## Related tools

- **SimpleDSFviewer** (Provides the built-in MC smoothing function applied to the raw melting curves and supplies the exemplar His-FGF7 + 50 µM heparin dataset for the raw-vs-smoothed comparison) — github.com/hscsun/SimpleDSFviewer-5.0
- **Matlab** (Environment in which SimpleDSFviewer's smoothing runs and in which per-point differences are computed, exported to CSV, and plotted as the Figure 4c surface plot)

## Examples

```
mc_diff = raw_mc - smoothed_mc; surf(mc_diff) % reproduce Figure 4c; max(mc_diff(folded_idx)) == 0.0392
```

## Evaluation signals

- Computed maximum raw-vs-smoothed difference in the folded-protein flat area equals the reported 0.0392
- Computed maximum raw-vs-smoothed difference in the denatured-protein high-peak region equals the reported 2.517
- mc_differences.csv exists and contains exactly one difference value per temperature point of the aligned curves
- Surface plot of the differences visually reproduces Figure 4c: flat low-amplitude region for the folded protein, high peaks for the denatured protein
- Raw and smoothed curves are confirmed to share the same temperature axis before differencing (no off-by-one point offsets)

## Limitations

- The 0.0392 and 2.517 reference extremes are specific to the exemplar His-FGF7 + 50 µM heparin dataset from the SimpleDSFviewer 5.0 package; other conditions will have different difference magnitudes
- Results depend on SimpleDSFviewer's built-in smoothing implementation in Matlab; a different smoothing algorithm will not reproduce these values
- Repository documentation has no changelog, so version-dependent behavior of the smoothing function cannot be tracked

## Evidence
