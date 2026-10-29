---
name: isobaric-peptide-cofragmentation-splitting
description: Use when MS/MS spectra of histone tails (middle-down/top-down) contain
  co-eluting isobaric peptides that share the same sequence but differ in PTM localization,
  and each species must be apportioned a share of the composite spectrum's total ion
  intensity.
when_to_use_negative:
- Bottom-up shot-gun proteomics data comparing unrelated peptide sequences — the README
  recommends isoScale for middle-down or top-down analyses suitable for direct comparison
  of single sequences with scrambled combinatorial PTMs
- Data not produced by Mascot database searching (inputs must be Mascot .csv output
  plus a Histone Coder result file)
- PTM localizations have not been validated by Histone Coder, so localization-diagnostic
  ions for FIRR apportionment cannot be trusted
edam_operation: http://edamontology.org/operation_3630
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: isoScale
  role: Performs the total-ion-intensity quantification and the FIRR-based splitting
    of co-fragmenting isobaric peptides
  repo: https://github.com/middle-down/Software/blob/master/isoScale-v2.jar?raw=true
- name: Histone Coder
  role: Produces the result file with validated, unambiguous PTM localizations that
    isoScale consumes to define the isobaric species
  repo: https://github.com/middle-down/Software/blob/master/Histone%20Coder.jar?raw=true
- name: Mascot
  role: Database searching engine whose .csv output supplies the MS/MS spectra, fragment-ion
    intensities and peptide identifications
  repo: github.com/middle-down/Software
- name: Java
  role: Runtime requirement to activate the isoScale/Histone Coder jar tools
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1002/pmic.201400084
    title: Middle‐down hybrid chromatography/tandem mass spectrometry workflow for
      characterization of combinatorial post‐translational modifications in histones
  source_content_hash: sha256:380fac2976f18a52bc289d191ffc32a1dee0f6f4fce8d2b607c923568f683e57
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
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# Isobaric peptide co-fragmentation splitting

## Summary

Quantifies histone peptides from Mascot-identified MS/MS spectra by summing total fragment-ion intensity, and splits composite spectra of two co-fragmenting isobaric peptides (same sequence, distinct PTM localizations) using the fragment ion relative ratio (FIRR), as implemented in isoScale from the middle-down proteomics workflow.

## When to use

Use when MS/MS spectra of histone tails (middle-down/top-down) contain co-eluting isobaric peptides that share the same sequence but differ in PTM localization, and each species must be apportioned a share of the composite spectrum's total ion intensity.

## When NOT to use

- Bottom-up shot-gun proteomics data comparing unrelated peptide sequences — the README recommends isoScale for middle-down or top-down analyses suitable for direct comparison of single sequences with scrambled combinatorial PTMs
- Data not produced by Mascot database searching (inputs must be Mascot .csv output plus a Histone Coder result file)
- PTM localizations have not been validated by Histone Coder, so localization-diagnostic ions for FIRR apportionment cannot be trusted

## Inputs

- Mascot .csv output file (data file)
- Histone Coder result file with validated PTM localizations
- User settings: fragmentation mode (CID/HCD or ETD), MS/MS ion tolerance, spectrum score and fragment ion type filters

## Outputs

- Quantification table (CSV) of per-peptide relative abundances, with isobaric species split via FIRR
- Per-histone normalized percentage abundance of each combinatorial mark

## How to apply

Load the Mascot .csv output file together with the Histone Coder result file containing validated PTM localizations into isoScale. Apply the spectrum filters (spectrum score, fragment ion types, MS/MS tolerance) and sum the total fragment-ion intensity of each Mascot-identified MS/MS spectrum as the peptide's absolute abundance. For composite spectra of two isobaric peptides sharing the same sequence but with distinct PTM localizations, select the fragmentation mode (CID/HCD or ETD) and the MS/MS ion tolerance, then search localization-diagnostic fragment ions and compute the fragment ion relative ratio (FIRR, principle from Pesavento et al., Analytical Chemistry 2006) to apportion the composite spectrum intensity between the two isobaric species. Finally normalize relative abundances per histone and write the quantification table (CSV), including the two-peptide demonstration from a single composite MS/MS spectrum.

## Related tools

- **isoScale** (Performs the total-ion-intensity quantification and the FIRR-based splitting of co-fragmenting isobaric peptides) — https://github.com/middle-down/Software/blob/master/isoScale-v2.jar?raw=true
- **Histone Coder** (Produces the result file with validated, unambiguous PTM localizations that isoScale consumes to define the isobaric species) — https://github.com/middle-down/Software/blob/master/Histone%20Coder.jar?raw=true
- **Mascot** (Database searching engine whose .csv output supplies the MS/MS spectra, fragment-ion intensities and peptide identifications) — github.com/middle-down/Software
- **Java** (Runtime requirement to activate the isoScale/Histone Coder jar tools)

## Examples

```
java -jar isoScale-v2.jar
```

## Evaluation signals

- The two demonstrated isobaric peptides carry non-negative FIRR-derived relative abundances that together account for the composite spectrum's total ion intensity (paper reports no numeric acceptance threshold, so a qualitative consistency check applies)
- Output CSV quantification table includes the two-peptide demonstration from a single composite MS/MS spectrum
- Significant binary-mark t-tests (one-tail homoscedastic, <5% significance level) recover ~37 significantly different binary marks in the crude mixture and ~128 in purified H3 between wt and Suz12-/- ESCs
- Significant binary marks show concordant abundance patterns between crude histone mixture and purified H3 preparations, consistent with Fig 4E
- Relative abundances normalize to percentage abundance per histone after summing all quantified peptides for that histone

## Limitations

- The paper reports no numeric acceptance threshold for FIRR-derived apportionment, so validation is qualitative
- Software is recommended for middle-down or top-down analyses; suitability for direct comparison relies on single sequences with scrambled combinatorial PTMs
- Works only for isobaric peptide pairs sharing the same sequence but with distinct PTM localizations; it cannot deconvolute cofragmenting species with different sequences
- Appropriate MS/MS ion tolerance must be chosen manually for the resolution of the data (suitable for high and low resolution data)

## Evidence

- [abstract] Workflow includes quantification of cofragmenting isobaric polypeptide species via the two software tools: [evidence span withheld — anchor status: not_found]
