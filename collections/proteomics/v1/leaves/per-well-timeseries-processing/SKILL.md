---
name: per-well-timeseries-processing
description: Use when you have normalized, smoothed DSF melting curves per well on
  a uniform temperature ramp and need derivative curves whose peaks locate each well's
  melting temperature (Tm).
when_to_use_negative:
- Input melting curves are raw and have not yet been normalized and smoothed by the
  upstream SimpleDSFviewer step, since Equation 2 is defined on the normalized, smoothed
  MCs
- The DSF acquisition has non-uniform temperature increments, so the constant Δt in
  the central-difference denominator is invalid
- The goal is Tm analysis directly on raw fluorescence rather than derivative-curve-based
  peak identification
edam_operation: http://edamontology.org/operation_2945
tools:
- name: SimpleDSFviewer
  role: Implements the MC normalization, smoothing, and FDC central-difference calculation
    (Equation 2) whose per-well output table is written to fdc_curves.csv
  repo: github.com/hscsun/SimpleDSFviewer-5.0
- name: Matlab
  role: Implementation language in which the vectorized per-well derivative computation
    is run (or via Matlab Runtime)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pro.3703
    title: 'SimpleDSFviewer: A tool to analyze and view differential scanning fluorimetry
      data for characterizing protein thermal stability and interactions'
  source_content_hash: sha256:64c00103102137ad8a3b1fbd60bb0a171098e5e462f70a5cdaca90fc6d29bcb5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Matlab
  - SimpleDSFviewer
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# per-well-timeseries-processing

## Summary

Compute per-well first-derivative curves (FDCs) from DSF melting curves using SimpleDSFviewer's central-difference Equation 2, FDC = (MC(n+1) − MC(n−1)) / (2Δt), then normalize the FDCs exactly like the MCs so derivative amplitudes are comparable across wells.

## When to use

Use when you have normalized, smoothed DSF melting curves per well on a uniform temperature ramp and need derivative curves whose peaks locate each well's melting temperature (Tm).

## When NOT to use

- Input melting curves are raw and have not yet been normalized and smoothed by the upstream SimpleDSFviewer step, since Equation 2 is defined on the normalized, smoothed MCs
- The DSF acquisition has non-uniform temperature increments, so the constant Δt in the central-difference denominator is invalid
- The goal is Tm analysis directly on raw fluorescence rather than derivative-curve-based peak identification

## Inputs

- normalized, smoothed exemplar melting curves (MCs) for every well, from the upstream SimpleDSFviewer smoothing step
- uniform DSF ramp temperature increment Δt between neighboring measurement points

## Outputs

- per-well first-derivative curves (FDCs), normalized like the MCs
- fdc_curves.csv — single table assembling the per-well FDC series

## How to apply

Load the smoothed, normalized melting curves (MCs) for every well produced by the upstream SimpleDSFviewer normalization/smoothing step, which is implemented in Matlab. For each well, compute the first-derivative curve by the central-difference formula FDC(n) = (MC(n+1) − MC(n−1)) / (2·Δt), where Δt is the uniform temperature difference between neighboring measurement points on the DSF ramp; in Matlab this is done with vectorized indexing over the temperature series. Apply the same MC normalization procedure to each computed FDC so derivative amplitudes are comparable across wells. Assemble the per-well FDC series into a single table and write it to fdc_curves.csv; the FDC peak position is the quantity downstream Tm analysis depends on.

## Related tools

- **SimpleDSFviewer** (Implements the MC normalization, smoothing, and FDC central-difference calculation (Equation 2) whose per-well output table is written to fdc_curves.csv) — github.com/hscsun/SimpleDSFviewer-5.0
- **Matlab** (Implementation language in which the vectorized per-well derivative computation is run (or via Matlab Runtime))

## Examples

```
fdc = (mc(3:end) - mc(1:end-2)) / (2*dt);  % Equation 2 central difference per well
```

## Evaluation signals

- Each output FDC series has the same number of points as its source MC series
- Each FDC's maximum lies at the derivative peak corresponding to the melting temperature of that well
- FDC values are normalized with the same procedure applied to the MCs, so amplitudes are comparable across wells
- The central-difference endpoints (first and last point) are accounted for consistently with the n±1 indexing of Equation 2
- fdc_curves.csv contains one FDC series per well in a single assembled table

## Limitations

- The central-difference formula assumes a uniform temperature increment Δt between neighboring measurement points; variable-ramp data would break the Δt denominator
- FDCs are only as good as the upstream normalization and smoothing — noisy or unnormalized MCs propagate into derivative peaks
- No changelog found for the SimpleDSFviewer repository, so version differences in the FD implementation may be undocumented

## Evidence
