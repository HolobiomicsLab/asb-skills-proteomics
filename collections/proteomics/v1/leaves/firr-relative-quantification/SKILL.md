---
name: firr-relative-quantification
description: Use when Mascot-identified MS/MS spectra contain two isobaric histone
  peptides sharing the same sequence but distinct PTM localizations, and each species
  needs a separate relative abundance from one composite spectrum.
when_to_use_negative:
- Spectra contain only a single peptide species (no co-fragmenting isobaric pair)
  — plain total-ion-intensity quantification suffices and FIRR is not needed
- Peptide identification/PTM localization has not been performed — Histone Coder/Mascot
  results are required inputs, and PTM localization ambiguity must be resolved before
  FIRR apportionment
- Bottom-up peptide-level workflows with distinct sequences per peak — the README
  recommends isoScale for middle-down or top-down analyses comparing single sequences
  with scrambled combinatorial PTMs
edam_operation: http://edamontology.org/operation_3630
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: isoScale
  role: Quantifies peptides from total MS/MS ion intensity and computes the fragment
    ion relative ratio (FIRR) to apportion composite spectra between co-fragmenting
    isobaric peptides; user selects MS/MS ion tolerance and CID/HCD vs ETD fragmentation
    mode
  repo: https://github.com/middle-down/Software/blob/master/isoScale-v2.jar?raw=true
- name: Mascot
  role: Database search engine whose .csv output supplies the identified MS/MS spectra
    and fragment ion intensities processed by isoScale
- name: Histone Coder
  role: Produces the result file with validated, unambiguously localized PTMs that
    isoScale consumes alongside the Mascot .csv data file
  repo: https://github.com/middle-down/Software/blob/master/Histone%20Coder.jar?raw=true
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pmic.201400084
    title: Middle‐down hybrid chromatography/tandem mass spectrometry workflow for
      characterization of combinatorial post‐translational modifications in histones
  source_content_hash: sha256:7accb2d5a50a3096e5d80dae9cef7cad79e53143025fe53f8bf280a5fe85baaf
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Histone Coder
  - Mascot
  - isoScale
  edam_operation: http://edamontology.org/operation_3630
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# firr-relative-quantification

## Summary

Apportioning the total fragment-ion intensity of a composite MS/MS spectrum between two co-fragmenting isobaric histone peptides (same sequence, distinct PTM localizations) using the fragment ion relative ratio (FIRR) computed by isoScale, as in the Sidoli et al. 2014 middle-down proteomics workflow.

## When to use

Use when Mascot-identified MS/MS spectra contain two isobaric histone peptides sharing the same sequence but distinct PTM localizations, and each species needs a separate relative abundance from one composite spectrum.

## When NOT to use

- Spectra contain only a single peptide species (no co-fragmenting isobaric pair) — plain total-ion-intensity quantification suffices and FIRR is not needed
- Peptide identification/PTM localization has not been performed — Histone Coder/Mascot results are required inputs, and PTM localization ambiguity must be resolved before FIRR apportionment
- Bottom-up peptide-level workflows with distinct sequences per peak — the README recommends isoScale for middle-down or top-down analyses comparing single sequences with scrambled combinatorial PTMs

## Inputs

- Mascot output in .csv extension (data file)
- Histone Coder result file with validated PTM localizations (Brno nomenclature, e.g. K4me3K14ac)

## Outputs

- Quantification table (CSV) of peptide absolute abundances (total MS/MS fragment ion intensity) and FIRR-apportioned relative abundances for the two isobaric species in composite spectra
- Per-histone percentage abundances of each combinatorial histone mark after normalization

## How to apply

Run isoScale on a Mascot .csv data file together with the Histone Coder result file containing validated PTM localizations. isoScale sums the total fragment-ion intensity of each Mascot-identified MS/MS spectrum as the peptide's absolute abundance, then identifies composite spectra holding two isobaric peptides. For each composite spectrum it searches localization-diagnostic fragment ions within the user-selected MS/MS ion tolerance (with the fragmentation mode set to CID/HCD or ETD in the interface) and computes the fragment ion relative ratio (FIRR) to split the spectrum's total intensity between the two species. The principle follows Pesavento et al. (Analytical Chemistry, 2006), and the MS/MS ion tolerance choice makes the FIRR search suitable for both high- and low-resolution data. Per histone, relative abundances are then normalized to yield the percentage abundance of each combinatorial mark.

## Related tools

- **isoScale** (Quantifies peptides from total MS/MS ion intensity and computes the fragment ion relative ratio (FIRR) to apportion composite spectra between co-fragmenting isobaric peptides; user selects MS/MS ion tolerance and CID/HCD vs ETD fragmentation mode) — https://github.com/middle-down/Software/blob/master/isoScale-v2.jar?raw=true
- **Mascot** (Database search engine whose .csv output supplies the identified MS/MS spectra and fragment ion intensities processed by isoScale)
- **Histone Coder** (Produces the result file with validated, unambiguously localized PTMs that isoScale consumes alongside the Mascot .csv data file) — https://github.com/middle-down/Software/blob/master/Histone%20Coder.jar?raw=true

## Examples

```
java -jar isoScale-v2.jar
```

## Evaluation signals

- The two FIRR-derived relative abundances of the demonstrated isobaric peptide pair are both non-negative and together account for the composite spectrum's total fragment ion intensity
- Output quantification table is written as CSV and covers every Mascot-identified MS/MS spectrum, including the two-peptide demonstration from a single composite spectrum
- Diagnostic-fragment search used the user-selected MS/MS ion tolerance consistent with the chosen fragmentation mode (CID/HCD or ETD) in the isoScale interface
- Per-histone relative abundances of all quantified combinatorial marks sum to 100% after normalization
- Inputs are exactly the two required files: the Mascot .csv data file and the Histone Coder result file (isoScale fails/behaves undefined without both)

## Limitations

- FIRR applies only to isobaric peptides that share the same sequence but have distinct PTM localizations co-fragmented in one MS/MS spectrum; it does not separate other kinds of co-eluting species
- The paper reports no numeric acceptance threshold for the two-peptide demonstration, so validation is a qualitative consistency check
- Quality depends on user-chosen filters (spectrum score, fragment ion types, MS/MS ion tolerance); a too-wide tolerance risks including non-diagnostic ions in the FIRR
- Requires Java to run the tools

## Evidence

- [abstract] abstract: [evidence span withheld — anchor status: not_found]
