---
name: plant-r-protein-prediction
description: Use when given plant protein sequences in FASTA format that need to be
  classified as R protein vs non-R protein, or when reproducing plant R protein prediction
  benchmarks like ACC 0.956 / F1 0.933 / AUC 0.997.
when_to_use_negative:
- The goal is HMMER-style annotation-based R protein identification — prPred-DRLF
  uses deep representation learning features instead, and HMMER negative training
  sequences are often annotated incompletely
- Input is already a numeric feature table or embedding matrix — only the classification/feature-selection
  portion applies, not sequence encoding
- Working with non-plant sequences or datasets where the deep representation learning
  compute cost (BiLSTM/UniRep embeddings) is prohibitive
edam_operation: http://edamontology.org/operation_0465
edam_topics:
- http://edamontology.org/topic_0621
- http://edamontology.org/topic_3474
tools:
- name: prPred-DRLF
  role: End-to-end Python toolkit that encodes sequences with deep representation
    learning features, applies LGBM feature selection, and predicts R vs non-R protein
  repo: https://github.com/Wangys-prog/prPred-DRLF
- name: CD-HIT
  role: Removes redundant sequences from the input dataset using a 30% identity cutoff
    before encoding
- name: eFeature
  role: Generates BiLSTM, UniRep, and TAPE-BERT embeddings of protein sequences as
    numerical vectors
  repo: http://lab.malab.cn/soft/eFeature/index.html
- name: LightGBM (LGBM)
  role: Classifier for R vs non-R protein prediction and one of the feature selection
    methods applied to the fused 5505D feature vector
- name: BiLSTM
  role: Deep representation learning model producing 3605D sequence embeddings used
    in the fused feature vector
- name: UniRep
  role: 1900-hidden-unit mLSTM pretrained on UniRef50, producing 1900D embeddings
    fused with BiLSTM features
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/pmic.202100161
    title: 'prPred‐DRLF: Plant R protein predictor using deep representation learning
      features'
  source_content_hash: sha256:5963dab075ae20f80145d556f44bfe38933b2607ee16a45588f53053b675ab93
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wangys-prog/prPred-DRLF
  tools:
  - BiLSTM
  - CD-HIT
  - LightGBM (LGBM)
  - Python
  - UniRep
  - eFeature
  - prPred-DRLF
  edam_operation: http://edamontology.org/operation_0465
  edam_topics:
  - http://edamontology.org/topic_0621
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# plant-r-protein-prediction

## Summary

Predict plant resistance (R) proteins by encoding sequences with deep representation learning features (BiLSTM 3605D fused with UniRep 1900D into a 5505D vector), selecting features with LGBM, and classifying with a LightGBM classifier, as implemented in the prPred-DRLF toolkit.

## When to use

Use when given plant protein sequences in FASTA format that need to be classified as R protein vs non-R protein, or when reproducing plant R protein prediction benchmarks like ACC 0.956 / F1 0.933 / AUC 0.997.

## When NOT to use

- The goal is HMMER-style annotation-based R protein identification — prPred-DRLF uses deep representation learning features instead, and HMMER negative training sequences are often annotated incompletely
- Input is already a numeric feature table or embedding matrix — only the classification/feature-selection portion applies, not sequence encoding
- Working with non-plant sequences or datasets where the deep representation learning compute cost (BiLSTM/UniRep embeddings) is prohibitive

## Inputs

- Plant protein sequences in FASTA format (e.g., dataset/test_data.fasta)
- Wang et al. plant R protein dataset (456 sequences: 152 positive, 304 negative) from https://github.com/Wangys-prog/prPred-DRLF/tree/master/dataset

## Outputs

- Per-sequence R protein / non-R protein predictions saved as CSV (e.g., predict_result.csv)
- Performance metrics on the independent test set: precision, recall, accuracy (ACC), F1-score, and AUC

## How to apply

Start from the Wang et al. plant R protein dataset (456 sequences: 152 positive, 304 negative after CD-HIT redundancy removal at a 30% identity cutoff, split 364 train / 92 independent test). Encode each protein sequence into numerical vectors with three deep representation learning models via eFeature: BiLSTM (3605D), UniRep (1900D, a 1900-hidden-unit mLSTM pretrained on UniRef50), and optionally TAPE-BERT (768D). Fuse the BiLSTM and UniRep vectors into a 5505D feature vector, then apply LGBM feature selection (alternatives: RF or MRMD3.0) to extract the optimal feature subset. Train the LightGBM classifier (alternatives compared: SVM, RF, ET) on the 364 training samples (1 = R protein, 0 = non-R protein) with fivefold cross-validation to check effectiveness, then run prPred-DRLF.py on the 92-sample independent test set. Evaluate with precision, recall, accuracy, F1-score, and AUC; the reference independent-test result is accuracy 0.956, precision 0.967, recall 0.905, F1-score 0.933, AUC 0.997.

## Related tools

- **prPred-DRLF** (End-to-end Python toolkit that encodes sequences with deep representation learning features, applies LGBM feature selection, and predicts R vs non-R protein) — https://github.com/Wangys-prog/prPred-DRLF
- **CD-HIT** (Removes redundant sequences from the input dataset using a 30% identity cutoff before encoding)
- **eFeature** (Generates BiLSTM, UniRep, and TAPE-BERT embeddings of protein sequences as numerical vectors) — http://lab.malab.cn/soft/eFeature/index.html
- **LightGBM (LGBM)** (Classifier for R vs non-R protein prediction and one of the feature selection methods applied to the fused 5505D feature vector)
- **BiLSTM** (Deep representation learning model producing 3605D sequence embeddings used in the fused feature vector)
- **UniRep** (1900-hidden-unit mLSTM pretrained on UniRef50, producing 1900D embeddings fused with BiLSTM features)

## Examples

```
python3 prPred-DRLF.py -i ./dataset/test_data.fasta -o ./dataset/predict_result
```

## Evaluation signals

- Independent test metrics should match the reported performance within a small tolerance: accuracy 0.956, precision 0.967, recall 0.905, F1-score 0.933, AUC 0.997
- After CD-HIT at a 30% identity cutoff, the dataset must contain 456 sequences (152 positive, 304 negative), split as 364 training and 92 independent test samples
- Fused BiLSTM + UniRep feature vector must have dimension 5505 (3605D + 1900D)
- The fused BiLSTM+UniRep features with LGBM should outperform other feature combinations and other classifiers (SVM, RF, ET) and beat prPred and HMMER on accuracy, F1-score, AUC, and recall
- Output CSV should contain per-sequence predictions labeled as R protein or non-R protein

## Limitations

- Deep representation learning feature methods require an elevated level of computing resources
- Parallel computing is not yet incorporated into the program; the authors plan to add it in future studies for high computational efficiency
- A GPU speeds up operation (README notes faster execution with GPU)
- Web server homepage exists (Figure 6) but no access URL is stated in the article text

## Evidence

- [abstract] we propose a new computational approach, called pr Pred-DRLF, which uses deep representation learning feature models to encode the amino acids as numerical vectors: [evidence span withheld — anchor status: not_found]
- [abstract] The results show that the fused features of bidirectional long short-term memory (BiLSTM) embedding and unified representation (Uni Rep) embedding have a better performance than other features: [evidence span withheld — anchor status: not_found]
