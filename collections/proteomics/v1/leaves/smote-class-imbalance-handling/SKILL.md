---
name: smote-class-imbalance-handling
description: Use when the GPCR benchmark dataset is class-imbalanced and you must
  compare classifiers under the SMOTE 300% positive-sample balancing protocol with
  5-fold cross-validation, e.g. reproducing the EMCBOW-GPCR vs. Liao baseline Table
  3 comparison.
when_to_use_negative:
- The dataset is already class-balanced or the evaluation protocol specifies a different
  balancing strategy than SMOTE 300% positive-sample oversampling.
- You are comparing methods under a different cross-validation design (e.g. hold-out
  or leave-one-out) that is not comparable to the paper's 5-fold CV with 20 repetitions
  protocol.
- The task is pure prediction on new sequences via the repository's Predict.py rather
  than benchmark comparison — SMOTE is a training-time step, not needed for inference.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: imbalanced-learn (SMOTE)
  role: Performs 300% positive-sample oversampling on the training partition of each
    fold
- name: XGBoost
  role: Final classifier trained on SMOTE-balanced EMCBOW-GPCR features
  repo: https://github.com/454170054/EMCBOW-GPCR
- name: Gensim
  role: Builds and trains the CBOW word-embedding models whose features are balanced
    with SMOTE
  repo: https://github.com/454170054/EMCBOW-GPCR/blob/main/code/GetCBOWFeatures.py
- name: scikit-learn (RandomForestClassifier)
  role: Classifier of the reimplemented Liao pipeline (SVM-Prot 188D + SMOTE + RF)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.044
    title: 'EMCBOW-GPCR: A method for identifying G-protein coupled receptors based
      on word embedding and wordbooks'
  source_content_hash: sha256:d7b1b6f3b6944c118d1fca3deaed9cb144892ed7406edc1a17cc7e009ec53435
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/454170054/EMCBOW-GPCR
  repo_url: https://github.com/454170054/EMCBOW-GPCR
  tools:
  - Gensim
  - XGBoost
  - imbalanced-learn (SMOTE)
  - scikit-learn (RandomForestClassifier)
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# smote-class-imbalance-handling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A protocol for handling class imbalance in GPCR benchmark data by applying SMOTE with 300% positive-sample oversampling inside a 5-fold cross-validation loop (20 repetitions), enabling a fair comparison between EMCBOW-GPCR (XGBoost) and the Liao baseline (SVM-Prot 188D + SMOTE + RF).

## When to use

Use when the GPCR benchmark dataset is class-imbalanced and you must compare classifiers under the SMOTE 300% positive-sample balancing protocol with 5-fold cross-validation, e.g. reproducing the EMCBOW-GPCR vs. Liao baseline Table 3 comparison.

## When NOT to use

- The dataset is already class-balanced or the evaluation protocol specifies a different balancing strategy than SMOTE 300% positive-sample oversampling.
- You are comparing methods under a different cross-validation design (e.g. hold-out or leave-one-out) that is not comparable to the paper's 5-fold CV with 20 repetitions protocol.
- The task is pure prediction on new sequences via the repository's Predict.py rather than benchmark comparison — SMOTE is a training-time step, not needed for inference.

## Inputs

- GPCR benchmark dataset with imbalanced positive/negative classes (FASTA-style sequences or precomputed feature tables)
- EMCBOW-GPCR features (BOW + word-embedding features reduced by the deep-learning layer)
- SVM-Prot 188-dimensional features for the Liao baseline reimplementation

## Outputs

- Per-fold accuracy and AUC values for each classifier over 5 folds x 20 repetitions
- Aggregated mean ± standard deviation metrics table (metrics_table.csv) with proposed Acc/AUC and Liao Acc/AUC rows

## How to apply

Load the GPCR benchmark dataset and, for each of the 5 cross-validation folds, fit SMOTE from imbalanced-learn with 300% positive-sample oversampling on the training partition only, leaving the test fold untouched. Train the proposed classifier on the SMOTE-balanced EMCBOW-GPCR features (BOW + Gensim CBOW word-embedding features reduced by the deep-learning layer) using XGBoost, which the paper selected because 'the Acc, Mcc and AUC value of XGBoost is the highest'. For the Liao reimplementation, apply the same SMOTE 300% oversampling to the same folds of the 188-dimensional SVM-Prot features and train a RandomForestClassifier. Follow the benchmark protocol of 5-fold cross-validation repeated 20 times, record per-fold accuracy and AUC, and aggregate as mean ± standard deviation (target: EMCBOW-GPCR Acc 92.91±0.10, AUC 95.56±0.13; Liao Acc 83.33±7.26, AUC 92.80±3.8). The key decision point is that oversampling must be applied per-fold to training data only to avoid leaking synthetic positives into evaluation.

## Related tools

- **imbalanced-learn (SMOTE)** (Performs 300% positive-sample oversampling on the training partition of each fold)
- **XGBoost** (Final classifier trained on SMOTE-balanced EMCBOW-GPCR features) — https://github.com/454170054/EMCBOW-GPCR
- **Gensim** (Builds and trains the CBOW word-embedding models whose features are balanced with SMOTE) — https://github.com/454170054/EMCBOW-GPCR/blob/main/code/GetCBOWFeatures.py
- **scikit-learn (RandomForestClassifier)** (Classifier of the reimplemented Liao pipeline (SVM-Prot 188D + SMOTE + RF))

## Examples

```
python GetResults.py
```

## Evaluation signals

- Proposed method mean Acc and AUC reproduce published values within stated SDs: Acc 92.91±0.10, AUC 95.56±0.13.
- Liao reimplementation mean Acc and AUC reproduce published values within stated SDs: Acc 83.33±7.26, AUC 92.80±3.8.
- metrics_table.csv contains all four metric rows (proposed Acc/AUC, Liao Acc/AUC) with mean and standard-deviation columns.
- All EMCBOW-GPCR metrics exceed the corresponding Liao metrics under the identical SMOTE 300% / 5-fold CV splits.
- SMOTE is fitted only on each fold's training partition; the test fold class distribution remains unmodified.

## Limitations

- SMOTE synthesizes minority-class samples, so oversampled positives do not represent new real GPCR sequences; results depend on the fold structure.
- Comparison is protocol-dependent: results are valid only under the same SMOTE 300% and 5-fold CV (20 repetitions) settings used by the paper.
- The classifier choice is sensitive: XGBoost was selected only after observing it had the highest Acc, Mcc and AUC among tested algorithms (CatBoost was faster to train but less accurate).

## Evidence
