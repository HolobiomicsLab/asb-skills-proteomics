---
name: histone-ptm-relative-quantification
description: Use when Mascot .csv search output and a Histone Coder result file are
  available for histone middle-down ETD spectra and relative abundances of single
  or combinatorial PTM peptide groups (e.g. 0-3 acetyl, 0-9 methyl) must be quantified
  across replicates.
when_to_use_negative:
- Bottom-up tryptic peptide data where extracted ion chromatograms (XIC) are the natural
  quantification route — isoScale is recommended for middle-down or top-down analyses
  comparing single sequences with scrambled combinatorial PTMs
- Data whose Mascot output cannot be converted to a Histone Coder result file (isoScale
  requires both inputs)
- Absolute quantification with isotope-labeled standards — isoScale provides only
  relative intensity-based quantification
edam_operation: http://edamontology.org/operation_3630
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0601
tools:
- name: isoScale
  role: Quantifies peptides from total ion intensity of MS/MS spectra and computes
    FIRR for co-fragmenting isobaric species; consumes Histone Coder result file plus
    Mascot .csv data file; user selects ETD fragmentation and MS/MS ion tolerance
  repo: https://github.com/middle-down/Software/blob/master/isoScale-v2.jar?raw=true
- name: Histone Coder
  role: 'Upstream validation: counts site-determining MS/MS ions for unambiguous PTM
    localization, converts Mascot output to Brno nomenclature, and emits the result
    file that isoScale consumes'
  repo: https://github.com/middle-down/Software/blob/master/Histone%20Coder.jar?raw=true
- name: Mascot
  role: Database search engine whose .csv output provides peptide-spectrum matches
    and PTM assignments fed into Histone Coder and isoScale
- name: Java
  role: Runtime requirement for both Histone Coder and isoScale jars
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1002/pmic.201400084
    title: Middle‐down hybrid chromatography/tandem mass spectrometry workflow for
      characterization of combinatorial post‐translational modifications in histones
  source_content_hash: sha256:7abd129ec3989e869deada974bbbb4f0acd990cef3cd760a6061de23933b4beb
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
  edam_operation: http://edamontology.org/operation_3630
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0601
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# histone-ptm-relative-quantification

## Summary

Relative quantification of histone PTM peptide forms from middle-down MS/MS data using isoScale, which sums total ion intensity of MS/MS spectra and resolves co-fragmenting isobaric species via the fragment ion relative ratio (FIRR), validated against Mascot/Histone Coder peptide-spectrum matches.

## When to use

Use when Mascot .csv search output and a Histone Coder result file are available for histone middle-down ETD spectra and relative abundances of single or combinatorial PTM peptide groups (e.g. 0-3 acetyl, 0-9 methyl) must be quantified across replicates.

## When NOT to use

- Bottom-up tryptic peptide data where extracted ion chromatograms (XIC) are the natural quantification route — isoScale is recommended for middle-down or top-down analyses comparing single sequences with scrambled combinatorial PTMs
- Data whose Mascot output cannot be converted to a Histone Coder result file (isoScale requires both inputs)
- Absolute quantification with isotope-labeled standards — isoScale provides only relative intensity-based quantification

## Inputs

- Mascot database search output in .csv format
- Histone Coder result file (validated, PTM-localized peptide-spectrum matches in Brno nomenclature)
- MS/MS spectra from middle-down RP/WCX-HILIC LC-ETD-MS/MS runs (e.g. histone H3 tails from wt and Suz12-/- ESCs, three technical replicates)

## Outputs

- Per-peptide-form relative abundances based on total ion intensity of MS/MS spectra (isoScale output tables, e.g. supplementary S3, S4, S7, S8)
- Fragment ion relative ratio (FIRR) values splitting co-fragmenting isobaric species
- Single-PTM abundance and fold-change tables per replicate (e.g. H3K27me2 ~75% wt vs <10% Suz12-/-, H3K36me2 67% to 40%, H3K9me2 ~5-fold increase, H4K16ac 7.1% to 11.8%)

## How to apply

First validate and localize PTMs by running Mascot (.csv output) through Histone Coder, which counts site-determining MS/MS ions and converts to Brno nomenclature (e.g. K4me3K14ac). Then run isoScale with the Histone Coder result file plus the Mascot .csv data file, selecting the ETD fragmentation option and setting the MS/MS ion tolerance for fragment-ion searching; isoScale quantifies each peptide from the total ion intensity of its MS/MS spectra and, for isobaric co-fragmented species sharing the same sequence but distinct PTM localizations, computes the fragment ion relative ratio (FIRR) to split the intensity. Aggregate per-peptide-form intensities into peptide groups (e.g. 0-3 acetyl, 0-9 methyl equivalents) per replicate, normalize relative abundances per replicate, and aggregate over all peptide forms carrying a target single mark (e.g. H3K27me2, H3K36me2, H3K9me2) to produce single-PTM abundance tables. Correctness is judged by agreement with an independent quantification route (extracted ion chromatograms) and by reproduction of published fold-changes such as the seven-fold H3K27me2/me3 reduction in Suz12-/- ESCs.

## Related tools

- **isoScale** (Quantifies peptides from total ion intensity of MS/MS spectra and computes FIRR for co-fragmenting isobaric species; consumes Histone Coder result file plus Mascot .csv data file; user selects ETD fragmentation and MS/MS ion tolerance) — https://github.com/middle-down/Software/blob/master/isoScale-v2.jar?raw=true
- **Histone Coder** (Upstream validation: counts site-determining MS/MS ions for unambiguous PTM localization, converts Mascot output to Brno nomenclature, and emits the result file that isoScale consumes) — https://github.com/middle-down/Software/blob/master/Histone%20Coder.jar?raw=true
- **Mascot** (Database search engine whose .csv output provides peptide-spectrum matches and PTM assignments fed into Histone Coder and isoScale)
- **Java** (Runtime requirement for both Histone Coder and isoScale jars)

## Evaluation signals

- isoScale quantification agrees with an independent extracted ion chromatogram (XIC) reference on at least 18 of 22 H3 peptide groups (0-3 acetyl, 0-9 methyl equivalents) across three technical replicates
- Computed single-PTM relative abundances match published values within rounding tolerance (e.g. H3K27me2+H3K27me3 seven-fold lower in Suz12-/- vs wt; H3K36me2 67%->40%; H3K9me2 ~5-fold up; H4K16ac 7.1%->11.8%)
- Normalization check: relative abundances sum to 100% per replicate after aggregation of peptide forms per mark
- FIRR values are produced for every isobaric peptide group sharing the same sequence but distinct PTM localizations, confirming the co-fragmentation correction ran
- Output tables (e.g. fig3_quantification_comparison.csv, table1_single_ptm_abundances.csv) contain a row per peptide group with wt and Suz12-/- abundances and fold-changes

## Limitations

- Spectral counting is inaccurate for unmodified and extensively modified histone peptides; intensity-based quantification should be preferred for these
- isoScale requires Mascot .csv output and a Histone Coder result file — it cannot run directly on raw data
- Quantification principle follows Pesavento et al. (Analytical Chemistry, 2006) and depends on the chosen MS/MS ion tolerance for fragment-ion searching; mis-set tolerance degrades FIRR estimates

## Evidence

- [other] isoScale outperformed spectral counting by quantifying 18 of the 22 peptide groups, with quantification values in better agreement with the XIC reference: [evidence span withheld — anchor status: not_found]
