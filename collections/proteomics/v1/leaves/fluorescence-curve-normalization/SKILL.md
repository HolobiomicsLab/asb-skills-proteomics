---
name: fluorescence-curve-normalization
description: Use when you have smoothed, normalized DSF melting curves (MCs) sampled
  at a uniform temperature increment and need the first-derivative curve (FDC) whose
  peak marks each well's melting temperature.
when_to_use_negative:
- The melting curves are still raw fluorescence and have not passed SimpleDSFviewer's
  normalization and smoothing steps — the FD formula assumes normalized, smoothed
  MC input
- The assay does not provide a uniform temperature increment (dt) between neighboring
  measurement points, so the central-difference denominator 2*dt is undefined
- You only need Tm peak calling from an already-computed FDC and do not need to reconstruct
  the derivative curve itself
tools:
- name: SimpleDSFviewer
  role: Defines and executes the FD computation (Equation 2 central difference) and
    FDC normalization on the smoothed MCs; source code at the project repository
  repo: github.com/hscsun/SimpleDSFviewer-5.0
- name: Matlab
  role: Implementation language in which the central-difference FDC calculation is
    performed with vectorized array operations; usable in the Matlab workspace
- name: Matlab Runtime
  role: Standalone execution option for running SimpleDSFviewer's FD calculation without
    a Matlab license
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pro.3703
    title: 'SimpleDSFviewer: A tool to analyze and view differential scanning fluorimetry
      data for characterizing protein thermal stability and interactions'
  source_content_hash: sha256:4bb193b6fa480dc2b45b02defa4710c3f90d2497dfc2a6e64fa00edcbc7f4970
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
yes_rate: 1.0
n_claims: 5
---

# fluorescence-curve-normalization

## Summary

Computes the first-derivative curve (FDC) from normalized, smoothed differential scanning fluorimetry (DSF) melting curves using SimpleDSFviewer's central-difference Equation (2), then normalizes the FDC with the same procedure as the MCs so derivative amplitudes are comparable across wells for melting-temperature (Tm) analysis.

## When to use

Use when you have smoothed, normalized DSF melting curves (MCs) sampled at a uniform temperature increment and need the first-derivative curve (FDC) whose peak marks each well's melting temperature.

## When NOT to use

- The melting curves are still raw fluorescence and have not passed SimpleDSFviewer's normalization and smoothing steps — the FD formula assumes normalized, smoothed MC input
- The assay does not provide a uniform temperature increment (dt) between neighboring measurement points, so the central-difference denominator 2*dt is undefined
- You only need Tm peak calling from an already-computed FDC and do not need to reconstruct the derivative curve itself

## Inputs

- Smoothed, normalized exemplar melting curves (MCs) per well from the upstream SimpleDSFviewer smoothing step
- Uniform temperature increment (dt) of the DSF ramp between neighboring measurement points

## Outputs

- Per-well first-derivative curves (FDCs) normalized in the same way as the MCs
- fdc_curves.csv — a single table of per-well FDC series

## How to apply

Load the smoothed, normalized exemplar melting curves (MCs) for every well from the upstream smoothing step of SimpleDSFviewer, which integrates MC normalization, smoothing, and Tm analysis. For each well, compute the first-derivative curve by the central-difference formula FDC(n) = (MC(n+1) - MC(n-1)) / (2*dt), where dt is the uniform temperature increment of the DSF ramp, using vectorized array operations in Matlab (SimpleDSFviewer's implementation language). Boundary points (first and last) lack two neighbors, so handle them explicitly (e.g., pad with NaN) to preserve point count. Normalize each computed FDC with the same normalization procedure applied to the MCs so derivative amplitudes are comparable across wells. Assemble the per-well FDC series into a single table and write it to fdc_curves.csv.

## Related tools

- **SimpleDSFviewer** (Defines and executes the FD computation (Equation 2 central difference) and FDC normalization on the smoothed MCs; source code at the project repository) — github.com/hscsun/SimpleDSFviewer-5.0
- **Matlab** (Implementation language in which the central-difference FDC calculation is performed with vectorized array operations; usable in the Matlab workspace)
- **Matlab Runtime** (Standalone execution option for running SimpleDSFviewer's FD calculation without a Matlab license)

## Examples

```
FDC = [NaN; (MC(3:end) - MC(1:end-2)) / (2*dt); NaN];
```

## Evaluation signals

- Each output FDC series has the same number of points as its source MC
- The maximum of each FDC lies at the derivative peak corresponding to the melting temperature (Tm) of that well
- The central-difference denominator is 2*dt with dt equal to the uniform temperature increment between neighboring measurement points
- FDCs are normalized with the identical procedure used for the MCs, so amplitudes are comparable across wells
- All per-well FDC series are present in fdc_curves.csv with one series per input well

## Limitations

- Boundary points of the MC series have only one neighbor, so the central difference is undefined there and must be handled explicitly (e.g., NaN padding)
- The FD formula assumes a uniform temperature increment between neighboring measurement points; irregular ramps break the 2*dt denominator
- FDC quality depends on the upstream normalization and smoothing of the MCs; noisy or unnormalized MCs propagate into the derivative

## Evidence

- [methods] The FD is computed from the normalized, smoothed melting curve using Equation (2), FDC = (MC(n+1) − MC(n−1)) / (2Δt): [evidence span withheld — anchor status: not_found]
