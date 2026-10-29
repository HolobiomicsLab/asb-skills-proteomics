---
name: gradient-boosting-classification
description: Use when protein sequences have been embedded into high-dimensional deep
  representation learning vectors (BiLSTM 3605D, UniRep 1900D, TAPE-BERT 768D) and
  a binary R protein / non-R protein classifier with feature selection is required.
when_to_use_negative:
- Input is already a curated feature table or the sequences are encoded with traditional
  handcrafted feature extraction methods rather than deep representation learning
  vectors (BiLSTM/UniRep/TAPE-BERT)
- The dataset lacks a defined training/independent-test split and labels, so supervised
  evaluation of classifier-feature combinations is impossible
- Computational resources are too limited for high-dimensional embedding, since deep
  representation learning features 'require an elevated level of computing resources'
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0080
tools:
- name: LightGBM (LGBM)
  role: Gradient boosting classifier trained on the selected deep representation learning
    feature subset; also used as a feature selection method
  repo: https://github.com/Wangys-prog/prPred-DRLF
- name: eFeature
  role: Produces the BiLSTM, UniRep, and TAPE-BERT numerical vector embeddings that
    serve as the classifier's input features
- name: prPred-DRLF
  role: End-to-end Python 3.7 pipeline (requirements include lightgbm-2.3.0) that
    runs LGBM feature selection and classification on input FASTA
  repo: https://github.com/Wangys-prog/prPred-DRLF
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pmic.202100161
    title: 'prPred‐DRLF: Plant R protein predictor using deep representation learning
      features'
  source_content_hash: sha256:2ce3fa948e07e47ff1eb9ed59cb7be804321ca3ca829ab3d7b19e6d8cafa8bf1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wangys-prog/prPred-DRLF
  tools:
  - BiLSTM
  - LightGBM (LGBM)
  - Python
  - eFeature
  - prPred-DRLF
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# gradient-boosting-classification

## Summary

Supervised binary classification of plant R vs non-R proteins using the LightGBM (LGBM) classifier on deep representation learning feature vectors (BiLSTM 3605D, UniRep 1900D, TAPE-BERT 768D and their fusions, e.g. BiLSTM+UniRep 5505D), with LGBM-based feature selection prior to training.

## When to use

Use when protein sequences have been embedded into high-dimensional deep representation learning vectors (BiLSTM 3605D, UniRep 1900D, TAPE-BERT 768D) and a binary R protein / non-R protein classifier with feature selection is required.

## When NOT to use

- Input is already a curated feature table or the sequences are encoded with traditional handcrafted feature extraction methods rather than deep representation learning vectors (BiLSTM/UniRep/TAPE-BERT)
- The dataset lacks a defined training/independent-test split and labels, so supervised evaluation of classifier-feature combinations is impossible
- Computational resources are too limited for high-dimensional embedding, since deep representation learning features 'require an elevated level of computing resources'

## Inputs

- FASTA file of plant R protein sequences (456 sequences after CD-HIT 30% identity cutoff: 152 positive, 304 negative)
- Numerical embedding vectors from eFeature: BiLSTM 3605D, UniRep 1900D, TAPE-BERT 768D, and fusions (BiLSTM+UniRep 5505D)
- Train/independent-test split: 364 training and 92 test samples with binary labels (1 = R protein, 0 = non-R protein)

## Outputs

- Trained LGBM classifier on the LGBM-selected feature subset
- Prediction results (R protein / non-R protein) per sequence
- Per-feature-combination metric rows (accuracy, precision, recall, F1-score, AUC), e.g. saved as feature_combination_metrics.csv

## How to apply

Encode the Wang et al. plant R protein FASTA set (456 sequences after CD-HIT at 30% identity: 152 positive, 304 negative; split 364 train / 92 independent test, labels 1 = R protein) into numerical vectors via eFeature using BiLSTM, UniRep, and TAPE-BERT, then concatenate fusions such as BiLSTM+UniRep (3605D+1900D = 5505D). Extract an optimal feature subset per combination using LGBM feature selection, comparing the three selection methods RF, LGBM, and MRMD3.0. Train the LGBM classifier (lightgbm-2.3.0, Python 3.7 per the README requirements) on each selected subset, using fivefold cross-validation on the training data to examine model effectiveness before independent testing. Evaluate each combination on the 92-sample independent test set computing precision, recall, accuracy, F1-score, and AUC, and optionally compare LGBM against SVM, RF, and extra tree (ET) classifiers. Expect the fused BiLSTM+UniRep features with the LGBM classifier to perform best, and record per-combination metrics (e.g., BiLSTM+UniRep: accuracy 0.956, precision 0.967, recall 0.905, F1 0.933, AUC 0.997).

## Related tools

- **LightGBM (LGBM)** (Gradient boosting classifier trained on the selected deep representation learning feature subset; also used as a feature selection method) — https://github.com/Wangys-prog/prPred-DRLF
- **eFeature** (Produces the BiLSTM, UniRep, and TAPE-BERT numerical vector embeddings that serve as the classifier's input features)
- **prPred-DRLF** (End-to-end Python 3.7 pipeline (requirements include lightgbm-2.3.0) that runs LGBM feature selection and classification on input FASTA) — https://github.com/Wangys-prog/prPred-DRLF

## Examples

```
python3 prPred-DRLF.py -i ./dataset/test_data.fasta -o ./dataset/predict_result
```

## Evaluation signals

- The reproduced BiLSTM+UniRep independent-test row must match reported values (accuracy 0.956, F1-score 0.933, AUC 0.997) within tolerance
- All seven feature combinations must yield complete metric rows with precision, recall, accuracy, F1-score, and AUC
- Fivefold cross-validation on the 364 training samples should be used to confirm the effectiveness of each model before independent testing
- Evaluation must be on the held-out 92-sample independent test set, not the training data
- Fused BiLSTM+UniRep features should outperform single-embedding and other fusion combinations under the LGBM classifier

## Limitations

- Deep representation learning feature methods require an elevated level of computing resources (stated limitation)
- Parallel computing is not yet incorporated into the program; the authors plan to add it for higher computational efficiency
- The pipeline is version-sensitive: Python 3.7 with pinned dependencies (lightgbm-2.3.0, numpy==1.19.2, pandas==1.2.0) per the README; GPU torch versions are CUDA-dependent

## Evidence

- [abstract] The model was evaluated using an independent test achieving an accuracy of 0.956, F1-scoreof0.933,andareaunderthereceiveroperatingcharacteristic(ROC)curve(AUC)of 0.997.: [evidence span withheld — anchor status: not_found]
