---
name: thermal-denaturation-curve-classification
description: Use when automating DSF Tm screening over many melting curves and needing
  rules to accept secondary Tms, reject noisy peaks, flag uninterpretable curves,
  or detect pre-denatured samples.
when_to_use_negative:
- The FDCs have not been normalized and smoothed first — raw derivative curves will
  produce noise peaks that fail or corrupt the 10% threshold logic
- Curves with 3 recognized peaks fall in a gap between the two-peak (dual Tm) and
  >=4-peak (no Tm) rules; interpret these manually rather than trusting the automated
  flag
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: SimpleDSFviewer
  role: 'Implements the automated screening: normalizes and smooths MCs, computes
    FDCs, runs peak finding, and applies the 10%-threshold and peak-count classification
    rules'
  repo:
  - evidence span withheld — anchor status: not_found
- name: Matlab
  role: Runtime environment in which findpeak-based screening of FDCs executes; SimpleDSFviewer
    can be used in the Matlab workspace
- name: Matlab Runtime
  role: Allows running the compiled SimpleDSFviewer screening workflow without a Matlab
    installation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/pro.3703
    title: 'SimpleDSFviewer: A tool to analyze and view differential scanning fluorimetry
      data for characterizing protein thermal stability and interactions'
  source_content_hash: sha256:f1e5aae72d9f06e7c8fd287f9ccff257c9873386adb190e2f9c8766c96a4308e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Matlab
  - Matlab Runtime
  - SimpleDSFviewer
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# thermal-denaturation-curve-classification

## Summary

Automated classification of DSF melting curves using peak finding on normalized, smoothed first derivative curves (FDCs): peaks must exceed 10% of the highest peak's width and height to qualify, >=4 qualifying peaks flag 'no Tm calculated', and declining MCs are classified as unfolded/denatured with the initial temperature reported.

## When to use

Use when automating DSF Tm screening over many melting curves and needing rules to accept secondary Tms, reject noisy peaks, flag uninterpretable curves, or detect pre-denatured samples.

## When NOT to use

- The FDCs have not been normalized and smoothed first — raw derivative curves will produce noise peaks that fail or corrupt the 10% threshold logic
- Curves with 3 recognized peaks fall in a gap between the two-peak (dual Tm) and >=4-peak (no Tm) rules; interpret these manually rather than trusting the automated flag

## Inputs

- Normalized and smoothed first derivative curves (FDCs) produced by SimpleDSFviewer from raw DSF fluorescence data
- Corresponding melting curves (MCs) for rise/decline classification
- Exemplar His-FGF10 DSF dataset including the 8 M urea sample

## Outputs

- tm_screening_results.csv with per-curve Tm values, second Tm values, no-Tm flags, and unfolded/denatured classifications with initial temperature
- Count of qualifying peaks per FDC (peaks exceeding 10% of the highest peak's width and height)

## How to apply

Run peak finding (findpeak) on each normalized and smoothed first derivative curve (FDC) in Matlab via SimpleDSFviewer. Count a peak as qualifying only if both its width and its height exceed 10% of the highest peak in the assay — smaller peaks are treated as noise. Report the Tm for single-peak curves and both the primary Tm and second Tm for curves with two qualifying peaks; if four or more peaks are recognized, flag the curve as 'no Tm calculated'. For curves whose melting curve (MC) declines rather than rises, classify the sample as unfolded/denatured and report the initial temperature (32 C for the His-FGF10 in 8 M urea exemplar). Write all per-curve outcomes (Tm values, second Tm, no-Tm flags, unfolded classifications) to tm_screening_results.csv.

## Related tools

- **SimpleDSFviewer** (Implements the automated screening: normalizes and smooths MCs, computes FDCs, runs peak finding, and applies the 10%-threshold and peak-count classification rules) — github.com/hscsun/SimpleDSFviewer-5.0
- **Matlab** (Runtime environment in which findpeak-based screening of FDCs executes; SimpleDSFviewer can be used in the Matlab workspace)
- **Matlab Runtime** (Allows running the compiled SimpleDSFviewer screening workflow without a Matlab installation)

## Examples

```
SimpleDSFviewer (in Matlab): load normalized/smoothed FDCs -> run findpeak-based screening -> apply 10% width/height threshold and >=4-peak rule -> export tm_screening_results.csv
```

## Evaluation signals

- The exemplar His-FGF10 curve in 8 M urea is classified as unfolded/denatured and returns the initial temperature of 32 C
- Curves with >=4 recognized peaks on the FDC carry a 'no Tm calculated' flag in tm_screening_results.csv
- Peaks whose width or height is <=10% of the highest peak in the assay are excluded from the qualifying-peak count (treated as noise)
- Curves with exactly two qualifying peaks yield both a primary Tm and a second Tm; single-peak curves yield only one Tm
- Every per-curve outcome in tm_screening_results.csv falls into exactly one of: Tm, Tm + second Tm, no-Tm flag, or unfolded/denatured with initial temperature

## Limitations

- DSF data analysis remains a challenge, requiring improved methods — the threshold rules are heuristic and may not generalize to all proteins
- The second-Tm recognition depends on the 10% width and height threshold relative to the highest peak, so genuine but small secondary transitions below this cutoff are discarded as noise
- The unfolded/denatured classification returns the initial temperature rather than a true Tm, so the reported value is a placeholder, not a stability measurement

## Evidence

- [supplementary] Second Tm recognition rule: [evidence span withheld — anchor status: not_found]
- [supplementary] Repository location: [evidence span withheld — anchor status: not_found]
