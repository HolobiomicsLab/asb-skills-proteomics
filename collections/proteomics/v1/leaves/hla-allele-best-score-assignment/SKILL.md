---
name: hla-allele-best-score-assignment
description: Use when evaluating predicted MHC-binder fractions for peptide lists
  from HLA samples with multiple HLA alleles (need -score-across-alleles assignment)
  or a mono-allelic line with one fixed allele, before applying a percentile-rank
  binder cutoff.
when_to_use_negative:
- Input peptides include lengths outside 8-11, which the paper excludes from the binding-affinity
  evaluation (lengths 8-25 were kept elsewhere)
- The sample's HLA alleles are unknown, so no allele-specific percentile rank can
  be assigned
- You need per-allele binder counts rather than one ratio per peptide set — best-score-across-alleles
  collapses allele information
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0625
tools:
- name: NetMHCpan
  role: Binding affinity prediction producing percentile rank scores (v4.0)
- name: MHCflurry
  role: Binding affinity prediction producing percentile rank scores (v1.6.0)
- name: HLAthena
  role: Binding affinity prediction producing percentile rank scores
- name: DeepRescore
  role: Source of the DeepRescore-only peptide lists being evaluated against search-engine-only
    lists
  repo: https://github.com/bzhanglab/DeepRescore
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pmic.201900334
    title: 'DeepRescore: Leveraging Deep Learning to Improve Peptide Identification
      in Immunopeptidomics'
  source_content_hash: sha256:306a8169cbc50f5f61f870f96a16ca050b6e13f8e5694d4a87789d154f7b538c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/bzhanglab/DeepRescore
  tools:
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
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# hla-allele-best-score-assignment

## Summary

Assigns each immunopeptidomics peptide a single binding-prediction score by taking the best (lowest) percentile rank across all HLA alleles of a multi-allelic sample (e.g. Mel15, D1), or the single known allele for mono-allelic cell lines (A*11:01, D2), so binder fractions from NetMHCpan, MHCflurry, and HLAthena are computed per identification method.

## When to use

Use when evaluating predicted MHC-binder fractions for peptide lists from HLA samples with multiple HLA alleles (need best-score-across-alleles assignment) or a mono-allelic line with one fixed allele, before applying a percentile-rank binder cutoff.

## When NOT to use

- Input peptides include lengths outside 8-11, which the paper excludes from the binding-affinity evaluation (lengths 8-25 were kept elsewhere)
- The sample's HLA alleles are unknown, so no allele-specific percentile rank can be assigned
- You need per-allele binder counts rather than one ratio per peptide set — best-score-across-alleles collapses allele information

## Inputs

- Filtered peptide lists (lengths 8-11) from DeepRescore *_pep_final.tsv outputs (q-value <= 0.01) and 1%-FDR-filtered lists from Comet, MS-GF+, X!Tandem, and MaxQuant
- HLA allele set for the sample (all alleles for D1/Mel15; HLA-A*11:01 for D2)

## Outputs

- Per-peptide best percentile rank score per prediction tool (NetMHCpan, MHCflurry, HLAthena)
- Binder classification per peptide (percentile rank <= 2%)
- Binder ratio table per search engine and per DeepRescore rescoring, saved as mhc_binder_ratios.csv

## How to apply

Run NetMHCpan (v4.0), MHCflurry (v1.6.0), and HLAthena on each peptide set restricted to lengths 8-11, as used in the paper's binding-affinity evaluation. For multi-allelic dataset D1 (Mel15, PRIDE PXD004894), assign each peptide the best (lowest) percentile rank score across all HLA alleles of the sample, so a peptide bound by any presented allele counts once. For mono-allelic dataset D2 (HLA-A*11:01, MassIVE MSV000084442), score against A*11:01 only. Classify a peptide as an MHC binder when its percentile rank score is 2% or less, separately per prediction tool and per identification method (each search engine alone vs. after DeepRescore rescoring), then compute the binder ratio (fraction of binders) and assemble the comparison table. The rationale is that allele-level best-score assignment avoids inflating or deflating binder fractions by allele multiplicity when only one ratio per peptide set is reported.

## Related tools

- **NetMHCpan** (Binding affinity prediction producing percentile rank scores (v4.0))
- **MHCflurry** (Binding affinity prediction producing percentile rank scores (v1.6.0))
- **HLAthena** (Binding affinity prediction producing percentile rank scores)
- **DeepRescore** (Source of the DeepRescore-only peptide lists being evaluated against search-engine-only lists) — https://github.com/bzhanglab/DeepRescore

## Examples

```
netMHCpan -p pep8to11_D2.txt -a HLA-A*11:01
```

## Evaluation signals

- Each peptide has exactly one percentile rank value per tool: minimum across all sample alleles for D1, the A*11:01 value for D2
- Binder is defined as percentile rank score <= 2%; ratios recomputed with a different cutoff should change monotonically
- Reproduced binder ratios should be higher for DeepRescore-only peptides (86%-93% in D1, 71%-86% in D2 reported) than for search-engine-only peptides across all three tools
- Only peptides of length 8-11 are evaluated; verify shorter/longer peptides are excluded before scoring
- mhc_binder_ratios.csv contains one ratio per (tool, identification-method, dataset) combination with no missing cells

## Limitations

- Taking the best score across alleles can overestimate binding when the presenting allele in the experiment is not the best-scoring one
- Best-score assignment was applied specifically to D1 (multi-allelic Mel15); D2 is mono-allelic so the 'best across alleles' logic reduces to a single allele
- The binder definition (percentile rank <= 2%) is a prediction-based proxy, not experimental validation of MHC binding

## Evidence

- [readme] netMHCpan availability: [evidence span withheld — anchor status: external_doc]
