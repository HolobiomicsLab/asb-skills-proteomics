---
name: mhc-binding-affinity-prediction
description: Use when you need to assess whether peptide identifications (e.g., from
  rescoring or different search engines) are enriched for true MHC ligands by comparing
  predicted binder ratios across tools.
when_to_use_negative:
- Peptides outside length 8-11 — the paper restricted the binding-affinity evaluation
  to this range, which covered at least 75% of identified peptides
- Non-immunopeptidomics (e.g., tryptic digest) identifications where MHC-binder enrichment
  is not a meaningful quality metric
- When experimental binding or elution validation data are available instead of predicted
  affinity
edam_operation: http://edamontology.org/operation_0416
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3520
tools:
- name: NetMHCpan
  role: Predicts MHC-peptide binding affinity (percentile rank scores) for the binder-ratio
    evaluation
- name: MHCflurry
  role: Second binding-affinity prediction tool (v1.6.0) used to compute binder ratios
- name: HLAthena
  role: Third binding-affinity prediction tool used to compute binder ratios
- name: DeepRescore
  role: Source of the rescued peptide identifications whose binder ratio is compared
    against search-engine-only identifications
  repo: https://github.com/bzhanglab/DeepRescore
- name: Comet
  role: Search engine providing baseline 1%-FDR peptide lists for comparison
- name: MS-GF+
  role: Search engine providing baseline 1%-FDR peptide lists for comparison
  repo: https://github.com/MSGFPlus/msgfplus
- name: X!Tandem
  role: Search engine providing baseline 1%-FDR peptide lists for comparison
- name: MaxQuant
  role: Search engine providing baseline 1%-FDR peptide lists for comparison
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pmic.201900334
    title: 'DeepRescore: Leveraging Deep Learning to Improve Peptide Identification
      in Immunopeptidomics'
  source_content_hash: sha256:75b26ee88c02119eebfd9fdb438e6aa811b62c6c4a20d25a9b51157220a260e7
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
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# mhc-binding-affinity-prediction

## Summary

Evaluate the quality of immunopeptidomics peptide identifications by predicting MHC-peptide binding affinity with NetMHCpan, MHCflurry, and HLAthena and computing the fraction of predicted binders per identification method, as done to validate DeepRescore-rescored peptides.

## When to use

Use when you need to assess whether peptide identifications (e.g., from rescoring or different search engines) are enriched for true MHC ligands by comparing predicted binder ratios across tools.

## When NOT to use

- Peptides outside length 8-11 — the paper restricted the binding-affinity evaluation to this range, which covered at least 75% of identified peptides
- Non-immunopeptidomics (e.g., tryptic digest) identifications where MHC-binder enrichment is not a meaningful quality metric
- When experimental binding or elution validation data are available instead of predicted affinity

## Inputs

- 1%-FDR-filtered peptide lists (q-value <= 0.01) from search engines (Comet, MS-GF+, X!Tandem, MaxQuant) and DeepRescore *_pep_final.tsv outputs
- HLA allele information per sample (e.g., multiple alleles for D1/Mel15; HLA-A*11:01 for D2)
- NetMHCpan v4.0, MHCflurry v1.6.0, and HLAthena prediction outputs (percentile rank scores)

## Outputs

- Per-tool, per-method binder ratios (fraction of peptides with percentile rank <= 2%)
- Comparison table of binder ratios (saved as mhc_binder_ratios.csv) for search-engine-only vs. DeepRescore-rescored peptides

## How to apply

Load 1%-FDR-filtered peptide lists (e.g., q-value <= 0.01 from DeepRescore *_pep_final.tsv outputs and corresponding Comet, MS-GF+, X!Tandem, MaxQuant results) and retain only peptides of length 8-11, the range used for the binding-affinity evaluation. Run NetMHCpan (v4.0), MHCflurry (v1.6.0), and HLAthena on each peptide set; for multi-allelic samples such as D1 (Mel15), assign each peptide the best (lowest) percentile rank across all sample HLA alleles, and use the sample allele (e.g., HLA-A*11:01 for D2) for mono-allelic data. Classify a peptide as an MHC binder when its percentile rank score is 2% or less, separately per tool and per identification method. Compute the binder ratio (fraction of binders among evaluated peptides) for each search engine alone versus after DeepRescore rescoring and assemble the ratios into a comparison table.

## Related tools

- **NetMHCpan** (Predicts MHC-peptide binding affinity (percentile rank scores) for the binder-ratio evaluation)
- **MHCflurry** (Second binding-affinity prediction tool (v1.6.0) used to compute binder ratios)
- **HLAthena** (Third binding-affinity prediction tool used to compute binder ratios)
- **DeepRescore** (Source of the rescued peptide identifications whose binder ratio is compared against search-engine-only identifications) — https://github.com/bzhanglab/DeepRescore
- **Comet** (Search engine providing baseline 1%-FDR peptide lists for comparison)
- **MS-GF+** (Search engine providing baseline 1%-FDR peptide lists for comparison) — https://github.com/MSGFPlus/msgfplus
- **X!Tandem** (Search engine providing baseline 1%-FDR peptide lists for comparison)
- **MaxQuant** (Search engine providing baseline 1%-FDR peptide lists for comparison)

## Examples

```
netMHCpan -p -a HLA-A*11:01 -f peptides_8to11.txt > netmhcpan_output.tsv  # then flag rows with %Rank_EL <= 2 as binders
```

## Evaluation signals

- Reproduced binder ratios should show DeepRescore-unique peptides have higher predicted binder fractions (86%-93% in D1, 71%-86% in D2) than peptides not identified by DeepRescore, across all three prediction tools
- Every evaluated peptide must be classified with a percentile rank score threshold of exactly 2% or less for binder status
- Peptide sets must be restricted to length 8-11 before prediction; the evaluated set should cover at least 75% of identified peptides
- For multi-allelic D1, each peptide must receive the minimum percentile rank across all sample HLA alleles, not a single-allele score
- Ratios must be computed separately per prediction tool and per identification method (each search engine alone and after rescoring)

## Limitations

- Binder ratios rely on predicted rather than measured binding affinity; percentile-rank-based binder definitions are proxy validations
- Only peptides of length 8-11 were considered, so peptides outside this range are excluded from the evaluation
- For multi-allelic samples, assigning the best percentile rank across alleles assumes the presenting allele is among the typed HLA alleles

## Evidence
