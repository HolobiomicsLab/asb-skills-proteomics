---
name: dsf-thermal-stability-analysis
description: Use when the input is DSF fluorescence melting-curve data (e.g. a 96-well
  exemplar plate ramp) and the goal is to compute normalized FD curves, extract per-well
  Tm values, or compare Tm from smoothed vs unsmoothed MCs.
when_to_use_negative:
- The input is not DSF melting-curve fluorescence data (e.g. it is a single-endpoint
  fluorescence readout with no temperature ramp, so no FDC or Tm can be defined).
- The temperature points of the ramp are non-uniform, so the central-difference formula
  FDC(n) = (MC(n+1) − MC(n−1)) / (2Δt) with a single Δt is invalid without per-pair
  spacing.
- The goal is protein–interaction quantification beyond thermal stability characterization,
  where this MC/FDC/Tm pipeline alone is not the analysis of record.
tools:
- name: SimpleDSFviewer
  role: User-friendly interface that performs MC normalization, smoothing, FDC computation
    (Equation 2), and Tm analysis with direct preview of analyzed data
  repo: github.com/hscsun/SimpleDSFviewer-5.0
- name: Matlab
  role: Implementation language of SimpleDSFviewer; used to run vectorized central-difference
    FDC calculations in the workspace
- name: Matlab Runtime
  role: Allows running the compiled SimpleDSFviewer without a Matlab installation
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1002/pro.3703
    title: 'SimpleDSFviewer: A tool to analyze and view differential scanning fluorimetry
      data for characterizing protein thermal stability and interactions'
  source_content_hash: sha256:3e30963024d26f0d1ab8a034357b2083f4bad0625ebd7a45f95cac38a0fdffdc
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

# dsf-thermal-stability-analysis

## Summary

Analyze differential scanning fluorimetry (DSF) melting curves with SimpleDSFviewer (a Matlab tool): normalize and smooth melting curves (MCs), compute first-derivative curves (FDCs) by central difference (Equation 2), and extract melting temperatures (Tm = temperature at max of FDC, Equation 3).

## When to use

Use when the input is DSF fluorescence melting-curve data (e.g. a 96-well exemplar plate ramp) and the goal is to compute normalized FD curves, extract per-well Tm values, or compare Tm from smoothed vs unsmoothed MCs.

## When NOT to use

- The input is not DSF melting-curve fluorescence data (e.g. it is a single-endpoint fluorescence readout with no temperature ramp, so no FDC or Tm can be defined).
- The temperature points of the ramp are non-uniform, so the central-difference formula FDC(n) = (MC(n+1) − MC(n−1)) / (2Δt) with a single Δt is invalid without per-pair spacing.
- The goal is protein–interaction quantification beyond thermal stability characterization, where this MC/FDC/Tm pipeline alone is not the analysis of record.

## Inputs

- DSF raw fluorescence melting curve data per well (e.g. 96-well exemplar plate)
- Smoothed, normalized exemplar melting curves (MCs) from the upstream smoothing step
- Uniform temperature increment Δt of the DSF ramp

## Outputs

- Per-well first-derivative curves (FDCs) tabulated as fdc_curves.csv
- Per-well Tm values (temperature at max of FDC)
- Comparison table of unsmoothed vs smoothed Tm per well (tm_unsmoothed_vs_smoothed.csv)

## How to apply

Load the raw or smoothed, normalized exemplar melting curves for every well into SimpleDSFviewer, which is developed in Matlab and runs either in the Matlab workspace or via Matlab Runtime. For each well, compute the first-derivative curve by the central-difference formula FDC(n) = (MC(n+1) − MC(n−1)) / (2Δt), where Δt is the temperature difference between neighboring measurement points of the DSF ramp (implemented with vectorized array operations in Matlab). Normalize each FDC with the same normalization procedure applied to the MCs so derivative amplitudes are comparable across wells. Take Tm as the temperature at max(FDC) per Equation 3; the article's Figure 5c workflow shows that Tm values of folded proteins calculated with smoothed data and unsmoothed data are very similar, so smoothing does not change Tm values. Tabulate per-well FDC series and paired smoothed/unsmoothed Tm values, writing outputs such as fdc_curves.csv and tm_unsmoothed_vs_smoothed.csv.

## Related tools

- **SimpleDSFviewer** (User-friendly interface that performs MC normalization, smoothing, FDC computation (Equation 2), and Tm analysis with direct preview of analyzed data) — github.com/hscsun/SimpleDSFviewer-5.0
- **Matlab** (Implementation language of SimpleDSFviewer; used to run vectorized central-difference FDC calculations in the workspace)
- **Matlab Runtime** (Allows running the compiled SimpleDSFviewer without a Matlab installation)

## Evaluation signals

- Each output FDC series has the same number of points as its source MC.
- The maximum of each FDC lies at the derivative peak corresponding to the melting temperature of that well.
- Tm values from smoothed and unsmoothed MCs are very similar for wells with folded proteins (Figure 5c expectation).
- FDC amplitudes are normalized with the same procedure applied to the MCs, making values comparable across the 96 wells.
- The analysis previews correctly inside SimpleDSFviewer (the tool 'directly previews analyzed data').

## Limitations

- The central-difference derivative assumes a uniform temperature increment Δt between neighboring measurement points.
- Data analysis of DSF remains a challenge requiring improved methods, per the article's abstract.
- No changelog is available for the SimpleDSFviewer-5.0 repository, so version differences must be checked manually.
- SimpleDSFviewer requires either a Matlab installation or the free Matlab Runtime to execute.

## Evidence

- [results] Tm comparison finding for smoothed vs unsmoothed MCs via the FD method: [evidence span withheld — anchor status: not_found]
