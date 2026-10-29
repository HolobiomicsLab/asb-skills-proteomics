---
name: histone-peptide-ion-intensity-quantification
description: Use when you need relative abundances of hypermodified histone-tail peptides
  (middle-down/top-down data) from Mascot .csv results, especially where isobaric
  peptides with different PTM localizations co-fragment and must be deconvoluted.
when_to_use_negative:
- For standard bottom-down (trypsin-digested) proteomics where isobaric PTM-localization
  scrambling is not the concern — isoScale is explicitly recommended for middle-down
  or top-down analyses
- When raw Mascot output has not been processed by Histone Coder, since isoScale requires
  the Histone Coder result file as one of its two inputs
edam_operation: http://edamontology.org/operation_3630
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: isoScale
  role: Quantifies peptides from total MS/MS ion intensity and performs FIRR-based
    relative quantification of cofragmenting isobaric peptides; consumes the Histone
    Coder result file and Mascot .csv
  repo: https://github.com/middle-down/Software/blob/master/isoScale-v2.jar?raw=true
- name: Histone Coder
  role: 'Predecessor step: validates MS/MS spectra, determines unambiguous PTM localization
    by counting site-determining ions, converts Mascot output to Brno nomenclature,
    and emits the result file consumed by isoScale'
  repo: https://github.com/middle-down/Software/blob/master/Histone%20Coder.jar?raw=true
- name: Mascot
  role: Database-searching engine whose .csv output assigns PTM localizations that
    isoScale quantifies
- name: Java
  role: Runtime requirement for the Histone Coder and isoScale software
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/pmic.201400084
    title: Middle‐down hybrid chromatography/tandem mass spectrometry workflow for
      characterization of combinatorial post‐translational modifications in histones
  source_content_hash: sha256:8b115708572684a50ba061a48603f3b3a38e84ed187a1d6ac7b2d7cae01ecf15
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Histone Coder
  - Java
  - Mascot
  - isoScale
  edam_operation: http://edamontology.org/operation_3630
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# histone-peptide-ion-intensity-quantification

## Summary

Quantify histone peptides from the total ion intensity of MS/MS spectra using isoScale, including deconvolution of cofragmenting isobaric species (same sequence, distinct PTM localizations) via the fragment ion relative ratio (FIRR). This underpins relative abundance comparison of combinatorial histone PTM marks between conditions, e.g. wt vs Suz12-/- murine ESCs.

## When to use

Use when you need relative abundances of hypermodified histone-tail peptides (middle-down/top-down data) from Mascot .csv results, especially where isobaric peptides with different PTM localizations co-fragment and must be deconvoluted.

## When NOT to use

- For standard bottom-down (trypsin-digested) proteomics where isobaric PTM-localization scrambling is not the concern — isoScale is explicitly recommended for middle-down or top-down analyses
- When raw Mascot output has not been processed by Histone Coder, since isoScale requires the Histone Coder result file as one of its two inputs

## Inputs

- Result file from Histone Coder (Mascot .xml output processed into Histone Coder format)
- Mascot database-search output in .csv extension (data file for isoScale)

## Outputs

- Per-peptide total-ion-intensity abundances for histone peptides
- FIRR-based relative abundances for isobaric cofragmenting peptides with distinct PTM localizations
- Quantification tables usable for between-condition statistical testing of combinatorial histone marks (e.g. binary two-mark abundances, binary_ptm_abundances.csv)

## How to apply

Run isoScale with two inputs: the result file produced by Histone Coder and the Mascot .csv output. Quantify each identified polypeptide from the total ion intensity of its MS/MS spectrum; for isobaric peptides sharing the same sequence but distinct PTM localizations, compute the fragment ion relative ratio (FIRR) to obtain relative quantification of the co-fragmenting species. In the isoScale interface, select the fragmentation method used in the experiment (CID/HCD or ETD) and set the MS/MS ion tolerance used to search the fragment ions required for the FIRR calculation, matching high- or low-resolution data. The quantification principle follows Pesavento et al. (Analytical Chemistry, 2006) and is recommended for direct comparison of single sequences with scrambled combinatorial PTMs. Aggregated per-peptide relative abundances can then be tested between conditions (e.g. one-tail homoscedastic t-test at <5% significance) to identify differentially abundant marks.

## Related tools

- **isoScale** (Quantifies peptides from total MS/MS ion intensity and performs FIRR-based relative quantification of cofragmenting isobaric peptides; consumes the Histone Coder result file and Mascot .csv) — https://github.com/middle-down/Software/blob/master/isoScale-v2.jar?raw=true
- **Histone Coder** (Predecessor step: validates MS/MS spectra, determines unambiguous PTM localization by counting site-determining ions, converts Mascot output to Brno nomenclature, and emits the result file consumed by isoScale) — https://github.com/middle-down/Software/blob/master/Histone%20Coder.jar?raw=true
- **Mascot** (Database-searching engine whose .csv output assigns PTM localizations that isoScale quantifies)
- **Java** (Runtime requirement for the Histone Coder and isoScale software)

## Examples

```
java -jar isoScale-v2.jar  # launch isoScale, then load the Histone Coder result file and Mascot .csv data file via the GUI (requires Java)
```

## Evaluation signals

- Each identified peptide receives an abundance derived from total MS/MS ion intensity, not spectral counts
- Isobaric peptides sharing a sequence but differing in PTM localization are split into distinct relative abundances via FIRR rather than summed
- FIRR fragment-ion search settings (ion tolerance, CID/HCD vs ETD) match the acquisition's fragmentation method and mass resolution
- Downstream t-tests (one-tail homoscedastic, <5% significance) recover biologically expected differentials, e.g. ~7-fold reduction of H3K27me2/me3 in Suz12-/- cells and consistent binary-mark results between purified H3 and crude histone mixture preparations

## Limitations

- isoScale only processes Mascot output files in .csv extension and requires the Histone Coder result file; other search engines or raw formats are not supported
- Quantification principle is described in Pesavento et al. (Analytical Chemistry, 2006) for cofragmenting isobaric species; accuracy depends on correct choice of MS/MS ion tolerance and fragment ion types
- Recommended for middle-down or top-down analyses; suitability for other experimental designs is not established

## Evidence
