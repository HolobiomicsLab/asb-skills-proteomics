---
name: cross-method-quantification-agreement-analysis
description: Use when the same set of peptide groups (e.g. 22 H3 groups spanning 0-3
  acetyl and 0-9 methyl PTM equivalents) has been quantified by two or more label-free
  approaches and you need per-group agreement against a reference quantification.
when_to_use_negative:
- The quantification relies on only one method or no reference quantification exists
  to compare against
- Input spectra are bottom-up digests rather than middle-down/top-down data — isoScale
  is recommended for direct comparison of single sequences with scrambled combinatorial
  PTMs in middle-down or top-down analyses
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: isoScale
  role: Quantifies peptides from the total ion intensity of the MS/MS spectra and
    computes FIRR for co-fragmenting isobaric peptides; run with ETD fragmentation
    and a chosen MS/MS ion tolerance
  repo: https://github.com/middle-down/Software/blob/master/isoScale-v2.jar?raw=true
- name: Histone Coder
  role: Validates PTM localization by counting site-determining MS/MS ions and produces
    the result file required as isoScale input
  repo: https://github.com/middle-down/Software/blob/master/Histone%20Coder.jar?raw=true
- name: Mascot
  role: Database search engine whose .csv output is the data file input for isoScale
- name: Java
  role: Runtime required to activate Histone Coder and isoScale
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.201400084
    title: Middle‐down hybrid chromatography/tandem mass spectrometry workflow for
      characterization of combinatorial post‐translational modifications in histones
  source_content_hash: sha256:4e2d4d23182676d2b42a728db13610a331460f40823d7b80cb02d7538033cbc6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Histone Coder
  - Java
  - Mascot
  - isoScale
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# cross-method-quantification-agreement-analysis

## Summary

Compares label-free quantification approaches (isoScale MS/MS total ion intensity, spectral counting, and extracted ion chromatogram (XIC)) across peptide groups to judge which method best agrees with a reference, using histone PTM peptide groups as the test case.

## When to use

Use when the same set of peptide groups (e.g. 22 H3 groups spanning 0-3 acetyl and 0-9 methyl PTM equivalents) has been quantified by two or more label-free approaches and you need per-group agreement against a reference quantification.

## When NOT to use

- The quantification relies on only one method or no reference quantification exists to compare against
- Input spectra are bottom-up digests rather than middle-down/top-down data — isoScale is recommended for direct comparison of single sequences with scrambled combinatorial PTMs in middle-down or top-down analyses

## Inputs

- Mascot database search output in .csv format
- Histone Coder result file (validated, PTM-localized PSMs)
- Spectral counts per peptide group from three technical replicates
- XIC quantification values per peptide group from three technical replicates

## Outputs

- Comparison table of isoScale, spectral-counting, and XIC relative abundances for all 22 peptide groups (fig3_quantification_comparison.csv)
- Per-group agreement status between isoScale and the XIC reference

## How to apply

Parse the Mascot .csv output together with the Histone Coder result file to obtain validated, PTM-localized peptide-spectrum matches, then run isoScale selecting ETD fragmentation and setting the MS/MS ion tolerance for fragment-ion searching so it quantifies peptides from total ion intensity of the MS/MS spectra and computes the fragment ion relative ratio (FIRR) for co-fragmenting isobaric species. Quantify the same peptide groups by spectral counting and by XIC across the three Suz12-/- technical replicates. Normalize relative abundances per replicate for each of the three approaches, then compare isoScale, XIC, and spectral-counting relative abundances per peptide group and record the agreement status per group. Accept the comparison when isoScale agrees with the XIC reference on at least 18 of the 22 peptide groups, and write the result table to fig3_quantification_comparison.csv.

## Related tools

- **isoScale** (Quantifies peptides from the total ion intensity of the MS/MS spectra and computes FIRR for co-fragmenting isobaric peptides; run with ETD fragmentation and a chosen MS/MS ion tolerance) — https://github.com/middle-down/Software/blob/master/isoScale-v2.jar?raw=true
- **Histone Coder** (Validates PTM localization by counting site-determining MS/MS ions and produces the result file required as isoScale input) — https://github.com/middle-down/Software/blob/master/Histone%20Coder.jar?raw=true
- **Mascot** (Database search engine whose .csv output is the data file input for isoScale)
- **Java** (Runtime required to activate Histone Coder and isoScale)

## Examples

```
java -jar isoScale-v2.jar  # GUI: select ETD fragmentation, set MS/MS ion tolerance, load Histone Coder result file + Mascot .csv data file
```

## Evaluation signals

- isoScale quantification agrees with the XIC reference on at least 18 of the 22 peptide groups (the acceptance threshold from the workflow)
- The comparison table is written as fig3_quantification_comparison.csv with agreement status recorded per peptide group
- isoScale quantifies 18 of the 22 peptide groups while spectral counting fails for unmodified and extensively modified peptides
- Relative abundances are normalized per replicate for all three approaches before comparison across the three technical replicates
- FIRR values are computed for isobaric species co-fragmented in the same MS/MS spectra

## Limitations

- Spectral counting is inaccurate for unmodified and extensively modified peptides, so it is a poor quantification method for those groups
- isoScale is designed for middle-down or top-down analyses; its suitability for comparing sequences with scrambled combinatorial PTMs is the stated scope
- Tool input format is rigid: isoScale requires the Histone Coder result file plus the Mascot .csv data file, and both tools require Java

## Evidence

- [other] isoScale outperformed spectral counting by quantifying 18 of the 22 peptide groups, with quantification values in better agreement with the XIC reference: [evidence span withheld — anchor status: not_found]
