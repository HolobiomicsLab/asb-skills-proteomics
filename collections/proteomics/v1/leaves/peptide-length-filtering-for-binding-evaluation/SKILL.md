---
name: peptide-length-filtering-for-binding-evaluation
description: Use when preparing peptide lists (e.g. from *_pep_final.tsv outputs at
  q-value <= 0.01) for NetMHCpan, MHCflurry, or HLAthena binding-affinity prediction
  on HLA class I data.
when_to_use_negative:
- The downstream analysis is general peptide identification or FDR assessment rather
  than MHC binding-affinity evaluation, where the article instead allows peptides
  of length 8–25
- The dataset contains MHC class II ligands or non-HLA-I peptides, for which the 8–11
  residue class I specificity window is inappropriate
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: NetMHCpan
  role: Binding-affinity prediction tool applied to the 8–11 aa filtered peptide sets
    (v4.0, percentile rank <= 2% defines binders)
- name: MHCflurry
  role: Binding-affinity prediction tool (v1.6.0) applied to the length-filtered peptide
    sets
- name: HLAthena
  role: Binding-affinity prediction tool applied to the length-filtered peptide sets
- name: DeepRescore
  role: Produces the *_pep_final.tsv peptide lists (q-value <= 0.01) that are length-filtered
    prior to binding evaluation
  repo: https://github.com/bzhanglab/DeepRescore
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pmic.201900334
    title: 'DeepRescore: Leveraging Deep Learning to Improve Peptide Identification
      in Immunopeptidomics'
  source_content_hash: sha256:ffce2fc6a1954847697892e2dac1d10648ac1000d8fe28b50f65ef697f9899b2
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# peptide-length-filtering-for-binding-evaluation

## Summary

Restrict identified immunopeptidomics peptide sets to 8–11 amino acids before MHC binding-affinity evaluation, mirroring the DeepRescore protocol in which this length window covers at least 75% of identified peptides and matches the allele binding specificity being predicted.

## When to use

Use when preparing peptide lists (e.g. from *_pep_final.tsv outputs at q-value <= 0.01) for NetMHCpan, MHCflurry, or HLAthena binding-affinity prediction on HLA class I data.

## When NOT to use

- The downstream analysis is general peptide identification or FDR assessment rather than MHC binding-affinity evaluation, where the article instead allows peptides of length 8–25
- The dataset contains MHC class II ligands or non-HLA-I peptides, for which the 8–11 residue class I specificity window is inappropriate

## Inputs

- Peptide lists from *_pep_final.tsv outputs (q-value <= 0.01) of DeepRescore-rescored Comet, MS-GF+, X!Tandem, and MaxQuant searches
- 1%-FDR-filtered peptide lists from each search engine alone (datasets D1: Mel15 PXD004894; D2: A*11:01 MSV000084442)

## Outputs

- Length-filtered (8–11 aa) peptide sets ready for NetMHCpan/MHCflurry/HLAthena binding-affinity prediction
- Per-method binder-ratio table (mhc_binder_ratios.csv) computed on the filtered peptides

## How to apply

After filtering peptide identifications at 1% FDR (q-value <= 0.01), remove peptides shorter than 8 or longer than 11 residues before running binding-affinity tools. The article first applies a broader 8–25 residue filter for general peptide identification, then narrows to 8–11 specifically for the binding-affinity evaluation, because this window covers at least 75% of peptides identified by Comet, MS-GF+, X!Tandem, and MaxQuant in datasets D1 (Mel15, PRIDE PXD004894) and D2 (HLA-A*11:01, MassIVE MSV000084442). Apply the same length window uniformly to DeepRescore-rescored lists and to each search engine's list so binder-ratio comparisons are computed over comparable peptide populations. After filtering, classify binders as peptides with a percentile rank score of 2% or less and compute the binder fraction per tool and per identification method.

## Related tools

- **NetMHCpan** (Binding-affinity prediction tool applied to the 8–11 aa filtered peptide sets (v4.0, percentile rank <= 2% defines binders))
- **MHCflurry** (Binding-affinity prediction tool (v1.6.0) applied to the length-filtered peptide sets)
- **HLAthena** (Binding-affinity prediction tool applied to the length-filtered peptide sets)
- **DeepRescore** (Produces the *_pep_final.tsv peptide lists (q-value <= 0.01) that are length-filtered prior to binding evaluation) — https://github.com/bzhanglab/DeepRescore

## Examples

```
awk 'length($1)>=8 && length($1)<=11' peptides_qval0.01.tsv > peptides_8to11_for_binding.tsv
```

## Evaluation signals

- All retained peptides have sequence length between 8 and 11 residues inclusive; no peptide outside this range remains in the evaluation input
- The length filter retains at least 75% of the identified peptides across search engines in D1 and D2, as reported in the paper (Figure S6 coverage)
- Identical length filtering is applied to DeepRescore and each search engine's peptide list so binder-ratio comparisons are over comparable populations
- Downstream binder classification uses the paper's threshold: percentile rank score of 2% or less defines an MHC binder

## Limitations

- The 8–11 residue window excludes up to ~25% of identified peptides, so binder ratios are not computed over the full identification set
- The filter is specific to HLA class I binding-affinity evaluation; the paper's general identification analysis used a broader 8–25 residue filter, so applying 8–11 elsewhere would bias sensitivity estimates

## Evidence
