---
name: per-point-signal-difference-computation
description: Use when you need to verify that MC smoothing preserves DSF melting-curve
  shape by quantifying point-wise raw-vs-smoothed residuals on His-FGF7 + 50 µM heparin
  exemplar data.
when_to_use_negative:
- The analysis goal is Tm estimation or MC normalization rather than quantifying how
  smoothing perturbs the curve shape
- Input data are not DSF melting curves from the SimpleDSFviewer exemplar or an equivalent
  raw/smoothed MC pair on a shared temperature axis
- Only one curve (raw or smoothed) is available — a paired raw-minus-smoothed difference
  cannot be computed
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: SimpleDSFviewer
  role: Provides the built-in MC smoothing function that generates the smoothed melting
    curves and the exemplar His-FGF7 + 50 µM heparin dataset
  repo: github.com/hscsun/SimpleDSFviewer-5.0
- name: Matlab
  role: Runs the smoothing, computes the per-point raw-minus-smoothed differences,
    and renders the Figure 4c difference surface plot
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pro.3703
    title: 'SimpleDSFviewer: A tool to analyze and view differential scanning fluorimetry
      data for characterizing protein thermal stability and interactions'
  source_content_hash: sha256:8226bb0e3a3b527c4f54d437723874d308e3a11c1657ae4549dfedecbda2e9a1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Matlab
  - SimpleDSFviewer
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# per-point-signal-difference-computation

## Summary

Quantifies how much SimpleDSFviewer's built-in MC smoothing alters differential scanning fluorimetry melting curves by computing per-point raw-minus-smoothed differences, then recovering regional maxima in the folded (flat) and denatured (high-peak) areas of the Figure 4c surface plot (0.0392 vs 2.517).

## When to use

Use when you need to verify that MC smoothing preserves DSF melting-curve shape by quantifying point-wise raw-vs-smoothed residuals on His-FGF7 + 50 µM heparin exemplar data.

## When NOT to use

- The analysis goal is Tm estimation or MC normalization rather than quantifying how smoothing perturbs the curve shape
- Input data are not DSF melting curves from the SimpleDSFviewer exemplar or an equivalent raw/smoothed MC pair on a shared temperature axis
- Only one curve (raw or smoothed) is available — a paired raw-minus-smoothed difference cannot be computed

## Inputs

- Exemplar raw DSF melting-curve dataset for His-FGF7 + 50 µM heparin shipped with SimpleDSFviewer 5.0
- Smoothed melting curves generated with SimpleDSFviewer's built-in MC smoothing function (Matlab)
- Shared temperature axis common to raw and smoothed curves

## Outputs

- Per-point raw-minus-smoothed difference series (mc_differences.csv, one difference value per temperature point)
- Regional maximum differences for folded (flat area) and denatured (high-peak) regions
- Surface plot of the differences reproducing Figure 4c (Matlab or Matplotlib)

## How to apply

Load the exemplar raw melting-curve dataset for the His-FGF7 + 50 µM heparin condition from the SimpleDSFviewer 5.0 package (github.com/hscsun/SimpleDSFviewer-5.0) and the corresponding smoothed MCs generated with SimpleDSFviewer's built-in MC smoothing function in Matlab. Align raw and smoothed curves on the shared temperature axis and compute the per-point difference (raw minus smoothed) at every temperature point. Scan the difference profile to locate the maximum difference within the folded-protein flat area and within the denatured-protein high-peak region. Validate by checking the recovered extremes against the reported values of 0.0392 (folded) and 2.517 (denatured), export the differences to mc_differences.csv (one value per temperature point), and reproduce the Figure 4c difference surface plot in Matlab or an equivalent Matplotlib surface plot.

## Related tools

- **SimpleDSFviewer** (Provides the built-in MC smoothing function that generates the smoothed melting curves and the exemplar His-FGF7 + 50 µM heparin dataset) — github.com/hscsun/SimpleDSFviewer-5.0
- **Matlab** (Runs the smoothing, computes the per-point raw-minus-smoothed differences, and renders the Figure 4c difference surface plot)

## Examples

```
diffMC = raw_mc - smoothed_mc; fprintf('folded %.4f denatured %.4f\n', max(diffMC(foldedIdx)), max(diffMC(denaturedIdx))); surf(T, wells, diffMC)
```

## Evaluation signals

- Computed maximum difference in the folded-protein flat area equals 0.0392
- Computed maximum difference in the denatured-protein high-peak region equals 2.517
- mc_differences.csv exists and contains exactly one difference value per temperature point on the shared axis
- Raw and smoothed curves are aligned on the same temperature axis before subtraction (no length or grid mismatch)
- Surface plot of differences visually matches Figure 4c: flat in the folded region, high peaks in the denatured region

## Limitations

- Validated extremes (0.0392 folded, 2.517 denatured) are specific to the His-FGF7 + 50 µM heparin exemplar data; other datasets will yield different maxima
- Difference values depend on SimpleDSFviewer's built-in smoothing implementation in Matlab; equivalent smoothing elsewhere may not reproduce the exact extremes
- No changelog found for the repository, so version-specific behavior of the smoothing function cannot be confirmed

## Evidence

- [results] The largest raw-vs-smoothed MC difference is 0.0392 for the folded protein (flat area of the surface plot) and 2.517 for the denatured protein (high peaks): [evidence span withheld — anchor status: not_found]
