---
name: isobaric-peptide-fragment-ratio-quantification
description: Use when quantifying co-isolated, co-fragmented isobaric histone peptide
  forms (same sequence, distinct PTM localizations) from middle-down or top-down ETD
  MS/MS spectra, where XIC or spectral counting is inadequate.
when_to_use_negative:
- Bottom-up digested peptides where sequence-level distinction (not PTM positional
  isomers) is the goal; the README recommends the software for middle-down or top-down
  analyses with scrambled combinatorial PTMs
- Input is raw Mascot .xml output that has not been validated and nomenclature-converted
  by Histone Coder — isoScale requires the Histone Coder result file as input
edam_operation: http://edamontology.org/operation_3630
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: isoScale
  role: Quantifies peptides from total MS/MS ion intensity and computes FIRR for co-fragmenting
    isobaric peptides; accepts ETD/CID/HCD selection and MS/MS ion tolerance settings
  repo: https://github.com/middle-down/Software/blob/master/isoScale-v2.jar?raw=true
- name: Histone Coder
  role: 'Upstream validation: counts site-determining MS/MS ions to confirm PTM localization
    and converts Mascot output to Brno nomenclature; its result file is a required
    isoScale input'
  repo: https://github.com/middle-down/Software/blob/master/Histone%20Coder.jar?raw=true
- name: Mascot
  role: Database search engine whose .csv output supplies the peptide-spectrum matches
    processed by isoScale
- name: Java
  role: Runtime requirement for activating the Histone Coder and isoScale software
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.201400084
    title: Middle‐down hybrid chromatography/tandem mass spectrometry workflow for
      characterization of combinatorial post‐translational modifications in histones
  source_content_hash: sha256:fbff6b84463d46674d721d169e6e03635c66c5e9bfcfa3247e9391bc5b5ac524
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
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# isobaric-peptide-fragment-ratio-quantification

## Summary

Quantify histone peptides and resolve co-fragmenting isobaric species by computing the fragment ion relative ratio (FIRR) from total MS/MS ion intensity, using isoScale on Mascot/Histone Coder output from middle-down ETD data.

## When to use

Use when quantifying co-isolated, co-fragmented isobaric histone peptide forms (same sequence, distinct PTM localizations) from middle-down or top-down ETD MS/MS spectra, where XIC or spectral counting is inadequate.

## When NOT to use

- Bottom-up digested peptides where sequence-level distinction (not PTM positional isomers) is the goal; the README recommends the software for middle-down or top-down analyses with scrambled combinatorial PTMs
- Input is raw Mascot .xml output that has not been validated and nomenclature-converted by Histone Coder — isoScale requires the Histone Coder result file as input

## Inputs

- Mascot database search output in .csv extension (data file)
- Histone Coder result file (validated PTM-localized PSMs)
- Middle-down or top-down ETD (or CID/HCD) MS/MS spectra of histone tails

## Outputs

- Peptide-level relative abundances based on total MS/MS ion intensity
- Fragment ion relative ratios (FIRR) apportioning intensity among co-fragmenting isobaric species
- Relative abundance tables per replicate for comparison against XIC or spectral counting references

## How to apply

Parse the Mascot database search .csv output together with the Histone Coder result file, so that only validated, PTM-localized peptide-spectrum matches feed quantification. Run isoScale selecting ETD fragmentation and setting the MS/MS ion tolerance for the fragment-ion search used to compute the fragment ion relative ratio (FIRR) of isobaric species. isoScale quantifies each peptide from the total ion intensity of its MS/MS spectra and apportions intensity among co-fragmented isobaric polypeptides sharing the same sequence but distinct PTM localizations, following the principle of Pesavento et al. (Analytical Chemistry, 2006). Normalize resulting relative abundances per replicate before cross-method or cross-condition comparison.

## Related tools

- **isoScale** (Quantifies peptides from total MS/MS ion intensity and computes FIRR for co-fragmenting isobaric peptides; accepts ETD/CID/HCD selection and MS/MS ion tolerance settings) — https://github.com/middle-down/Software/blob/master/isoScale-v2.jar?raw=true
- **Histone Coder** (Upstream validation: counts site-determining MS/MS ions to confirm PTM localization and converts Mascot output to Brno nomenclature; its result file is a required isoScale input) — https://github.com/middle-down/Software/blob/master/Histone%20Coder.jar?raw=true
- **Mascot** (Database search engine whose .csv output supplies the peptide-spectrum matches processed by isoScale)
- **Java** (Runtime requirement for activating the Histone Coder and isoScale software)

## Examples

```
java -jar isoScale-v2.jar --result HistoneCoder_result.csv --data Mascot_output.csv --fragmentation ETD --ion-tolerance 0.02
```

## Evaluation signals

- isoScale quantification should agree with an XIC reference quantification on the majority of target peptide groups (e.g. ≥18 of 22 histone H3 PTM groups in the Suz12-/- benchmark)
- isoScale should quantify more peptide groups than spectral counting, particularly for unmodified and extensively modified peptides where spectral counting is inaccurate
- FIRR values for isobaric pairs should sum so that apportioned intensities reconcile with the total MS/MS ion intensity of the co-fragmented spectra
- Output should cover peptides in the standard Brno nomenclature (e.g. K4me3K14ac) as converted by Histone Coder
- Relative abundances should be normalized per replicate before cross-replicate or cross-method comparison

## Limitations

- Requires Mascot .csv output plus a Histone Coder result file — cannot run on raw vendor files or unvalidated Mascot .xml
- Recommended for middle-down or top-down analyses; not intended for standard bottom-up peptide quantification
- FIRR accuracy depends on an appropriately chosen MS/MS ion tolerance for the fragment-ion search, which the user must set according to data resolution (high vs low resolution)

## Evidence
