---
name: melting-curve-numerical-differentiation
description: Use when you have smoothed, normalized DSF melting curves (fluorescence
  vs temperature per well) and need derivative curves whose peak locates each well's
  melting temperature (Tm).
when_to_use_negative:
- Input melting curves are raw and unprocessed — SimpleDSFviewer requires normalization
  and smoothing before differentiation (Equation 2 operates on smoothed, normalized
  MCs).
- 'Non-uniform temperature sampling: the central-difference formula assumes a fixed
  temperature difference dt between neighboring measurement points.'
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SimpleDSFviewer
  role: Source implementation that integrates melting curve (MC) normalization, smoothing,
    and melting temperature (Tm) analysis; its Equation (2) central-difference routine
    is the reference for this skill
  repo: https://github.com/hscsun/SimpleDSFviewer-5.0
- name: Matlab
  role: Language in which SimpleDSFviewer is developed and in which the central-difference
    FDC computation is implemented/run (directly or via Matlab Runtime)
- name: Matlab Runtime
  role: Execution environment allowing SimpleDSFviewer's FDC computation to run without
    a full Matlab installation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pro.3703
    title: 'SimpleDSFviewer: A tool to analyze and view differential scanning fluorimetry
      data for characterizing protein thermal stability and interactions'
  source_content_hash: sha256:5b98c246a45705c80ae4468eba8b584e26eb2782c05c207193606303985ca66d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hscsun/SimpleDSFviewer-5.0
  tools:
  - Matlab
  - Matlab Runtime
  - SimpleDSFviewer
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# melting-curve-numerical-differentiation

## Summary

Computes the first-derivative curve (FDC) of a differential scanning fluorimetry (DSF) melting curve using the central-difference formula of Equation (2), then normalizes the FDC the same way as the melting curves (MCs), as implemented in SimpleDSFviewer for Tm identification.

## When to use

Use when you have smoothed, normalized DSF melting curves (fluorescence vs temperature per well) and need derivative curves whose peak locates each well's melting temperature (Tm).

## When NOT to use

- Input melting curves are raw and unprocessed — SimpleDSFviewer requires normalization and smoothing before differentiation (Equation 2 operates on smoothed, normalized MCs).
- Non-uniform temperature sampling: the central-difference formula assumes a fixed temperature difference dt between neighboring measurement points.

## Inputs

- Smoothed, normalized melting curves (MCs) per DSF well, fluorescence intensity vs temperature, with uniform temperature increment dt

## Outputs

- Per-well first-derivative curves (FDCs), normalized like the MCs, e.g. assembled as fdc_curves.csv

## How to apply

Load the smoothed, normalized exemplar melting curves (MCs) for every well produced by the upstream normalization/smoothing step of SimpleDSFviewer. For each well, compute the first-derivative curve by the central-difference formula FDC(n) = (MC(n+1) - MC(n-1)) / (2*dt), where dt is the uniform temperature increment between neighboring measurement points of the DSF ramp; implement this with vectorized array operations in Matlab, the language SimpleDSFviewer is written in. Normalize each computed FDC with the same normalization procedure applied to the MCs so derivative amplitudes are comparable across wells. Assemble the per-well FDC series into a single table (e.g. fdc_curves.csv); the FDC maximum marks the derivative peak corresponding to that well's melting temperature, which downstream Tm analysis uses.

## Related tools

- **SimpleDSFviewer** (Source implementation that integrates melting curve (MC) normalization, smoothing, and melting temperature (Tm) analysis; its Equation (2) central-difference routine is the reference for this skill) — https://github.com/hscsun/SimpleDSFviewer-5.0
- **Matlab** (Language in which SimpleDSFviewer is developed and in which the central-difference FDC computation is implemented/run (directly or via Matlab Runtime))
- **Matlab Runtime** (Execution environment allowing SimpleDSFviewer's FDC computation to run without a full Matlab installation)

## Examples

```
FDC = (MC(2:end-1,:) - MC(1:end-2,:)) ./ (2*dt); % central-difference first derivative per well in Matlab, then normalize as for MCs
```

## Evaluation signals

- Each output FDC series has the same number of points as its source MC.
- The maximum of each FDC lies at the derivative peak corresponding to the melting temperature of that well.
- FDC amplitudes are on the same scale across wells because the MC normalization procedure was applied to the derivatives.
- Spot-check the finite-difference arithmetic: FDC(n) equals (MC(n+1) - MC(n-1)) / (2*dt) at interior points, with boundary conventions consistent for n = first and last index.

## Limitations

- Central differencing cannot produce a derivative at the first and last measurement points, so FDC series are effectively shorter at the boundaries unless boundary conventions are chosen.
- Accuracy depends on prior normalization and smoothing of the MCs; derivative curves amplify noise in raw or unsmoothed data.
- The formula assumes a uniform temperature increment dt across the DSF ramp.

## Evidence

- [methods] FDC = (MC(n+1) − MC(n−1)) / (2Δt), where Δt is the temperature difference between neighboring measurement points, and the FDC is normalized in the same way as the MCs.: [evidence span withheld — anchor status: not_found]
