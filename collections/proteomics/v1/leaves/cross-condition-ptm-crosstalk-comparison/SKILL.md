---
name: cross-condition-ptm-crosstalk-comparison
description: Use when you have Mascot-searched middle-down/top-down histone MS/MS
  data (Histone Coder result file + Mascot .csv) and need to compare co-detection
  fractions of specific PTM pairs between two or more biological conditions.
when_to_use_negative:
- Data are bottom-up tryptic peptides rather than middle-down/top-down histone tails
  — isoScale is explicitly recommended for 'direct comparison of single sequences
  with scrambled combinatorial PTMs' in middle-down or top-down analyses
- You need single-mark abundance changes across conditions (e.g. the seven-fold H3K27me2/me3
  reduction in Suz12-/-) rather than co-detection fractions of PTM pairs
- PTM localization is ambiguous — pairs must be counted only after Histone Coder has
  confirmed site-determining ions, otherwise co-existence is confounded with mislocalization
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Histone Coder
  role: Validates MS/MS spectra by counting site-determining ions for unambiguous
    PTM localization and converts Mascot output to standard Brno nomenclature (e.g.
    K4me3K14ac) before pair counting
  repo: https://github.com/middle-down/Software/blob/master/Histone%20Coder.jar?raw=true
- name: isoScale
  role: Quantifies peptides by total MS/MS ion intensity and provides relative quantification
    of co-fragmented isobaric peptides sharing the same sequence but distinct PTM
    localizations (FIRR), enabling accurate co-existence fractions
  repo: https://github.com/middle-down/Software/blob/master/isoScale-v2.jar?raw=true
- name: Mascot
  role: Database search engine producing the .csv identifications of histone polypeptides
    that Histone Coder and isoScale consume
- name: Cross-talk database
  role: Data repository hosting the combinatorial PTM data used for cross-talk/co-existence
    queries
  repo: http://crosstalkdb.bmb.sdu.dk/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/pmic.201400084
    title: Middle‐down hybrid chromatography/tandem mass spectrometry workflow for
      characterization of combinatorial post‐translational modifications in histones
  source_content_hash: sha256:94754d910e04a92dfe0ae734d572965c2aece69f834f203fc7020933b18d7dcb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Cross-talk database
  - Histone Coder
  - Mascot
  - isoScale
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# cross-condition-ptm-crosstalk-comparison

## Summary

Quantify whether pairs of post-translational modifications (e.g. H3K9me3K27me3, H3K27me3K36me3, H3K18acK23ac) co-exist on the same histone tail across experimental conditions (wt vs Suz12-/- murine ESCs), using isoScale MS/MS total-ion-intensity quantification of Histone Coder-validated Mascot identifications in Brno nomenclature.

## When to use

Use when you have Mascot-searched middle-down/top-down histone MS/MS data (Histone Coder result file + Mascot .csv) and need to compare co-detection fractions of specific PTM pairs between two or more biological conditions.

## When NOT to use

- Data are bottom-up tryptic peptides rather than middle-down/top-down histone tails — isoScale is explicitly recommended for 'direct comparison of single sequences with scrambled combinatorial PTMs' in middle-down or top-down analyses
- You need single-mark abundance changes across conditions (e.g. the seven-fold H3K27me2/me3 reduction in Suz12-/-) rather than co-detection fractions of PTM pairs
- PTM localization is ambiguous — pairs must be counted only after Histone Coder has confirmed site-determining ions, otherwise co-existence is confounded with mislocalization

## Inputs

- Histone Coder result file (validated MS/MS spectra with PTM localization counts, Brno nomenclature)
- Mascot .csv output from middle-down LC-MS/MS runs (e.g. ETD fragmentation of histone H3/H4/H2A tails)
- Set of PTM pairs of interest per histone type (e.g. K9me3/K27me3, K27me3/K36me3, K18ac/K23ac)
- Peptide abundance table from isoScale for each condition (wt, Suz12-/-)

## Outputs

- ptm_coexistence_stats.csv with columns: histone type, PTM pair, wt co-existence percentage, Suz12-/- co-existence percentage
- Side-by-side cross-condition percentages for each quantified combinatorial mark pair

## How to apply

Load the relative peptide abundance table produced by isoScale, which quantifies histone peptides based on the total ion intensity of MS/MS spectra, and parse the PTM strings (Brno nomenclature, e.g. K4me3K14ac) written by Histone Coder from the Mascot .csv output. For each PTM pair of interest, compute the fraction of quantified peptide spectra in which both marks are co-detected, with the denominator being all spectra containing either mark of the pair; before counting, apply isoScale's relative quantification of isobaric peptides co-fragmented in the same MS/MS spectrum (FIRR, fragment ion relative ratio) to deconvolute peptides sharing the same sequence but distinct PTM localizations. Run the computation separately per condition (e.g. wild-type and Suz12-/- murine ESC datasets) and report percentages side by side, then write a table with columns for histone type, PTM pair, and per-condition co-existence percentage. Validation uses paper-specific expected values: H3K9me3K27me3 never co-detected, H3K27me3K36me3 <0.5% in wt and <0.2% in Suz12-/-, H3K18ac/H3K23ac co-detected in 54% of relevant peptides, and H4S1acK20me2 at 54.1% (wt) vs 60.2% (Suz12-/-).

## Related tools

- **Histone Coder** (Validates MS/MS spectra by counting site-determining ions for unambiguous PTM localization and converts Mascot output to standard Brno nomenclature (e.g. K4me3K14ac) before pair counting) — https://github.com/middle-down/Software/blob/master/Histone%20Coder.jar?raw=true
- **isoScale** (Quantifies peptides by total MS/MS ion intensity and provides relative quantification of co-fragmented isobaric peptides sharing the same sequence but distinct PTM localizations (FIRR), enabling accurate co-existence fractions) — https://github.com/middle-down/Software/blob/master/isoScale-v2.jar?raw=true
- **Mascot** (Database search engine producing the .csv identifications of histone polypeptides that Histone Coder and isoScale consume)
- **Cross-talk database** (Data repository hosting the combinatorial PTM data used for cross-talk/co-existence queries) — http://crosstalkdb.bmb.sdu.dk/

## Evaluation signals

- H3K9me3K27me3 never co-detected in either condition (0% in wt and Suz12-/-)
- H3K27me3K36me3 below 0.5% in wt and below 0.2% in Suz12-/- ESCs
- H3K18ac and H3K23ac co-detected in 54% of relevant peptides
- H4S1acK20me2 co-existence at 54.1% (wt) and 60.2% (Suz12-/-) reproduced side by side
- Output CSV has the required schema: histone type, PTM pair, wt percentage, Suz12-/- percentage

## Limitations

- Co-existence is measured from co-detected spectra, so cross-checking depends on deconvolution of co-fragmenting isobaric species via FIRR; MS/MS ion tolerance choice (CID/HCD vs ETD, high vs low resolution data) affects the calculation
- Histone Coder's supported PTM set is limited to phosphorylation (S,T,Y), acetylation (K), mono- and dimethylation (K,R) and trimethylation (K); other modification types cannot be counted
- No changelog found for the software; results depend on the 2014 jar releases from the middle-down/Software repository

## Evidence

- [results] Only trace amounts of the combination H3K27me3K36me3 were quantified (<0.5% in wt, <0.2% in Suz12-/- ESCs): [evidence span withheld — anchor status: not_found]
