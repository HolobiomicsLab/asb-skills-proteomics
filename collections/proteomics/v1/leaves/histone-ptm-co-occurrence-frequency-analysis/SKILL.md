---
name: histone-ptm-co-occurrence-frequency-analysis
description: Use when middle-down histone MS/MS data (Mascot .csv + Histone Coder
  Brno-nomenclature PTM strings) must be tested for co-existence or mutual exclusivity
  of specific mark pairs, e.g. H3K9me3K27me3 vs H3K18acK23ac.
when_to_use_negative:
- 'Input is bottom-up short-chained peptides: isoScale is explicitly recommended for
  middle-down or top-down analyses suited to comparing single sequences with scrambled
  combinatorial PTMs'
- The aim is absolute abundance of individual marks (e.g. seven-fold H3K27me2/me3
  reduction), not binary co-detection fractions between mark pairs
- Data were not searched with Mascot — Histone Coder requires Mascot .xml output and
  isoScale requires the Mascot .csv plus Histone Coder result file
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: isoScale
  role: Quantifies histone peptides by total ion intensity of MS/MS spectra and deconvolutes
    co-fragmenting isobaric species (FIRR) so co-detected PTM pairs are counted on
    correctly localized peptides
  repo: https://github.com/middle-down/Software/blob/master/isoScale-v2.jar?raw=true
- name: Histone Coder
  role: Validates PTM localization by counting site-determining MS/MS ions and converts
    Mascot output into Brno nomenclature PTM strings used for pair detection
  repo: https://github.com/middle-down/Software/blob/master/Histone%20Coder.jar?raw=true
- name: Mascot
  role: Database search engine whose .csv/.xml output is the upstream input assigning
    PTM localizations to histone tail spectra
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/pmic.201400084
    title: Middle‐down hybrid chromatography/tandem mass spectrometry workflow for
      characterization of combinatorial post‐translational modifications in histones
  source_content_hash: sha256:2b9313e68e73c3a6131e1b189a8a01c98abd344e1904435a0c511200dd8ba74d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Histone Coder
  - Mascot
  - isoScale
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# histone-ptm-co-occurrence-frequency-analysis

## Summary

Quantifies how frequently two post-translational modifications co-exist on the same hypermodified histone N-terminal tail by computing the fraction of quantified MS/MS peptide spectra containing both marks of a PTM pair, using isoScale-intensity peptide tables from middle-down ETD data.

## When to use

Use when middle-down histone MS/MS data (Mascot .csv + Histone Coder Brno-nomenclature PTM strings) must be tested for co-existence or mutual exclusivity of specific mark pairs, e.g. H3K9me3K27me3 vs H3K18acK23ac.

## When NOT to use

- Input is bottom-up short-chained peptides: isoScale is explicitly recommended for middle-down or top-down analyses suited to comparing single sequences with scrambled combinatorial PTMs
- The aim is absolute abundance of individual marks (e.g. seven-fold H3K27me2/me3 reduction), not binary co-detection fractions between mark pairs
- Data were not searched with Mascot — Histone Coder requires Mascot .xml output and isoScale requires the Mascot .csv plus Histone Coder result file

## Inputs

- isoScale relative peptide abundance table (total ion intensity of MS/MS spectra per peptide)
- Histone Coder result file with PTM strings in Brno nomenclature (e.g. K4me3K14ac)
- Mascot .csv output file
- List of PTM pairs of interest (e.g. K9me3/K27me3, K27me3/K36me3, K18ac/K23ac)

## Outputs

- ptm_coexistence_stats.csv with columns: histone type, PTM pair, wt co-existence percentage, Suz12-/- co-existence percentage
- Side-by-side per-condition co-existence percentages for each PTM pair

## How to apply

Load the relative peptide abundance table produced by isoScale, which quantifies histone peptides by total ion intensity of MS/MS spectra. Parse each peptide's PTM string in standard Brno nomenclature (e.g. K4me3K14ac) as converted by Histone Coder from the Mascot .csv output. For each PTM pair of interest, compute the fraction of quantified spectra in which both marks are co-detected, using as denominator all spectra containing either mark of the pair; first apply isoScale's relative quantification of co-fragmenting isobaric species (FIRR) to deconvolute peptides sharing a sequence but differing in PTM localization before counting. Repeat the computation per condition (e.g. wt vs Suz12-/- murine ESC) and report percentages side by side. Export a table with histone type, PTM pair, and per-condition co-existence percentages.

## Related tools

- **isoScale** (Quantifies histone peptides by total ion intensity of MS/MS spectra and deconvolutes co-fragmenting isobaric species (FIRR) so co-detected PTM pairs are counted on correctly localized peptides) — https://github.com/middle-down/Software/blob/master/isoScale-v2.jar?raw=true
- **Histone Coder** (Validates PTM localization by counting site-determining MS/MS ions and converts Mascot output into Brno nomenclature PTM strings used for pair detection) — https://github.com/middle-down/Software/blob/master/Histone%20Coder.jar?raw=true
- **Mascot** (Database search engine whose .csv/.xml output is the upstream input assigning PTM localizations to histone tail spectra)

## Examples

```
frac = df[df.ptm.str.contains('K27me3|K36me3')].ptm.str.contains('K27me3').eq(df.ptm.str.contains('K36me3')) ; coexistence_pct = 100*frac[df.ptm.str.contains('K27me3|K36me3')].mean()
```

## Evaluation signals

- H3K9me3K27me3 is never co-detected (0 spectra) in both wt and Suz12-/- ESC datasets
- H3K27me3K36me3 co-existence is <0.5% in wt and <0.2% in Suz12-/- ESCs
- H3K18ac/H3K23ac co-detected in 54% of relevant peptides
- H4S1acK20me2 co-existence reproduces at 54.1% (wt) and 60.2% (Suz12-/-)
- Output CSV schema contains exactly histone type, PTM pair, wt percentage, and Suz12-/- percentage, and each fraction denominator equals all spectra containing either mark of the pair

## Limitations

- Trace-level combinations (e.g. <0.5% of spectra) approach detection limits, so absence of H3K9me3K27me3/K36me3 pairs could reflect sensitivity rather than true absence
- Isobaric peptides co-fragmenting in the same MS/MS spectrum share sequence but differ in PTM localization and must be deconvoluted via FIRR before counting; incorrect ion tolerance settings bias the split
- Histone Coder's PTM script covers only phosphorylation (S,T,Y), acetylation (K), mono-/dimethylation (K,R) and trimethylation (K); other modification types are not handled

## Evidence
