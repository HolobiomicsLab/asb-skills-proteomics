---
name: percentile-rank-binder-classification
description: Use when evaluating whether peptide identifications from immunopeptidomics
  searches (e.g., DeepRescore *_pep_final.tsv outputs or 1%-FDR-filtered search-engine
  peptide lists) are enriched for predicted MHC binders using binding-affinity prediction
  tools.
when_to_use_negative:
- When peptides fall outside length 8-11 — the paper restricts the binding-affinity
  evaluation to this range (it covered at least 75% of identified peptides), so results
  on other lengths are out of scope
- When no HLA allele information is available for the sample — the percentile rank
  scoring requires assigning alleles (per-allele best rank for multi-allelic D1, A*11:01
  for D2)
- When the goal is to evaluate PSM-level or spectrum-level quality rather than predicted
  MHC binding of peptide sequences
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: NetMHCpan
  role: Predicts MHC-peptide binding affinity; percentile rank scores are thresholded
    at 2% to define binders
- name: MHCflurry
  role: Predicts MHC-peptide binding affinity (v1.6.0) used alongside NetMHCpan for
    binder-ratio comparison
- name: HLAthena
  role: Third binding-affinity prediction tool used to compute binder ratios as a
    cross-tool validation
- name: DeepRescore
  role: Source of the *_pep_final.tsv peptide outputs (q-value <= 0.01) whose unique
    peptides are evaluated for binder enrichment
  repo: https://github.com/bzhanglab/DeepRescore
- name: Comet
  role: Search engine whose 1%-FDR-filtered peptide lists provide the baseline (non-rescored)
    binder-ratio comparison
- name: MS-GF+
  role: Search engine whose 1%-FDR-filtered peptide lists provide the baseline (non-rescored)
    binder-ratio comparison
  repo: https://github.com/MSGFPlus/msgfplus
- name: X!Tandem
  role: Search engine whose 1%-FDR-filtered peptide lists provide the baseline (non-rescored)
    binder-ratio comparison
- name: MaxQuant
  role: Search engine whose 1%-FDR-filtered peptide lists provide the baseline (non-rescored)
    binder-ratio comparison
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pmic.201900334
    title: 'DeepRescore: Leveraging Deep Learning to Improve Peptide Identification
      in Immunopeptidomics'
  source_content_hash: sha256:306b0cf69641f3a840f2ea6b6d355bf5052d713c662d61d83ffa24903d9d3961
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/bzhanglab/DeepRescore
  tools:
  - Comet
  - DeepRescore
  - HLAthena
  - MHCflurry
  - MS-GF+
  - MaxQuant
  - NetMHCpan
  - X!Tandem
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# percentile-rank-binder-classification

## Summary

Classifies MHC-presented peptides as binders by applying a percentile-rank-score threshold (<=2%) to NetMHCpan, MHCflurry, and HLAthena binding-affinity predictions, and compares binder ratios across peptide identification methods (Comet, MS-GF+, X!Tandem, MaxQuant, vs. DeepRescore-rescored).

## When to use

Use when evaluating whether peptide identifications from immunopeptidomics searches (e.g., DeepRescore *_pep_final.tsv outputs or 1%-FDR-filtered search-engine peptide lists) are enriched for predicted MHC binders using binding-affinity prediction tools.

## When NOT to use

- When peptides fall outside length 8-11 — the paper restricts the binding-affinity evaluation to this range (it covered at least 75% of identified peptides), so results on other lengths are out of scope
- When no HLA allele information is available for the sample — the percentile rank scoring requires assigning alleles (per-allele best rank for multi-allelic D1, A*11:01 for D2)
- When the goal is to evaluate PSM-level or spectrum-level quality rather than predicted MHC binding of peptide sequences

## Inputs

- DeepRescore *_pep_final.tsv peptide outputs (q-value <= 0.01) for datasets D1 (Mel15, PRIDE PXD004894) and D2 (HLA-A*11:01, MassIVE MSV000084442)
- 1%-FDR-filtered peptide lists from Comet, MS-GF+, X!Tandem, and MaxQuant
- HLA allele information (all alleles of Mel15 for D1; HLA-A*11:01 for D2)
- NetMHCpan v4.0, MHCflurry v1.6.0, and HLAthena binding-affinity prediction outputs with percentile rank scores

