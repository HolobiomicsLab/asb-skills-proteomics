---
name: isobaric-peptide-fragment-ion-ratio-correction
description: Use when middle-down or top-down histone MS/MS data contain isobaric
  polypeptide species that co-fragment — i.e., identical peptide sequences with distinct
  PTM localizations (e.g., K4me3 vs K14ac positional isomers) — and relative abundances
  per localization are needed.
when_to_use_negative:
- Bottom-up digest data with uniquely localized, non-isobaric peptides — standard
  peptide intensity quantification suffices, and isoScale is explicitly recommended
  for middle-down or top-down analyses
- PTM localization has not yet been validated — cofragmented isobaric species must
  first be resolved by Histone Coder site-determining ion counting before FIRR apportionment
  is meaningful
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0601
tools:
- name: isoScale
  role: Performs the total-ion-intensity quantification and calculates the fragment
    ion relative ratio (FIRR) for cofragmenting isobaric histone tail peptides
  repo: https://github.com/middle-down/Software/blob/master/isoScale-v2.jar?raw=true
- name: Histone Coder
  role: 'Upstream step: counts site-determining MS/MS ions to unambiguously localize
    PTMs and converts Mascot output to Brno nomenclature, producing the result file
    consumed by isoScale'
  repo: https://github.com/middle-down/Software/blob/master/Histone%20Coder.jar?raw=true
- name: Mascot
  role: Database search engine whose .csv output supplies the identified polypeptides
    for quantification
- name: Java
  role: Runtime required to execute the Histone Coder and isoScale .jar tools
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pmic.201400084
    title: Middle‐down hybrid chromatography/tandem mass spectrometry workflow for
      characterization of combinatorial post‐translational modifications in histones
  source_content_hash: sha256:715ca93932c06bbbb6678c5669b0c0a747827dae8877ac8c154315c9ae9651da
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Histone Coder
  - Iso Scale
  - Java
  - Mascot
  - isoScale
  - pandas
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0601
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# isobaric-peptide-fragment-ion-ratio-correction

## Summary

Deconvolves cofragmenting isobaric histone tail peptides (same sequence, distinct PTM localizations) by computing the fragment ion relative ratio (FIRR) from MS/MS spectra, enabling relative quantification of positional isomers in middle-down proteomics.

## When to use

Use when middle-down or top-down histone MS/MS data contain isobaric polypeptide species that co-fragment — i.e., identical peptide sequences with distinct PTM localizations (e.g., K4me3 vs K14ac positional isomers) — and relative abundances per localization are needed.

## When NOT to use

- Bottom-up digest data with uniquely localized, non-isobaric peptides — standard peptide intensity quantification suffices, and isoScale is explicitly recommended for middle-down or top-down analyses
- PTM localization has not yet been validated — cofragmented isobaric species must first be resolved by Histone Coder site-determining ion counting before FIRR apportionment is meaningful

## Inputs

- Histone Coder result file (validated PTM localizations in Brno nomenclature, derived from Mascot .xml output)
- Mascot database-search output in .csv format (data file for isoScale)
- Raw MS/MS spectra acquired with ETD (or CID/HCD) fragmentation at high mass accuracy

## Outputs

- Relative quantification of individual histone tail peptide forms based on total ion intensity of MS/MS spectra
- FIRR-based relative abundances for co-fragmented isobaric peptides sharing the same sequence but with distinct PTM localizations
- Aggregated single-PTM relative abundance tables (e.g., percentages of H3K27me2/me3, H3K36me2, H3K9me2 across samples)

## How to apply

Run isoScale (isoScale-v2.jar from the middle-down/Software repo) on two inputs: a result file produced by Histone Coder (which validated PTM localization by counting site-determining MS/MS ions and converted Mascot output to Brno nomenclature such as K4me3K14ac) and a Mascot .csv data file. Select the fragmentation type used in the experiment (CID/HCD or ETD) and set the MS/MS ion tolerance appropriate to the instrument resolution, since FIRR is calculated by searching fragment ions within that tolerance. isoScale then quantifies peptides by total ion intensity of the MS/MS spectra and computes the fragment ion relative ratio (FIRR) to apportion intensity between isobaric species sharing the same sequence but differing in PTM position, following the principle of Pesavento et al. (Analytical Chemistry, 2006). Aggregate the resulting per-peptide-form intensities over all forms carrying a given mark to obtain relative single-PTM abundances (e.g., H3K27me2, H3K36me2) for downstream wt vs knockout comparisons.

## Related tools

- **isoScale** (Performs the total-ion-intensity quantification and calculates the fragment ion relative ratio (FIRR) for cofragmenting isobaric histone tail peptides) — https://github.com/middle-down/Software/blob/master/isoScale-v2.jar?raw=true
- **Histone Coder** (Upstream step: counts site-determining MS/MS ions to unambiguously localize PTMs and converts Mascot output to Brno nomenclature, producing the result file consumed by isoScale) — https://github.com/middle-down/Software/blob/master/Histone%20Coder.jar?raw=true
- **Mascot** (Database search engine whose .csv output supplies the identified polypeptides for quantification)
- **Java** (Runtime required to execute the Histone Coder and isoScale .jar tools)

## Examples

```
java -jar isoScale-v2.jar --result Histone_Coder_result.csv --data mascot_output.csv --fragmentation ETD
```

## Evaluation signals

- The MS/MS ion tolerance and fragmentation mode (ETD vs CID/HCD) selected in isoScale match the acquisition settings of the dataset
- FIRR-apportioned relative abundances of isobaric positional isomers sum to ~100% for each co-fragmented peptide group
- Aggregated single-PTM percentages reproduce published values (e.g., H3K27me2 ~75% in wt ESCs vs <10% in Suz12-/-; H3K36me2 67% → 40%; H4K16ac 7.1% → 11.8%) within rounding tolerance
- Each quantified peptide form carries an unambiguous Histone Coder localization backed by site-determining ion counts
- The result file (Histone Coder output) and Mascot .csv data file are correctly paired as isoScale's two required inputs

## Limitations

- FIRR-based deconvolution depends on sufficient site-determining fragment ions within the chosen MS/MS ion tolerance; low-resolution data may leave isobaric species unresolved
- Quantification principle assumes cofragmenting isobaric species share the same sequence and differ only in PTM localization; other isobaric interferences are outside its model
- The workflow requires sequential execution (Mascot .xml → Histone Coder → isoScale with Mascot .csv); a missing or mismatched intermediate file breaks the pipeline

## Evidence
