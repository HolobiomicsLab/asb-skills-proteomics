---
name: ptm-abundance-fold-change-comparison-between-cell-states
description: Use when Mascot/Histone Coder/isoScale middle-down results exist for
  two conditions (e.g. wt and knockout ESCs) and the goal is to identify which single
  histone PTMs changed significantly in relative abundance, beyond a headline change
  such as the seven-fold H3K27me2/me3 drop.
when_to_use_negative:
- Input is raw Mascot .xml output that has not yet been validated and converted to
  Brno nomenclature by Histone Coder, or .csv files not yet quantified by isoScale.
- The analysis targets bottom-up peptide-level quantification rather than middle-down/top-down
  intact-tail data, for which isoScale-based total-ion-intensity quantification of
  cofragmenting isobaric species is recommended.
- Only combinatorial multi-mark profiles are of interest and no single-PTM aggregation
  across peptide forms is required.
edam_operation: http://edamontology.org/operation_3638
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0601
tools:
- name: Iso Scale
  role: Quantifies peptides by total ion intensity of MS/MS spectra and provides relative
    quantification of isobaric peptides with distinct PTM localizations; its output
    tables are aggregated per single PTM
  repo: https://github.com/middle-down/Software/blob/master/isoScale-v2.jar?raw=true
- name: Histone Coder
  role: Validates PTM localization by counting site-determining MS/MS ions and converts
    Mascot output to Brno nomenclature, producing the result file consumed by isoScale
  repo: https://github.com/middle-down/Software/blob/master/Histone%20Coder.jar?raw=true
- name: Mascot
  role: Database searching engine that assigns PTM localization; its .xml output feeds
    Histone Coder and its .csv output feeds isoScale
- name: pandas
  role: Loads and aggregates the isoScale supplementary tables (S3, S4, S7, S8) to
    compute per-PTM wt vs Suz12-/- abundances and fold-changes
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pmic.201400084
    title: Middle‐down hybrid chromatography/tandem mass spectrometry workflow for
      characterization of combinatorial post‐translational modifications in histones
  source_content_hash: sha256:b0ba852ab33bdf0f9038099a2763240fc633fa9d4b938920aacbcbe98e48748d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Histone Coder
  - Iso Scale
  - Java
  - Mascot
  - pandas
  edam_operation: http://edamontology.org/operation_3638
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0601
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# ptm-abundance-fold-change-comparison-between-cell-states

## Summary

Compares relative abundances of single histone PTMs (e.g. H3K27me2/me3, H3K36me2, H3K9me2, H4K16ac) between two cell states (wt vs Suz12-/- ESCs) by aggregating isoScale total-ion-intensity quantifications over all peptide forms carrying each mark and computing fold-changes.

## When to use

Use when Mascot/Histone Coder/isoScale middle-down results exist for two conditions (e.g. wt and knockout ESCs) and the goal is to identify which single histone PTMs changed significantly in relative abundance, beyond a headline change such as the seven-fold H3K27me2/me3 drop.

## When NOT to use

- Input is raw Mascot .xml output that has not yet been validated and converted to Brno nomenclature by Histone Coder, or .csv files not yet quantified by isoScale.
- The analysis targets bottom-up peptide-level quantification rather than middle-down/top-down intact-tail data, for which isoScale-based total-ion-intensity quantification of cofragmenting isobaric species is recommended.
- Only combinatorial multi-mark profiles are of interest and no single-PTM aggregation across peptide forms is required.

## Inputs

- isoScale quantification output tables for wt and Suz12-/- ESCs (supplementary tables S3, S4, S7, S8)
- Histone Coder result file derived from Mascot .xml output
- Mascot .csv data file of identified polypeptides

## Outputs

- table1_single_ptm_abundances.csv: per-mark wt and Suz12-/- relative abundances and fold-changes for single PTMs (H3K27me2, H3K27me3, H3K36me2, H3K9me2, H4K16ac)
- Validation check confirming the seven-fold reduction of combined H3K27me2+H3K27me3 in Suz12-/- cells

## How to apply

Load the isoScale output supplementary tables (e.g. S3, S4, S7, S8), which were computed from a Histone Coder result file plus a Mascot .csv data file, into pandas. For each target single PTM (H3K27me2, H3K27me3, H3K36me2, H3K9me2, H4K16ac), sum the relative abundance over all peptide forms carrying that mark, since isoScale quantifies peptides by total ion intensity of MS/MS spectra and resolves cofragmenting isobaric species via the fragment ion relative ratio (FIRR). Compute wt vs Suz12-/- relative abundances and fold-changes per mark, expecting H3K27me2 ~75% in wt vs <10% in Suz12-/-, H3K36me2 dropping from 67% to 40%, H3K9me2 rising ~5-fold, and H4K16ac increasing from 7.1% to 11.8%. Write the aggregated single-PTM abundance and fold-change table to a CSV (e.g. table1_single_ptm_abundances.csv) for comparison against the reported Table 1 values.

## Related tools

- **Iso Scale** (Quantifies peptides by total ion intensity of MS/MS spectra and provides relative quantification of isobaric peptides with distinct PTM localizations; its output tables are aggregated per single PTM) — https://github.com/middle-down/Software/blob/master/isoScale-v2.jar?raw=true
- **Histone Coder** (Validates PTM localization by counting site-determining MS/MS ions and converts Mascot output to Brno nomenclature, producing the result file consumed by isoScale) — https://github.com/middle-down/Software/blob/master/Histone%20Coder.jar?raw=true
- **Mascot** (Database searching engine that assigns PTM localization; its .xml output feeds Histone Coder and its .csv output feeds isoScale)
- **pandas** (Loads and aggregates the isoScale supplementary tables (S3, S4, S7, S8) to compute per-PTM wt vs Suz12-/- abundances and fold-changes)

## Examples

```
import pandas as pd; s3=pd.read_csv('S3.csv'); agg=s3.groupby('PTM')['rel_abundance'].sum(); fc=agg['Suz12KO']/agg['WT']; agg.to_csv('table1_single_ptm_abundances.csv')
```

## Evaluation signals

- Computed H3K27me2+H3K27me3 abundance in Suz12-/- cells is reduced approximately seven-fold relative to wt.
- H3K36me2 drops from ~67% (wt) to ~40% (Suz12-/-) and H3K9me2 increases ~5-fold.
- H4K16ac rises from 7.1% to 11.8%; each reproduced single-PTM percentage matches the reported Table 1 value within rounding tolerance.
- Per-PTM totals are the sum over all peptide forms carrying that mark, with cofragmenting isobaric species resolved by FIRR before aggregation.
- Output CSV contains rows for H3K27me2, H3K27me3, H3K36me2, H3K9me2, and H4K16ac with wt and Suz12-/- abundances and fold-change columns.

## Limitations

- isoScale quantification relies on total ion intensity of MS/MS spectra and FIRR for isobaric species; accuracy depends on chosen MS/MS ion tolerance and fragment ion type filters.
- Requires Mascot .csv input plus a Histone Coder result file — raw .xml Mascot output alone is not directly processable by isoScale.
- Significant changes were reported for 16 other single PTMs of histone H3 beyond H3K27 methylation; aggregation choices over peptide forms affect which marks appear significantly regulated.

## Evidence
