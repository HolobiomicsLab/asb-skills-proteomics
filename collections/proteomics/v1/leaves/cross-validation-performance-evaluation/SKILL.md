---
name: cross-validation-performance-evaluation
description: Use when comparing GPCR identification methods (e.g., EMCBOW-GPCR vs.
  Liao's SVM-Prot 188D + SMOTE + RF) under a stated SMOTE-balancing protocol, so that
  per-fold Acc/AUC can be aggregated into a fair side-by-side metrics table.
when_to_use_negative:
- Evaluation target is a single held-out independent test set rather than the 5-fold
  cross-validation with 20 repetitions protocol used in the paper's Table 3 comparison
- Input is already an aggregated metrics table — re-running CV would duplicate completed
  benchmarking
- Task involves non-GPCR sequence classification where the GPCR benchmark dataset,
  SVM-Prot 188D features, and the CBOW wordbooks do not apply
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: XGBoost
  role: Final classifier trained on the processed (DL-reduced) features; selected
    because its Acc, Mcc and AUC were the highest among tested algorithms
- name: Gensim
  role: Builds and trains the CBOW word-embedding models whose segment representations
    feed the feature extraction evaluated by cross-validation
- name: imbalanced-learn (SMOTE)
  role: Performs 300% positive-sample oversampling on the training partition of each
    cross-validation fold to balance classes
- name: scikit-learn (RandomForestClassifier)
  role: Classifier for the reimplemented Liao baseline (SVM-Prot 188D + SMOTE + RF)
    run on identical cross-validation splits
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.044
    title: 'EMCBOW-GPCR: A method for identifying G-protein coupled receptors based
      on word embedding and wordbooks'
  source_content_hash: sha256:ebfae5c24b0e2d4a7cdae210328a362a4d9d03ab4d5f2205eafbda74f73380af
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/454170054/EMCBOW-GPCR
  tools:
  - Gensim
  - XGBoost
  - imbalanced-learn (SMOTE)
  - scikit-learn (RandomForestClassifier)
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# cross-validation-performance-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark protocol from EMCBOW-GPCR: evaluate GPCR predictors with 5-fold cross-validation repeated 20 times on the benchmark dataset, applying SMOTE 300% positive-sample oversampling to each training fold and reporting Acc and AUC as mean ± standard deviation.

## When to use

Use when comparing GPCR identification methods (e.g., EMCBOW-GPCR vs. Liao's SVM-Prot 188D + SMOTE + RF) under a stated SMOTE-balancing protocol, so that per-fold Acc/AUC can be aggregated into a fair side-by-side metrics table.

## When NOT to use

- Evaluation target is a single held-out independent test set rather than the 5-fold cross-validation with 20 repetitions protocol used in the paper's Table 3 comparison
- Input is already an aggregated metrics table — re-running CV would duplicate completed benchmarking
- Task involves non-GPCR sequence classification where the GPCR benchmark dataset, SVM-Prot 188D features, and the CBOW wordbooks do not apply

## Inputs

- GPCR benchmark dataset (sequence labels for the GPCR prediction task)
- Trained CBOW word-embedding models (Ql=2 model after the 74th training, Ql=3 after the 100th, Ql=4 after the 280th, per convergence of the CBOW objective function)
- EMCBOW-GPCR feature vectors (BOW + word-embedding features passed through the deep-learning dimension-reduction layer)
- Liao reimplementation features (SVM-Prot 188-dimensional features + SMOTE + Random Forest)
- FASTA file of GPCR sequences (repository input format for prediction)

## Outputs

- Per-fold accuracy and AUC values for each of the 5 folds, repeated 20 times
- Aggregated mean ± standard deviation metrics (target: proposed Acc 92.91±0.10, AUC 95.56±0.13; Liao Acc 83.33±7.26, AUC 92.80±3.8)
- metrics_table.csv with all four metric rows (proposed Acc/AUC, Liao Acc/AUC) and mean/standard-deviation columns

## How to apply

Load the GPCR benchmark dataset and trained CBOW word-embedding models from the 454170054/EMCBOW-GPCR repository. For each fold, extract EMCBOW-GPCR features (BOW + word-embedding features reduced by the deep-learning layer) and the 188-dimensional SVM-Prot features for the Liao reimplementation, then apply SMOTE with 300% positive-sample oversampling to the training partition only. Run 5-fold cross-validation with 20 repetitions, using XGBoost as the proposed classifier (selected because the Acc, Mcc and AUC value of XGBoost is the highest among tested algorithms) and Random Forest for the Liao pipeline on identical splits. Record per-fold accuracy and AUC for both methods and aggregate as mean ± standard deviation, writing the four metric rows (proposed Acc/AUC, Liao Acc/AUC) with mean and standard-deviation columns to metrics_table.csv.

## Related tools

- **XGBoost** (Final classifier trained on the processed (DL-reduced) features; selected because its Acc, Mcc and AUC were the highest among tested algorithms)
- **Gensim** (Builds and trains the CBOW word-embedding models whose segment representations feed the feature extraction evaluated by cross-validation)
- **imbalanced-learn (SMOTE)** (Performs 300% positive-sample oversampling on the training partition of each cross-validation fold to balance classes)
- **scikit-learn (RandomForestClassifier)** (Classifier for the reimplemented Liao baseline (SVM-Prot 188D + SMOTE + RF) run on identical cross-validation splits)

## Examples

```
python GetResults.py && python Predict.py "test.fasta"
```

## Evaluation signals

- metrics_table.csv contains all four metric rows (proposed Acc/AUC, Liao Acc/AUC) with mean and standard-deviation columns
- Proposed method means fall within stated standard deviations: Acc 92.91±0.10, AUC 95.56±0.13
- Liao baseline means fall within stated standard deviations: Acc 83.33±7.26, AUC 92.80±3.8
- Both methods are evaluated on the same 5-fold splits with SMOTE 300% oversampling applied only to training partitions
- Cross-validation is repeated 20 times on the benchmark dataset, matching the paper's protocol

## Limitations

- Software version numbers (Gensim, XGBoost, etc.) are not stated in the article; consult requirements.txt in the repository
- The word-embedding hyperparameter N is flexible and hard to determine; it may significantly influence feature quality and thus CV metrics
- The DL model structure (number of layers and units per layer) is flexible and difficult to optimize
- The repository README requires a Python 3.7.0 virtual environment with packages from requirements.txt, and GetResults.py must be run first to generate models

## Evidence

- [results] 5-fold cross-validation with 20 times on benchmark dataset: [evidence span withheld — anchor status: not_found]
