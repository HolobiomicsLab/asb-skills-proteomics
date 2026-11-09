---
name: heparin-protein-thermal-shift-interpretation
description: Decision support for choosing between HMD and FD Tm determination when
  analyzing 96-well DSF melting curves — especially heparin-binding thermal shift
  data where curves may contain two protein species of different Tm values or high
  initial fluorescence.
when_to_use_negative:
- The melting curve is a simple single-species transition — HMD and FD will agree
  and the comparison adds no information.
- The goal is routine normalization/smoothing/previewing of melting curves rather
  than HMD-vs-FD Tm discrepancy assessment (SimpleDSFviewer's core MC normalization,
  smoothing, and Tm-analysis functions suffice).
- Input data are not DSF fluorescence-versus-temperature melting curves (e.g. already-summarized
  Tm tables with no raw curves).
tools:
- name: SimpleDSFviewer
  role: User-friendly Matlab interface for loading, analyzing, and viewing the 96-well
    DSF melting curves; integrates melting curve (MC) normalization, smoothing, and
    melting temperature (Tm) analysis.
  repo: github.com/hscsun/SimpleDSFviewer-5.0
- name: Matlab
  role: Development/runtime environment in which SimpleDSFviewer is executed to compute
    HMD Tm values per Equation 4 and join them with FD Tm values.
  repo: github.com/hscsun/SimpleDSFviewer-5.0
- name: Matlab Runtime
  role: Alternative execution mode for SimpleDSFviewer for users without a full Matlab
    installation.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/pro.3703
    title: 'SimpleDSFviewer: A tool to analyze and view differential scanning fluorimetry
      data for characterizing protein thermal stability and interactions'
  source_content_hash: sha256:d70e41bacd0226985bda0b5a44663bb86329c97321cecfc68abe523b8eb64069
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
n_claims: 10
---

# heparin-protein-thermal-shift-interpretation

## Summary

Interprets 96-well differential scanning fluorimetry (DSF) melting curves of heparin–protein interactions by comparing HMD (half-maximal denaturation) Tm values against FD (first derivative) Tm values, flagging the two known HMD failure modes: curves containing two protein species with different Tm values (where HMD yields a single averaged Tm) and high-initial-fluorescence curves where HMD cannot calculate a Tm at all.

## When to use

Decision support for choosing between HMD and FD Tm determination when analyzing 96-well DSF melting curves — especially heparin-binding thermal shift data where curves may contain two protein species of different Tm values or high initial fluorescence.

## When NOT to use

- The melting curve is a simple single-species transition — HMD and FD will agree and the comparison adds no information.
- The goal is routine normalization/smoothing/previewing of melting curves rather than HMD-vs-FD Tm discrepancy assessment (SimpleDSFviewer's core MC normalization, smoothing, and Tm-analysis functions suffice).
- Input data are not DSF fluorescence-versus-temperature melting curves (e.g. already-summarized Tm tables with no raw curves).

## Inputs

- Raw 96-well DSF fluorescence-versus-temperature melting curve data (compatible with SimpleDSFviewer/Matlab)
- Previously computed FD-method Tm values per well

## Outputs

- hmd_fd_tm_comparison.csv — per-well joined HMD vs FD Tm table with discrepancy annotations
- Annotations flagging low-heparin two-species wells (HMD-averaged Tm) and high-initial-fluorescence wells where HMD is not calculable

## How to apply

Load the raw 96-well fluorescence-versus-temperature melting curve data into the SimpleDSFviewer/Matlab environment. For each well, compute the HMD Tm as the temperature at half maximal fluorescence according to Equation 4 of the paper, then join these values well-by-well with the Tm values previously obtained by the FD method, which takes the maximum of the first derivative as Tm. Annotate the two documented discrepancy classes: low-heparin wells where HMD averages a two-species curve into a single Tm instead of resolving the two peaks, and curves with high initial fluorescence where HMD cannot calculate a Tm although FD succeeds. Write the joined per-well comparison table with discrepancy annotations to hmd_fd_tm_comparison.csv. The rationale is that FD peak maxima can distinguish two species' peaks while HMD collapses them to an averaged Tm, so FD is the appropriate default for mixed-species and high-background curves.

## Related tools

- **SimpleDSFviewer** (User-friendly Matlab interface for loading, analyzing, and viewing the 96-well DSF melting curves; integrates melting curve (MC) normalization, smoothing, and melting temperature (Tm) analysis.) — github.com/hscsun/SimpleDSFviewer-5.0
- **Matlab** (Development/runtime environment in which SimpleDSFviewer is executed to compute HMD Tm values per Equation 4 and join them with FD Tm values.) — github.com/hscsun/SimpleDSFviewer-5.0
- **Matlab Runtime** (Alternative execution mode for SimpleDSFviewer for users without a full Matlab installation.)

## Examples

```
Load raw 96-well melting curves in SimpleDSFviewer under Matlab Runtime, compute HMD Tm per Equation 4 for each well, then join with FD Tm and write hmd_fd_tm_comparison.csv.
```

## Evaluation signals

- The output file hmd_fd_tm_comparison.csv exists and contains one joined row per well across all 96 wells.
- HMD Tm for each well equals the temperature at half maximal fluorescence per Equation 4 of the paper.
- The two documented discrepancy classes are annotated: low-heparin wells where HMD yields a single averaged Tm for two species, and high-initial-fluorescence wells where HMD Tm is not calculable but FD succeeds.
- For two-species wells, the FD Tm takes the maximum of the first derivative and distinguishes the two peaks, whereas the HMD value is an average — the table should make this difference visible.
- Every well where HMD cannot calculate a Tm is explicitly included (flagged) rather than silently dropped.

## Limitations

- The HMD method cannot appropriately handle data with two protein species of different Tm values — it yields a single averaged Tm rather than resolving both species.
- Melting curves with high initial fluorescence prevent the HMD method from calculating a Tm, even though the FD method (maximum of the first derivative) succeeds.
- DSF data analysis remains a challenge requiring improved methods, so the HMD/FD comparison covers known failure modes but is not a complete solution.

## Evidence

- [results] HMD averaging of two-species curves and FD peak resolution: [evidence span withheld — anchor status: not_found]
- [results] HMD failure on high initial fluorescence: [evidence span withheld — anchor status: not_found]
