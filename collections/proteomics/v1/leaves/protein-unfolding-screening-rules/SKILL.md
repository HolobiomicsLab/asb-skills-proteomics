---
name: protein-unfolding-screening-rules
description: Use when classifying DSF melting curves from normalized, smoothed first
  derivative curves (FDCs) — to decide whether a small peak is a genuine second Tm
  versus noise, whether a curve yields no Tm, or whether a sample is already unfolded/denatured.
when_to_use_negative:
- Input is raw DSF fluorescence rather than normalized, smoothed first derivative
  curves (FDCs) — the 10% width/height threshold is defined on FDC peaks, not raw
  traces
- The goal is deconvoluting complex multi-transition curves with expert manual inspection
  — the rule simply flags curves with >=4 recognized peaks as 'no Tm calculated' rather
  than fitting them
- The analysis requires thermodynamic parameters beyond Tm (e.g. ΔH, folding kinetics)
  — this skill only classifies curves and reports Tm / second Tm / unfolded status
tools:
- name: SimpleDSFviewer
  role: 'Runs the automated data screening: peak finding on FDCs, applies the 10%
    width/height qualifying rule and the >=4-peak no-Tm rule, detects declining MCs
    (unfolded), and writes tm_screening_results.csv'
  repo: https://github.com/hscsun/SimpleDSFviewer-5.0
- name: Matlab
  role: Development and execution environment in which SimpleDSFviewer's findpeak-based
    screening is run
- name: Matlab Runtime
  role: Alternative execution path allowing SimpleDSFviewer's screening to run without
    a full Matlab installation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/pro.3703
    title: 'SimpleDSFviewer: A tool to analyze and view differential scanning fluorimetry
      data for characterizing protein thermal stability and interactions'
  source_content_hash: sha256:20c282be404f9712e98d251e507bf85430eed3f72e90df5c8143ccae5e2645bc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hscsun/SimpleDSFviewer-5.0
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
yes_rate: 0.8888888888888888
n_claims: 9
---

# protein-unfolding-screening-rules

## Summary

Encodes SimpleDSFviewer's automated DSF screening rules: a peak on a first derivative curve (FDC) qualifies only when its width and height each exceed 10% of the highest peak, >=4 qualifying peaks flag 'no Tm calculated', and a declining melting curve calls the sample unfolded/denatured and returns the initial temperature.

## When to use

Use when classifying DSF melting curves from normalized, smoothed first derivative curves (FDCs) — to decide whether a small peak is a genuine second Tm versus noise, whether a curve yields no Tm, or whether a sample is already unfolded/denatured.

## When NOT to use

- Input is raw DSF fluorescence rather than normalized, smoothed first derivative curves (FDCs) — the 10% width/height threshold is defined on FDC peaks, not raw traces
- The goal is deconvoluting complex multi-transition curves with expert manual inspection — the rule simply flags curves with >=4 recognized peaks as 'no Tm calculated' rather than fitting them
- The analysis requires thermodynamic parameters beyond Tm (e.g. ΔH, folding kinetics) — this skill only classifies curves and reports Tm / second Tm / unfolded status

## Inputs

- Normalized and smoothed first derivative curves (FDCs) from SimpleDSFviewer (Matlab workspace or Matlab Runtime)
- Corresponding melting curves (MCs) to detect declining/decline-rather-than-rise behavior
- Exemplar His-FGF10 DSF dataset including the 8 M urea sample (validation input)

## Outputs

- Per-curve Tm values and second Tm calls
- 'No Tm calculated' flags for curves with >=4 recognized peaks
- Unfolded/denatured classifications with the initial temperature as the reported value (e.g. 32 C for 8 M urea His-FGF10)
- tm_screening_results.csv aggregating all per-curve screening outcomes

## How to apply

Load the normalized and smoothed FDCs produced by SimpleDSFviewer in Matlab and run peak finding on each curve. Count only peaks whose width and height each exceed 10% of the highest peak in the assay as qualifying peaks; sub-threshold peaks are treated as noise. For a single qualifying peak report the corresponding Tm; for two qualifying peaks report both the primary Tm and the second Tm. If four or more peaks are recognized on a curve, flag it as 'no Tm calculated'. For curves whose melting curve (MC) declines rather than rises, classify the sample as unfolded/denatured and return the initial temperature as the reported value (e.g. 32 C for His-FGF10 in 8 M urea). Write all per-curve outcomes (Tm values, second Tm, no-Tm flags, unfolded classifications with initial temperature) to tm_screening_results.csv.

## Related tools

- **SimpleDSFviewer** (Runs the automated data screening: peak finding on FDCs, applies the 10% width/height qualifying rule and the >=4-peak no-Tm rule, detects declining MCs (unfolded), and writes tm_screening_results.csv) — https://github.com/hscsun/SimpleDSFviewer-5.0
- **Matlab** (Development and execution environment in which SimpleDSFviewer's findpeak-based screening is run)
- **Matlab Runtime** (Alternative execution path allowing SimpleDSFviewer's screening to run without a full Matlab installation)

## Evaluation signals

- The exemplar His-FGF10 curve in 8 M urea is classified as unfolded/denatured and returns the initial temperature of 32 C
- Curves with >=4 recognized peaks carry a 'no Tm calculated' flag in tm_screening_results.csv
- A sub-tallest peak is called a second Tm only when both its width and height exceed 10% of the highest peak in the assay; below-threshold peaks are not reported
- Single-qualifying-peak curves yield exactly one Tm; two-qualifying-peak curves yield both primary Tm and second Tm
- tm_screening_results.csv exists and contains one outcome row per curve with Tm, second Tm, no-Tm flag, and unfolded classification fields

## Limitations

- Peaks below the 10% width/height threshold are treated as noise, so genuine but small secondary transitions may be discarded
- Curves with four or more recognized peaks yield no Tm at all — the rule flags rather than resolves complex multi-transition curves
- For declining melting curves, the reported value is the initial temperature (e.g. 32 C for His-FGF10 in 8 M urea), which is an unfolded-status marker, not a measured Tm

## Evidence

- [methods] Ten percent threshold rule: [evidence span withheld — anchor status: not_found]
