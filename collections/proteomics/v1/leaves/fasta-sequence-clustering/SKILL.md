---
name: fasta-sequence-clustering
description: Use when a labeled protein dataset in FASTA format (e.g., plant R protein
  positives and negatives) must be deduplicated before feature extraction and classifier
  training.
when_to_use_negative:
- The input is already a feature table or embedding matrix (e.g., BiLSTM/UniRep vectors)
  rather than raw FASTA sequences — clustering is a sequence-level pre-step.
- The dataset is very small or sequences are already known to be non-redundant; aggressive
  30% identity clustering could discard genuinely distinct homologs.
- The goal is family-level classification where sequence redundancy carries biological
  signal rather than being an artifact to remove.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: CD-HIT
  role: Clusters FASTA sequences at a 30% identity cutoff and retains cluster representatives
    to remove redundancy from the positive and negative sets
- name: prPred-DRLF
  role: Source toolkit whose dataset directory provides the Wang et al. [9] plant
    R protein FASTA files that CD-HIT filtering is applied to
  repo: https://github.com/Wangys-prog/prPred-DRLF
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/pmic.202100161
    title: 'prPred‐DRLF: Plant R protein predictor using deep representation learning
      features'
  source_content_hash: sha256:07c7b7b96035398dc29c0f0ccbb1f32cce8fb528a96b8584184b1c9919dd1cd9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wangys-prog/prPred-DRLF
  tools:
  - CD-HIT
  - prPred-DRLF
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# fasta-sequence-clustering

## Summary

Removes redundant sequences from FASTA-format protein sets by clustering with CD-HIT at a low sequence-identity cutoff (30%) so that downstream machine-learning models are trained on non-redundant representatives.

## When to use

Use when a labeled protein dataset in FASTA format (e.g., plant R protein positives and negatives) must be deduplicated before feature extraction and classifier training.

## When NOT to use

- The input is already a feature table or embedding matrix (e.g., BiLSTM/UniRep vectors) rather than raw FASTA sequences — clustering is a sequence-level pre-step.
- The dataset is very small or sequences are already known to be non-redundant; aggressive 30% identity clustering could discard genuinely distinct homologs.
- The goal is family-level classification where sequence redundancy carries biological signal rather than being an artifact to remove.

## Inputs

- Positive-class protein sequences in FASTA format (R proteins)
- Negative-class protein sequences in FASTA format (non-R proteins)
- Sequence identity cutoff parameter (0.3)

## Outputs

- cdhit_filtered.fasta — deduplicated FASTA with 456 sequences (152 positive, 304 negative)
- Training/test split derived from the filtered set (364 training, 92 independent test samples)

## How to apply

Obtain the positive and negative FASTA files (here the Wang et al. [9] plant R protein dataset from the prPred-DRLF GitHub dataset directory) and run CD-HIT separately on each set with the sequence identity cutoff set to 0.3 (30% identity), so CD-HIT clusters near-identical sequences and retains one representative per cluster. After filtering, verify the deduplicated dataset contains 456 protein sequences: 152 positive (R protein) and 304 negative (non-R protein). Save the representative sequences (e.g., to cdhit_filtered.fasta) and partition them as in the paper: 364 training samples and 92 independent test samples. The 30% identity threshold is deliberately strict — it removes even remotely similar homologs, which is standard practice to prevent train/test leakage and to keep the LGBM-based prPred-DRLF evaluation honest.

## Related tools

- **CD-HIT** (Clusters FASTA sequences at a 30% identity cutoff and retains cluster representatives to remove redundancy from the positive and negative sets)
- **prPred-DRLF** (Source toolkit whose dataset directory provides the Wang et al. [9] plant R protein FASTA files that CD-HIT filtering is applied to) — https://github.com/Wangys-prog/prPred-DRLF

## Examples

```
cd-hit -i positive.fasta -o positive_cdhit.fasta -c 0.3 ; cd-hit -i negative.fasta -o negative_cdhit.fasta -c 0.3
```

## Evaluation signals

- The filtered FASTA contains exactly 456 sequences, matching the paper's post-CD-HIT count.
- Class composition is preserved as 152 positive and 304 negative samples after filtering.
- The split yields 364 training samples and 92 independent test samples, as reported in the methods.
- CD-HIT was run separately on the positive and negative files (not pooled) with -c 0.3.
- Output file (cdhit_filtered.fasta) parses as valid FASTA and every sequence is a cluster representative of the original input.

## Limitations

- A 30% identity cutoff is very aggressive and may remove distantly related but biologically distinct R proteins, shrinking the dataset before feature extraction.
- Deep representation learning feature methods (BiLSTM, UniRep, TAPE-BERT) used downstream require elevated computing resources; a GPU speeds up the pipeline but is not required for the CD-HIT step itself.
- Parallel computing is not yet incorporated into the prPred-DRLF program, limiting computational efficiency of the full pipeline.

## Evidence

- [methods] The dataset employed in this study is based on Wang et al. [9]: [evidence span withheld — anchor status: not_found]
