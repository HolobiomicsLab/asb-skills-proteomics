---
name: histone-binary-ptm-abundance-statistics
description: Use when comparing relative abundances of binary (two-mark) histone peptides
  between conditions (e.g., wt vs Suz12-/- ESCs) across two histone preparations,
  to identify significantly changed marks with a t-test.
when_to_use_negative:
- The analysis targets single PTMs or global mark counts rather than co-existing binary
  (two-mark) peptides — the task here is specifically cross-preparation consistency
  of binary marks
- Input data are bottom-up histone peptide spectra, since isoScale quantification
  via FIRR is recommended for middle-down or top-down analyses
- Abundance values have not been deconvoluted for isobaric cofragmenting peptides,
  making raw ion intensities incomparable across localization isomers
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0601
tools:
- name: isoScale
  role: Quantifies peptides from total MS/MS ion intensity and computes FIRR to relatively
    quantify isobaric peptides with distinct PTM localizations; consumes the Histone
    Coder result file and Mascot .csv output
  repo: https://github.com/middle-down/Software/blob/master/isoScale-v2.jar?raw=true
- name: Histone Coder
  role: Validates PTM localization by counting site-determining MS/MS ions and converts
    Mascot output to Brno nomenclature, producing the result file required by isoScale
  repo: https://github.com/middle-down/Software/blob/master/Histone%20Coder.jar?raw=true
- name: Mascot
  role: Database search engine assigning PTM localizations; its .csv output is the
    isoScale data file
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/pmic.201400084
    title: Middle‐down hybrid chromatography/tandem mass spectrometry workflow for
      characterization of combinatorial post‐translational modifications in histones
  source_content_hash: sha256:d75d63f8560f8dad55db5b07798c4e8db276462f6def4aac83329a3faf36dd85
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Histone Coder
  - Java
  - Mascot
  - isoScale
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0601
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# histone-binary-ptm-abundance-statistics

## Summary

Statistical comparison of co-existing (binary, two-mark) histone PTM abundances between genotypes, quantified from middle-down MS/MS data with isoScale, to test whether significant marks are consistent across histone preparations (e.g., crude histone mixture vs purified H3).

## When to use

Use when comparing relative abundances of binary (two-mark) histone peptides between conditions (e.g., wt vs Suz12-/- ESCs) across two histone preparations, to identify significantly changed marks with a t-test.

## When NOT to use

- The analysis targets single PTMs or global mark counts rather than co-existing binary (two-mark) peptides — the task here is specifically cross-preparation consistency of binary marks
- Input data are bottom-up histone peptide spectra, since isoScale quantification via FIRR is recommended for middle-down or top-down analyses
- Abundance values have not been deconvoluted for isobaric cofragmenting peptides, making raw ion intensities incomparable across localization isomers

## Inputs

- Histone Coder result file (output of Mascot .xml processing with PTM localization validation)
- Mascot database-search output in .csv extension
- Aggregated binary (two-mark) histone peptide relative abundances for crude histone mixture and purified H3 in wt and Suz12-/- murine ESCs

## Outputs

- binary_ptm_abundances.csv with binary-mark abundances and t-test significance calls
- List of significantly different binary marks per preparation (expected ~37 in crude mixture, ~128 in purified H3)
- Concordance assessment of significant binary marks between preparations (Fig 4E-style comparison)

## How to apply

Run isoScale on the Histone Coder result file plus the Mascot .csv output to quantify peptides by total MS/MS ion intensity, using the fragment ion relative ratio (FIRR) to deconvolute isobaric cofragmenting peptides that share a sequence but differ in PTM localization. Aggregate the corrected relative abundances per binary (two-mark) histone peptide for each preparation (crude mixture and purified H3) and genotype (wt and Suz12-/- murine ESCs). Apply a one-tail homoscedastic t-test at the <5% significance level to each binary mark to find marks significantly different between genotypes in each preparation. Then verify which significant binary marks are concordant between the crude mixture and purified H3 — purified H3 is the more sensitive preparation, so it should recover a superset of the significant marks. Write binary-mark abundances and significance calls to binary_ptm_abundances.csv.

## Related tools

- **isoScale** (Quantifies peptides from total MS/MS ion intensity and computes FIRR to relatively quantify isobaric peptides with distinct PTM localizations; consumes the Histone Coder result file and Mascot .csv output) — https://github.com/middle-down/Software/blob/master/isoScale-v2.jar?raw=true
- **Histone Coder** (Validates PTM localization by counting site-determining MS/MS ions and converts Mascot output to Brno nomenclature, producing the result file required by isoScale) — https://github.com/middle-down/Software/blob/master/Histone%20Coder.jar?raw=true
- **Mascot** (Database search engine assigning PTM localizations; its .csv output is the isoScale data file)

## Evaluation signals

- One-tail homoscedastic t-test at <5% significance recovers approximately 37 significantly different binary marks in the crude histone mixture
- The same test recovers approximately 128 significantly different binary marks in purified H3
- Significant binary marks show concordant abundance patterns between crude mixture and purified H3, consistent with Fig 4E
- binary_ptm_abundances.csv is produced with abundance and significance columns for each binary mark
- Isobaric peptide contributions are deconvoluted via FIRR before aggregation, so localization-isomer abundances sum sensibly

## Limitations

- isoScale is recommended for middle-down or top-down analyses, not bottom-up data
- Purified H3 is the more sensitive preparation; crude mixture analysis detects fewer significant marks, so absence from crude mixture does not contradict the purified-H3 result
- FIRR calculation depends on the chosen MS/MS ion tolerance and fragment ion types selected in the interface (CID/HCD vs ETD)

## Evidence
