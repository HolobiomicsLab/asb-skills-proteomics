---
name: protein-sequence-embedding-generation
description: Use when raw protein sequences in FASTA format must be converted into
  fixed-length numerical feature vectors for machine-learning classification, following
  the deep representation learning approach of prPred-DRLF.
when_to_use_negative:
- Input is already a numerical feature table or embedding matrix — this skill operates
  on raw amino acid FASTA sequences
- The analysis targets non-protein sequences (e.g., nucleotides) or tasks outside
  plant R protein identification where these pretrained protein embeddings are unvalidated
- Computing resources are severely limited — the article notes deep representation
  learning features 'usually require an elevated level of computing resources'
edam_operation: http://edamontology.org/operation_3094
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0080
tools:
- name: eFeature
  role: Toolkit used to run the pretrained BiLSTM, UniRep, and TAPE-BERT models and
    embed protein sequences into numerical vectors
- name: prPred-DRLF
  role: Python pipeline that wraps embedding generation, feature fusion, and prediction;
    sequences and scripts obtained from its GitHub repository
  repo: https://github.com/Wangys-prog/prPred-DRLF
- name: BiLSTM
  role: Deep representation learning model producing 3605D per-sequence embeddings
- name: TAPE (tasks assessing protein embedding)
  role: BERT-based semi-supervised protein embedding model producing 768D vectors,
    used for alternative feature combinations
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_006
  - task_005
  source_papers:
  - doi: 10.1002/pmic.202100161
    title: 'prPred‐DRLF: Plant R protein predictor using deep representation learning
      features'
  source_content_hash: sha256:211706192c8b3ac408b84c2a89e5c25b3f2eef8b6a9bc3ba29f086720b5a0bbe
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wangys-prog/prPred-DRLF
  tools:
  - BiLSTM
  - CD-HIT
  - Python
  - TAPE (tasks assessing protein embedding)
  - eFeature
  - prPred-DRLF
  edam_operation: http://edamontology.org/operation_3094
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# protein-sequence-embedding-generation

## Summary

Encode raw plant protein amino acid sequences into high-dimensional numerical vectors using deep representation learning models (BiLSTM 3605D, UniRep 1900D, TAPE-BERT 768D) via eFeature, then fuse BiLSTM+UniRep into a 5505D vector for downstream LGBM-based R protein classification.

## When to use

Use when raw protein sequences in FASTA format must be converted into fixed-length numerical feature vectors for machine-learning classification, following the deep representation learning approach of prPred-DRLF.

## When NOT to use

- Input is already a numerical feature table or embedding matrix — this skill operates on raw amino acid FASTA sequences
- The analysis targets non-protein sequences (e.g., nucleotides) or tasks outside plant R protein identification where these pretrained protein embeddings are unvalidated
- Computing resources are severely limited — the article notes deep representation learning features 'usually require an elevated level of computing resources'

## Inputs

- Protein sequences in FASTA format (e.g., dataset/test_data.fasta from the Wang et al. plant R protein dataset)
- Pretrained BiLSTM, UniRep, and TAPE-BERT embedding models accessible through eFeature

## Outputs

- Per-model numerical feature matrices: BiLSTM 3605D, UniRep 1900D, TAPE-BERT 768D
- Fused BiLSTM+UniRep 5505D feature matrix with sequence labels (1 = R protein, 0 = non-R protein), e.g., fused_embeddings.csv

## How to apply

Load the CD-HIT-filtered FASTA dataset (456 sequences after a 30% identity cutoff: 152 positive, 304 negative) from the prPred-DRLF repository. Run each sequence through the pretrained BiLSTM embedding model and the pretrained UniRep (1900-hidden-unit mLSTM pretrained on UniRef50) model via the eFeature toolkit, yielding 3605D and 1900D vectors respectively; TAPE-BERT (768D, BERT-based semi-supervised embedding per Rao et al.) can also be generated for comparison. Concatenate per-sequence BiLSTM and UniRep vectors into a fused 5505D feature vector, and save the fused feature matrix with class labels (1 = R protein, 0 = non-R protein) as a CSV. Validate that the fused matrix has exactly 5505 columns and one row per input sequence before passing to LGBM feature selection and classification.

## Related tools

- **eFeature** (Toolkit used to run the pretrained BiLSTM, UniRep, and TAPE-BERT models and embed protein sequences into numerical vectors)
- **prPred-DRLF** (Python pipeline that wraps embedding generation, feature fusion, and prediction; sequences and scripts obtained from its GitHub repository) — https://github.com/Wangys-prog/prPred-DRLF
- **BiLSTM** (Deep representation learning model producing 3605D per-sequence embeddings)
- **TAPE (tasks assessing protein embedding)** (BERT-based semi-supervised protein embedding model producing 768D vectors, used for alternative feature combinations)

## Examples

```
python3 prPred-DRLF.py -i ./dataset/test_data.fasta -o ./dataset/predict_result
```

## Evaluation signals

- The fused BiLSTM+UniRep feature matrix must have exactly 5505 feature columns (3605 + 1900) and one row per input sequence (456 after CD-HIT filtering)
- Per-model dimensionality checks: BiLSTM = 3605D, UniRep = 1900D, TAPE-BERT = 768D
- Every sequence receives a finite numeric vector; no rows dropped during embedding
- Downstream check: feeding the fused features into the LGBM classifier on the 92-sample independent test set should reproduce accuracy 0.956, F1 0.933, AUC 0.997 within tolerance

## Limitations

- Deep representation learning feature methods require an elevated level of computing resources; a GPU speeds up embedding (README recommends CUDA-enabled PyTorch builds)
- Parallel computing is not yet incorporated into the program, limiting computational efficiency
- Embedding is environment-sensitive: the README pins Python 3.7 and specific package versions (torch 1.2.0/1.3.1, tape_proteins 0.5, lightgbm 2.3.0), and directs users to a GitLab mirror if the model errors

## Evidence
