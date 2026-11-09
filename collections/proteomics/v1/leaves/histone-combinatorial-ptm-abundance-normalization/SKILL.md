---
name: histone-combinatorial-ptm-abundance-normalization
description: Use when MS/MS spectra of hypermodified histone tails have been identified
  by Mascot (e.g. middle-down or top-down analyses with scrambled combinatorial PTMs)
  and you need relative abundance per combinatorial mark.
when_to_use_negative:
- Bottom-up analyses where single sequences with scrambled combinatorial PTMs are
  not directly comparable — the authors recommend isoScale for middle-down or top-down
  analyses
- Quantification from MS1 precursor intensities or chromatographic peak areas rather
  than MS/MS total ion intensity
- Spectra whose PTM localizations have not been validated by Histone Coder (site-determining
  ion counting), since the result file is a required input
edam_operation: http://edamontology.org/operation_3630
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: isoScale
  role: Performs MS/MS total-ion-intensity quantification, FIRR-based splitting of
    co-fragmenting isobaric peptides, and per-histone normalization to percentage
    abundance; consumes Mascot .csv and Histone Coder result files
  repo: https://github.com/middle-down/Software/blob/master/isoScale-v2.jar?raw=true
- name: Histone Coder
  role: Produces the result file (output format) that supplies validated PTM localizations
    needed before quantification
  repo: https://github.com/middle-down/Software/blob/master/Histone%20Coder.jar?raw=true
- name: Mascot
  role: Database search engine whose .csv output provides the identified MS/MS spectra
    and scores that are quantified and filtered
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pmic.201400084
    title: Middle‐down hybrid chromatography/tandem mass spectrometry workflow for
      characterization of combinatorial post‐translational modifications in histones
  source_content_hash: sha256:c2bcb1bf8107a0f673a629e0bda837510ea6a3a7e7532ac3be85ef2881ee5707
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
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# histone-combinatorial-ptm-abundance-normalization

## Summary

Quantifies Mascot-identified histone tail peptides by summing total fragment-ion intensity of each MS/MS spectrum (including FIRR-based splitting of co-fragmenting isobaric species) and normalizes each peptide's intensity by the sum of all quantified peptides of that histone to yield percentage abundance of each combinatorial PTM mark.

## When to use

Use when MS/MS spectra of hypermodified histone tails have been identified by Mascot (e.g. middle-down or top-down analyses with scrambled combinatorial PTMs) and you need relative abundance per combinatorial mark.

## When NOT to use

- Bottom-up analyses where single sequences with scrambled combinatorial PTMs are not directly comparable — the authors recommend isoScale for middle-down or top-down analyses
- Quantification from MS1 precursor intensities or chromatographic peak areas rather than MS/MS total ion intensity
- Spectra whose PTM localizations have not been validated by Histone Coder (site-determining ion counting), since the result file is a required input

## Inputs

- Mascot .csv output file (data file)
- Histone Coder result file containing validated PTM localizations in Brno nomenclature
- User-selected spectrum score filter, fragment ion types, MS/MS ion tolerance, and fragmentation mode (CID/HCD or ETD)

## Outputs

- Quantification table (CSV) of relative peptide abundances per histone
- Percentage abundance of each combinatorial PTM mark per histone
- FIRR-derived relative abundances for two isobaric peptides apportioned from a single composite MS/MS spectrum

## How to apply

Run isoScale on two inputs: a Histone Coder result file holding validated PTM localizations (Brno nomenclature, e.g. K4me3K14ac) and the Mascot .csv output file. From the interface, set filters for spectrum score and fragment ion types, choose the fragmentation mode (CID/HCD or ETD), and set the MS/MS ion tolerance used to find localization-diagnostic fragment ions. The software sums the total ion intensity of each identified MS/MS spectrum as the peptide's absolute abundance; for composite spectra containing two isobaric peptides with identical sequence but distinct PTM localizations, it computes the fragment ion relative ratio (FIRR) to apportion the composite intensity between the two species. Finally, intensities are normalized by the sum of all quantified peptides for each histone, so every combinatorial mark is expressed as a percentage of the histone's total signal. The quantification principle follows Pesavento et al. (Analytical Chemistry, 2006).

## Related tools

- **isoScale** (Performs MS/MS total-ion-intensity quantification, FIRR-based splitting of co-fragmenting isobaric peptides, and per-histone normalization to percentage abundance; consumes Mascot .csv and Histone Coder result files) — https://github.com/middle-down/Software/blob/master/isoScale-v2.jar?raw=true
- **Histone Coder** (Produces the result file (output format) that supplies validated PTM localizations needed before quantification) — https://github.com/middle-down/Software/blob/master/Histone%20Coder.jar?raw=true
- **Mascot** (Database search engine whose .csv output provides the identified MS/MS spectra and scores that are quantified and filtered)

## Examples

```
java -jar isoScale-v2.jar --data mascot_output.csv --result histone_coder_result.csv --fragmentation ETD --ion-tolerance <tolerance> --filters score,ion_types
```

## Evaluation signals

- For each histone (e.g. H3, H4, H2A), the normalized percentage abundances of all quantified combinatorial marks should sum to 100%
- For a composite spectrum of two isobaric peptides, the two FIRR-derived relative abundances must be non-negative and together equal the composite spectrum's total fragment-ion intensity
- The quantification CSV should only contain spectra passing the selected spectrum score and fragment ion type filters
- Quantified marks should carry Brno nomenclature PTM strings (e.g. K4me3K14ac) consistent with the Histone Coder result file
- Ground-truth comparison: the paper's application reproduced a seven-fold reduction of H3K27me2/me3 in Suz12-/- murine ESCs

## Limitations

- The software is Java-based and designed around Mascot output (.xml for Histone Coder, .csv for isoScale); other search engines' formats are not handled
- The paper reports no numeric acceptance threshold for the two-peptide demonstration from a composite spectrum — only a qualitative consistency check is applied
- FIRR computation depends on the user-chosen MS/MS ion tolerance and fragmentation mode (CID/HCD vs ETD), so results are sensitive to these settings
- No changelog was found for the software, and no numeric acceptance threshold is reported for validating FIRR apportionment

## Evidence
