---
name: peptide-set-binder-ratio-comparison
description: Use when you have two or more 8-11 aa peptide lists from immunopeptidomics
  searches (e.g., q-value <= 0.01 DeepRescore outputs vs. 1%-FDR search engine lists)
  and need an orthogonal quality check via predicted MHC binder fractions.
when_to_use_negative:
- Peptides are longer than 11 aa or shorter than 8 aa — the paper restricts the binding-affinity
  evaluation to peptides of length 8-11 (which covered at least 75% of identified
  peptides)
- The analysis goal is sensitivity/quantity of identifications rather than false-positive
  assessment — use identification-count comparisons instead of binder-ratio comparisons
- Sample HLA alleles are unknown, since per-peptide best-allele percentile ranks or
  single-allele predictions cannot be assigned
edam_operation: http://edamontology.org/operation_0416
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: NetMHCpan
  role: Predicts MHC-peptide binding affinity; percentile rank <= 2% defines a binder
- name: MHCflurry
  role: Second binding-affinity prediction tool (v1.6.0) used to cross-validate binder
    ratios
- name: HLAthena
  role: Third binding-affinity prediction tool used to cross-validate binder ratios
- name: DeepRescore
  role: Provides the *_pep_final.tsv peptide outputs (q-value <= 0.01) whose binder
    ratios are compared against search-engine-only identifications
  repo: https://github.com/bzhanglab/DeepRescore
- name: Comet
  role: Source of 1%-FDR-filtered baseline peptide identifications for comparison
- name: MS-GF+
  role: Source of 1%-FDR-filtered baseline peptide identifications for comparison
  repo: https://github.com/MSGFPlus/msgfplus
- name: X!Tandem
  role: Source of 1%-FDR-filtered baseline peptide identifications for comparison
- name: MaxQuant
  role: Source of 1%-FDR-filtered baseline peptide identifications for comparison
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pmic.201900334
    title: 'DeepRescore: Leveraging Deep Learning to Improve Peptide Identification
      in Immunopeptidomics'
  source_content_hash: sha256:c24b231c70e0ae7a360eb7c0ebebe9a1224c1383dac193c3b2fe41489de02d84
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
  edam_operation: http://edamontology.org/operation_0416
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# peptide-set-binder-ratio-comparison

## Summary

Evaluates the biological plausibility of immunopeptidomics peptide identification sets by predicting MHC-peptide binding affinity with NetMHCpan, MHCflurry, and HLAthena, classifying binders at percentile rank <= 2%, and comparing binder ratios across identification methods (e.g., DeepRescore-rescored vs. search-engine-only peptide lists).

## When to use

Use when you have two or more 8-11 aa peptide lists from immunopeptidomics searches (e.g., q-value <= 0.01 DeepRescore outputs vs. 1%-FDR search engine lists) and need an orthogonal quality check via predicted MHC binder fractions.

## When NOT to use

- Peptides are longer than 11 aa or shorter than 8 aa — the paper restricts the binding-affinity evaluation to peptides of length 8-11 (which covered at least 75% of identified peptides)
- The analysis goal is sensitivity/quantity of identifications rather than false-positive assessment — use identification-count comparisons instead of binder-ratio comparisons
- Sample HLA alleles are unknown, since per-peptide best-allele percentile ranks or single-allele predictions cannot be assigned

## Inputs

- Peptide lists from DeepRescore *_pep_final.tsv outputs filtered at q-value <= 0.01
- 1%-FDR-filtered peptide lists (target-decoy, PSM and peptide levels) from Comet, MS-GF+, X!Tandem, and MaxQuant
- HLA allele information per dataset (multiple alleles for D1/Mel15; A*11:01 for D2)
- NetMHCpan v4.0, MHCflurry v1.6.0, and HLAthena prediction outputs (percentile rank scores)

## Outputs

- Per-tool, per-method binder classifications (percentile rank <= 2%)
- mhc_binder_ratios.csv: binder ratios for each search engine alone and after DeepRescore rescoring, per prediction tool and per dataset

## How to apply

Load peptide lists from DeepRescore *_pep_final.tsv outputs (filter q-value <= 0.01) and the corresponding 1%-FDR-filtered peptide lists from Comet, MS-GF+, X!Tandem, and MaxQuant, retaining only peptides of length 8-11 as in the paper's binding-affinity evaluation. Run NetMHCpan (v4.0), MHCflurry (v1.6.0), and HLAthena on each set; for multi-allelic samples such as D1 (Mel15, PRIDE PXD004894) assign each peptide the best (lowest) percentile rank across all sample HLA alleles, and use the single A*11:01 allele for D2 (MassIVE MSV000084442). Classify a peptide as an MHC binder when its percentile rank score is 2% or less, separately per tool and per identification method. Compute the binder ratio (fraction of binders among evaluated peptides) for each search engine alone and after DeepRescore rescoring, and assemble the ratios into a comparison table saved as mhc_binder_ratios.csv. The rationale is that a correct identification pipeline for HLA ligands should yield a higher proportion of predicted binders, so higher ratios among DeepRescore-only peptides indicate fewer false positives.

## Related tools

- **NetMHCpan** (Predicts MHC-peptide binding affinity; percentile rank <= 2% defines a binder)
- **MHCflurry** (Second binding-affinity prediction tool (v1.6.0) used to cross-validate binder ratios)
- **HLAthena** (Third binding-affinity prediction tool used to cross-validate binder ratios)
- **DeepRescore** (Provides the *_pep_final.tsv peptide outputs (q-value <= 0.01) whose binder ratios are compared against search-engine-only identifications) — https://github.com/bzhanglab/DeepRescore
- **Comet** (Source of 1%-FDR-filtered baseline peptide identifications for comparison)
- **MS-GF+** (Source of 1%-FDR-filtered baseline peptide identifications for comparison) — https://github.com/MSGFPlus/msgfplus
- **X!Tandem** (Source of 1%-FDR-filtered baseline peptide identifications for comparison)
- **MaxQuant** (Source of 1%-FDR-filtered baseline peptide identifications for comparison)

## Examples

```
netMHCpan -p -a HLA-A11:01 -f peptides_8to11aa.txt > netmhcpan_A1101_output.tsv
```

## Evaluation signals

- Reproduced binder ratios should be higher for DeepRescore-unique peptides than for peptides not identified by DeepRescore, across all three tools and both datasets (paper: 86%-93% in D1 and 71%-86% in D2 for DeepRescore-unique peptides)
- Only peptides of length 8-11 are included, matching the paper's criterion of covering at least 75% of identified peptides
- Binder calls are consistent with the <= 2% percentile rank threshold applied uniformly per prediction tool
- For D1, each peptide's assigned score equals the minimum percentile rank across all sample HLA alleles; for D2, all predictions use A*11:01
- Output table mhc_binder_ratios.csv contains one ratio per (prediction tool, identification method, dataset) combination

## Limitations

- The binder ratio is a prediction-based proxy, not experimental validation; it assumes the binding predictors are accurate for the sample's HLA alleles
- Restricting to peptides of length 8-11 excludes a minority of identifications (the length filter covered at least 75% of identified peptides, not all)
- For multi-allelic samples like D1, using the best (lowest) percentile rank across alleles can overestimate binder fractions relative to true allele-of-origin assignments

## Evidence
