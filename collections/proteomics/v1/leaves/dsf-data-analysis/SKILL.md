---
name: dsf-data-analysis
description: Use when analyzing DSF melting curves (e.g., from His-FGF10 thermal stability
  assays) to normalize, smooth, and extract Tm values from first derivative curves,
  including automated second-Tm recognition and unfolded/denatured classification.
when_to_use_negative:
- Data are raw DSF fluorescence traces not yet normalized or smoothed — run SimpleDSFviewer's
  normalization and smoothing steps first
- The goal is qualitative visual inspection only, without Tm extraction or automated
  peak-based screening
edam_operation: http://edamontology.org/operation_0322
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: SimpleDSFviewer
  role: Matlab-based interface used to normalize, smooth, and analyze DSF melting
    curves and run automated Tm screening with peak detection on first derivative
    curves
  repo: github.com/hscsun/SimpleDSFviewer-5.0
- name: Matlab
  role: Development environment in which SimpleDSFviewer is run for DSF analysis
- name: Matlab Runtime
  role: Alternative runtime allowing SimpleDSFviewer to be used without a Matlab installation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/pro.3703
    title: 'SimpleDSFviewer: A tool to analyze and view differential scanning fluorimetry
      data for characterizing protein thermal stability and interactions'
  source_content_hash: sha256:b9ac6a7c312b8e858ee938a3e421dd9731c590507c5dc96ee12b477bc49616f9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Matlab
  - Matlab Runtime
  - SimpleDSFviewer
  edam_operation: http://edamontology.org/operation_0322
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# dsf-data-analysis

## Summary

Automated differential scanning fluorimetry (DSF) data analysis using SimpleDSFviewer, which integrates melting curve (MC) normalization, smoothing, and melting temperature (Tm) analysis with direct preview of analyzed results. Includes findpeak-based automated screening of first derivative curves (FDCs) to classify melting curves as single-Tm, two-Tm, no-Tm, or unfolded/denatured.

## When to use

Use when analyzing DSF melting curves (e.g., from His-FGF10 thermal stability assays) to normalize, smooth, and extract Tm values from first derivative curves, including automated second-Tm recognition and unfolded/denatured classification.

## When NOT to use

- Data are raw DSF fluorescence traces not yet normalized or smoothed — run SimpleDSFviewer's normalization and smoothing steps first
- The goal is qualitative visual inspection only, without Tm extraction or automated peak-based screening

## Inputs

- Normalized and smoothed first derivative curves (FDCs) from DSF raw fluorescence data
- Melting curves (MCs) from DSF assays, e.g., exemplar His-FGF10 dataset including 8 M urea samples
- DSF raw fluorescence data files for protein thermal stability and interaction experiments

## Outputs

- tm_screening_results.csv with per-curve Tm values, second Tm values, no-Tm flags, and unfolded/denatured classifications with initial temperature
- Classified melting curves (single-Tm, two-Tm, no-Tm, unfolded/denatured)
- Analyzed/previewed DSF data plots within SimpleDSFviewer

## How to apply

Load normalized and smoothed first derivative curves (FDCs) produced by SimpleDSFviewer in Matlab (or via Matlab Runtime). Run peak finding on each FDC and count only peaks whose width and height each exceed 10% of the highest peak in the assay as qualifying peaks; smaller peaks are treated as noise. For curves with one qualifying peak report the corresponding Tm; with two qualifying peaks report both the primary Tm and the second Tm; with four or more recognized peaks flag the curve as 'no Tm calculated'. For curves whose melting curve declines rather than rises, classify the sample as unfolded/denatured and return the initial temperature as the reported value (e.g., 32 C for His-FGF10 in 8 M urea). Write all per-curve outcomes (Tm values, second Tm, no-Tm flags, unfolded classifications) to tm_screening_results.csv.

## Related tools

- **SimpleDSFviewer** (Matlab-based interface used to normalize, smooth, and analyze DSF melting curves and run automated Tm screening with peak detection on first derivative curves) — github.com/hscsun/SimpleDSFviewer-5.0
- **Matlab** (Development environment in which SimpleDSFviewer is run for DSF analysis)
- **Matlab Runtime** (Alternative runtime allowing SimpleDSFviewer to be used without a Matlab installation)

## Evaluation signals

- Exemplar His-FGF10 curve in 8 M urea is classified as unfolded/denatured and returns the initial temperature of 32 C
- Curves with >= 4 recognized peaks carry a 'no Tm calculated' flag in tm_screening_results.csv
- Peaks with width or height below 10% of the highest peak in the assay are excluded (treated as noise) and do not count as qualifying peaks
- Curves with two qualifying peaks yield both a primary Tm and a second Tm; single-peak curves yield one Tm
- Output file tm_screening_results.csv contains per-curve outcomes matching the classification rules

## Limitations

- No changelog found for the SimpleDSFviewer repository, so version-dependent behavior changes cannot be tracked
- Data analysis of DSF data remains a challenge requiring improved methods; the 10% peak threshold and >=4-peak rule are heuristics for this classifier
- Requires Matlab or Matlab Runtime to execute

## Evidence

- [abstract] SimpleDSFviewer integrates melting curve normalization, smoothing, and melting temperature analysis: "SimpleDSFviewer integrates melting curve (MC) normalization, smoothing, and melting temperature (Tm) analysis and directly previews analyzed data, providing an efficient analysis tool for DSF."
- [methods] Peak recognition threshold and classification rule from source card: [evidence span withheld — anchor status: not_found]
- [methods] Unfolded classification with initial temperature: [evidence span withheld — anchor status: not_found]
- [discussion] No changelog in repository: [evidence span withheld — anchor status: not_found]
