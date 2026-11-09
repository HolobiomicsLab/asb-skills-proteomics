---
name: fluorescence-melting-curve-quality-interpretation
description: 'Decision support for validating DSF melting-temperature calls across
  a multi-well plate: when per-well HMD and FD Tm values must be joined and discrepancy
  classes (averaged two-species Tm, not-calculable Tm) flagged before reporting results.'
when_to_use_negative:
- Melting curves containing only a single protein species with low initial fluorescence
  — HMD and FD will agree and no discrepancy analysis is needed
- Analysis goals that need only normalized or smoothed curves (SimpleDSFviewer's MC
  normalization/smoothing) rather than Tm-method cross-validation
- Datasets from instruments/file formats other than the DSF fluorescence melting curves
  supported by SimpleDSFviewer/Matlab
tools:
- name: SimpleDSFviewer
  role: Interface in which the 96-well DSF melting curves are loaded, HMD Tm values
    are computed per well, and curves are previewed for quality interpretation
  repo: github.com/hscsun/SimpleDSFviewer-5.0
- name: Matlab
  role: Runtime environment in which SimpleDSFviewer executes the melting-curve analysis
    and the HMD-vs-FD comparison workflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/pro.3703
    title: 'SimpleDSFviewer: A tool to analyze and view differential scanning fluorimetry
      data for characterizing protein thermal stability and interactions'
  source_content_hash: sha256:8135da1018244534905a2431ece6a851b027ac348363830568c6f536e0d29a14
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
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# fluorescence-melting-curve-quality-interpretation

## Summary

Interprets differential scanning fluorimetry (DSF) melting curves by comparing half-maximal-derivative (HMD) Tm values against first-derivative (FD) Tm values to flag curves the HMD method handles incorrectly, e.g. two-species curves that yield a single averaged Tm or high-initial-fluorescence curves where Tm is not calculable.

## When to use

Decision support for validating DSF melting-temperature calls across a multi-well plate: when per-well HMD and FD Tm values must be joined and discrepancy classes (averaged two-species Tm, not-calculable Tm) flagged before reporting results.

## When NOT to use

- Melting curves containing only a single protein species with low initial fluorescence — HMD and FD will agree and no discrepancy analysis is needed
- Analysis goals that need only normalized or smoothed curves (SimpleDSFviewer's MC normalization/smoothing) rather than Tm-method cross-validation
- Datasets from instruments/file formats other than the DSF fluorescence melting curves supported by SimpleDSFviewer/Matlab

## Inputs

- Raw 96-well DSF fluorescence-versus-temperature melting curve data
- Per-well FD-method Tm values previously computed (max of first derivative)

## Outputs

- Joined per-well HMD vs FD Tm comparison table (e.g. hmd_fd_tm_comparison.csv) with discrepancy annotations (averaged two-species Tm; not-calculable wells)

## How to apply

Load the raw 96-well DSF fluorescence-versus-temperature melting curves into SimpleDSFviewer in the Matlab workspace. For each well, compute the HMD Tm as the temperature at half maximal fluorescence (Equation 4 of the paper) and join it, well by well, with the Tm previously obtained by the FD method, which defines Tm as the temperature of maximum derivative. Annotate the two discrepancy classes described in the paper: (1) curves containing two protein species with different Tm values, where HMD averages the species into a single Tm while FD resolves two peaks by taking the maximum of the FD; and (2) melting curves with high initial fluorescence, where HMD cannot calculate a Tm although the FD method succeeds. Write the per-well HMD-vs-FD comparison table including these annotations so downstream users see which wells' HMD calls are trustworthy and which require FD-based or manual inspection.

## Related tools

- **SimpleDSFviewer** (Interface in which the 96-well DSF melting curves are loaded, HMD Tm values are computed per well, and curves are previewed for quality interpretation) — github.com/hscsun/SimpleDSFviewer-5.0
- **Matlab** (Runtime environment in which SimpleDSFviewer executes the melting-curve analysis and the HMD-vs-FD comparison workflow)

## Evaluation signals

- The output table contains one row per well of the 96-well plate with both HMD and FD Tm columns populated (or explicitly flagged as not-calculable)
- Two-species wells with different Tm values show a single HMD Tm that sits between the two FD-derived peak temperatures (an averaged Tm), and are annotated as such
- Wells with high initial fluorescence are flagged as 'not calculable' under HMD even though the FD method yields a Tm
- For single-species wells, HMD and FD Tm values agree within expected experimental variation, confirming the join was done well-by-well
- Recomputing the HMD Tm as the temperature at half maximal fluorescence (Equation 4) reproduces the values in the output table

## Limitations

- The HMD method cannot appropriately handle data with two protein species of different Tm values — it yields an averaged Tm instead of resolving both peaks
- Melting curves with high initial fluorescence prevent the HMD method from calculating a Tm, although the FD method succeeds
- The repository has no changelog, so version differences in behavior cannot be traced

## Evidence

- [other] HMD fails on two-species curves; FD takes the maximum of the FD as Tm: [evidence span withheld — anchor status: not_found]
