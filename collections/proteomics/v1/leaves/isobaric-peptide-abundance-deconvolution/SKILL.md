---
name: isobaric-peptide-abundance-deconvolution
description: Use when middle-down (or top-down) histone MS/MS data contain co-fragmenting
  isobaric polypeptide species sharing the same sequence but different PTM localizations
  (e.g. H3K18ac vs H3K23ac), and per-localization relative abundances are needed for
  PTM cross-talk analysis.
when_to_use_negative:
- Data are from standard bottom-up tryptic workflows where peptides do not share the
  same sequence with scrambled combinatorial PTMs — isoScale is explicitly recommended
  only for middle-down or top-down analyses
- Spectra were assigned by a search engine other than Mascot — Histone Coder and isoScale
  take Mascot .xml/.csv outputs as input
- The PTM localization reported by Mascot is ambiguous — validate/resolve localizations
  with Histone Coder site-determining ion counts before attempting FIRR deconvolution
edam_operation: http://edamontology.org/operation_3630
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: isoScale
  role: Quantifies peptides by total MS/MS ion intensity and performs relative quantification
    of co-fragmented isobaric species via the fragment ion relative ratio (FIRR)
  repo: https://github.com/middle-down/Software/blob/master/isoScale-v2.jar?raw=true
- name: Histone Coder
  role: Validates unambiguous PTM localization by counting site-determining MS/MS
    ions and converts Mascot output to Brno nomenclature before isoScale quantification
  repo: https://github.com/middle-down/Software/blob/master/Histone%20Coder.jar?raw=true
- name: Mascot
  role: Database search engine whose .csv/.xml output files are the required input
    format for both tools
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/pmic.201400084
    title: Middle‐down hybrid chromatography/tandem mass spectrometry workflow for
      characterization of combinatorial post‐translational modifications in histones
  source_content_hash: sha256:666a0b74eb6f997280a82a935ad13c63e595c2345cbaaf47fb9197f0e486eb58
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
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# isobaric-peptide-abundance-deconvolution

## Summary

Deconvolutes the relative abundances of isobaric histone peptide species that co-fragment in the same MS/MS spectrum but carry distinct PTM localizations, using isoScale's fragment ion relative ratio (FIRR) calculation on Mascot/Histone Coder outputs from middle-down proteomics data.

## When to use

Use when middle-down (or top-down) histone MS/MS data contain co-fragmenting isobaric polypeptide species sharing the same sequence but different PTM localizations (e.g. H3K18ac vs H3K23ac), and per-localization relative abundances are needed for PTM cross-talk analysis.

## When NOT to use

- Data are from standard bottom-up tryptic workflows where peptides do not share the same sequence with scrambled combinatorial PTMs — isoScale is explicitly recommended only for middle-down or top-down analyses
- Spectra were assigned by a search engine other than Mascot — Histone Coder and isoScale take Mascot .xml/.csv outputs as input
- The PTM localization reported by Mascot is ambiguous — validate/resolve localizations with Histone Coder site-determining ion counts before attempting FIRR deconvolution

## Inputs

- Mascot .csv output file (data file) from database searching of middle-down histone MS/MS spectra
- Histone Coder result file (validated PTM localizations in Brno nomenclature)
- MS/MS ion tolerance setting and choice of fragmentation mode (CID/HCD or ETD)

## Outputs

- Relative peptide abundance table quantified by total MS/MS ion intensity, with isobaric species deconvoluted by FIRR
- Per-peptide PTM strings in Brno nomenclature enabling pairwise co-existence statistics (e.g. ptm_coexistence_stats.csv with wt and Suz12-/- percentages)

## How to apply

First run Histone Coder on the Mascot .xml output to validate PTM localization by counting site-determining MS/MS ions and converting mod strings to Brno nomenclature (e.g. K4me3K14ac); then feed the Histone Coder result file plus the Mascot .csv data file into isoScale. In isoScale, select ETD (or CID/HCD) fragmentation, set the MS/MS ion tolerance for searching fragment ions, and let the software compute the fragment ion relative ratio (FIRR) to split the total ion intensity of each MS/MS spectrum among the co-fragmenting isobaric species. Quantify each peptide from the total ion intensity of its MS/MS spectra, then combine the deconvoluted, localization-resolved abundances to compute co-existence fractions of PTM pairs (e.g. H3K27me3K36me3) with all spectra containing either mark of the pair as denominator. The rationale is that cofragmenting isobaric species otherwise inflate a single intensity value, so FIRR-based deconvolution (principle from Pesavento et al., Analytical Chemistry, 2006) is required before counting pairwise co-detected marks.

## Related tools

- **isoScale** (Quantifies peptides by total MS/MS ion intensity and performs relative quantification of co-fragmented isobaric species via the fragment ion relative ratio (FIRR)) — https://github.com/middle-down/Software/blob/master/isoScale-v2.jar?raw=true
- **Histone Coder** (Validates unambiguous PTM localization by counting site-determining MS/MS ions and converts Mascot output to Brno nomenclature before isoScale quantification) — https://github.com/middle-down/Software/blob/master/Histone%20Coder.jar?raw=true
- **Mascot** (Database search engine whose .csv/.xml output files are the required input format for both tools)

## Examples

```
java -jar "isoScale-v2.jar"  # GUI: load Histone Coder result file + Mascot .csv, choose ETD fragmentation, set MS/MS ion tolerance, compute FIRR
```

## Evaluation signals

- Reproduced pairwise co-existence statistics should match reported values: H3K18ac/H3K23ac co-detected in ~54% of relevant peptides, and H4S1acK20me2 at 54.1% (wt) and 60.2% (Suz12-/-) ESCs
- Trace/repressed-mark pairs should fall at expected levels: H3K27me3K36me3 below 0.5% (wt) and 0.2% (Suz12-/-), and H3K9me3K27me3 never co-detected on histone H3 tails
- Sum of the FIRR-deconvoluted intensities of co-fragmenting isobaric species should reconcile with the total ion intensity of the shared MS/MS spectrum
- Each PTM string in the output should carry an unambiguous localization supported by a non-zero count of site-determining ions from Histone Coder
- PTM strings in the output should follow standard Brno nomenclature (e.g. K4me3K14ac)

## Limitations

- Deconvolution accuracy depends on the chosen MS/MS ion tolerance for the fragment-ion search used in the FIRR calculation; suitable for high and low resolution data but the tolerance must be set per dataset
- The software quantifies only what Mascot identified; peptides never assigned by the search cannot be quantified or deconvoluted
- Requires Java and the two-step Histone Coder → isoScale pipeline with specific file formats (.xml into Histone Coder, .csv plus result file into isoScale)

## Evidence
