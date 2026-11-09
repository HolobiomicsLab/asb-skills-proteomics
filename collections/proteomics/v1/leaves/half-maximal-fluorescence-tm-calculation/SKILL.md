---
name: half-maximal-fluorescence-tm-calculation
description: Use when 96-well DSF fluorescence-versus-temperature melting curves need
  Tm determination by the half-maximal-defluorescence (HMD) method, especially to
  benchmark HMD against FD-derived Tm values and flag two-species or high-background
  failure cases.
when_to_use_negative:
- Melting curves containing a single protein species where the FD maximum alone already
  resolves the Tm — the HMD comparison adds no discriminatory value
- Data that are not DSF fluorescence-versus-temperature melt curves (e.g., CD melts
  or DSC thermograms), since Equation 4 is defined for fluorescence half-maximal defluorescence
- When the goal is only to preview normalized/smoothed curves rather than compute
  and compare Tm values per well
tools:
- name: SimpleDSFviewer
  role: User-friendly interface in which the 96-well melting curves are loaded, normalized,
    smoothed, and analyzed for Tm
  repo: github.com/hscsun/SimpleDSFviewer-5.0
- name: Matlab
  role: Environment in which SimpleDSFviewer runs and where the per-well HMD-vs-FD
    Tm join and comparison table are computed
- name: Matlab Runtime
  role: Standalone runtime allowing SimpleDSFviewer to be executed without a full
    Matlab installation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/pro.3703
    title: 'SimpleDSFviewer: A tool to analyze and view differential scanning fluorimetry
      data for characterizing protein thermal stability and interactions'
  source_content_hash: sha256:416f80f04ab3c37fe8a8a0374009b8f0615439ccadfcc055e0c83cf67687af11
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
yes_rate: 0.6666666666666666
n_claims: 9
---

# half-maximal-fluorescence-tm-calculation

## Summary

Computes melting temperatures (Tm) from differential scanning fluorimetry (DSF) curves using the half-maximal-defluorescence (HMD) method — the temperature at half maximal fluorescence per Equation 4 of the SimpleDSFviewer paper — and joins these values with first-derivative (FD) Tm values to expose HMD failure cases.

## When to use

Use when 96-well DSF fluorescence-versus-temperature melting curves need Tm determination by the half-maximal-defluorescence (HMD) method, especially to benchmark HMD against FD-derived Tm values and flag two-species or high-background failure cases.

## When NOT to use

- Melting curves containing a single protein species where the FD maximum alone already resolves the Tm — the HMD comparison adds no discriminatory value
- Data that are not DSF fluorescence-versus-temperature melt curves (e.g., CD melts or DSC thermograms), since Equation 4 is defined for fluorescence half-maximal defluorescence
- When the goal is only to preview normalized/smoothed curves rather than compute and compare Tm values per well

## Inputs

- Raw 96-well DSF fluorescence-versus-temperature melting curve data (one curve per well)
- Per-well Tm values previously computed by the FD method

## Outputs

- hmd_fd_tm_comparison.csv — per-well table of HMD Tm vs FD Tm with discrepancy annotations (two-species averaged-Tm wells; high-initial-fluorescence wells where HMD is not calculable)

## How to apply

Load the raw 96-well DSF fluorescence-versus-temperature melting curve data into the SimpleDSFviewer/Matlab environment. For each of the 96 wells, calculate the HMD Tm as the temperature at half maximal fluorescence according to Equation 4 of the paper. Join the computed HMD Tm values, well by well, with the Tm values previously obtained by the FD method (where FD takes the maximum of the first derivative as Tm). Flag the two reported discrepancy classes: low-heparin wells where HMD averages two-species curves into a single averaged Tm instead of resolving two peaks, and curves with high initial fluorescence where HMD cannot calculate a Tm although the FD method succeeds. Write the joined per-well HMD-vs-FD table, including discrepancy annotations and not-calculable wells, to hmd_fd_tm_comparison.csv.

## Related tools

- **SimpleDSFviewer** (User-friendly interface in which the 96-well melting curves are loaded, normalized, smoothed, and analyzed for Tm) — github.com/hscsun/SimpleDSFviewer-5.0
- **Matlab** (Environment in which SimpleDSFviewer runs and where the per-well HMD-vs-FD Tm join and comparison table are computed)
- **Matlab Runtime** (Standalone runtime allowing SimpleDSFviewer to be executed without a full Matlab installation)

## Examples

```
addpath('SimpleDSFviewer-5.0'); % load 96-well DSF melt curves, compute per-well HMD Tm (Equation 4: T at half maximal F), join with FD Tm, writetable(...,'hmd_fd_tm_comparison.csv')
```

## Evaluation signals

- hmd_fd_tm_comparison.csv contains exactly 96 rows, one per well, each with both an HMD Tm (where calculable) and the FD Tm
- Low-heparin two-species wells show a single averaged HMD Tm while the FD curve resolves two distinct derivative peaks
- Wells with high initial fluorescence have a missing/not-calculable HMD Tm entry but a successful FD Tm, and are annotated as discrepancy cases
- Recomputing each HMD Tm via Equation 4 (temperature at half maximal fluorescence) on the raw curves reproduces the table values
- Every discrepancy well carries an explicit annotation distinguishing the two failure classes (two-species averaging vs high initial fluorescence)

## Limitations

- The HMD method cannot appropriately handle data with two protein species of different Tm values — it yields an averaged Tm, whereas the FD method takes the maximum of the FD as Tm and can distinguish the two species' peaks
- For melting curves with high initial fluorescence, the HMD method cannot calculate a Tm at all, although the FD method succeeds
- No changelog found for the SimpleDSFviewer repository, so version differences in behavior are not documented

## Evidence
