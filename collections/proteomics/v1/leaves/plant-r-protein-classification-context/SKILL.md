---
name: plant-r-protein-classification-context
description: Use when plant protein FASTA sequences must be labeled as R protein vs
  non-R protein, and prediction should leverage pretrained deep representation embeddings
  (BiLSTM, UniRep) rather than traditional hand-crafted features.
when_to_use_negative:
- Input is already a numeric feature table (embeddings) rather than raw protein sequences
  in FASTA format — the toolkit expects FASTA and performs embedding itself
- HMMER-based profile matching of annotated R protein domains is the goal — prPred-DRLF
  is a machine-learning predictor, and HMMER negatives were largely incompletely annotated
- Non-plant resistance protein prediction outside the scope of the Wang et al. [9]
  plant R protein dataset
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0080
tools:
- name: prPred-DRLF
  role: End-to-end Python toolkit that embeds FASTA sequences with deep representation
    learning models and predicts plant R proteins via LGBM
  repo: https://github.com/Wangys-prog/prPred-DRLF
- name: LightGBM (LGBM)
  role: Classifier on the fused BiLSTM + UniRep feature vector and feature-selection
    method that reduces the 5505D fused vector to an optimal subset
  repo: https://github.com/Wangys-prog/prPred-DRLF
- name: CD-HIT
  role: Redundancy removal of training sequences with a 30% identity cutoff before
    embedding and classification
- name: TAPE (tasks assessing protein embedding)
  role: BERT-based semi-supervised protein sequence embedding model (768D) evaluated
    as an alternative representation to BiLSTM and UniRep
  repo: https://github.com/Wangys-prog/prPred-DRLF
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/pmic.202100161
    title: 'prPred‐DRLF: Plant R protein predictor using deep representation learning
      features'
  source_content_hash: sha256:2c1fa0157780172e92a4800853a76482755d57a3c5999ed6cba2fa52d847d0ae
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wangys-prog/prPred-DRLF
  tools:
  - CD-HIT
  - LightGBM (LGBM)
  - Python
  - TAPE (tasks assessing protein embedding)
  - prPred-DRLF
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# plant-r-protein-classification-context

## Summary

Classify plant resistance (R) proteins using deep representation learning features — fused BiLSTM (3605D) and UniRep (1900D) embeddings — reduced by LGBM feature selection and fed to a light gradient boosting machine (LGBM) classifier, as implemented in the prPred-DRLF Python toolkit.

## When to use

Use when plant protein FASTA sequences must be labeled as R protein vs non-R protein, and prediction should leverage pretrained deep representation embeddings (BiLSTM, UniRep) rather than traditional hand-crafted features.

## When NOT to use

- Input is already a numeric feature table (embeddings) rather than raw protein sequences in FASTA format — the toolkit expects FASTA and performs embedding itself
- HMMER-based profile matching of annotated R protein domains is the goal — prPred-DRLF is a machine-learning predictor, and HMMER negatives were largely incompletely annotated
- Non-plant resistance protein prediction outside the scope of the Wang et al. [9] plant R protein dataset

## Inputs

- Plant protein sequences in FASTA format (e.g. ./dataset/test_data.fasta)
- Plant R protein dataset from Wang et al. [9]: 456 sequences (152 positive, 304 negative), downloadable from the prPred-DRLF GitHub dataset directory
- Pretrained BiLSTM, UniRep, and TAPE-BERT embedding models accessed via eFeature

## Outputs

- CSV prediction file labeling each input sequence as R protein or non-R protein (path passed via -o)
- Feature-selected 5505D-derived subset matrix from LGBM feature selection usable for downstream visualization (e.g. 2D UMAP scatter separating R vs non-R clusters)

## How to apply

Clone the prPred-DRLF repository and set up a Python 3.7 environment (numpy 1.19.2, pandas 1.2.0, lightgbm-2.3.0, tape_proteins 0.5, torch 1.2.0/1.3.1). Prepare input sequences in FASTA format; the reference dataset derives from Wang et al. [9] with 456 sequences (152 positive R proteins, 304 negatives) after CD-HIT filtering at a 30% identity cutoff, split into 364 training and 92 independent-test samples. Embed sequences via the three deep representation learning models (BiLSTM 3605D, UniRep 1900D, TAPE-BERT 768D), fuse BiLSTM + UniRep into a 5505D vector, then apply LGBM feature selection before LGBM classification. Rationale: the article reports the fused BiLSTM + UniRep features outperform other feature combinations, and the independent test achieved accuracy 0.956, F1-score 0.933, and AUC 0.997. Evaluate with precision, recall, accuracy, F1-score, and AUC under fivefold cross-validation plus independent testing, and benchmark against prPred and HMMER.

## Related tools

- **prPred-DRLF** (End-to-end Python toolkit that embeds FASTA sequences with deep representation learning models and predicts plant R proteins via LGBM) — https://github.com/Wangys-prog/prPred-DRLF
- **LightGBM (LGBM)** (Classifier on the fused BiLSTM + UniRep feature vector and feature-selection method that reduces the 5505D fused vector to an optimal subset) — https://github.com/Wangys-prog/prPred-DRLF
- **CD-HIT** (Redundancy removal of training sequences with a 30% identity cutoff before embedding and classification)
- **TAPE (tasks assessing protein embedding)** (BERT-based semi-supervised protein sequence embedding model (768D) evaluated as an alternative representation to BiLSTM and UniRep) — https://github.com/Wangys-prog/prPred-DRLF

## Examples

```
python3 prPred-DRLF.py -i ./dataset/test_data.fasta -o ./dataset/predict_result
```

## Evaluation signals

- Independent test reaches accuracy 0.956, F1-score 0.933, and AUC 0.997 on the 92-sample held-out set
- Fused BiLSTM + UniRep features (3605D + 1900D = 5505D) outperform BiLSTM-only, UniRep-only, and TAPE-BERT feature combinations under fivefold cross-validation
- prPred-DRLF shows overall improvement in accuracy, F1-score, AUC, and recall compared with prPred and HMMER
- 2D UMAP of LGBM-feature-selected vectors separates non-R and R proteins into distinct clusters (Figure 4)
- Prediction output CSV is produced for every FASTA input sequence via prPred-DRLF.py -i/-o

## Limitations

- Deep representation learning feature methods require an elevated level of computing resources
- Parallel computing is not yet incorporated into the program; the authors plan to add it to achieve high computational efficiency
- Python 3.7 with specific pinned package versions (lightgbm-2.3.0, numpy 1.19.2, pandas 1.2.0, tape_proteins 0.5) and PyTorch 1.2.0/1.3.1 is required; GPU speeds up the model but CPU-only installs need the +cpu wheels

## Evidence

- [abstract] Independent test metrics: [evidence span withheld — anchor status: not_found]
