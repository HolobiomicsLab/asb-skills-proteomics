---
name: sequence-identity-thresholding
description: Use when a labeled protein sequence dataset (e.g., positive/negative
  FASTA files of plant R proteins) must be deduplicated before feature extraction
  and model construction to remove redundant homologs.
when_to_use_negative:
- The input is already a deduplicated or non-redundant sequence set and no homolog
  clustering is needed
- The goal is to preserve fine-grained sequence diversity for analysis rather than
  train a classifier on non-redundant representatives
- Data are not biological sequences (e.g., a feature table or numeric matrix), where
  CD-HIT identity clustering does not apply
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: CD-HIT
  role: Clusters sequences at the 30% identity cutoff and outputs non-redundant cluster
    representatives for both the positive and negative sets
- name: prPred-DRLF
  role: Provides the source plant R protein dataset (Wang et al. [9]) in its GitHub
    dataset directory that is subjected to CD-HIT filtering
  repo: https://github.com/Wangys-prog/prPred-DRLF
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/pmic.202100161
    title: 'prPred‐DRLF: Plant R protein predictor using deep representation learning
      features'
  source_content_hash: sha256:3c91795e32c8991e7acae60769d735da51df43665657cabdf3db2f91d46cb80e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wangys-prog/prPred-DRLF
  tools:
  - CD-HIT
  - prPred-DRLF
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# sequence-identity-thresholding

## Summary

Deduplicate a protein FASTA dataset by clustering sequences with CD-HIT at a low sequence-identity cutoff (30%) and retaining cluster representatives, so that downstream feature extraction and classifier training are not biased by near-identical sequences.

## When to use

Use when a labeled protein sequence dataset (e.g., positive/negative FASTA files of plant R proteins) must be deduplicated before feature extraction and model construction to remove redundant homologs.

## When NOT to use

- The input is already a deduplicated or non-redundant sequence set and no homolog clustering is needed
- The goal is to preserve fine-grained sequence diversity for analysis rather than train a classifier on non-redundant representatives
- Data are not biological sequences (e.g., a feature table or numeric matrix), where CD-HIT identity clustering does not apply

## Inputs

- Positive-set protein sequences in FASTA format (R proteins)
- Negative-set protein sequences in FASTA format (non-R proteins)

## Outputs

- cdhit_filtered.fasta: deduplicated FASTA containing exactly 456 sequences (152 positive, 304 negative)
- Sequence counts per class for downstream 364/92 training/test splitting

## How to apply

Run CD-HIT separately on the positive and negative FASTA files with the sequence identity threshold set to 0.3 (30% identity), letting CD-HIT cluster redundant sequences and keep one representative per cluster. Retain the cluster representatives as the deduplicated dataset and save them as a filtered FASTA (e.g., cdhit_filtered.fasta). In the prPred-DRLF study this step was applied to the plant R protein dataset of Wang et al. [9] taken from the prPred-DRLF GitHub dataset directory, and the 30% identity cutoff was chosen to aggressively remove homologous redundancy before deep representation learning feature extraction (BiLSTM, UniRep, TAPE-BERT) and LGBM classification. Validate the result by counting sequences: the filtered set must contain 456 protein sequences (152 positive, 304 negative), which was then split into 364 training and 92 independent test samples.

## Related tools

- **CD-HIT** (Clusters sequences at the 30% identity cutoff and outputs non-redundant cluster representatives for both the positive and negative sets)
- **prPred-DRLF** (Provides the source plant R protein dataset (Wang et al. [9]) in its GitHub dataset directory that is subjected to CD-HIT filtering) — https://github.com/Wangys-prog/prPred-DRLF

## Examples

```
cd-hit -i positive.fasta -o cdhit_filtered.fasta -c 0.3 ; cd-hit -i negative.fasta -o negative_cdhit.fasta -c 0.3
```

## Evaluation signals

- The deduplicated FASTA contains exactly 456 sequences, matching the counts reported after CD-HIT filtering
- Class composition is preserved: 152 positive (R protein) and 304 negative (non-R protein) samples
- The downstream split yields 364 training and 92 independent test samples
- CD-HIT was run separately on positive and negative sets with the identity cutoff set to 0.3
- Output is a valid FASTA (cdhit_filtered.fasta) whose sequences are cluster representatives

## Limitations

- A 30% identity cutoff is very aggressive; clusters at this threshold may merge distantly related homologs, so rare sequence variants may be discarded
- CD-HIT does not incorporate parallel computing in the prPred-DRLF workflow, and deep representation learning downstream requires elevated computing resources
- The fixed 456-sequence size is specific to the Wang et al. [9] plant R protein dataset; other inputs will yield different counts

## Evidence

- [methods] Dataset source for filtering: [evidence span withheld — anchor status: not_found]
