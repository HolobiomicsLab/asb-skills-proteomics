---
name: relative-peptide-abundance-quantification
description: Use when raw Mascot .csv/.xml output from ETD middle-down histone experiments
  must be converted into relative peptide abundances, including deconvolution of isobaric
  peptides sharing a sequence but with distinct PTM localizations.
when_to_use_negative:
- Data are bottom-up tryptic peptide identifications where isobaric species with distinct
  PTM localizations on long tails are not the quantification target — isoScale is
  recommended for middle-down or top-down analyses
- Raw spectra have not been searched with Mascot, since both tools consume Mascot
  .xml/.csv outputs
- Absolute (rather than relative) peptide quantities are required; isoScale reports
  relative abundances based on total ion intensity
edam_operation: http://edamontology.org/operation_3630
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0601
tools:
- name: isoScale
  role: Quantifies peptides by total ion intensity of MS/MS spectra and performs relative
    quantification of co-fragmented isobaric peptides via FIRR
  repo: https://github.com/middle-down/Software/blob/master/isoScale-v2.jar?raw=true
- name: Histone Coder
  role: Validates PTM localization by counting site-determining ions and converts
    Mascot output to Brno nomenclature; its result file is the required input to isoScale
  repo: https://github.com/middle-down/Software/blob/master/Histone%20Coder.jar?raw=true
- name: Mascot
  role: Database search engine whose .xml/.csv outputs are the starting point for
    spectral validation and quantification
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/pmic.201400084
    title: Middle‐down hybrid chromatography/tandem mass spectrometry workflow for
      characterization of combinatorial post‐translational modifications in histones
  source_content_hash: sha256:ecea10a26b1553c0ea06e6efdcfc05ac9bd243891cb547215b2f8d1f46855ae3
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
yes_rate: 0.875
n_claims: 16
---

# relative-peptide-abundance-quantification

## Summary

Quantifies relative abundance of histone peptides and cofragmenting isobaric species from Mascot-identified middle-down MS/MS data using isoScale, which sums total ion intensity of MS/MS spectra per PTM-combinatorial peptide.

## When to use

Use when raw Mascot .csv/.xml output from ETD middle-down histone experiments must be converted into relative peptide abundances, including deconvolution of isobaric peptides sharing a sequence but with distinct PTM localizations.

## When NOT to use

- Data are bottom-up tryptic peptide identifications where isobaric species with distinct PTM localizations on long tails are not the quantification target — isoScale is recommended for middle-down or top-down analyses
- Raw spectra have not been searched with Mascot, since both tools consume Mascot .xml/.csv outputs
- Absolute (rather than relative) peptide quantities are required; isoScale reports relative abundances based on total ion intensity

## Inputs

- Mascot .xml output file (input to Histone Coder)
- Mascot .csv output file (data file input to isoScale)
- Histone Coder result file (validated PTM-localization output feeding isoScale)

## Outputs

- Table of relative peptide abundances per combinatorial PTM string (Brno nomenclature), quantified by total MS/MS ion intensity
- Relative quantification of co-fragmenting isobaric peptide species via FIRR
- Derived statistics such as PTM-pair co-existence percentages (e.g. ptm_coexistence_stats.csv)

## How to apply

First run Histone Coder on the Mascot .xml output to validate PTM localization by counting site-determining MS/MS ions and to convert assignments to standard Brno nomenclature (e.g. K4me3K14ac). Then feed the Histone Coder result file plus the Mascot .csv data file into isoScale, selecting the fragmentation mode (CID/HCD or ETD) and an MS/MS ion tolerance appropriate to the instrument resolution. isoScale quantifies each peptide as the total ion intensity of its MS/MS spectra, and for isobaric peptides co-fragmented in the same MS/MS spectrum it computes the fragment ion relative ratio (FIRR) to apportion intensity between species with distinct PTM localizations. The resulting relative abundance table (e.g. for 256 combinatorial marks on H3, H4 and H2A) can then be parsed to compute co-existence percentages of PTM pairs across wt and Suz12-/- datasets. The quantification principle follows Pesavento et al. (Analytical Chemistry, 2006) and is recommended for middle-down/top-down data where scrambled combinatorial PTMs on a single sequence must be compared directly.

## Related tools

- **isoScale** (Quantifies peptides by total ion intensity of MS/MS spectra and performs relative quantification of co-fragmented isobaric peptides via FIRR) — https://github.com/middle-down/Software/blob/master/isoScale-v2.jar?raw=true
- **Histone Coder** (Validates PTM localization by counting site-determining ions and converts Mascot output to Brno nomenclature; its result file is the required input to isoScale) — https://github.com/middle-down/Software/blob/master/Histone%20Coder.jar?raw=true
- **Mascot** (Database search engine whose .xml/.csv outputs are the starting point for spectral validation and quantification)

## Examples

```
java -jar isoScale-v2.jar --result HistoneCoder_result.csv --data mascot_output.csv --fragmentation ETD --tolerance 0.02Da
```

## Evaluation signals

- PTM strings in the output follow standard Brno nomenclature (e.g. K4me3K14ac)
- Reproduced co-existence statistics match reported values: H3K27me3K36me3 <0.5% (wt) and <0.2% (Suz12-/-), H3K18ac/H3K23ac co-detected in 54%, H4S1acK20me2 at 54.1% (wt) and 60.2% (Suz12-/-)
- H3K9me3K27me3 and H3K9me3K36me3 combinations are never detected on histone H3 tails
- Quantified combinatorial mark counts are consistent with the reported scale (256 combinatorial marks on H3/H4/H2A; 713 identified on purified H3)
- Isobaric species sharing the same sequence but distinct PTM localizations are deconvoluted (FIRR computed) rather than summed into one intensity

## Limitations

- isoScale is recommended for middle-down or top-down analyses; suitability for standard bottom-up workflows is not claimed
- Both tools require Mascot output (.xml for Histone Coder, .csv for isoScale) and Java runtime
- Accurate isobaric deconvolution depends on choosing an appropriate MS/MS ion tolerance for the instrument resolution (CID/HCD vs ETD, high vs low resolution)

## Evidence
