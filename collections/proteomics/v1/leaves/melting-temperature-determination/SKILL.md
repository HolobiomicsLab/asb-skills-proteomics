---
name: melting-temperature-determination
description: Use when raw 96-well DSF fluorescence-versus-temperature melting curves
  must be converted to per-well Tm values via the first-derivative method, with smoothing,
  second-Tm detection, and unfolded-curve classification.
when_to_use_negative:
- Curves known to contain two protein species of different Tm values must not rely
  on the HMD method, which returns a single averaged Tm — the FD method is required
  instead
- Data that are not DSF fluorescence melting curves (e.g. DSC thermograms or already-summarized
  Tm tables) offer no melting curve on which to run the FDC maximum analysis
- Curves with extremely high initial fluorescence where the HMD method cannot calculate
  a Tm should be routed to the FD method rather than re-run with HMD settings
edam_topics:
- http://edamontology.org/topic_0154
tools:
- name: SimpleDSFviewer
  role: User-friendly Matlab interface that performs melting curve normalization,
    smoothing, and Tm analysis, including findpeak-based automated screening of first
    derivative curves
  repo: github.com/hscsun/SimpleDSFviewer-5.0
- name: Matlab
  role: Environment in which SimpleDSFviewer runs to compute FDC maxima, apply the
    10% peak threshold rule, and tabulate per-well Tm values
- name: Matlab Runtime
  role: Allows SimpleDSFviewer to be used without a full Matlab installation when
    running the Tm analysis workflow
provenance:
  source_task_ids:
  - task_005
  - task_006
  - task_007
  source_papers:
  - doi: 10.1002/pro.3703
    title: 'SimpleDSFviewer: A tool to analyze and view differential scanning fluorimetry
      data for characterizing protein thermal stability and interactions'
  source_content_hash: sha256:2f0d8e5b60beb8d81f96683352d7b314671d0e313f5c3c3bd5b7e1d8a42769fd
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Matlab
  - Matlab Runtime
  - SimpleDSFviewer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# melting-temperature-determination

## Summary

Determine melting temperatures (Tm) from differential scanning fluorimetry (DSF) melting curves using the first-derivative curve (FDC) maximum method, as implemented and automated in SimpleDSFviewer (developed in Matlab), including second-Tm recognition and unfolded/denatured classification.

## When to use

Use when raw 96-well DSF fluorescence-versus-temperature melting curves must be converted to per-well Tm values via the first-derivative method, with smoothing, second-Tm detection, and unfolded-curve classification.

## When NOT to use

- Curves known to contain two protein species of different Tm values must not rely on the HMD method, which returns a single averaged Tm — the FD method is required instead
- Data that are not DSF fluorescence melting curves (e.g. DSC thermograms or already-summarized Tm tables) offer no melting curve on which to run the FDC maximum analysis
- Curves with extremely high initial fluorescence where the HMD method cannot calculate a Tm should be routed to the FD method rather than re-run with HMD settings

## Inputs

- Raw 96-well DSF fluorescence-versus-temperature melting curve data (e.g. exemplar His-FGF10 plate, including 8 M urea wells)
- SimpleDSFviewer-normalized and smoothed first derivative curves (FDCs) exported to Matlab

## Outputs

- Per-well Tm table (FD-method Tm, optionally smoothed vs unsmoothed pairs and second Tm values)
- Screening annotations: second-Tm flags, 'no Tm calculated' flags (>=4 recognized peaks), unfolded/denatured classifications with initial temperature
- CSV tables such as tm_unsmoothed_vs_smoothed.csv, hmd_fd_tm_comparison.csv, tm_screening_results.csv

## How to apply

Load the raw 96-well DSF fluorescence melting curve data (e.g. exemplar His-FGF10 plate) into SimpleDSFviewer in the Matlab workspace or via Matlab Runtime, which integrates melting curve normalization, smoothing, and Tm analysis with direct preview. For each well, compute the first derivative curve (FDC) and take Tm as the temperature at max(FDC) per Equation 3; smoothing of the MC can be applied without materially changing Tm (smoothed vs unsmoothed Tm values are very similar for folded proteins). In the automated findpeak screening, count a peak as a second Tm only if its width AND height each exceed 10% of the highest peak in the assay; peaks below that threshold are treated as noise. If four or more peaks are recognized on an FDC, flag 'no Tm calculated'; if the melting curve declines rather than rises, classify the sample as unfolded/denatured and return the initial temperature (e.g. 32 C for His-FGF10 in 8 M urea). Prefer the FD method over the half-maximal-distance (HMD, Equation 4) method when curves may contain two protein species of different Tm values (HMD averages them into one Tm) or high initial fluorescence (HMD cannot calculate a Tm).

## Related tools

- **SimpleDSFviewer** (User-friendly Matlab interface that performs melting curve normalization, smoothing, and Tm analysis, including findpeak-based automated screening of first derivative curves) — github.com/hscsun/SimpleDSFviewer-5.0
- **Matlab** (Environment in which SimpleDSFviewer runs to compute FDC maxima, apply the 10% peak threshold rule, and tabulate per-well Tm values)
- **Matlab Runtime** (Allows SimpleDSFviewer to be used without a full Matlab installation when running the Tm analysis workflow)

## Evaluation signals

- Smoothed and unsmoothed FD-method Tm values for folded-protein wells are very similar (smoothing should not change Tm) — compare the paired columns of tm_unsmoothed_vs_smoothed.csv
- A second Tm is only reported when both width and height of a secondary FDC peak exceed 10% of the highest peak; smaller peaks must be ignored as noise
- Curves with >= 4 recognized FDC peaks carry a 'no Tm calculated' flag in tm_screening_results.csv
- The exemplar His-FGF10 curve in 8 M urea is classified as unfolded/denatured and returns the initial temperature of 32 C
- Wells with two protein species show distinct FDC peaks under the FD method, whereas the HMD comparison table exposes a single averaged Tm as the discrepancy

## Limitations

- The HMD method (Equation 4) cannot appropriately handle data with two protein species of different Tm values, yielding an averaged Tm
- Melting curves with high initial fluorescence prevent HMD Tm calculation even though the FD method succeeds
- DSF data analysis remains a challenge, requiring improved methods

## Evidence
