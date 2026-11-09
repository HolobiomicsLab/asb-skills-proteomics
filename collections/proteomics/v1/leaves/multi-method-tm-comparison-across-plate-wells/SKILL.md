---
name: multi-method-tm-comparison-across-plate-wells
description: Use when a full 96-well DSF fluorescence-versus-temperature dataset needs
  Tm values compared between the HMD and FD methods to reveal wells where one method
  misassigns or cannot determine Tm.
when_to_use_negative:
- Curves contain a single protein species with one clean unfolding transition — HMD
  and FD will largely agree and the comparison adds little
- The input is not plate-based DSF melting-curve data (e.g. already-summarized Tm
  tables without raw fluorescence-versus-temperature traces), since Equation 4 half-maximal
  interpolation cannot be recomputed
edam_operation: http://edamontology.org/operation_0250
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: SimpleDSFviewer
  role: Loads and previews the 96-well DSF melting curves and supports the HMD half-maximal-fluorescence
    Tm calculation per Equation 4 for the comparison
  repo: github.com/hscsun/SimpleDSFviewer-5.0
- name: Matlab
  role: Runtime environment in which SimpleDSFviewer is developed and run to compute
    and join the per-well HMD and FD Tm values
  repo: github.com/hscsun/SimpleDSFviewer-5.0
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/pro.3703
    title: 'SimpleDSFviewer: A tool to analyze and view differential scanning fluorimetry
      data for characterizing protein thermal stability and interactions'
  source_content_hash: sha256:ab6573c2768671f12d46ae46df3519e418e4c6892c8cbbbb45823b579c216e28
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Matlab
  - SimpleDSFviewer
  edam_operation: http://edamontology.org/operation_0250
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# multi-method-tm-comparison-across-plate-wells

## Summary

Computes and joins per-well melting temperatures (Tm) from the HMD (half-maximal fluorescence) and FD (first-derivative maximum) methods across a 96-well differential scanning fluorimetry (DSF) plate, flagging discrepancy classes such as two-species curves that HMD averages and high-initial-fluorescence curves where HMD fails but FD succeeds.

## When to use

Use when a full 96-well DSF fluorescence-versus-temperature dataset needs Tm values compared between the HMD and FD methods to reveal wells where one method misassigns or cannot determine Tm.

## When NOT to use

- Curves contain a single protein species with one clean unfolding transition — HMD and FD will largely agree and the comparison adds little
- The input is not plate-based DSF melting-curve data (e.g. already-summarized Tm tables without raw fluorescence-versus-temperature traces), since Equation 4 half-maximal interpolation cannot be recomputed

## Inputs

- Raw 96-well DSF fluorescence-versus-temperature melting curve data (loadable in SimpleDSFviewer/Matlab)
- Per-well FD-method Tm values (previously computed, Tm = temperature of maximum first derivative)

## Outputs

- hmd_fd_tm_comparison.csv — per-well joined HMD vs FD Tm table with discrepancy annotations (two-species averaged-Tm wells, high-initial-fluorescence wells) and not-calculable wells

## How to apply

Load the raw 96-well DSF fluorescence-versus-temperature melting curves into SimpleDSFviewer (Matlab) and, for each well, compute the HMD Tm as the temperature at half maximal fluorescence according to Equation 4 of the paper. Join the computed HMD Tm values, well by well, with Tm values previously obtained by the FD method, where Tm is taken as the maximum of the first derivative. Flag two discrepancy classes: low-heparin wells whose curves contain two protein species with different Tm values — HMD yields a single averaged Tm while FD resolves the two species' peaks — and melting curves with high initial fluorescence where HMD cannot calculate a Tm although FD succeeds. Write the joined per-well HMD-vs-FD table with annotations and not-calculable wells to hmd_fd_tm_comparison.csv.

## Related tools

- **SimpleDSFviewer** (Loads and previews the 96-well DSF melting curves and supports the HMD half-maximal-fluorescence Tm calculation per Equation 4 for the comparison) — github.com/hscsun/SimpleDSFviewer-5.0
- **Matlab** (Runtime environment in which SimpleDSFviewer is developed and run to compute and join the per-well HMD and FD Tm values) — github.com/hscsun/SimpleDSFviewer-5.0

## Examples

```
% In Matlab with SimpleDSFviewer on path: SimpleDSFviewer; % load 96-well melting curves, compute HMD Tm (Eq. 4), join with FD Tm, writetable(joined, 'hmd_fd_tm_comparison.csv')
```

## Evaluation signals

- hmd_fd_tm_comparison.csv contains exactly one row per well for all 96 wells of the plate
- Two-species (low-heparin) wells show a single averaged HMD Tm but two distinguishable FD peaks, matching the reported discrepancy class
- High-initial-fluorescence wells are annotated as HMD not-calculable while an FD Tm value is present for the same well
- HMD Tm values correspond to the temperature at half maximal fluorescence per Equation 4 and FD Tm values correspond to the maximum of the first derivative
- Discrepancy annotations are present for both reported classes and not-calculable wells are explicitly marked rather than silently dropped

## Limitations

- The HMD method cannot appropriately handle data with two protein species of different Tm values, yielding a single averaged Tm instead of resolving both
- For melting curves with high initial fluorescence, HMD cannot calculate a Tm even though the FD method succeeds

## Evidence
