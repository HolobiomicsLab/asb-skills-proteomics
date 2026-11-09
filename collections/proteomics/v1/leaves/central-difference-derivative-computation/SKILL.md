---
name: central-difference-derivative-computation
description: Use when normalized, smoothed DSF melting curves are available and a
  per-well first-derivative curve is needed to locate the derivative peak corresponding
  to the melting temperature (Tm).
when_to_use_negative:
- Input is raw (un-normalized, un-smoothed) DSF fluorescence data — SimpleDSFviewer's
  MC normalization and smoothing must be applied first
- The goal is Tm estimation from the raw MC directly rather than from a derivative
  peak, since this skill only produces the derivative curve
- The temperature ramp is non-uniform, so Δt is not well-defined between neighboring
  measurement points
tools:
- name: SimpleDSFviewer
  role: Implements the Equation (2) central-difference FDC computation and MC-consistent
    FDC normalization within its DSF analysis interface
  repo: github.com/hscsun/SimpleDSFviewer-5.0
- name: Matlab
  role: Implementation language used to compute the FDC via vectorized array operations,
    runnable in the Matlab workspace
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pro.3703
    title: 'SimpleDSFviewer: A tool to analyze and view differential scanning fluorimetry
      data for characterizing protein thermal stability and interactions'
  source_content_hash: sha256:e4edd374278d7a40fb3dc03b1e512b0da04688018d28537d8c6df43878b00af6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Matlab
  - SimpleDSFviewer
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# central-difference-derivative-computation

## Summary

Computes the first-derivative curve (FDC) of a differential scanning fluorimetry melting curve using the central-difference formula of Equation (2), FDC = (MC(n+1) − MC(n−1)) / (2Δt), then normalizes the FDC with the same procedure applied to the MCs, as implemented in SimpleDSFviewer (Matlab).

## When to use

Use when normalized, smoothed DSF melting curves are available and a per-well first-derivative curve is needed to locate the derivative peak corresponding to the melting temperature (Tm).

## When NOT to use

- Input is raw (un-normalized, un-smoothed) DSF fluorescence data — SimpleDSFviewer's MC normalization and smoothing must be applied first
- The goal is Tm estimation from the raw MC directly rather than from a derivative peak, since this skill only produces the derivative curve
- The temperature ramp is non-uniform, so Δt is not well-defined between neighboring measurement points

## Inputs

- Smoothed, normalized melting curves (MCs) per well from the upstream SimpleDSFviewer smoothing step
- Uniform temperature increment Δt of the DSF ramp between neighboring measurement points

## Outputs

- Per-well first-derivative curves (FDCs) with one point per MC measurement point
- Normalized FDC table (e.g. fdc_curves.csv) with derivative peaks corresponding to per-well melting temperatures

## How to apply

Load the smoothed, normalized melting curves (MCs) for every well from the upstream SimpleDSFviewer normalization/smoothing step. For each well, apply the central-difference formula FDC(n) = (MC(n+1) − MC(n−1)) / (2Δt), where Δt is the temperature difference between neighboring measurement points of the DSF ramp, implemented with vectorized array operations in Matlab (SimpleDSFviewer's implementation language). Normalize each resulting FDC with the same normalization procedure used for the MCs so derivative amplitudes are comparable across wells. Assemble the per-well FDC series into a table (e.g. fdc_curves.csv) and check that each FDC has the same number of points as its source MC and that its maximum sits at the derivative peak marking that well's Tm.

## Related tools

- **SimpleDSFviewer** (Implements the Equation (2) central-difference FDC computation and MC-consistent FDC normalization within its DSF analysis interface) — github.com/hscsun/SimpleDSFviewer-5.0
- **Matlab** (Implementation language used to compute the FDC via vectorized array operations, runnable in the Matlab workspace)

## Examples

```
FDC = [NaN; (MC(3:end,:) - MC(1:end-2,:)) / (2*dt); NaN];
```

## Evaluation signals

- Each output FDC series has exactly the same number of points as its source MC
- Each FDC maximum lies at the derivative peak corresponding to the melting temperature (Tm) of that well
- FDC amplitudes are normalized with the same procedure as the MCs, so derivative magnitudes are comparable across wells
- Endpoint behavior is consistent with the central-difference stencil (no derivative defined for the first and last measurement points, where MC(n+1) or MC(n−1) is absent)

## Limitations

- The formula requires a uniform temperature increment Δt between neighboring measurement points; irregular ramps break the 2Δt denominator
- Edge points (first and last) cannot be computed by the central difference since MC(n±1) is undefined there
- Derivative quality depends on the upstream normalization and smoothing; noisy or un-normalized MCs will propagate into the FDC

## Evidence

- [abstract] SimpleDSFviewer integrates melting curve (MC) normalization, smoothing, and melting temperature (Tm) analysis and directly previews analyzed data: "SimpleDSFviewer integrates melting curve (MC) normalization, smoothing, and melting temperature (Tm) analysis and directly previews analyzed data"
- [abstract] SimpleDSFviewer is developed in Matlab, and it is freely available for all users to use in Matlab workspace or with Matlab Runtime.: "SimpleDSFviewer is developed in Matlab, and it is freely available for all users to use in Matlab workspace or with Matlab Runtime."
- [abstract] However, data analysis remains a challenge, requiring improved methods.: "However, data analysis remains a challenge, requiring improved methods."
