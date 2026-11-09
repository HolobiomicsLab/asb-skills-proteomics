---
name: plant-resistance-protein-dataset-preparation
description: Use when constructing a plant resistance (R) protein training/test benchmark
  from FASTA sequences, or when a classifier dataset requires redundancy removal at
  a low (30%) identity cutoff before feature extraction.
when_to_use_negative:
- The input is already a deduplicated feature table or numeric embedding matrix —
  CD-HIT deduplication operates on raw FASTA sequences before feature extraction
- The task requires pairwise identity above 30% to be tolerated (e.g. constructing
  a hard-negative set of close homologs) — this skill removes sequences above the
  30% identity threshold
- The dataset is not protein sequence data (e.g. nucleotide FASTA) — CD-HIT must then
  be substituted with a nucleotide-aware clustering tool
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0780
- http://edamontology.org/topic_3474
tools:
- name: CD-HIT
  role: Clusters and removes redundant sequences from the positive and negative FASTA
    sets at a 30% identity cutoff, retaining cluster representatives
- name: prPred-DRLF
  role: Source toolkit whose GitHub dataset directory hosts the Wang et al. [9] plant
    R protein positive/negative FASTA files and whose downstream pipeline consumes
    the filtered dataset
  repo: https://github.com/Wangys-prog/prPred-DRLF
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/pmic.202100161
    title: 'prPred‐DRLF: Plant R protein predictor using deep representation learning
      features'
  source_content_hash: sha256:848ee9c1900caa325e779075e60fbe273bd5c462ea8a32b508f81d7d62811c1e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wangys-prog/prPred-DRLF
  tools:
  - CD-HIT
  - prPred-DRLF
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0780
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# plant-resistance-protein-dataset-preparation

## Summary

Prepares the plant R protein benchmark used by prPred-DRLF: obtain the Wang et al. [9] plant resistance (R) protein dataset from the prPred-DRLF GitHub dataset directory, deduplicate positive and negative FASTA sets separately with CD-HIT at a 30% sequence identity cutoff, and verify the resulting 456-sequence dataset (152 positive, 304 negative; 364 train / 92 independent test).

## When to use

Use when constructing a plant resistance (R) protein training/test benchmark from FASTA sequences, or when a classifier dataset requires redundancy removal at a low (30%) identity cutoff before feature extraction.

## When NOT to use

- The input is already a deduplicated feature table or numeric embedding matrix — CD-HIT deduplication operates on raw FASTA sequences before feature extraction
- The task requires pairwise identity above 30% to be tolerated (e.g. constructing a hard-negative set of close homologs) — this skill removes sequences above the 30% identity threshold
- The dataset is not protein sequence data (e.g. nucleotide FASTA) — CD-HIT must then be substituted with a nucleotide-aware clustering tool

## Inputs

- Positive (plant R protein) sequences in FASTA format from the Wang et al. [9] dataset (prPred-DRLF GitHub dataset directory)
- Negative (non-R protein) sequences in FASTA format from the same prPred-DRLF dataset directory

## Outputs

- cdhit_filtered.fasta: deduplicated FASTA containing exactly 456 protein sequences (152 positive, 304 negative)
- Training and independent test partitions of the deduplicated dataset (364 training and 92 test samples)

## How to apply

Download the Wang et al. [9] plant R protein dataset (positive R protein and negative non-R protein FASTA files) from the prPred-DRLF GitHub repository's dataset directory (https://github.com/Wangys-prog/prPred-DRLF/tree/master/dataset). Run CD-HIT separately on the positive and negative FASTA files with the sequence identity threshold set to 0.3 (30% identity) so that sequences sharing more than 30% identity are clustered and only cluster representatives are retained; running the two classes separately preserves the class balance for downstream LGBM classification. Save the representatives as a deduplicated FASTA (e.g. cdhit_filtered.fasta). The rationale is that near-identical homologs inflate cross-validation and independent-test performance, so a strict 30% cutoff enforces low sequence similarity between samples before deep representation learning feature extraction (BiLSTM, UniRep, TAPE-BERT) and model construction.

## Related tools

- **CD-HIT** (Clusters and removes redundant sequences from the positive and negative FASTA sets at a 30% identity cutoff, retaining cluster representatives)
- **prPred-DRLF** (Source toolkit whose GitHub dataset directory hosts the Wang et al. [9] plant R protein positive/negative FASTA files and whose downstream pipeline consumes the filtered dataset) — https://github.com/Wangys-prog/prPred-DRLF

## Examples

```
cd-hit -i positive.fasta -o positive_cdhit.fasta -c 0.3 ; cd-hit -i negative.fasta -o negative_cdhit.fasta -c 0.3
```

## Evaluation signals

- The deduplicated FASTA contains exactly 456 sequences
- Class composition of the filtered dataset is exactly 152 positive and 304 negative samples
- The dataset splits into 364 training samples and 92 independent test samples
- Positive and negative sets were filtered separately (each retains only CD-HIT cluster representatives at -c 0.3), not pooled before clustering
- No pair of retained sequences within a class exceeds the 30% identity threshold applied by CD-HIT

## Limitations

- A 30% identity cutoff is very stringent and can substantially reduce dataset size, limiting the amount of training data for deep representation learning models
- The deep representation learning feature methods used downstream 'usually require an elevated level of computing resources', and parallel computing is not yet incorporated into the prPred-DRLF program, which affects the efficiency of the pipeline that consumes this dataset
- The HMMER baseline could only be evaluated on the positive training set because most negative training set sequences are annotated incompletely, so dataset quality checks should not rely on functional annotation of negatives

## Evidence
