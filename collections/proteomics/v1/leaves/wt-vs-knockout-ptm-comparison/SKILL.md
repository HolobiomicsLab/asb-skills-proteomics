---
name: wt-vs-knockout-ptm-comparison
description: Use when two genotypes (e.g., wt vs a knockout like Suz12-/-) have been
  quantified for combinatorial histone marks in more than one histone preparation
  (purified H3 vs crude mixture), and you must test which binary marks change significantly
  and whether the preparations agree.
when_to_use_negative:
- Data are bottom-up tryptic histone peptides rather than middle-down histone tails
  — isoScale quantification via FIRR is recommended specifically for middle-down or
  top-down analyses comparing single sequences with scrambled combinatorial PTMs.
- Input is still raw Mascot output in .xml — that must first be processed by Histone
  Coder (localization validation and Brno nomenclature conversion) before binary-mark
  abundance comparison.
- The question concerns single PTMs or three-plus-mark combinatorial states rather
  than co-existing binary (two-mark) marks — this skill's t-test and concordance validation
  are defined on binary marks only.
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: isoScale
  role: Quantifies peptides from total ion intensity of MS/MS spectra and provides
    relative quantification of isobaric cofragmenting peptides via the fragment ion
    relative ratio (FIRR), yielding the binary-mark abundances compared across wt
    and Suz12-/- and across preparations
  repo: https://github.com/middle-down/Software/blob/master/isoScale-v2.jar?raw=true
- name: Histone Coder
  role: Validates unambiguous PTM localization by counting site-determining MS/MS
    ions and converts Mascot output to Brno nomenclature; its result file is a required
    input for isoScale
  repo: https://github.com/middle-down/Software/blob/master/Histone%20Coder.jar?raw=true
- name: Mascot
  role: Database search engine assigning PTM localizations; its .csv output is the
    data file consumed by isoScale
- name: Java
  role: Runtime requirement for activating Histone Coder and isoScale
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/pmic.201400084
    title: Middle‐down hybrid chromatography/tandem mass spectrometry workflow for
      characterization of combinatorial post‐translational modifications in histones
  source_content_hash: sha256:13c7095a29314828c28494d6c4ed99b56e51c0cdc599c6fcd2fb5ee3f3f724a3
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
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# wt-vs-knockout-ptm-comparison

## Summary

Cross-preparation comparison of co-existing binary histone PTM mark abundances between wild-type and Suz12-/- murine ESCs, testing whether significant genotype differences agree between purified histone H3 (higher sensitivity) and crude histone mixture preparations analyzed by middle-down LC-MS/MS with Histone Coder and isoScale.

## When to use

Use when two genotypes (e.g., wt vs a knockout like Suz12-/-) have been quantified for combinatorial histone marks in more than one histone preparation (purified H3 vs crude mixture), and you must test which binary marks change significantly and whether the preparations agree.

## When NOT to use

- Data are bottom-up tryptic histone peptides rather than middle-down histone tails — isoScale quantification via FIRR is recommended specifically for middle-down or top-down analyses comparing single sequences with scrambled combinatorial PTMs.
- Input is still raw Mascot output in .xml — that must first be processed by Histone Coder (localization validation and Brno nomenclature conversion) before binary-mark abundance comparison.
- The question concerns single PTMs or three-plus-mark combinatorial states rather than co-existing binary (two-mark) marks — this skill's t-test and concordance validation are defined on binary marks only.

## Inputs

- isoScale-corrected relative abundance table of binary histone peptide marks for crude histone mixture, wt and Suz12-/- ESCs
- isoScale-corrected relative abundance table of binary histone peptide marks for purified histone H3, wt and Suz12-/- ESCs
- Histone Coder result file (localization-validated Mascot output in Brno nomenclature)
- Mascot .csv output data file

## Outputs

- binary_ptm_abundances.csv containing per-binary-mark relative abundances and t-test significance calls for wt vs Suz12-/- in each preparation
- List of significantly different binary marks per preparation (~37 in crude mixture, ~128 in purified H3)
- Cross-preparation concordance assessment of significant binary marks (Fig 4E-style comparison)

## How to apply

Load the isoScale-quantified relative abundances (derived from total ion intensity of MS/MS spectra, with isobaric cofragmenting peptides deconvoluted via the fragment ion relative ratio, FIRR) for both the crude histone mixture and purified H3 in wt and Suz12-/- samples. For each binary (two-mark) histone peptide in Brno nomenclature (e.g., K4me3K14ac), apply a one-tail homoscedastic t-test at the <5% significance level to identify marks significantly different between genotypes within each preparation. Then intersect the significant binary-mark sets across the two preparations and compare their relative abundance patterns to verify concordance, using purified H3 as the more sensitive preparation against which the crude mixture is judged. Rationale: binary marks significantly different in both experiments should be quantified similarly in both, demonstrating that the middle-down workflow (RP trap column + WCX-HILIC-ETD MS/MS, Histone Coder localization validation, isoScale FIRR correction) is robust across sample complexity.

## Related tools

- **isoScale** (Quantifies peptides from total ion intensity of MS/MS spectra and provides relative quantification of isobaric cofragmenting peptides via the fragment ion relative ratio (FIRR), yielding the binary-mark abundances compared across wt and Suz12-/- and across preparations) — https://github.com/middle-down/Software/blob/master/isoScale-v2.jar?raw=true
- **Histone Coder** (Validates unambiguous PTM localization by counting site-determining MS/MS ions and converts Mascot output to Brno nomenclature; its result file is a required input for isoScale) — https://github.com/middle-down/Software/blob/master/Histone%20Coder.jar?raw=true
- **Mascot** (Database search engine assigning PTM localizations; its .csv output is the data file consumed by isoScale)
- **Java** (Runtime requirement for activating Histone Coder and isoScale)

## Examples

```
java -jar isoScale-v2.jar  # GUI: supply Histone Coder result file + Mascot .csv, choose ETD fragmentation and MS/MS ion tolerance, then t-test exported FIRR-corrected binary-mark abundances (wt vs Suz12-/-)
```

## Evaluation signals

- One-tail homoscedastic t-test at p < 0.05 recovers approximately 37 significantly different binary marks in the crude mixture and approximately 128 in purified H3.
- Binary marks significant in both preparations show concordant relative abundance patterns between crude mixture and purified H3, consistent with Fig 4E.
- The known positive control is reproduced: H3K27me2/me3 shows a seven-fold reduction in Suz12-/- cells, plus significant changes in 16 other single PTMs of histone H3.
- All binary marks are labeled in standard Brno nomenclature (e.g., K4me3K14ac) and trace back to Histone Coder-validated PTM localizations with site-determining ion counts.
- Abundances derive from FIRR-corrected isoScale quantification, so isobaric cofragmenting species with distinct PTM localizations are deconvoluted rather than summed.

## Limitations

- The t-test is one-tail and homoscedastic, assuming equal variances between genotypes; deviations from this assumption can inflate or deflate significance calls.
- FIRR deconvolution quality depends on the chosen MS/MS ion tolerance and fragment ion type filters; mis-set tolerances can misattribute intensity between isobaric species with different PTM localizations.
- Validation covers binary (two-mark) marks; concordance of the crude mixture with purified H3 is not demonstrated here for higher-order combinatorial marks, and purified H3 identified 713 marks versus 256 quantified across H3, H4, and H2A, so coverage differs by preparation.

## Evidence
