---
name: extracted-ion-chromatogram-quantification
description: Use when relative abundances of histone N-terminal tail peptide groups
  (e.g. H3 species spanning 0-3 acetyl and 0-9 methyl PTM equivalents) must be quantified
  from LC-ESI-MS/MS runs and compared across technical replicates or against other
  quantification approaches.
when_to_use_negative:
- Standard bottom-up shotgun proteomics with sequence-unique peptides — isoScale/XIC-style
  co-isobaric handling is explicitly recommended for middle-down or top-down analyses
  where the same sequence carries scrambled combinatorial PTMs
- Input is already a normalized abundance or feature table — re-extracting XICs would
  duplicate existing quantification
- Spectra were not searched with Mascot or no Histone Coder result file exists, since
  isoScale requires the Mascot .csv output plus the Histone Coder result file as inputs
edam_operation: http://edamontology.org/operation_3630
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: isoScale
  role: Quantifies validated peptides from total MS/MS ion intensity and computes
    FIRR for co-fragmenting isobaric species, benchmarked against the XIC reference
  repo: https://github.com/middle-down/Software/blob/master/isoScale-v2.jar?raw=true
- name: Histone Coder
  role: Validates unambiguous PTM localization by counting site-determining MS/MS
    ions and converts Mascot output to Brno nomenclature, producing the result file
    required by isoScale
  repo: https://github.com/middle-down/Software/blob/master/Histone%20Coder.jar?raw=true
- name: Mascot
  role: Database search engine whose .csv output supplies the identified peptides
    feeding Histone Coder and isoScale quantification
- name: Java
  role: Runtime required to launch the Histone Coder and isoScale .jar tools
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.201400084
    title: Middle‐down hybrid chromatography/tandem mass spectrometry workflow for
      characterization of combinatorial post‐translational modifications in histones
  source_content_hash: sha256:17899d66d23e6c6750db19d46f991251a3fbc9659a9582d88fbcce8c5806d7ff
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
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# extracted-ion-chromatogram-quantification

## Summary

Quantifies relative abundances of histone tail peptide groups by extracted ion chromatograms (XIC) across technical replicates, serving as the MS1-based reference against which MS/MS-intensity methods (isoScale) and spectral counting are benchmarked in the middle-down histone PTM workflow.

## When to use

Use when relative abundances of histone N-terminal tail peptide groups (e.g. H3 species spanning 0-3 acetyl and 0-9 methyl PTM equivalents) must be quantified from LC-ESI-MS/MS runs and compared across technical replicates or against other quantification approaches.

## When NOT to use

- Standard bottom-up shotgun proteomics with sequence-unique peptides — isoScale/XIC-style co-isobaric handling is explicitly recommended for middle-down or top-down analyses where the same sequence carries scrambled combinatorial PTMs
- Input is already a normalized abundance or feature table — re-extracting XICs would duplicate existing quantification
- Spectra were not searched with Mascot or no Histone Coder result file exists, since isoScale requires the Mascot .csv output plus the Histone Coder result file as inputs

## Inputs

- Raw LC-ESI-MS/MS data (RP-WCX-HILIC separation, ETD fragmentation) of purified histone H3 from Suz12-/- ESCs, three technical replicates
- Mascot database search output in .csv format
- Histone Coder result file with validated, PTM-localized peptide-spectrum matches
- Definition of the peptide-grouping scheme (22 peptide groups: 0-3 acetyl, 0-9 methyl PTM equivalents)

## Outputs

- Per-replicate XIC-derived relative abundances for each of the 22 peptide groups
- Normalized abundance table aligning XIC, isoScale (with FIRR), and spectral counting values per peptide group and replicate
- Comparison table written to fig3_quantification_comparison.csv with per-group agreement status versus the XIC reference

## How to apply

Separate histone tails by an RP trap column followed by weak cation exchange-hydrophilic interaction LC interfaced to high mass accuracy ESI MS/MS with ETD, then search spectra with Mascot and validate PTM localization with Histone Coder before quantification. For each validated peptide group (here 22 groups covering 0-3 acetyl and 0-9 methyl PTM equivalents on purified histone H3), integrate the extracted ion chromatogram (XIC) for every technical replicate (three Suz12-/- replicates in the benchmark). Normalize relative abundances per replicate so methods are directly comparable, and repeat the same grouping for spectral counting and for isoScale (isoScale using ETD fragmentation and a chosen MS/MS ion tolerance to compute the fragment ion relative ratio, FIRR, of co-fragmenting isobaric species). Judge success by agreement with the XIC reference: in the source study, isoScale was accepted when it agreed with XIC on at least 18 of the 22 peptide groups, while spectral counting was inaccurate for unmodified and extensively modified peptides.

## Related tools

- **isoScale** (Quantifies validated peptides from total MS/MS ion intensity and computes FIRR for co-fragmenting isobaric species, benchmarked against the XIC reference) — https://github.com/middle-down/Software/blob/master/isoScale-v2.jar?raw=true
- **Histone Coder** (Validates unambiguous PTM localization by counting site-determining MS/MS ions and converts Mascot output to Brno nomenclature, producing the result file required by isoScale) — https://github.com/middle-down/Software/blob/master/Histone%20Coder.jar?raw=true
- **Mascot** (Database search engine whose .csv output supplies the identified peptides feeding Histone Coder and isoScale quantification)
- **Java** (Runtime required to launch the Histone Coder and isoScale .jar tools)

## Examples

```
java -jar isoScale-v2.jar  # select ETD fragmentation and set MS/MS ion tolerance; input: Histone Coder result file + Mascot .csv
```

## Evaluation signals

- isoScale quantification agrees with the XIC reference on at least 18 of the 22 peptide groups (acceptance criterion of the benchmark)
- isoScale quantifies more peptide groups than spectral counting (18 vs fewer of the 22 groups), especially recovering unmodified and extensively modified peptides that spectral counting missed
- Relative abundances are normalized per replicate for all three quantification approaches before comparison
- FIRR values are produced for co-fragmenting isobaric species sharing the same sequence but distinct PTM localizations
- Final comparison table is written to fig3_quantification_comparison.csv with an agreement status recorded per peptide group

## Limitations

- XIC comparison values come from the same three Suz12-/- technical replicates; results are specific to purified histone H3 and may not transfer to other histones or cell states
- isoScale quantification relies on total ion intensity of MS/MS spectra, so co-fragmenting isobaric polypeptide species must be deconvoluted via FIRR, which depends on a correctly chosen MS/MS ion tolerance
- Software is GUI-based Java tools accepting only Mascot outputs (.xml for Histone Coder, .csv for isoScale); no changelog was available for the repository

## Evidence

- [abstract] We present Histone Coder and Iso Scale software to extract, filter, and analyze MS/MS data, including quantification of cofragmenting isobaric polypeptide species.: [evidence span withheld — anchor status: not_found]
