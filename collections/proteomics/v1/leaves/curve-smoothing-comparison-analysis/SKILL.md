---
name: curve-smoothing-comparison-analysis
description: Use when you have 96-well (or multi-well plate) DSF raw fluorescence
  melting curves and need to verify that a smoothing routine does not shift Tm values
  derived from the FDC maximum before adopting it for Tm analysis.
when_to_use_negative:
- The goal is Tm determination itself rather than validating whether the smoothing
  step biases Tm values
- The input data are already normalized/smoothed processed curves, so no unsmoothed-vs-smoothed
  pairing can be formed
- Analysis targets other DSF outputs (e.g. curve normalization alone) rather than
  FDC-maximum Tm values
edam_operation: http://edamontology.org/operation_2409
tools:
- name: SimpleDSFviewer
  role: Loads the 96-well raw DSF melting curves, provides the melting curve smoothing
    routine, and supports FDC-based Tm computation used for the smoothed-vs-unsmoothed
    comparison
  repo: github.com/hscsun/SimpleDSFviewer-5.0
- name: Matlab
  role: Environment in which the per-well max(FDC) Tm computations are run in the
    Matlab workspace
- name: Matlab Runtime
  role: Allows SimpleDSFviewer to be executed without a Matlab installation when running
    the smoothed-vs-unsmoothed Tm comparison
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/pro.3703
    title: 'SimpleDSFviewer: A tool to analyze and view differential scanning fluorimetry
      data for characterizing protein thermal stability and interactions'
  source_content_hash: sha256:2ce3a94da5da5e48f626e47a6b966225325a0196601a9741990162ebd9fadd99
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
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# curve-smoothing-comparison-analysis

## Summary

A validation skill for differential scanning fluorimetry (DSF) analysis that quantifies whether melting curve smoothing alters Tm estimates. It computes Tm per well as the temperature at the maximum of the first derivative curve (FDC, Equation 3) from both unsmoothed and smoothed melting curves and tabulates the paired values for direct comparison (e.g. Figure 5c of SimpleDSFviewer).

## When to use

Use when you have 96-well (or multi-well plate) DSF raw fluorescence melting curves and need to verify that a smoothing routine does not shift Tm values derived from the FDC maximum before adopting it for Tm analysis.

## When NOT to use

- The goal is Tm determination itself rather than validating whether the smoothing step biases Tm values
- The input data are already normalized/smoothed processed curves, so no unsmoothed-vs-smoothed pairing can be formed
- Analysis targets other DSF outputs (e.g. curve normalization alone) rather than FDC-maximum Tm values

## Inputs

- Raw DSF fluorescence melting curve data for a 96-well plate (exemplar plate data loadable via SimpleDSFviewer)
- SimpleDSFviewer melting curve smoothing routine settings as implemented in the tool

## Outputs

- Per-well paired Tm table (unsmoothed vs smoothed) for all 96 wells
- Comparison file tm_unsmoothed_vs_smoothed.csv reproducing the smoothed-vs-unsmoothed Tm comparison (Figure 5c)

## How to apply

Load the exemplar plate's raw fluorescence melting curves into Matlab via SimpleDSFviewer (or run SimpleDSFviewer under Matlab Runtime). For each of the 96 wells, compute the first derivative curve (FDC) of the unsmoothed melting curve and take Tm as the temperature at max(FDC) per Equation 3. Then apply the SimpleDSFviewer melting curve smoothing routine and recompute the max(FDC) Tm on the smoothed curves. Tabulate the paired unsmoothed vs smoothed Tm values per well and compare the two sets: for wells with folded proteins the Tm values should be very similar, demonstrating that smoothing does not change Tm values. Write the comparison to a CSV such as tm_unsmoothed_vs_smoothed.csv, mirroring Figure 5c of the article.

## Related tools

- **SimpleDSFviewer** (Loads the 96-well raw DSF melting curves, provides the melting curve smoothing routine, and supports FDC-based Tm computation used for the smoothed-vs-unsmoothed comparison) — github.com/hscsun/SimpleDSFviewer-5.0
- **Matlab** (Environment in which the per-well max(FDC) Tm computations are run in the Matlab workspace)
- **Matlab Runtime** (Allows SimpleDSFviewer to be executed without a Matlab installation when running the smoothed-vs-unsmoothed Tm comparison)

## Evaluation signals

- The output table contains paired Tm values for all 96 wells (two Tm columns, one per well row)
- For wells with folded proteins, unsmoothed and smoothed Tm values are very similar (smoothing does not change Tm values)
- Each Tm equals the temperature at the maximum of the first derivative curve (FDC), per Equation 3
- The comparison reproduces the pattern reported in Figure 5c of the article

## Limitations

- The similarity finding is reported for folded proteins; behavior for poorly folded or unfolding-incompatible wells is not established by this comparison
- DSF data analysis remains a challenge requiring improved methods, so the comparison validates only the smoothing step within the SimpleDSFviewer pipeline
- No changelog was found for the SimpleDSFviewer-5.0 repository, so tool version differences in the smoothing routine may not be traceable

## Evidence

- [abstract] integration of smoothing and Tm analysis: "SimpleDSFviewer integrates melting curve (MC) normalization, smoothing, and melting temperature (Tm) analysis and directly previews analyzed data, providing an efficient analysis tool for DSF."
- [abstract] tool implementation and runtime: "SimpleDSFviewer is developed in Matlab, and it is freely available for all users to use in Matlab workspace or with Matlab Runtime."
- [full_text] core finding on smoothing effect: [evidence span withheld — anchor status: not_found]