## Outputs

- Per-tool, per-method binder classifications (binder if percentile rank score <= 2%) for peptides of length 8-11
- Comparison table of binder ratios (e.g., mhc_binder_ratios.csv) for each search engine alone and after DeepRescore rescoring

## How to apply

Load peptide lists filtered at q-value <= 0.01 from DeepRescore *_pep_final.tsv outputs and the corresponding 1%-FDR-filtered peptide lists from Comet, MS-GF+, X!Tandem, and MaxQuant, retaining only peptides of length 8-11 as used in the paper's binding-affinity evaluation. Run NetMHCpan (v4.0), MHCflurry (v1.6.0), and HLAthena on each peptide set; for dataset D1 (Mel15, PRIDE PXD004894), which has multiple HLA alleles, assign each peptide the best (lowest) percentile rank score across all HLA alleles of the sample, and use the A*11:01 allele for dataset D2 (HLA-A*11:01 mono-allelic cell line, MassIVE MSV000084442). Classify a peptide as an MHC binder when its percentile rank score is 2% or less, separately for each prediction tool and each identification method. Compute the binder ratio (fraction of binders among evaluated peptides) per search engine alone and after DeepRescore rescoring, then assemble the ratios into a comparison table (e.g., mhc_binder_ratios.csv). The rationale is that a higher binder fraction among DeepRescore-unique peptides indicates fewer false positives among DeepRescore-only identifications.

## Related tools

- **NetMHCpan** (Predicts MHC-peptide binding affinity; percentile rank scores are thresholded at 2% to define binders)
- **MHCflurry** (Predicts MHC-peptide binding affinity (v1.6.0) used alongside NetMHCpan for binder-ratio comparison)
- **HLAthena** (Third binding-affinity prediction tool used to compute binder ratios as a cross-tool validation)
- **DeepRescore** (Source of the *_pep_final.tsv peptide outputs (q-value <= 0.01) whose unique peptides are evaluated for binder enrichment) — https://github.com/bzhanglab/DeepRescore
- **Comet** (Search engine whose 1%-FDR-filtered peptide lists provide the baseline (non-rescored) binder-ratio comparison)
- **MS-GF+** (Search engine whose 1%-FDR-filtered peptide lists provide the baseline (non-rescored) binder-ratio comparison) — https://github.com/MSGFPlus/msgfplus
- **X!Tandem** (Search engine whose 1%-FDR-filtered peptide lists provide the baseline (non-rescored) binder-ratio comparison)
- **MaxQuant** (Search engine whose 1%-FDR-filtered peptide lists provide the baseline (non-rescored) binder-ratio comparison)

## Examples

```
df = pd.read_csv('mhc_binder_ratios.csv'); df['binder'] = df['percentile_rank'] <= 2.0; print(df.groupby(['tool','method'])['binder'].mean())
```

## Evaluation signals

- DeepRescore-unique peptides should show higher binder ratios than peptides not identified by DeepRescore across all three tools: 86%-93% in D1 and 71%-86% in D2 (vs. lower ratios for non-DeepRescore identifications)
- Every peptide classified must have a length between 8 and 11 residues
- A peptide counts as a binder only when its percentile rank score is 2% or less, applied consistently per tool and per identification method
- For D1 peptides, the assigned score must be the best (lowest) percentile rank across all sample HLA alleles; for D2 it must come from HLA-A*11:01 only
- The output table must include one binder ratio per combination of prediction tool (NetMHCpan, MHCflurry, HLAthena) and identification method (Comet, MS-GF+, X!Tandem, MaxQuant, DeepRescore-rescored)

## Limitations

- The binder ratio is a prediction-based proxy for correctness, not experimental validation; it assumes higher predicted-binder fraction implies fewer false positives
- The length 8-11 filter covers at least 75% but not all identified peptides, so the evaluation excludes longer identifications
- For multi-allelic samples (D1), taking the best percentile rank across all alleles can overestimate binding for individual peptides
- Results depend on the specific tool versions (NetMHCpan v4.0, MHCflurry v1.6.0) and HLA allele assignments

## Evidence

- [readme] NetMHCpan is installed as part of the DeepRescore/NeoFlow pipeline: [evidence span withheld — anchor status: external_doc]
