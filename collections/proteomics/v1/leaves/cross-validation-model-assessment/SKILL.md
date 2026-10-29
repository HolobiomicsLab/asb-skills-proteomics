---
name: cross-validation-model-assessment
description: Use when a trained classifier (e.g., LGBM on BiLSTM/UniRep/TAPE-BERT
  feature subsets) must be validated before reporting performance, requiring both
  cross-validation and an independent test evaluation.
when_to_use_negative:
- The input is an already-computed metric table or published benchmark — no model
  needs to be trained or evaluated
- The dataset has no held-out independent test split and the goal is only exploratory
  cross-validation, not final performance reporting
- The task targets negative-set sequence annotation (e.g., HMMER evaluation), which
  is restricted to the positive training set because most negative training set sequences
  are annotated incompletely
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0080
tools:
- name: LightGBM (LGBM)
  role: Gradient-boosting classifier used for plant R protein identification and also
    for feature selection prior to cross-validated and independent-test evaluation
  repo: https://github.com/Wangys-prog/prPred-DRLF
- name: prPred-DRLF
  role: Python toolkit (Python 3.7) that implements the full prediction pipeline whose
    models are assessed via cross-validation and independent testing
  repo: https://github.com/Wangys-prog/prPred-DRLF
- name: eFeature
  role: Embedding service producing BiLSTM (3605D), UniRep (1900D), and TAPE-BERT
    (768D) numerical feature vectors that feed the evaluated models
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pmic.202100161
    title: 'prPred‐DRLF: Plant R protein predictor using deep representation learning
      features'
  source_content_hash: sha256:499dd03fb6b9c7d0183b041f86b56d890e1e10ecdcaf2c3603344fc581314d06
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# cross-validation-model-assessment

## Summary

Assess plant R protein prediction models using fivefold cross-validation on the 364-sample training set to examine model effectiveness, followed by evaluation on a 92-sample independent test set with precision, recall, accuracy, F1-score, and AUC metrics.

## When to use

Use when a trained classifier (e.g., LGBM on BiLSTM/UniRep/TAPE-BERT feature subsets) must be validated before reporting performance, requiring both cross-validation and an independent test evaluation.

## When NOT to use

- The input is an already-computed metric table or published benchmark — no model needs to be trained or evaluated
- The dataset has no held-out independent test split and the goal is only exploratory cross-validation, not final performance reporting
- The task targets negative-set sequence annotation (e.g., HMMER evaluation), which is restricted to the positive training set because most negative training set sequences are annotated incompletely

## Inputs

- Wang et al. plant R protein dataset (456 CD-HIT-filtered sequences: 152 positive, 304 negative; FASTA format from https://github.com/Wangys-prog/prPred-DRLF/tree/master/dataset)
- Train/test split: 364 training samples and 92 independent test samples with labels 1 = R protein, 0 = non-R protein
- Selected feature subsets (e.g., fused BiLSTM+UniRep 5505D vectors after LGBM feature selection)
- Trained LGBM classifier model

## Outputs

- Fivefold cross-validation effectiveness estimates per model/feature combination
- Independent-test metric rows: accuracy, precision, recall, F1-score, and AUC for each feature combination (e.g., BiLSTM+UniRep: 0.956, 0.967, 0.905, 0.933, 0.997)
- Comparison table saved as feature_combination_metrics.csv
- ROC curves / AUC values for the independent test set

## How to apply

Train the LGBM classifier on the selected feature subset from the Wang et al. plant R protein dataset (456 sequences: 152 positive, 304 negative; split 364 training / 92 independent test). Apply fivefold cross-validation on the training data to examine the effectiveness of each model/feature combination before committing to a configuration. Then evaluate every feature combination on the held-out 92-sample independent test set, computing precision (TP/(TP+FP)), recall (TP/(TP+FN)), accuracy ((TP+TN)/(TP+FP+TN+FN)), F1-score, and the area under the ROC curve (AUC). Compile the per-combination metric rows (e.g., BiLSTM+UniRep with LGBM feature selection: accuracy 0.956, precision 0.967, recall 0.905, F1 0.933, AUC 0.997) into a comparison table such as feature_combination_metrics.csv. The rationale is that cross-validation guards against overfitting during feature/classifier selection, while the independent test provides an unbiased estimate of generalization for the final reported performance.

## Related tools

- **LightGBM (LGBM)** (Gradient-boosting classifier used for plant R protein identification and also for feature selection prior to cross-validated and independent-test evaluation) — https://github.com/Wangys-prog/prPred-DRLF
- **prPred-DRLF** (Python toolkit (Python 3.7) that implements the full prediction pipeline whose models are assessed via cross-validation and independent testing) — https://github.com/Wangys-prog/prPred-DRLF
- **eFeature** (Embedding service producing BiLSTM (3605D), UniRep (1900D), and TAPE-BERT (768D) numerical feature vectors that feed the evaluated models)

## Examples

```
python3 prPred-DRLF.py -i ./dataset/test_data.fasta -o ./dataset/predict_result
```

## Evaluation signals

- The reproduced BiLSTM+UniRep independent-test row must match the reported values (accuracy 0.956, F1-score 0.933, AUC 0.997) within tolerance
- All seven feature combinations must yield complete metric rows (accuracy, precision, recall, F1-score, AUC) in feature_combination_metrics.csv
- Metric formulas must be implemented exactly: precision = TP/(TP+FP), recall = TP/(TP+FN), accuracy = (TP+TN)/(TP+FP+TN+FN)
- AUC values on the independent test set should approach 0.99 for the best fused feature combination; substantially lower values indicate an evaluation-pipeline error
- Cross-validation must be run on the 364-sample training set only; the 92-sample independent test set must never be used for feature selection or tuning

## Limitations

- Deep representation learning feature methods require an elevated level of computing resources
- Parallel computing is not yet incorporated into the program; the authors plan this in future studies to achieve high computational efficiency
- The independent test set is small (92 samples), so per-metric estimates carry non-trivial variance

## Evidence

- [abstract] Independent test performance of prPred-DRLF: [evidence span withheld — anchor status: not_found]
