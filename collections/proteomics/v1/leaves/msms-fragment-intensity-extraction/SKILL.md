---
name: msms-fragment-intensity-extraction
description: Use when quantifying histone peptides from Mascot .csv output by MS/MS
  total ion intensity, especially when composite spectra contain two isobaric peptides
  sharing the same sequence but distinct PTM localizations that must be deconvoluted.
when_to_use_negative:
- Input is raw Mascot output in .xml extension — that is the input format for Histone
  Coder (PTM localization validation), not isoScale, which requires the Mascot .csv
  file plus the Histone Coder result file
- Peptides do not share the same sequence with distinct PTM localizations (i.e., no
  isobaric co-fragmentation) — FIRR-based deconvolution is specifically for isobaric
  species
- Bottom-up proteomics — the README recommends isoScale for middle-down or top-down
  analyses where single sequences with scrambled combinatorial PTMs are compared directly
edam_operation: http://edamontology.org/operation_3630
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0601
tools:
- name: isoScale
  role: Sums total ion intensity of each Mascot-identified MS/MS spectrum, computes
    FIRR for isobaric co-fragmenting peptides, and normalizes per-histone relative
    abundances; run as isoScale-v2.jar (Java)
  repo: https://github.com/middle-down/Software/blob/master/isoScale-v2.jar?raw=true
- name: Mascot
  role: Database searching engine whose .csv output supplies the identified MS/MS
    spectra and intensities that isoScale quantifies
- name: Histone Coder
  role: Upstream tool whose result file provides validated PTM localizations required
    as isoScale's second input
  repo: https://github.com/middle-down/Software/blob/master/Histone%20Coder.jar?raw=true
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pmic.201400084
    title: Middle‐down hybrid chromatography/tandem mass spectrometry workflow for
      characterization of combinatorial post‐translational modifications in histones
  source_content_hash: sha256:e8e7252527713306f5b95ffd5db533d843028095ea4e878e2a9cb33046d994c5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Histone Coder
  - Mascot
  - isoScale
  edam_operation: http://edamontology.org/operation_3630
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0601
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# msms-fragment-intensity-extraction

## Summary

Quantify peptides from Mascot-identified MS/MS spectra by summing total fragment-ion intensity per spectrum using isoScale, including apportioning composite spectra of co-fragmenting isobaric histone peptides via the fragment ion relative ratio (FIRR). This converts MS/MS identification data into relative abundance of combinatorial histone marks.

## When to use

Use when quantifying histone peptides from Mascot .csv output by MS/MS total ion intensity, especially when composite spectra contain two isobaric peptides sharing the same sequence but distinct PTM localizations that must be deconvoluted.

## When NOT to use

- Input is raw Mascot output in .xml extension — that is the input format for Histone Coder (PTM localization validation), not isoScale, which requires the Mascot .csv file plus the Histone Coder result file
- Peptides do not share the same sequence with distinct PTM localizations (i.e., no isobaric co-fragmentation) — FIRR-based deconvolution is specifically for isobaric species
- Bottom-up proteomics — the README recommends isoScale for middle-down or top-down analyses where single sequences with scrambled combinatorial PTMs are compared directly

## Inputs

- Mascot output data file in .csv extension (Mascot .csv output)
- Histone Coder result file containing validated PTM localizations (output format of Histone Coder)
- User-selected settings: spectrum score filter, fragment ion types, MS/MS ion tolerance, fragmentation mode (CID/HCD or ETD)

## Outputs

- Quantification table (CSV) with per-peptide total MS/MS fragment-ion intensities
- FIRR-based relative abundances apportioned between two isobaric co-fragmenting peptides in composite spectra
- Normalized percentage abundance of each combinatorial histone mark per histone

## How to apply

Load the Mascot .csv data file together with the Histone Coder result file containing validated PTM localizations into isoScale. Apply the spectrum filters selectable from the interface (spectrum score, fragment ion types, MS/MS ion tolerance) and sum the total fragment-ion intensity of each Mascot-identified MS/MS spectrum as the peptide's absolute abundance. For composite spectra of two co-fragmenting isobaric peptides, select the fragmentation mode (CID/HCD or ETD) and the MS/MS ion tolerance, search localization-diagnostic fragment ions, and compute the fragment ion relative ratio (FIRR) to apportion the composite intensity between the two isobaric species. Finally normalize relative abundances by the sum of all quantified peptides per histone to yield percentage abundance of each combinatorial mark, and write the quantification table (CSV). The quantification principle follows Pesavento et al. (Analytical Chemistry, 2006), and the software is recommended for middle-down or top-down analyses suitable for direct comparison of single sequences with scrambled combinatorial PTMs.

## Related tools

- **isoScale** (Sums total ion intensity of each Mascot-identified MS/MS spectrum, computes FIRR for isobaric co-fragmenting peptides, and normalizes per-histone relative abundances; run as isoScale-v2.jar (Java)) — https://github.com/middle-down/Software/blob/master/isoScale-v2.jar?raw=true
- **Mascot** (Database searching engine whose .csv output supplies the identified MS/MS spectra and intensities that isoScale quantifies)
- **Histone Coder** (Upstream tool whose result file provides validated PTM localizations required as isoScale's second input) — https://github.com/middle-down/Software/blob/master/Histone%20Coder.jar?raw=true

## Examples

```
java -jar isoScale-v2.jar --data mascot_output.csv --result histone_coder_result --mode HCD --tolerance 0.02
```

## Evaluation signals

- Each quantified peptide's abundance equals the summed total fragment-ion intensity of its Mascot-identified MS/MS spectrum after score and fragment-ion-type filters are applied
- For the two-peptide demonstration from a single composite MS/MS spectrum, both isobaric peptides carry non-negative FIRR-derived relative abundances that together account for the composite spectrum's total ion intensity (paper reports no numeric acceptance threshold, so a qualitative consistency check is applied)
- Normalized per-histone relative abundances sum to 100% (percentages of all quantified peptides per histone)
- Output CSV uses the standard Brno nomenclature for PTMs (e.g. K4me3K14ac) inherited from the Histone Coder result file
- The chosen MS/MS ion tolerance is appropriate for the data resolution (software supports both high and low resolution data via the tolerance setting)

## Limitations

- The paper reports no numeric acceptance threshold for the FIRR consistency check, so validation of isobaric deconvolution is qualitative
- The software processes only Mascot output files in .csv extension and requires the Histone Coder result file as a companion input
- Recommended for middle-down or top-down analyses; suitability for direct comparison depends on single sequences with scrambled combinatorial PTMs
- Requires Java; the quantification principle is that of Pesavento et al. (Analytical Chemistry, 2006), not an independent method

## Evidence

- [abstract] Quantification principle and FIRR purpose: [evidence span withheld — anchor status: not_found]
